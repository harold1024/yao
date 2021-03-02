<template>
  <div class="container">
    <Menu v-if="showMenu" :context-pos="contextPos" @menyItemCmd="menyItemCmd" />
    <table class="table">
      <tbody class="table-body">
        <tr
          v-for="(row, index) in tableData.rows"
          :key="index"
          class="table-list"
        >
          <td
            v-for="(col, index) in tableData.cols"
            v-show="isNeedShow(row - 1, col - 1)"
            :key="index"
            class="table-item"
            :class="{
              selected: selectedCells.includes(
                (row - 1) * tableData.cols + col - 1
              ),
            }"
            :colspan="
              tableData.layoutDetail[(row - 1) * tableData.cols + col - 1] &&
                tableData.layoutDetail[(row - 1) * tableData.cols + col - 1][
                  'colSpan'
                ]
            "
            :rowspan="
              tableData.layoutDetail[(row - 1) * tableData.cols + col - 1] &&
                tableData.layoutDetail[(row - 1) * tableData.cols + col - 1][
                  'rowSpan'
                ]
            "
            :contentEditable="{
              contentEditable: selectedCells1.includes(
                (row - 1) * tableData.cols + col - 1
              ),
            }"
            @mousedown="(e) => handleCellMousedown(e, row, col)"
            @mouseenter.stop.prevent="handleCellMouseenter(row, col)"
            @mouseup="handleMouseUp"
            @contextmenu.prevent.stop="handleContendMenu"
            @focus="isLocked = true"
            @blur="isLocked = false"
            @input="changeText"
            @dblclick="(e) => dblclick(e, row, col)"
            v-html="row"
          >
            >
            {{ row }}--{{ col }}
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
<script>
import Menu from './Menu'
export default {
  components: {
    Menu
  },
  data() {
    return {
      innerText: '',
      isLocked: false,
	  contentEditable: false,
	  selectedCells1: [],
      // 这块其实初始设置 tableData： {cols: 3, rows: 2} 就可以 把tabelDate设置成计算属性，layoutDetail 用js生成更方便
      tableData: {
        cols: 3,
        rows: 2,
        layoutDetail: [
          {
            colSpan: 1,
            rowSpan: 1
          },
          {
            colSpan: 1,
            rowSpan: 1
          },
          {
            colSpan: 1,
            rowSpan: 1
          },
          {
            colSpan: 1,
            rowSpan: 1
          },
          {
            colSpan: 1,
            rowSpan: 1
          },
          {
            colSpan: 1,
            rowSpan: 1
          }
        ]
      },
      selectedCells: [],
      // mousedown的时候设置为其他值 否则都是-1
      selectionHold: -1,
      startX: -1,
      startY: -1,
      endX: -1,
      endY: -1,
      showMenu: false,
      contextPos: {
        l: 0,
        t: 0
      }
    }
  },
  computed: {
    hiddenTdMaps() {
      const hiddenTdMaps = {}
      const tableData = this.tableData
      for (let i = 0; i < tableData.rows; i++) {
        for (let j = 0; j < tableData.cols; j++) {
          if (tableData.layoutDetail[i * tableData.cols + j]) {
            const colInfo = tableData.layoutDetail[i * tableData.cols + j]
            if (
              (colInfo.colSpan && colInfo.colSpan > 1) ||
              (colInfo.rowSpan && colInfo.rowSpan > 1)
            ) {
              for (let row = i; row < i + (colInfo.rowSpan || 1); row++) {
                // col = (row === i ? j + 1 : j) 是为了避开自己
                for (
                  let col = row === i ? j + 1 : j;
                  col < j + (colInfo.colSpan || 1);
                  col++
                ) {
                  hiddenTdMaps[`${row}_${col}`] = true
                }
              }
            }
          }
        }
      }
      return hiddenTdMaps
    }
  },
  watch: {
    value() {
      if (!this.isLocked || !this.innerText) {
        this.innerText = this.value
      }
    }
  },
  created() {},
  methods: {
    changeText() {
      this.$emit('input', this.$el.innerHTML)
    },
    dblclick(e, x, y) {
      console.log(12333)
      const cellIndex = (x - 1) * this.tableData.cols + y - 1
      this.selectedCells1 = [cellIndex]
      // this.contentEditable = true;
    },
    clearSelection() {
      this.selectedCells = []
    },
    changeShowMenu() {
      this.showMenu = !this.showMenu
    },
    isNeedShow(row, col) {
      return !this.hiddenTdMaps[`${row}_${col}`]
    },
    handleCellMousedown(e, x, y) {
      this.showMenu = false
      if (e.which === 1) {
        this.endX = this.endY = -1
      }
      // e.witch = 1 是鼠标左键
      if (e.which !== 1) return
      const cellIndex = (x - 1) * this.tableData.cols + y - 1
      this.startX = x
      this.startY = y
      this.selectedCells = [cellIndex]
      // mousedown标志
      this.selectionHold = cellIndex
    },
    handleCellMouseenter(x, y) {
      if (this.selectionHold !== -1) {
        this.endX = x
        this.endY = y
        this.rendSelectedCell()
      }
    },
    rendSelectedCell() {
      const startX = Math.min(this.startX, this.endX)
      const startY = Math.min(this.startY, this.endY)
      const endX = Math.max(this.startX, this.endX)
      const endY = Math.max(this.startY, this.endY)
      const tableData = this.tableData
      const selectedCells = []
      for (let row = 1; row <= tableData.cols; row++) {
        for (let col = 1; col <= tableData.cols; col++) {
          if (row >= startX && row <= endX && col >= startY && col <= endY) {
            selectedCells.push((row - 1) * this.tableData.cols + col - 1)
          }
        }
      }
      this.selectedCells = selectedCells
    },
    handleMouseUp(e) {
      this.selectionHold = -1
    },
    handleContendMenu(e) {
      console.log(e)
      this.showMenu = true
      this.contextPos.l = e.pageX
      this.contextPos.t = e.pageY
    },
    reRenderTableLayout() {
      const arr = []
      for (let i = 0; i < this.tableData.cols * this.tableData.rows; i++) {
        arr.push({
          colSpan: '',
          rowSpan: ''
        })
      }
      this.tableData.layoutDetail = arr
    },
    menyItemCmd(cmd) {
      const tableData = this.tableData
      const startX = Math.min(this.startX, this.endX)
      const startY = Math.min(this.startY, this.endY)
      const endX = Math.max(this.startX, this.endX)
      const endY = Math.max(this.startY, this.endY)
      switch (cmd) {
        case 'merge':
          if (
            startX === -1 ||
            startY === -1 ||
            endX === -1 ||
            endY === -1 ||
            (startX === endX && startY === endY)
          ) { return }
          const startIndex = (startX - 1) * tableData.cols + startY - 1
          this.tableData.layoutDetail[startIndex].rowSpan = endX - startX + 1
          this.tableData.layoutDetail[startIndex].colSpan = endY - startY + 1
          break
        case 'split':
          this.tableData.layoutDetail.forEach((v) => {
            v.rowSpan = 1
            v.colSpan = 1
          })
          break
        case 'delRow':
          this.tableData.rows = this.tableData.rows - 1
          // 行号 列号变化时候  需要重新渲染 this.tableData.layoutDetail
          this.reRenderTableLayout()
          break
        case 'delCol':
          this.tableData.cols = this.tableData.cols - 1
          this.reRenderTableLayout()
          break
        case 'addRow':
          this.tableData.rows = this.tableData.rows + 1
          this.reRenderTableLayout()
          break
        case 'addCol':
          this.tableData.cols = this.tableData.cols + 1
          this.reRenderTableLayout()
          break
        case 'clearSelection':
          this.clearSelection()
          break
      }
      this.changeShowMenu()
    }
  }
}
</script>

<style lang='scss'>
.container {
  -webkit-touch-callout: none; /* iOS Safari */
  -webkit-user-select: none; /* Chrome/Safari/Opera */
  -khtml-user-select: none; /* Konqueror */
  -moz-user-select: none; /* Firefox */
  -ms-user-select: none; /* Internet Explorer/Edge */
  user-select: none; /* Non-prefixed version, currently*/

  .table {
    border-collapse: collapse;
    width: 100%;

    .table-body {
      .table-list {
        height: 50px;
        line-height: 50px;

        &:hover {
          //  background:#eef1f6;
        }
        td {
          padding: 0 10px;
          border: 1px solid #dfe6ec;

          &.selected {
            background: #d4e7f5;
          }
        }
      }
    }
  }
}
</style>
