<template>
  <div
    :class="[
      'fixed border border-solid border-gray-200 p-[10px] bg-blue-50 transition-[width] duration-300',
      isBlockHidden ? 'w-[485px]' : 'w-[600px]',
    ]"
    :style="style"
    ref="container"
  >
    <div class="grid grid-cols-2">
      <div class="flex justify-start items-center">
        <h2 class="m-0 ml-1 font-normal text-xl cursor-default">
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
      :class="[
        'overflow-hidden transition-[opacity] duration-300',
        isFolded ? 'opacity-0' : 'opacity-100',
        { folding: isFolding },
      ]"
      :style="{ height: isFolded ? '0px' : storedHeight + 'px' }"
      ref="content"
    >
      <div class="mt-1 grid grid-cols-3">
        <div class="flex justify-start items-center">
          <el-button-group data-drag-protected class="ml-1">
            <el-button
              icon="el-icon-plus"
              size="small"
              @click="onAddPageSize"
              plain
            >
            </el-button>
            <el-button
              icon="el-icon-minus"
              size="small"
              @click="onSubPageSize"
              plain
            >
            </el-button>
          </el-button-group>
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
            @keyup.native.enter="onSearch"
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
          :data="tableData"
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
      <div class="mt-1 flex justify-end items-center">
        <el-pagination
          background
          :page-size="pageSize"
          :current-page="currentPage"
          :page-count="pageCount"
          :pager-count="5"
          @current-change="onPageChange"
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
import { useDraggable, useIntervalFn, useDebounceFn } from '@vueuse/core'
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
    const content = ref(null)
    const isDraggable = ref(true)
    const isFolded = ref(false)
    const isFolding = ref(false)
    const isBlockHidden = ref(true)
    const inputText = ref('')
    const keyWord = ref(/.*/)
    const rawData = ref([])
    const storedHeight = ref(275)
    const currentPage = ref(1)
    const pageSize = ref(3)

    const { style } = useDraggable(container, {
      initialValue: { x: props.initX, y: props.initY },
      preventDefault: true,
      stopPropagation: true,
      onStart(_position, event) {
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
        container.value.classList.remove('moving')
        event.target.style.cursor = ''
      },
    })
    const endFolding = useDebounceFn(() => {
      isFolding.value = false
    }, 310)
    useIntervalFn(getTableData, 2500, {
      immediateCallback: true,
    })

    const filteredData = computed(() => {
      return rawData.value.filter((item) => {
        return (
          item.cluster.match(keyWord.value) ||
          item.department.match(keyWord.value)
        )
      })
    })
    const pageCount = computed(() => {
      return Math.ceil(filteredData.value.length / pageSize.value)
    })
    const tableData = computed(() => {
      let page = Math.min(currentPage.value, pageCount.value)
      return filteredData.value.slice(
        (page - 1) * pageSize.value,
        page * pageSize.value
      )
    })

    function onLock() {
      isDraggable.value = !isDraggable.value
    }
    function onFold() {
      isFolding.value = true
      if (!isFolded.value) storedHeight.value = content.value.scrollHeight
      isFolded.value = !isFolded.value
      endFolding()
    }
    function onSearch() {
      keyWord.value = new RegExp(inputText.value.trim().split(/\s+/).join('|'))
      currentPage.value = 1
    }
    function onReset() {
      inputText.value = ''
      keyWord.value = /.*/
      currentPage.value = 1
    }
    function onPageChange(page) {
      currentPage.value = page
    }
    function onAddPageSize() {
      if (pageSize.value < 15) {
        pageSize.value++
        storedHeight.value += 48
      }
    }
    function onSubPageSize() {
      if (pageSize.value > 3) {
        pageSize.value--
        storedHeight.value -= 48
      }
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
      content,
      storedHeight,
      isDraggable,
      isFolded,
      isFolding,
      isBlockHidden,
      inputText,
      style,
      currentPage,
      pageSize,
      pageCount,
      tableData,
      onSearch,
      onReset,
      onLock,
      onFold,
      onPageChange,
      onAddPageSize,
      onSubPageSize,
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
.folding {
  @apply transition-[opacity,height] duration-300;
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
.el-button-group >>> i {
  @apply font-semibold;
}
</style>
