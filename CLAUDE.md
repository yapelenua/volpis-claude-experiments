# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # start dev server (Vite HMR)
npm run build     # vue-tsc type-check then Vite production build
npm run preview   # serve the production build locally
```

There is no test runner or linter configured yet.

## Architecture

### Auto-imports — do not manually import these

`unplugin-auto-import` and `unplugin-vue-components` run at build time and inject imports automatically. **Never manually import:**

- Vue composables: `ref`, `computed`, `watch`, `onMounted`, etc.
- Pinia: `defineStore`, `storeToRefs`
- Vue Router composables: `useRouter`, `useRoute`
- Element Plus components: `ElButton`, `ElTable`, `ElDialog`, etc. — use `<el-button>` directly in templates

Generated type declarations live in `src/auto-imports.d.ts` and `src/components.d.ts` (regenerated on build).

### CSS loading order (critical)

`src/assets/styles/main.css` controls load order deliberately:

1. `element-plus/dist/index.css` — full EP stylesheet
2. `element-theme.css` — CSS variable overrides for EP theming
3. Tailwind `theme` + `utilities` layers — **no preflight**, so EP base styles are not reset

Do not reorder these imports. Tailwind preflight is intentionally omitted. The `ElementPlusResolver` in `vite.config.ts` is set to `importStyle: false` because the full EP stylesheet is already loaded here.

To customise Element Plus colours, radii, or fonts — edit `src/assets/styles/element-theme.css` using `--el-*` CSS variables. Never override EP styles with Tailwind utility classes (`text-*`, `bg-*`, etc.) applied directly to `el-*` components.

### Routing and layout

Every route is a child of `DefaultLayout.vue`, which provides the sidebar and top `<el-menu>` header. To add a new page:

1. Create `src/pages/YourPage.vue`
2. Add a lazy-loaded route entry in `src/router/index.ts` as a child of the `/` layout route
3. Optionally add a nav link to the `navItems` array in `DefaultLayout.vue`

### Pinia stores

Stores use the **setup-store** style (`defineStore('id', () => { ... })`). One store per domain — no god stores. Consumers must use `storeToRefs()` when destructuring reactive state. Never mutate store state directly outside of actions.

### TypeScript

Strict mode (`strict: true`, `noImplicitAny`). The `@/` alias maps to `src/`. Shared interfaces go in `src/types/index.ts`.

- Every function must have an explicit return type
- Props typed with `defineProps<{...}>()`
- Emits typed with `defineEmits<{...}>()`
- `any` and `@ts-ignore` are bugs, not workarounds

### Vue components

- `<script setup lang="ts">` only — Options API is not used in this project
- Composables go in `src/composables/`, named `use*.ts`
- No business logic directly in templates — extract to `computed` or functions
- Flag components over 200 lines for splitting

## What to flag

- `any` type or `@ts-ignore`
- Options API usage
- Element Plus style overrides via Tailwind utilities
- Missing error handling in async functions
- Direct store state mutation outside actions
- Components over 200 lines
- `console.log` left in committed code
