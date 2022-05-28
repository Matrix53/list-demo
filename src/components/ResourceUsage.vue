<template>
  <div
    :class="[
      'fixed rounded-[10px] border border-solid border-gray-200 p-[10px] bg-blue-50 transition-[width] duration-300',
      isBlockHidden ? 'w-[485px]' : 'w-[600px]',
    ]"
    :style="style"
    ref="container"
  >
    <div class="grid grid-cols-2">
      <div class="flex justify-start items-center">
        <h2 class="m-0 ml-3 font-normal text-xl cursor-default">
          Resource Usage
        </h2>
      </div>
      <div class="flex justify-end items-center">
        <span
          class="mr-1 round-shadow h-6 w-6 p-1 cursor-pointer select-none"
          @click="onLock"
          data-drag-protected
        >
          <Icon size="20" color="#2080f0FF">
            <LockOpenOutline v-if="isDraggable" />
            <LockClosedOutline v-else />
          </Icon>
        </span>
        <span
          class="round-shadow h-6 w-6 p-1 cursor-pointer select-none"
          @click="onFold"
          data-drag-protected
        >
          <Icon size="24" color="#2080f0FF">
            <CaretBackCircleOutline v-if="isFolded" />
            <CaretDownCircleOutline v-else />
          </Icon>
        </span>
      </div>
    </div>
    <div
      ref="tableContainer"
      :class="[
        'overflow-hidden transition-all duration-300',
        isFolded ? 'opacity-0' : 'opacity-100',
      ]"
      :style="{ height: isFolded ? '0px' : storedHeight + 'px' }"
    >
      <div class="mt-1 grid grid-cols-3">
        <div class="flex justify-start items-center">
          <el-switch
            v-model="isBlockHidden"
            active-text="Hide Blocked"
            class="ml-3"
            data-drag-protected
          >
          </el-switch>
        </div>
        <div class="col-span-2 flex justify-end items-center">
          <el-input
            :placeholder="
              isBlockHidden ? 'Clus. or Dept.' : 'Search Clus. or Dept.'
            "
            v-model="inputText"
            size="medium"
            :class="[
              'mr-2 transition-[width] duration-300',
              isBlockHidden ? 'w-[200px]' : 'w-[250px]',
            ]"
            @keyup.enter="onSearch"
            data-drag-protected
          >
            <el-button
              slot="append"
              id="search-btn"
              @click="onSearch"
              data-drag-protected
            >
              Search
            </el-button>
          </el-input>
          <el-button
            size="medium"
            class="mr-[2px] px-[12px]"
            @click="onReset"
            data-drag-protected
          >
            Reset
          </el-button>
        </div>
      </div>
      <div class="mt-2">
        <el-table
          border
          empty-text="No Data"
          :class="{ 'block-hidden': isBlockHidden }"
          data-drag-protected
        >
          <el-table-column prop="cluster" label="Cluster" width="110">
          </el-table-column>
          <el-table-column prop="department" label="Department" width="144">
          </el-table-column>
          <el-table-column prop="reserved" label="Reserved" width="115">
          </el-table-column>
          <el-table-column prop="spotUsed" label="SpotUsed" width="115">
          </el-table-column>
          <el-table-column prop="blocked" label="Blocked" width="115">
          </el-table-column>
        </el-table>
      </div>
      <div class="mt-1 flex justify-end">
        <el-pagination
          background
          layout="prev, pager, next"
          class="relative left-2 pb-0"
          data-drag-protected
        >
        </el-pagination>
      </div>
    </div>
  </div>
</template>

<script>
import { defineComponent } from '@vue/runtime-core'
import { ref, computed } from '@vue/composition-api'
import { useDraggable, useIntervalFn, useElementSize } from '@vueuse/core'
import {
  CaretBackCircleOutline,
  CaretDownCircleOutline,
  LockClosedOutline,
  LockOpenOutline,
} from '@v2icons/ionicons5'
import { Icon } from '@v2icons/utils'

