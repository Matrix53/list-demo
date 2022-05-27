<template>
  <div class="container" ref="container">
    <div class="h-60 w-60"></div>
    <div ref="tableContainer"></div>
  </div>
</template>

<script>
import { defineComponent } from '@vue/runtime-core'
import { ref, onMounted, computed, onUpdated } from '@vue/composition-api'
import { useDraggable, useIntervalFn, useElementSize } from '@vueuse/core'
import {
  CaretBackCircleOutline,
  CaretDownCircleOutline,
  LockClosedOutline,
  LockOpenOutline,
} from '@v2icons/ionicons5'

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
    const storedHeight = ref(421)

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
      let mockData = this.generateData()
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

    return { container, tableContainer }
  },
})
</script>

<style scoped>
.container {
  position: fixed;
  border: 1px solid #ddd;
  border-radius: 10px;
  width: 485px;
  padding: 10px 0px 0px 10px;
  background-color: aliceblue;
}
</style>
