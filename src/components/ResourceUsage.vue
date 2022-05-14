<script setup lang="ts">
import { ref } from 'vue'
import { useDraggable } from '@vueuse/core'
import {
  NGrid,
  NGi,
  NH2,
  NButton,
  NPagination,
  NDataTable,
  NInput,
  NIcon,
  NInputGroup,
} from 'naive-ui'
import {
  CaretBackCircleOutline,
  CaretDownCircleOutline,
  LockClosedOutline,
  LockOpenOutline,
} from '@vicons/ionicons5'

const props = defineProps<{ initX: number; initY: number }>()

const container = ref<HTMLElement | null>(null)
const isDraggable = ref(true)
const isFolded = ref(false)

const { style } = useDraggable(container, {
  initialValue: { x: props.initX, y: props.initY },
  onStart(_position, event) {
    if (!isDraggable.value) return false
    for (
      let tmp = event.target as HTMLElement;
      tmp !== container.value;
      tmp = tmp.parentElement!
    ) {
      if (tmp.dataset.dragProtected !== undefined) return false
    }
  },
})

function onLock(event: MouseEvent) {
  isDraggable.value = !isDraggable.value
  ;(event.currentTarget as HTMLElement).blur()
}
function onFold(event: MouseEvent) {
  isFolded.value = !isFolded.value
  ;(event.currentTarget as HTMLElement).blur()
}
</script>

<template>
  <div class="container" ref="container" :style="style">
    <n-grid :cols="24">
      <n-gi :span="7" :offset="2">
        <n-h2 data-drag-protected class="mb-2"> Resource Usage </n-h2>
      </n-gi>
      <n-gi :span="3" :offset="12">
        <n-button
          circle
          type="info"
          quaternary
          @click="onLock"
          data-drag-protected
        >
          <template #icon>
            <n-icon>
              <lock-open-outline v-if="isDraggable" />
              <lock-closed-outline v-else />
            </n-icon>
          </template>
        </n-button>
        <n-button
          circle
          type="info"
          quaternary
          @click="onFold"
          data-drag-protected
          class="ml-1"
        >
          <template #icon>
            <n-icon>
              <caret-back-circle-outline v-if="isFolded" />
              <caret-down-circle-outline v-else />
            </n-icon>
          </template>
        </n-button>
      </n-gi>
      <n-gi :span="10" :offset="11">
        <n-input-group>
          <n-input data-drag-protected />
          <n-button data-drag-protected primary type="info">查找</n-button>
        </n-input-group>
      </n-gi>
      <n-gi :span="3">
        <n-button data-drag-protected class="ml-3" type="info" ghost>
          重置
        </n-button>
      </n-gi>
      <n-gi :span="24" class="mt-2">
        <n-data-table data-drag-protected />
      </n-gi>
      <n-gi :span="12" :offset="12" class="mt-2">
        <n-pagination data-drag-protected />
      </n-gi>
    </n-grid>
  </div>
</template>

<style scoped>
.container {
  position: fixed;
  border: 1px solid gray;
  border-radius: 10px;
  width: 600px;
  height: 400px;
  padding: 10px;
  background-color: aliceblue;
}
</style>
