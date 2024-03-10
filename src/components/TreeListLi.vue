<template>
  <ul>
    <li v-for="item in data" :key="item.key">
      <span @click="toggleTree(item)">{{ item.text }}</span>
      <tree-list-li :tree-data="item.children" v-if="item.children && item.isOpen" />
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
  setup(props) {
    const cloneTreeData = ref([])
    cloneTreeData.value = JSON.parse(JSON.stringify(props.treeData))
    const toggleTree = (item) => {
      if (item.children) {
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
    }
    return {
      data: cloneTreeData,
      toggleTree
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
}
span {
  display: inline-block;
  margin-bottom: 10px;
}
</style>
