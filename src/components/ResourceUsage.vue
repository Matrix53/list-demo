<script setup lang="tsx">
import { ref, reactive, onMounted, computed } from 'vue'
import { useDraggable, useIntervalFn } from '@vueuse/core'
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
  spotUsed: number
  block: number
}

const props = defineProps<{ initX: number; initY: number }>()

const container = ref<HTMLElement | null>(null)
const dataTable = ref<InstanceType<typeof NDataTable> | null>(null)
const isDraggable = ref(true)
const isFolded = ref(false)
const pagination = reactive({
  pageSize: 6,
  pageSlot: 5,
})
const inputText = ref('')
const keyWord = ref<RegExp>(/.*/)
const rawData = ref<RowDataItem[]>([])

const palette = [
  '#eff6ff',
  '#dbeafe',
  '#bfdbfe',
  '#93c5fd',
  '#60a5fa',
  '#3b82f6',
  '#2563eb',
]
const columns: DataTableColumn[] = [
  {
    title: 'Cluster',
    key: 'cluster',
    width: 110,
    defaultSortOrder: 'ascend',
    sorter: 'default',
  },
  {
    title: 'Department',
    key: 'department',
    sorter: 'default',
  },
  {
    title: 'Reserved',
    key: 'reserved',
    width: 115,
    sorter: createSorter('reserved', 3),
    render: renderCell('reserved'),
    cellProps: addCellProps('reserved'),
  },
  {
    title: 'SpotUsed',
    key: 'spotUsed',
    width: 115,
    sorter: createSorter('spotUsed', 2),
    render: renderCell('spotUsed'),
    cellProps: addCellProps('spotUsed'),
  },
  {
    title: 'Block',
    key: 'block',
    width: 115,
    sorter: createSorter('block', 1),
    render: renderCell('block'),
    cellProps: addCellProps('block'),
  },
]

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
const { pause, resume, isActive } = useIntervalFn(getTableData, 2500, {
  immediateCallback: true,
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

onMounted(() => {
  document
    .querySelector('#data-table .n-data-table-wrapper')
    ?.setAttribute('data-drag-protected', '')
  document
    .querySelector('#data-table .n-pagination')
    ?.setAttribute('data-drag-protected', '')
})

function onLock(event: MouseEvent) {
  isDraggable.value = !isDraggable.value
  ;(event.currentTarget as HTMLElement).blur()
}
function onFold(event: MouseEvent) {
  isFolded.value = !isFolded.value
  ;(event.currentTarget as HTMLElement).blur()
}
function onSearch() {
  keyWord.value = new RegExp(inputText.value.trim().split(/\s+/).join('|'))
}
function resetFilter() {
  inputText.value = ''
  keyWord.value = /.*/
  dataTable.value?.clearSorter()
  dataTable.value?.page(1)
}
function getTableData() {
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
  let randomData: RowDataItem[] = []
  clusterList.forEach((cluster) => {
    departmentList.forEach((department) => {
      randomData.push({
        cluster,
        department,
        reserved: Math.floor(Math.random() * 100),
        spotUsed: Math.floor(Math.random() * 100),
        block: Math.floor(Math.random() * 100),
      })
    })
  })
  rawData.value = randomData
}
function createSorter(attr: keyof RowDataItem, multiple: number) {
  return {
    compare: (row1: any, row2: any) =>
      ((row1 as RowDataItem)[attr] as number) -
      ((row2 as RowDataItem)[attr] as number),
    multiple,
  }
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
        backgroundColor: palette[index],
      },
    }
  }
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
            <n-input
              data-drag-protected
              placeholder="Search Clus. or Dept."
              v-model:value="inputText"
              type="text"
              @keyup.enter="onSearch"
            />
            <n-button data-drag-protected primary type="info" @click="onSearch">
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
            :pagination="pagination"
            id="data-table"
            ref="dataTable"
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
#data-table {
  min-height: 310px;
}
#data-table:deep(.n-data-table__pagination) {
  margin-top: 9px;
}
#data-table:deep(.n-data-table-empty) {
  padding-top: 100px;
  padding-bottom: 100px;
}
</style>
