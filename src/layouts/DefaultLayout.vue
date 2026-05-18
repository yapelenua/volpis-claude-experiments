<script setup lang="ts">
import { ref } from 'vue'

const activeMenu = ref('home')

const navItems = [
  { index: 'home', label: 'Home', route: '/' },
]
</script>

<template>
  <div class="flex h-screen bg-gray-50">
    <!-- Sidebar -->
    <aside class="w-56 flex-shrink-0 bg-gray-900 flex flex-col">
      <div class="h-14 flex items-center px-4 border-b border-gray-700">
        <span class="text-white font-semibold text-base tracking-wide">MyApp</span>
      </div>
      <nav class="flex-1 py-4 px-2 space-y-1">
        <router-link
          v-for="item in navItems"
          :key="item.index"
          :to="item.route"
          class="flex items-center gap-2 px-3 py-2 rounded-md text-gray-300 hover:text-white hover:bg-gray-700 transition-colors text-sm"
          active-class="bg-gray-700 text-white"
        >
          {{ item.label }}
        </router-link>
      </nav>
    </aside>

    <!-- Right panel -->
    <div class="flex flex-col flex-1 overflow-hidden">
      <!-- Header -->
      <header class="h-14 bg-white border-b border-gray-200 flex items-center px-4">
        <el-menu
          :default-active="activeMenu"
          mode="horizontal"
          :ellipsis="false"
          class="border-none flex-1"
          @select="(key: string) => (activeMenu = key)"
        >
          <el-menu-item index="home">Dashboard</el-menu-item>
          <el-menu-item index="settings">Settings</el-menu-item>
        </el-menu>
        <div class="flex items-center gap-2 ml-auto">
          <el-avatar size="small" class="bg-blue-500">U</el-avatar>
        </div>
      </header>

      <!-- Main content -->
      <main class="flex-1 overflow-auto p-6">
        <router-view />
      </main>
    </div>
  </div>
</template>
