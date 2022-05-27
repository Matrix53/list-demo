<template>
  <ResourceUsage :initX="300" :initY="150" />
</template>

<script>
import { defineComponent } from '@vue/runtime-core'
import ResourceUsage from './components/ResourceUsage.vue'

export default defineComponent({
  name: 'App',
  components: {
    ResourceUsage,
  },
})
</script>

<style>
body {
  background-color: azure;
}
</style>
