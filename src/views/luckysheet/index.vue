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
      <el-table-column prop="date" label="间隔名称" class="table-column">
        <template slot-scope="scope">
          <span
            v-if="
              scope.row.index === tabClickIndex && tabClickLabel === '间隔名称'
            "
          >
            <el-input
              class="inputSty"
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
      <el-table-column prop="name" label="信号描述" class="table-column">
        <template slot-scope="scope">
          <span
            v-if="
              scope.row.index === tabClickIndex && tabClickLabel === '信号描述'
            "
          >
            <el-input
              class="inputSty"
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
      <el-table-column
        prop="address"
        label="备注"
        width="280"
        class="table-column"
      >
        <template slot-scope="scope">
          <span
            v-if="scope.row.index === tabClickIndex && tabClickLabel === '备注'"
          >
            <el-input
              class="inputSty"
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
    <el-drawer
      title="添加新模块"
      :visible.sync="drawer"
      :direction="direction"
      :before-close="handleClose"
      custom-class="demo-drawer"
      ref="drawer"
    >
      <!-- <span>我来啦!</span> -->
      <div class="demo-drawer__content">
        <el-form :model="form">
          <!-- <el-form-item label="活动名称" :label-width="formLabelWidth">
            <el-input v-model="form.name" autocomplete="off"></el-input>
          </el-form-item> -->
          <el-form-item label="选择模块" :label-width="formLabelWidth">
            <el-select v-model="form.region" placeholder="请选择新模块">
              <el-option
                v-for="item in modularOptions"
                :key="item.itemValue"
                :label="item.itemLabel"
                :value="item.itemValue"
              />
            </el-select>
          </el-form-item>
        </el-form>
        <div class="demo-drawer__footer">
          <el-button @click="cancelForm">取 消</el-button>
          <el-button
            type="primary"
            @click="submitDrawer()"
            :loading="loading"
            >{{ loading ? "提交中 ..." : "确 定" }}</el-button
          >
        </div>
      </div>
    </el-drawer>
  </div>
