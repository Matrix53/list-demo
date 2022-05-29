<template>
  <div
    :class="[
      'fixed rounded-lg border border-solid border-gray-200 p-[10px] bg-blue-50 transition-[width] duration-300',
      isBlockHidden ? 'w-[440px]' : 'w-[540px]',
    ]"
    :style="style"
    ref="container"
  >
    <div class="grid grid-cols-2">
      <div class="flex justify-start items-center">
        <h2 class="m-0 ml-1 font-normal text-xl cursor-default select-none">
          Resource Usage
        </h2>
      </div>
      <div class="flex justify-end items-center">
        <el-tooltip
          :content="isDraggable ? 'Pin' : 'Unpin'"
          :open-delay="300"
          placement="top-start"
        >
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
        </el-tooltip>
        <el-tooltip
          :content="isFolded ? 'Unfold' : 'Fold'"
          :open-delay="300"
          placement="top-start"
        >
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
        </el-tooltip>
      </div>
    </div>
    <div
      :class="[
        'flex flex-col justify-between overflow-hidden transition-[opacity] duration-300',
        isFolded ? 'opacity-0' : 'opacity-100',
        { folding: isFolding },
      ]"
      :style="{ height: isFolded ? '0px' : contentHeight + 'px' }"
      ref="content"
    >
      <div>
        <div class="mt-1 grid grid-cols-3">
          <div class="flex justify-start items-center">
            <el-button-group data-drag-protected class="ml-1">
              <el-tooltip
                content="Add Row"
                :open-delay="300"
                placement="top-start"
              >
                <el-button
                  icon="el-icon-plus"
                  size="small"
                  @click="onAddPageSize"
                  plain
                >
                </el-button>
              </el-tooltip>
              <el-tooltip
                content="Reduce Row"
                :open-delay="300"
                placement="top-start"
              >
                <el-button
                  icon="el-icon-minus"
                  size="small"
                  @click="onSubPageSize"
                  plain
                >
                </el-button>
              </el-tooltip>
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
            :span-method="addRowSpan"
            data-drag-protected
          >
            <el-table-column
              prop="cluster"
              label="Cluster"
              width="100"
              align="center"
              class-name="cluster-col"
              :resizable="false"
            >
              <template v-slot="scope">
                <el-tag
                  disable-transitions
                  :type="tagTypeList[scope.row.clusterNumber % 4]"
                >
                  {{ scope.row.cluster }}
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column
              prop="department"
              label="Department"
              width="120"
              :resizable="false"
            >
            </el-table-column>
            <el-table-column
              prop="reserved"
              label="Reserved"
              class-name="reserved-col"
              width="110"
              :resizable="false"
            >
              <template v-slot="scope">
                <div
                  class="h-full w-full pl-[10px] leading-[47px]"
                  :style="usageToStyle(scope.row.usage)"
                >
                  {{ scope.row.reserved }}
                </div>
              </template>
            </el-table-column>
            <el-table-column
              prop="spotUsed"
              label="SpotUsed"
              width="110"
              :resizable="false"
            >
              <template #header>
                <template v-if="!isBlockHidden">SpotUsed</template>
                <template v-else>
                  SpotUsed
                  <el-tooltip
                    content="Show Blocked"
                    :open-delay="300"
                    placement="top-start"
                  >
                    <el-button
                      type="text"
                      icon="el-icon-caret-right"
                      class="pt-0 pb-0 switch-btn"
                      @click="onShowBlocked"
                    >
                    </el-button>
                  </el-tooltip>
                </template>
              </template>
            </el-table-column>
            <el-table-column
              prop="blocked"
              label="Blocked"
              width="100"
              :resizable="false"
            >
              <template #header>
                <template v-if="isBlockHidden">Blocked</template>
                <template v-else>
                  Blocked
                  <el-tooltip
                    content="Hide Blocked"
                    :open-delay="300"
                    placement="top-start"
                  >
                    <el-button
                      type="text"
                      icon="el-icon-caret-left"
                      class="pt-0 pb-0 switch-btn"
                      @click="onHideBlocked"
                    >
                    </el-button>
                  </el-tooltip>
                </template>
              </template>
            </el-table-column>
          </el-table>
        </div>
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
    const contentHeight = ref(275)
    const currentPage = ref(1)
    const pageSize = ref(3)

    const tagTypeList = ['', 'success', 'info', 'warning']

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

    function onLock(event) {
      isDraggable.value = !isDraggable.value
      event.currentTarget.blur()
    }
    function onFold(event) {
      isFolding.value = true
      if (!isFolded.value) contentHeight.value = content.value.scrollHeight - 2
      isFolded.value = !isFolded.value
      endFolding()
      event.currentTarget.blur()
    }
    function onSearch() {
      keyWord.value = new RegExp(inputText.value.trim().split(/\s+/).join('|'))
      currentPage.value = 1
    }
    function onReset(event) {
      inputText.value = ''
      keyWord.value = /.*/
      currentPage.value = 1
      event.currentTarget.blur()
    }
    function onShowBlocked() {
      isBlockHidden.value = false
    }
    function onHideBlocked() {
      isBlockHidden.value = true
    }
    function onPageChange(page) {
      currentPage.value = page
    }
    function onAddPageSize(event) {
      if (pageSize.value < 15) {
        pageSize.value++
        contentHeight.value += 48
      }
      event.currentTarget.blur()
    }
    function onSubPageSize(event) {
      if (pageSize.value > 3) {
        pageSize.value--
        contentHeight.value -= 48
      }
      event.currentTarget.blur()
    }
    function usageToStyle(usage) {
      let percentage = 1 - usage
      let red = Math.floor(percentage * 255)
      let green = Math.floor(128 + percentage * 127)
      return {
        backgroundColor:
          percentage === 0 ? `rgb(255,88,91)` : `rgb(${red}, ${green}, 255)`,
        color: percentage > 0.6 ? '#606266' : '#fff',
      }
    }
    function addRowSpan({ rowIndex, columnIndex }) {
      if (columnIndex === 0) {
        if (
          rowIndex !== 0 &&
          tableData.value[rowIndex].cluster ===
            tableData.value[rowIndex - 1].cluster
        )
          return { rowspan: 0, colspan: 0 }
        let endIndex = rowIndex + 1
        while (endIndex < tableData.value.length) {
          if (
            tableData.value[endIndex].cluster !==
            tableData.value[rowIndex].cluster
          ) {
            break
          }
          endIndex++
        }
        return { rowspan: endIndex - rowIndex, colspan: 1 }
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
          let total = Math.floor(Math.random() * 100) + 1
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
      let originData = generateData()
      let tmpData = []
      for (let cluster in originData) {
        for (let department in originData[cluster]) {
          let tmp = originData[cluster][department]
          tmpData.push({
            cluster,
            department,
            reserved: `${tmp.RESERVED_USED}/${tmp.RESERVED_TOTAL}`,
            spotUsed: tmp.SPOT_USED,
            blocked: tmp.RESERVED_BLOCKED,
            usage: tmp.RESERVED_USED / tmp.RESERVED_TOTAL,
            clusterNumber: 0,
          })
        }
      }
      tmpData.sort((a, b) => {
        return (
          a.cluster.localeCompare(b.cluster) ||
          a.department.localeCompare(b.department)
        )
      })
      let clusterNumber = 0
      for (let i = 1; i < tmpData.length; i++) {
        if (tmpData[i].cluster !== tmpData[i - 1].cluster) {
          clusterNumber++
        }
        tmpData[i].clusterNumber = clusterNumber
      }
      rawData.value = tmpData
    }

    return {
      container,
      content,
      contentHeight,
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
      onShowBlocked,
      onHideBlocked,
      onPageChange,
      onAddPageSize,
      onSubPageSize,
      usageToStyle,
      addRowSpan,
      tagTypeList,
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
.el-input >>> .el-input-group__append {
  @apply border-r-0;
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
.el-table >>> td.reserved-col {
  @apply h-[48px] p-0;
}
.el-table >>> td.reserved-col .cell {
  @apply h-full w-full p-0;
}
.el-table >>> td.cluster-col {
  @apply py-[6px];
}
.block-hidden >>> .el-table__empty-text {
  @apply right-[45px];
}
.el-button-group >>> i {
  @apply font-semibold;
}
.switch-btn >>> i {
  @apply font-black;
}
</style>
