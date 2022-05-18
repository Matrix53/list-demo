<script setup lang="tsx">
import { ref, reactive, onMounted, computed, onUpdated } from 'vue'
import { useDraggable, useIntervalFn, useElementSize } from '@vueuse/core'
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

import type { DataTableColumn } from 'naive-ui'
interface RowDataItem {
  cluster: string
  department: string
  reserved: number
  reservedUsed: number
  spotUsed: number
  block: number
}

const props = defineProps<{ initX: number; initY: number }>()

const container = ref<HTMLDivElement | null>(null)
const tableContainer = ref<HTMLDivElement | null>(null)
const dataTable = ref<InstanceType<typeof NDataTable> | null>(null)
const isDraggable = ref(true)
const isFolded = ref(false)
const inputText = ref('')
const keyWord = ref<RegExp>(/.*/)
const rawData = ref<RowDataItem[]>([])
const storedHeight = ref(421)

const palette = [
  ['#eff6ff', '#000'],
  ['#dbeafe', '#000'],
  ['#bfdbfe', '#000'],
  ['#93c5fd', '#000'],
  ['#60a5fa', '#fff'],
  ['#3b82f6', '#fff'],
  ['#2563eb', '#fff'],
]
const columns: DataTableColumn[] = [
  {
    title: 'Cluster',
    key: 'cluster',
    width: 110,
  },
  {
    title: 'Department',
    key: 'department',
    width: 120,
  },
  {
    title: 'Reserved',
    key: 'reserved',
    width: 115,
    render: renderCell('reserved'),
    cellProps: addCellProps('reserved'),
  },
  {
    title: 'SpotUsed',
    key: 'spotUsed',
    width: 115,
    render: renderCell('spotUsed'),
    cellProps: addCellProps('spotUsed'),
  },
  {
    title: 'Block',
    key: 'block',
    width: 115,
    render: renderCell('block'),
    cellProps: addCellProps('block'),
  },
]

