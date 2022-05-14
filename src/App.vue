<script setup lang="ts">
import { NConfigProvider } from 'naive-ui'
import type { GlobalThemeOverrides } from 'naive-ui'
import ResourceUsage from '@/components/ResourceUsage.vue'

const themeOverrides: GlobalThemeOverrides = {
  common: {
    infoColor: '#2080f0',
    primaryColor: '#2080f0FF',
    primaryColorHover: '#4098FCFF',
    primaryColorPressed: '#1060C9FF',
    primaryColorSuppl: '#4098FCFF',
  },
}
</script>

<template>
  <n-config-provider :theme-overrides="themeOverrides">
    <resource-usage :init-x="300" :init-y="200" />
  </n-config-provider>
</template>

<style>
body {
  background-color: azure;
}
</style>
