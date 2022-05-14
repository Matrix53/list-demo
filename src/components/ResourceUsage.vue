<script setup lang="ts">
import { ref, reactive } from 'vue'
import { useDraggable } from '@vueuse/core'
import {
  NGrid,
  NGi,
  NH2,
  NButton,
  NDataTable,
  NInput,
  NIcon,
  NInputGroup,
  NCollapseTransition,
  NTooltip,
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
const columns = ref([])
const data = ref([])
const pagination = reactive({})

const { style } = useDraggable(container, {
  initialValue: { x: props.initX, y: props.initY },
  preventDefault: true,
  stopPropagation: true,
  onStart(_position, event) {
    if (!isDraggable.value) return false
    for (
      let tmp = event.target as HTMLElement;
      tmp !== container.value;
      tmp = tmp.parentElement!
    ) {
      if (tmp.dataset.dragProtected !== undefined) return false
    }
    container.value.classList.add('moving')
    ;(event.target as HTMLElement).style.cursor = 'move'
  },
  onEnd(_position, event) {
    container.value?.classList.remove('moving')
    ;(event.target as HTMLElement).style.cursor = ''
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
      <n-gi :span="7" :offset="1">
        <n-h2 class="mb-0 cursor-default">Resource Usage</n-h2>
      </n-gi>
      <n-gi :span="3" :offset="13">
        <n-tooltip trigger="hover" :duration="50" :delay="650">
          <template #trigger>
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
          </template>
          {{ isDraggable ? 'Pin' : 'Unpin' }}
        </n-tooltip>
        <n-tooltip
          trigger="hover"
          :duration="50"
          :delay="650"
          display-directive="show"
        >
          <template #trigger>
            <n-button
              circle
              type="info"
              quaternary
              @click="onFold"
              data-drag-protected
              class="ml-1 fold-button"
            >
              <template #icon>
                <n-icon>
                  <caret-back-circle-outline v-if="isFolded" />
                  <caret-down-circle-outline v-else />
                </n-icon>
              </template>
            </n-button>
          </template>
          {{ isFolded ? 'Unfold' : 'Fold' }}
        </n-tooltip>
      </n-gi>
    </n-grid>
    <n-collapse-transition :show="!isFolded">
      <n-grid :cols="24" class="mt-2">
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
          <n-data-table
            data-drag-protected
            :columns="columns"
            :data="data"
            :pagination="pagination"
            class="data-table"
          />
        </n-gi>
      </n-grid>
    </n-collapse-transition>
  </div>
</template>

<style scoped>
.container {
  position: fixed;
  border: 1px solid #ddd;
  border-radius: 10px;
  width: 600px;
  padding: 10px;
  background-color: aliceblue;
}
.moving {
  box-shadow: 0 0 7px #ddd;
}
.fold-button:deep(.n-button__icon) {
  width: 24px;
  max-width: 24px;
  height: 24px;
  font-size: 24px;
}
.fold-button:deep(.n-icon-slot) {
  height: 24px;
  width: 24px;
}
.fold-button {
  position: relative;
  top: 3px;
}
.data-table {
  min-height: 310px;
}
.data-table:deep(.n-data-table__pagination) {
  margin-top: 9px;
}
</style>
