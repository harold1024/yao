<template>
  <div
    class="context-menu"
    :style="{ left: `${contextPos.l}px`, top: `${contextPos.t}px` }"
  >
    <ul v-if="contextPos.c == 1">
      <li v-for="(item, index) in operations" :key="index">
        <a @click="menuItemClick(item.code)">
          {{ item.name }}
        </a>
      </li>
    </ul>
    <ul v-else>
      <li v-for="(item, index) in operationsA" :key="index">
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
          name: "向上添加模块",
          code: "upAddModular",
        },
        {
          name: "向下添加模块",
          code: "downAddModular",
        },
        {
          name: "删除模块",
          code: "delModular",
        },
      ],
    },
    operationsA: {
      type: Array,
      default: () => [
        {
          name: "向上添加行",
          code: "upAddRow",
        },
        {
          name: "向下添加行",
          code: "downAddRow",
        },
        {
          name: "删除行",
          code: "delRow",
        },
      ],
    },
    parentTableData: {
      type: Object,
      default: () => {},
    },
    contextPos: {
      type: Object,
      default: () => ({ l: 0, t: 0, c: "" }),
    },
  },
  mounted() {},
  methods: {
    menuItemClick(cmd) {
      this.$emit("menyItemCmd", cmd);
    },
  },
};
</script>
<style lang="scss" scoped>
.context-menu {
  position: fixed;
  width: 130px;
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