</template>
<script>
const modularDate = [
  {
    modular: "A",
    modularMain: [
      {
        date: "A模块",
        name: "66kV华朝甲线76314开关合/分",
        address: "",
      },
    ],
  },
  {
    modular: "B",
    modularMain: [
      {
        date: "B模块",
        name: "66kV华朝甲线76314开关合/分",
        address: "",
      },
      {
        date: "B模块",
        name: "66kV华朝甲线76314开关合/分",
        address: "",
      },
    ],
  },
  {
    modular: "C",
    modularMain: [
      {
        date: "C模块",
        name: "66kV华朝甲线76314开关合/分",
        address: "",
      },
      {
        date: "C模块",
        name: "66kV华朝甲线76314开关合/分",
        address: "",
      },
      {
        date: "C模块",
        name: "66kV华朝甲线76314开关合/分",
        address: "",
      },
    ],
  },
  {
    modular: "D",
    modularMain: [
      {
        date: "D模块",
        name: "66kV华朝甲线76314开关合/分",
        address: "",
      },
      {
        date: "D模块",
        name: "66kV华朝甲线76314开关合/分",
        address: "",
      },
      {
        date: "D模块",
        name: "66kV华朝甲线76314开关合/分",
        address: "",
      },
      {
        date: "D模块",
        name: "66kV华朝甲线76314开关合/分",
        address: "",
      },
    ],
  },
];
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
          date: "1",
          name: "66kV华朝甲线76314开关合/分",
          address: "",
        },
        {
          date: "2",
          name: "66kV华朝甲线76314开关合/分",
          address: "",
        },
        {
          date: "2",
          name: "66kV华朝甲线76314开关合/分",
          address: "",
        },
        {
          date: "3",
          name: "66kV华朝甲线76314开关合/分",
          address: "",
        },
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
        c: "",
      },
      row: "",
      column: "",
      drawer: false,
      direction: "rtl",
      formLabelWidth: "80px",
      loading: false,
      form: {
        region: "",
      },
      modularOptions: [
        {
          itemValue: "A",
          itemLabel: "A",
        },
        {
          itemValue: "B",
          itemLabel: "B",
        },
        {
          itemValue: "C",
          itemLabel: "C",
        },
        {
          itemValue: "D",
          itemLabel: "D",
        },
      ],
      AddModular:"downAddModular",
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
        // return "border: 1px solid #ccc";
      } else {
        return "background-color:#fff;";
        // return "border: 1px solid #fff;";
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
      if (column.label == "间隔名称") {
        this.contextPos.c = 1;
      } else {
        this.contextPos.c = 0;
      }
    },
    changeShowMenu() {
      this.showMenu = !this.showMenu;
    },
    menyItemCmd(cmd) {
      switch (cmd) {
        case "upAddRow":
          var newRow = {
            date: this.row.date,
            name: this.row.name,
            address: "",
          };
          this.tableData.splice(this.row.index, 0, newRow);
          this.getSpanArrFirst();
          break;
        case "downAddRow":
          var newRow = {
            date: this.row.date,
            name: this.row.name,
            address: "",
          };
          this.tableData.splice(this.row.index + 1, 0, newRow);
          this.getSpanArrFirst();
          break;
        case "delRow":
          this.tableData.splice(this.row.index, 1);
          this.getSpanArrFirst();
          break;
        case "upAddModular":
          this.drawer = true;
          this.AddModular="upAddModular"
          // var newRow = {
          //   date: this.row.date,
          //   name: this.row.name,
          //   address: "",
          // };
          // this.tableData.splice(this.row.index, 0, newRow);
          // this.getSpanArrFirst();
          break;
        case "downAddModular":
          this.drawer = true;
          this.AddModular="downAddModular"
          // var newRow = {
          //   date: this.row.date,
          //   name: this.row.name,
          //   address: "",
          // };
          // this.tableData.splice(this.row.index, 0, newRow);
          // this.getSpanArrFirst();
          break;
        case "delModular":
          // this.tableData.splice(this.row.index, 1);
          // this.getSpanArrFirst();
          // console.log(this.row);
          let num = 0;
          this.tableData.map((item) => {
            if (item.date == this.row.date) {
              num++;
            }
          });
          this.tableData.splice(this.row.index, num);
          this.getSpanArrFirst();
          break;
      }
      this.changeShowMenu();
    },
    submitDrawer(done) {
      this.$confirm("确定要添加此模块吗？")
        .then((_) => {
          this.loading = true;
          // this.form.region
          // modularDate
          var result = modularDate.filter((item) => {
            return item["modular"] == this.form.region;
          });
          let num = 0;
          this.tableData.map((item) => {
            if (item.date == this.row.date) {
              num++;
            }
          });
          if(this.AddModular == "downAddModular"){
            this.tableData.splice(this.row.index+num, 0, ...result[0].modularMain);
          } else {
            this.tableData.splice(this.row.index, 0, ...result[0].modularMain);
          }
          this.getSpanArrFirst();
          this.drawer = false;
          this.loading = false;
        })
        .catch((_) => {});
    },
    cancelForm() {
      this.loading = false;
      this.drawer = false;
    },
    handleClose(done) {
      this.$confirm("确认关闭？")
        .then((_) => {
          done();
        })
        .catch((_) => {});
    },
  },
};
</script>
<style lang="scss" scoped>
.demo-drawer__content {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding: 20px;
}
.demo-drawer__content form {
  flex: 1;
}
.demo-drawer__footer {
  display: flex;
  justify-content: flex-end;
}

::v-deep {
  .el-table--medium td {
    padding: 0 !important;
  }
  .inputSty {
    // border: 1px solid red;
    .el-input__inner {
      height: 23px;
      line-height: 23px;
      font-size: 12px;
      border: none;
      background: #ccc;
      padding: 0;
    }
  }
}
</style>
