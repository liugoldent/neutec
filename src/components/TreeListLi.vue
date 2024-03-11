<template>
  <ul>
    <li v-for="item in data" :key="item.key">
      <span @click="toggleTree(item)" :class="{ [$style.highLight]: item.highlight }"
        >{{ item.text }}</span
      >
      <tree-list-li
        :tree-data="item.children"
        v-if="item.children && item.isOpen"
        @cancelFatherHightLight="cancelFatherHightLight(item)"
      />
    </li>
  </ul>
</template>

<script>
import { ref } from 'vue'
export default {
  components: {},
  props: {
    treeData: {
      type: Array
    }
  },
  setup(props, { emit }) {
    const cloneTreeData = ref([])
    cloneTreeData.value = JSON.parse(JSON.stringify(props.treeData))
    const toggleTree = (item) => {
      clearHighlight(cloneTreeData.value) // 先清掉這組件所有的hightLight
      emit('cancelFatherHightLight') // 再清掉父元素的hightLight
      item.highlight = true // 最後此次hightLight高亮
      if (!item.isOpen) {
        // 如果點擊的是已經開啟的項目，則直接關閉
        cloneTreeData.value.forEach((i) => {
          if (i !== item && i.isOpen) {
            i.isOpen = false
          }
        })
      }
      item.isOpen = !item.isOpen // 切換 isOpen 屬性
    }
    // 因為toggle是針對children做切換與hightLight，所以也要emit上去讓父資料不要有hightLight
    const cancelFatherHightLight = function (item) {
      item.highlight = false
    }
    // 將本次的所有資料都先不要hightLight
    const clearHighlight = function (items) {
      items.forEach((item) => {
        item.highlight = false
        if (item.children) {
          clearHighlight(item.children)
        }
      })
    }
    return {
      data: cloneTreeData,
      toggleTree,
      cancelFatherHightLight
    }
  }
}
</script>

<style lang="scss" module>
ul {
  list-style-type: none;
  padding-left: 20px;
}
li {
  margin-bottom: 10px;
  color: white;
}
span {
  display: inline-block;
  margin-bottom: 10px;
}
// 點選到的項目，要highLight
.highLight {
  color: yellow;
}
</style>