const { style } = useDraggable(container, {
  initialValue: { x: props.initX, y: props.initY },
  preventDefault: true,
  stopPropagation: true,
  onStart(position, event) {
    if (
      !isFolded.value &&
      position.x + 15 >= container.value!.clientWidth &&
      position.y + 15 >= container.value!.clientHeight
    )
      return false
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
useIntervalFn(getTableData, 2500, {
  immediateCallback: true,
})
const { height } = useElementSize(tableContainer)

const pageSize = computed(() => {
  return Math.floor((height.value - 139) / 47)
})
const filteredData = computed(() => {
  return rawData.value.filter((item) => {
    return (
      item.cluster.match(keyWord.value) || item.department.match(keyWord.value)
    )
  })
})
const maxData = computed(() => {
  return rawData.value.reduce((max, item) => {
    let tmp = Math.max(item.block, item.reserved, item.spotUsed)
    return Math.max(max, tmp)
  }, 0)
})

onMounted(addDragProtection)
onUpdated(addDragProtection)

function onLock(event: MouseEvent) {
  isDraggable.value = !isDraggable.value
  ;(event.currentTarget as HTMLElement).blur()
}
function onFold(event: MouseEvent) {
  if (!isFolded.value) storedHeight.value = height.value
  isFolded.value = !isFolded.value
  ;(event.currentTarget as HTMLElement).blur()
}
function onSearch() {
  keyWord.value = new RegExp(inputText.value.trim().split(/\s+/).join('|'))
}
function addDragProtection() {
  document
    .querySelector('#data-table .n-data-table-wrapper')
    ?.setAttribute('data-drag-protected', '')
  document
    .querySelector('#data-table .n-pagination')
    ?.setAttribute('data-drag-protected', '')
}
function resetFilter() {
  inputText.value = ''
  keyWord.value = /.*/
  dataTable.value?.clearSorter()
  dataTable.value?.page(1)
}
function generateData() {
  let clusterList = [
    'sh38',
    'bj15',
    'sh231',
    'bh154',
    'ck23',
    'ml68',
    'pku39',
    'vl27',
    'tc39',
  ]
  let departmentList = [
    'Model',
    'STC',
    'ToolChain',
    'Thoery',
    'DataSet',
    'Algorithm',
    'Production',
    'Test',
    'Hardware',
  ]
  let res: Record<string, Record<string, Record<string, number>>> = {}
  clusterList.forEach((cluster) => {
    let tmp = {}
    departmentList.forEach((department) => {
      Object.defineProperty(tmp, department, {
        value: {
          RESERVED_TOTAL: Math.floor(Math.random() * 100),
          RESERVED_USED: Math.floor(Math.random() * 100),
          RESERVED_IDLE: Math.floor(Math.random() * 100),
          SPOT_USED: Math.floor(Math.random() * 100),
          RESERVED_BLOCKED: Math.floor(Math.random() * 100),
        },
        writable: true,
        enumerable: true,
      })
    })
    Object.defineProperty(res, cluster, {
      value: tmp,
      writable: true,
      enumerable: true,
    })
  })
  return res
}
function getTableData() {
  let mockData = generateData()
  let randomData: RowDataItem[] = []
  for (let cluster in mockData) {
    for (let department in mockData[cluster]) {
      randomData.push({
        cluster,
        department,
        reservedUsed: mockData[cluster][department].RESERVED_USED,
        reserved: mockData[cluster][department].RESERVED_TOTAL,
        spotUsed: mockData[cluster][department].SPOT_USED,
        block: mockData[cluster][department].RESERVED_BLOCKED,
      })
    }
  }
  randomData.sort((a, b) => {
    return (
      a.cluster.localeCompare(b.cluster) ||
      a.department.localeCompare(b.department)
    )
  })
  rawData.value = randomData
}
function renderCell(attr: keyof RowDataItem) {
  return (rowData: object) => {
    return <>{(rowData as RowDataItem)[attr]}</>
  }
}
function addCellProps(attr: keyof RowDataItem) {
  return (rowData: object) => {
    let index = Math.floor(
      (((rowData as RowDataItem)[attr] as number) * (palette.length - 1)) /
        maxData.value
    )
    return {
      style: {
        backgroundColor: palette[index][0],
        color: palette[index][1],
      },
    }
  }
}
</script>

<template>
  <div
    class="container"
    ref="container"
    :style="[style, { paddingBottom: isFolded ? '10px' : '0px' }]"
  >
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
      <div
        :style="{
          paddingBottom: '10px',
          paddingRight: '10px',
          height: storedHeight + 'px',
          resize: 'vertical',
          overflow: 'hidden',
          minHeight: '280px',
        }"
        ref="tableContainer"
      >
        <n-grid :cols="24" class="mt-2">
          <n-gi :span="10" :offset="11">
            <n-input-group>
              <n-input
                data-drag-protected
                placeholder="Search Clus. or Dept."
                v-model:value="inputText"
                type="text"
                @keyup.enter="onSearch"
              />
              <n-button
                data-drag-protected
                primary
                type="info"
                @click="onSearch"
              >
                Search
              </n-button>
            </n-input-group>
          </n-gi>
          <n-gi :span="3">
            <n-button
              data-drag-protected
              class="ml-2"
              type="info"
              ghost
              @click="resetFilter"
            >
              Reset
            </n-button>
          </n-gi>
          <n-gi :span="24" class="mt-2">
            <n-data-table
              :columns="columns"
              :data="filteredData"
              :single-line="false"
              :pagination="{
                pageSize,
                pageSlot: 5,
              }"
              id="data-table"
              ref="dataTable"
            />
          </n-gi>
        </n-grid>
      </div>
    </n-collapse-transition>
  </div>
</template>

<style scoped>
.container {
  position: fixed;
  border: 1px solid #ddd;
  border-radius: 10px;
  width: 600px;
  padding: 10px 0px 0px 10px;
  background-color: aliceblue;
}
.moving {
  box-shadow: 0 0 10px #ddd;
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
#data-table:deep(.n-data-table__pagination) {
  margin-top: 9px;
}
#data-table:deep(.n-data-table-empty) {
  padding-top: 100px;
  padding-bottom: 100px;
}
</style>
