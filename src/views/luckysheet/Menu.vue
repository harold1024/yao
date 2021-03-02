<template>
  <div
    class="context-menu"
    :style="{ left: `${contextPos.l}px`, top: `${contextPos.t}px` }"
  >
    <ul>
      <li v-for="(item, index) in operations" :key="index">
        <a @click="menuItemClick(item.code)">
          {{ item.name }}
        </a>
      </li>
    </ul>
  </div>
</template>
<script>
export default {
  props: {
    operations: {
      type: Array,
      default: () => [
        {
          name: '添加行',
          code: 'addRow'
        },
        {
          name: '添加模块',
          code: 'merge'
        },
        {
          name: '删除行',
          code: 'delRow'
        },
        {
          name: '删除模块',
          code: 'merge'
        }
        // {
        //   name: '拆分',
        //   code: 'split'
        // },
        // {
        //   name: '删除行',
        //   code: 'delRow'
        // },
        // {
        //   name: "删除列",
        //   code: "delCol",
        // },
        // {
        //   name: '添加行',
        //   code: 'addRow'
        // },
        // {
        //   name: '添加列',
        //   code: 'addCol'
        // },
        // {
        //   name: '清空选择',
        //   code: 'clearSelection'
        // }
      ]
    },
    parentTableData: {
      type: Object,
      default: () => {}
    },
    contextPos: {
      type: Object,
      default: () => ({ l: 0, t: 0 })
    }
  },
  mounted() {},
  methods: {
    menuItemClick(cmd) {
      this.$emit('menyItemCmd', cmd)
    }
  }
}
</script>
<style lang="scss" scoped>
.context-menu {
  position: fixed;
  width: 96px;
  background-color: #fff;
  z-index: 100;
  border: 1px solid rgba(0, 0, 0, 0.15);
  font-size: 12px;
  border-radius: 6px;
  box-shadow: 5px 5px 5px 0px rgba(0, 0, 0, 0.2);

  ul {
    margin: 0;
    padding: 0;

    li {
      width: 100%;
      height: 26px;
      text-align: center;
      list-style: none;
      &:nth-child(2n + 3) {
        border-top: 1px solid rgba(0, 0, 0, 0.15);
      }
      a {
        padding: 5px 20px;
        display: block;
        color: #666;
        line-height: 20px;
        &.disabled {
          cursor: not-allowed;
        }
        &:hover:not(.disabled) {
          background-color: #42b983;
          color: #fff;
        }
      }
    }
  }
}
</style>