export default defineComponent({
  name: 'ResourceUsage',
  props: {
    initX: {
      type: Number,
      default: 300,
    },
    initY: {
      type: Number,
      default: 150,
    },
  },
  components: {
    CaretBackCircleOutline,
    CaretDownCircleOutline,
    LockClosedOutline,
    LockOpenOutline,
    Icon,
  },
  setup(props) {
    const container = ref(null)
    const tableContainer = ref(null)
    const dataTable = ref(null)
    const isDraggable = ref(true)
    const isFolded = ref(false)
    const isBlockHidden = ref(true)
    const inputText = ref('')
    const keyWord = ref(/.*/)
    const rawData = ref([])
    const storedHeight = ref(191)

    const { style } = useDraggable(container, {
      initialValue: { x: props.initX, y: props.initY },
      preventDefault: true,
      stopPropagation: true,
      onStart(position, event) {
        if (
          !isFolded.value &&
          position.x + 15 >= container.value.clientWidth &&
          position.y + 15 >= container.value.clientHeight
        )
          return false
        if (!isDraggable.value) return false
        for (
          let tmp = event.target;
          tmp !== container.value;
          tmp = tmp.parentElement
        ) {
          if (tmp.dataset.dragProtected !== undefined) return false
        }
        container.value.classList.add('moving')
        event.target.style.cursor = 'move'
      },
      onEnd(_position, event) {
        container.value?.classList.remove('moving')
        event.target.style.cursor = ''
      },
    })
    useIntervalFn(getTableData, 2500, {
      immediateCallback: true,
    })
    const { height } = useElementSize(tableContainer)

    function onLock() {
      isDraggable.value = !isDraggable.value
    }
    function onFold() {
      if (!isFolded.value)
        storedHeight.value = tableContainer.value.scrollHeight
      isFolded.value = !isFolded.value
    }
    function onSearch() {
      keyWord.value = new RegExp(inputText.value.trim().split(/\s+/).join('|'))
    }
    function onReset() {
      inputText.value = ''
      keyWord.value = /.*/
      // dataTable.value?.clearSorter()
      // dataTable.value?.page(1)
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
      let res = {}
      clusterList.forEach((cluster) => {
        let tmp = {}
        departmentList.forEach((department) => {
          let total = Math.floor(Math.random() * 100)
          Object.defineProperty(tmp, department, {
            value: {
              RESERVED_TOTAL: total,
              RESERVED_USED: Math.floor(Math.random() * total) + 1,
              RESERVED_IDLE: Math.floor(Math.random() * total) + 1,
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
      let randomData = []
      for (let cluster in mockData) {
        for (let department in mockData[cluster]) {
          randomData.push({
            cluster,
            department,
            reservedUsed: mockData[cluster][department].RESERVED_USED,
            reserved: mockData[cluster][department].RESERVED_TOTAL,
            spotUsed: mockData[cluster][department].SPOT_USED,
            blocked: mockData[cluster][department].RESERVED_BLOCKED,
            index: 0,
          })
        }
      }
      randomData.sort((a, b) => {
        return (
          a.cluster.localeCompare(b.cluster) ||
          a.department.localeCompare(b.department)
        )
      })
      randomData.forEach((item, index) => {
        item.index = index
      })
      rawData.value = randomData
    }

    return {
      container,
      tableContainer,
      storedHeight,
      isDraggable,
      isFolded,
      isBlockHidden,
      inputText,
      style,
      onSearch,
      onReset,
      onLock,
      onFold,
    }
  },
})
</script>

<style scoped>
.round-shadow {
  @apply rounded-full flex justify-center items-center transition duration-300 hover:bg-slate-300/50;
}
#search-btn {
  @apply px-[12px] pt-[11px] pb-[10px] rounded-l-none text-white bg-[#409EFF] hover:bg-[#66b1ff];
}
.moving {
  @apply shadow-lg;
}
.el-table >>> .el-table__body-wrapper {
  @apply overflow-x-hidden;
}
.el-table >>> .el-table__empty-text {
  @apply relative right-0 transition-[right] duration-300;
}
.block-hidden >>> .el-table__empty-text {
  @apply right-[45px];
}
</style>
