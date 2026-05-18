<script setup lang="ts">
import { ref } from 'vue'
import { storeToRefs } from 'pinia'
import { useCounterStore } from '@/stores/counter'
import type { TableRow } from '@/types'

const counter = useCounterStore()
const { count } = storeToRefs(counter)

// el-input demo
const inputValue = ref('')

// el-dialog demo
const dialogVisible = ref(false)

// el-table mock data
const tableData: TableRow[] = [
  { id: 1, name: 'Alice Martin',  role: 'Engineer',  email: 'alice@example.com' },
  { id: 2, name: 'Bob Chen',      role: 'Designer',  email: 'bob@example.com'   },
  { id: 3, name: 'Carol Okonkwo', role: 'PM',        email: 'carol@example.com' },
]

console.log('Hello, world! second this time');
</script>

<template>
  <div class="space-y-8">
    <!-- Page title -->
    <h1 class="text-2xl font-bold text-gray-800">Component Demo</h1>

    <!-- ── Buttons ─────────────────────────────────────────── -->
    <section class="bg-white rounded-lg p-6 shadow-sm">
      <h2 class="text-sm font-semibold text-gray-500 uppercase tracking-wider mb-4">Buttons</h2>
      <div class="flex flex-wrap gap-3">
        <el-button type="primary">Primary</el-button>
        <el-button>Default</el-button>
        <el-button type="danger">Danger</el-button>
        <el-button type="primary" plain>Plain Primary</el-button>
        <el-button type="primary" :loading="true">Loading</el-button>
      </div>
    </section>

    <!-- ── Input ──────────────────────────────────────────── -->
    <section class="bg-white rounded-lg p-6 shadow-sm">
      <h2 class="text-sm font-semibold text-gray-500 uppercase tracking-wider mb-4">Input</h2>
      <div class="flex flex-col gap-3 max-w-sm">
        <el-input v-model="inputValue" placeholder="Type something…" clearable />
        <p class="text-sm text-gray-600">
          Value: <span class="font-mono text-blue-600">{{ inputValue || '—' }}</span>
        </p>
      </div>
    </section>

    <!-- ── Table ──────────────────────────────────────────── -->
    <section class="bg-white rounded-lg p-6 shadow-sm">
      <h2 class="text-sm font-semibold text-gray-500 uppercase tracking-wider mb-4">Table</h2>
      <el-table :data="tableData" stripe class="w-full">
        <el-table-column prop="id"    label="ID"    width="60" />
        <el-table-column prop="name"  label="Name"  />
        <el-table-column prop="role"  label="Role"  />
        <el-table-column prop="email" label="Email" />
      </el-table>
    </section>

    <!-- ── Dialog ─────────────────────────────────────────── -->
    <section class="bg-white rounded-lg p-6 shadow-sm">
      <h2 class="text-sm font-semibold text-gray-500 uppercase tracking-wider mb-4">Dialog</h2>
      <el-button type="primary" @click="dialogVisible = true">Open Dialog</el-button>

      <el-dialog v-model="dialogVisible" title="Example Dialog" width="420px">
        <p class="text-gray-600">
          This dialog was triggered by a button click. You can put any content here.
        </p>
        <template #footer>
          <div class="flex justify-end gap-2">
            <el-button @click="dialogVisible = false">Cancel</el-button>
            <el-button type="primary" @click="dialogVisible = false">Confirm</el-button>
          </div>
        </template>
      </el-dialog>
    </section>

    <!-- ── Pinia Counter ──────────────────────────────────── -->
    <section class="bg-white rounded-lg p-6 shadow-sm">
      <h2 class="text-sm font-semibold text-gray-500 uppercase tracking-wider mb-4">
        Pinia Counter Store
      </h2>
      <div class="flex items-center gap-4">
        <el-button circle @click="counter.decrement">−</el-button>
        <el-tag size="large" type="primary" class="text-lg px-4">{{ count }}</el-tag>
        <el-button circle type="primary" @click="counter.increment">+</el-button>
        <el-button text @click="counter.reset">Reset</el-button>
      </div>
    </section>

    <!-- ── Tailwind + EP side-by-side ─────────────────────── -->
    <section class="bg-white rounded-lg p-6 shadow-sm">
      <h2 class="text-sm font-semibold text-gray-500 uppercase tracking-wider mb-4">
        Tailwind + Element Plus coexistence
      </h2>
      <div class="grid grid-cols-1 sm:grid-cols-3 gap-4">
        <div class="flex flex-col items-center gap-2 p-4 bg-blue-50 rounded-lg border border-blue-100">
          <el-tag type="primary">Vue 3</el-tag>
          <span class="text-xs text-gray-500">Composition API</span>
        </div>
        <div class="flex flex-col items-center gap-2 p-4 bg-green-50 rounded-lg border border-green-100">
          <el-tag type="success">Tailwind v4</el-tag>
          <span class="text-xs text-gray-500">Utility classes</span>
        </div>
        <div class="flex flex-col items-center gap-2 p-4 bg-purple-50 rounded-lg border border-purple-100">
          <el-tag type="warning">Element Plus</el-tag>
          <span class="text-xs text-gray-500">Auto-imported</span>
        </div>
      </div>
    </section>
  </div>
</template>
