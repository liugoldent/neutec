<template>
  <ul>
    <li v-for="(item, index) in data" :key="item.key">
      <span @click="toggleTree(item)" :class="{ [$style.highLight]: item.highlight }"
        >{{ item.text }}{{ item.isOpen }}</span
      >
      <tree-list-li
        :recur-index="item.no"
        :tree-data="item.children"
        v-if="item.children && item.isOpen"
        @cancelFatherHightLight="cancelFatherHightLight(item)"
        @updateTreeNewValue="updateTreeNewValue"
      />
    </li>
  </ul>
</template>

<script>
import { ref, watch, watchEffect, onMounted, inject } from 'vue'
export default {
  components: {},
  props: {
    treeData: {
      type: Array
    },
    recurIndex: {
      type: String,
      default: `0`
    }
  },
  setup(props, { emit }) {
    const cloneTreeData = ref([])
    cloneTreeData.value = JSON.parse(JSON.stringify(props.treeData))
    onMounted(() => {
      cloneTreeData.value.forEach((item, index) => {
        item.no = `${props.recurIndex}-${index}`
      })
    })
    const clickItem = ref({})
    const toggleTree = (item) => {
      clickItem.value = item
      // 先清掉這組件所有的hightLight
      clearHighlight(cloneTreeData.value)
      // 再清掉父元素的hightLight
      emit('cancelFatherHightLight')
      // 最後此次hightLight高亮
      item.highlight = true
      // 如果點擊的是已經開啟的項目，則直接關閉
      if (!item.isOpen) {
        cloneTreeData.value.forEach((i) => {
          if (i !== item && i.isOpen) {
            i.isOpen = false
          }
        })
      }
      // 切換 isOpen 屬性
      item.isOpen = !item.isOpen
      console.log(cloneTreeData.value)
      updateTreeNewValue()
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
    const data = inject('saveArr')
    // 先刪掉這次要刪掉的點（同層的）
    // 最後再刪掉比自己no大的數據
    const findThisTimeCloseIndex = function (newObj) {
      if (!newObj) {
        return
      }
      const index = data.value.findIndex((item) => {
        return item.no.length === newObj.no.length
      })
      // 如果找到跟自己同一階層的，就砍掉同階層的
      if (index > -1) {
        data.value.splice(index, 1)
      }
      console.log('---------',data.value)
      // 如果找到比自己還長的，就砍掉
      // for (let i = data.value.length - 1; i >= 0; i--) {
      //   if (data.value[i].no.length >= newObj.no.length) {
      //     data.value.splice(i, 1)
      //   }
      // }
    }
    // 更新走過開啟的節點
    // 先刪除，之後加上
    const updateTreeNewValue = function () {
      const newObj = cloneTreeData.value.find((item) => item.isOpen)
      console.log(newObj)
      if (newObj) {
        findThisTimeCloseIndex(newObj)
        data.value.unshift(newObj)
        console.log(newObj)
        console.log(data.value)
        if(data.value.length > 1){
          data.value = data.value.filter(item => {
            console.log(item.no.length > newObj.no.length)
            return item.no.length >= newObj.no.length
          })
        }

        emit('updateTreeNewValue', data.value)
      }else{
        console.log(clickItem.value)
        data.value = data.value.filter(item => item.key !== clickItem.value.key && item.no.length < clickItem.value.no.length)
      }
      console.log(data.value)
    }

    return {
      data: cloneTreeData,
      toggleTree,
      cancelFatherHightLight,
      updateTreeNewValue
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
