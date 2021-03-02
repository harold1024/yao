<template>
  <div class="container">
    <Menu
      v-if="showMenu"
      :context-pos="contextPos"
      @menyItemCmd="menyItemCmd"
    />
    <el-table
      :data="tableData"
      border
      height="850"
      style="width: 100%"
      :span-method="objectSpanMethod"
      :row-class-name="tableRowClassName"
      @cell-dblclick="celldblClick"
      @cell-click="cellClick"
      @row-contextmenu="handleContendMenu"
      @mouseup="handleMouseUp"
      :cell-style="tableCellStyle"
    >
      >
      <el-table-column type="index" />
      <el-table-column prop="date" label="间隔名称">
        <template slot-scope="scope">
          <span
            v-if="
              scope.row.index === tabClickIndex && tabClickLabel === '间隔名称'
            "
          >
            <el-input
              v-model="scope.row.date"
              maxlength="300"
              placeholder="请输入原因"
              size="mini"
              ref="atlas-name-input"
              @blur="inputBlur"
            />
          </span>
          <span v-else>{{ scope.row.date }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="name" label="信号描述">
        <template slot-scope="scope">
          <span
            v-if="
              scope.row.index === tabClickIndex && tabClickLabel === '信号描述'
            "
          >
            <el-input
              v-model="scope.row.name"
              placeholder="请输入信号描述"
              size="mini"
              ref="atlas-name-input"
              @blur="inputBlur"
            />
          </span>
          <span v-else>{{ scope.row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="address" label="备注" width="280">
        <template slot-scope="scope">
          <span
            v-if="scope.row.index === tabClickIndex && tabClickLabel === '备注'"
          >
            <el-input
              v-model="scope.row.address"
              maxlength="300"
              placeholder="请输入备注"
              size="mini"
              ref="atlas-name-input"
              @blur="inputBlur"
            />
          </span>
          <span v-else>{{ scope.row.address }}</span>
        </template>
      </el-table-column>
      <!-- <el-table-column label=" 单元操作" width="180">
        <template slot-scope="scope">
          <el-button
            size="mini"
            @click="handleEdit(scope.$index, scope.row)"
          >编辑</el-button>
          <el-button
            size="mini"
            type="danger"
            @click="handleDelete(scope.$index, scope.row)"
          >删除</el-button>
        </template>
      </el-table-column>
      <el-table-column label="模块操作" width="180">
        <template slot-scope="scope">
          <el-button
            size="mini"
            @click="handleEdit(scope.$index, scope.row)"
          >编辑</el-button>
          <el-button
            size="mini"
            type="danger"
            @click="handleDelete(scope.$index, scope.row)"
          >删除</el-button>
        </template>
      </el-table-column> -->
    </el-table>
    <!-- <Table /> -->
  </div>
</template>

<script>
import Menu from "./Menu";
import Table from "./Table";
export default {
  components: {
    Menu,
    Table,
  },
  data() {
    return {
      tableData: [
        {
          date: "66kV",
          name: "66kV华朝甲线76314开关合/分",
          address: "",
        },
        {
          date: "66kV华朝甲线76314",
          name: "66kV华朝甲线76314开关合/分",
          address: "",
        },
        {
          date: "66kV华朝甲线76314",
          name: "66kV华朝甲线76314乙刀闸合/分",
          address: "",
        },
        {
          date: "66kV华朝甲线76314",
          name: "66kV华朝甲线76314甲刀闸合/分",
          address: "",
        },
        {
          date: "66kV华朝甲线76314",
          name: "66kV华朝甲线76314J1接地刀闸合/分",
          address: "",
        },
        {
          date: "66kV华朝甲线76314",
          name: "66kV华朝甲线76314J2接地刀闸合/分",
          address: "",
        },
        {
          date: "66kV华朝甲线76314",
          name: "66kV华朝甲线76314J3接地刀闸合/分",
          address: "",
        },
        {
          date: "66kV华朝乙线76316",
          name: "66kV华朝甲线76314开关合/分",
          address: "",
        },
        {
          date: "66kV华朝乙线76316",
          name: "66kV华朝甲线76314乙刀闸合/分",
          address: "",
        },
        {
          date: "66kV华朝乙线76316",
          name: "66kV华朝甲线76314甲刀闸合/分",
          address: "",
        },
        {
          date: "66kV华朝乙线76316",
          name: "66kV华朝甲线76314J1接地刀闸合/分",
          address: "",
        },
        {
          date: "66kV华朝乙线76316",
          name: "66kV华朝甲线76314J2接地刀闸合/分",
          address: "",
        },
        {
          date: "66kV华朝乙线76316",
          name: "66kV华朝甲线76314J3接地刀闸合/分",
          address: "",
        },
        {
          date: "66kV分段76310",
          name: "66kV华朝甲线76314J1接地刀闸合/分",
          address: "",
        },
        {
          date: "66kV分段76310",
          name: "66kV华朝甲线76314J2接地刀闸合/分",
          address: "",
        },
        {
          date: "66kV分段76310",
          name: "66kV华朝甲线76314J3接地刀闸合/分",
          address: "",
        },
        {
          date: "一组一次76322",
          name: "66kV华朝甲线76314J3接地刀闸合/分",
          address: "",
        },
        {
          date: "一组一次76322",
          name: "66kV华朝甲线76314J1接地刀闸合/分",
          address: "",
        },
        {
          date: "一组一次76322",
          name: "66kV华朝甲线76314J2接地刀闸合/分",
          address: "",
        },
        {
          date: "一组一次76322",
          name: "66kV华朝甲线76314J3接地刀闸合/分",
          address: "",
        },
      ],
      spanArrFirst: [],
      posFirst: 0,
      tabClickIndex: null, // 点击的单元格
      tabClickLabel: "", // 当前点击的列名
      startX: -1,
      startY: -1,
      endX: -1,
      endY: -1,
      showMenu: false,
      contextPos: {
        l: 0,
        t: 0,
      },
      row: "",
      column: "",
    };
  },
  mounted() {
    this.getSpanArrFirst();
  },
  methods: {
    objectSpanMethod({ row, column, rowIndex, columnIndex }) {
      if (columnIndex === 1 || columnIndex === 5) {
        const _row = this.spanArrFirst[rowIndex];
        const _col = _row > 0 ? 1 : 0;
        return {
          rowspan: _row,
          colspan: _col,
        };
      }
    },
    // 合并一级指标
    getSpanArrFirst() {
      this.spanArrFirst = [];
      this.posFirst = 0;
      for (var i = 0; i < this.tableData.length; i++) {
        if (i === 0) {
          this.spanArrFirst.push(1);
          this.posFirst = 0;
        } else {
          // 判断当前元素与上一个元素是否相同
          if (this.tableData[i].date === this.tableData[i - 1].date) {
            this.spanArrFirst[this.posFirst] += 1;
            this.spanArrFirst.push(0);
          } else {
            this.spanArrFirst.push(1);
            this.posFirst = i;
          }
        }
      }
    },

    tableRowClassName({ row, rowIndex }) {
      // 把每一行的索引放进row
      row.index = rowIndex;
    },
    // 双击显示输入框
    celldblClick(row, column, cell, event) {
      this.showMenu = false;
      switch (column.label) {
        case "间隔名称":
          this.tabClickIndex = row.index;
          this.tabClickLabel = column.label;
          break;
        case "信号描述":
          this.tabClickIndex = row.index;
          this.tabClickLabel = column.label;
          break;
        case "备注":
          this.tabClickIndex = row.index;
          this.tabClickLabel = column.label;
          break;
        default:
          return;
      }
      this.$nextTick(() => {
        //自动获取焦点 element组件autofocus失效
        this.$refs["atlas-name-input"].$refs.input.focus();
      });
    },
    cellClick(row, column, cell, event) {
      this.showMenu = false;
      this.row = row;
      this.column = column;
    },
    tableCellStyle(row, rowIndex, column) {
      if (this.row && this.columnName) {
      }
      if (this.row === row.row && this.column === row.column) {
        return "background-color:#ccc;";
      } else {
        return "background-color:#fff;";
      }
    },
    // 失去焦点初始化
    inputBlur(row, event, column) {
      this.tabClickIndex = null;
      this.tabClickLabel = "";
    },
    handleMouseUp(e) {
      console.log(123);
      // this.selectionHold = -1;
    },
    // 右键显示
    handleContendMenu(row, column, e) {
      this.row = row;
      this.column = column;
      event.preventDefault(); // 关闭浏览器右键默认事件
      this.showMenu = true;
      this.contextPos.l = e.pageX;
      this.contextPos.t = e.pageY;
    },
    changeShowMenu() {
      this.showMenu = !this.showMenu;
    },
    menyItemCmd(cmd) {
      const tableData = this.tableData;
      const startX = Math.min(this.startX, this.endX);
      const startY = Math.min(this.startY, this.endY);
      const endX = Math.max(this.startX, this.endX);
      const endY = Math.max(this.startY, this.endY);
      switch (cmd) {
        case "merge":
          if (
            startX === -1 ||
            startY === -1 ||
            endX === -1 ||
            endY === -1 ||
            (startX === endX && startY === endY)
          ) {
            return;
          }
          const startIndex = (startX - 1) * tableData.cols + startY - 1;
          this.tableData.layoutDetail[startIndex].rowSpan = endX - startX + 1;
          this.tableData.layoutDetail[startIndex].colSpan = endY - startY + 1;
          break;
        case "split":
          this.tableData.layoutDetail.forEach((v) => {
            v.rowSpan = 1;
            v.colSpan = 1;
          });
          break;
        case "delRow":
          var newRow = {
            date: this.row.date,
            name: this.row.name,
            address: "",
          };
          this.tableData.splice(this.row.index, 1);
          this.getSpanArrFirst();
          break;
        case "delCol":
          this.tableData.cols = this.tableData.cols - 1;
          this.reRenderTableLayout();
          break;
        case "addRow":
          var newRow = {
            date: this.row.date,
            name: this.row.name,
            address: "",
          };
          this.tableData.splice(this.row.index, 0, newRow);
          this.getSpanArrFirst();
          break;
        case "addCol":
          this.tableData.cols = this.tableData.cols + 1;
          this.reRenderTableLayout();
          break;
        case "clearSelection":
          this.clearSelection();
          break;
      }
      this.changeShowMenu();
    },
  },
};
</script>
