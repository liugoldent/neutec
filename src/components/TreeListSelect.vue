<template>
  <div class="tree-select">
    <select v-model="selectedItem" @change="selectItemFunc2">
      <option value="">請選擇</option>
      <template v-for="item in data" :key="item.key">
        <option :value="item">{{ item.text }}</option>
        <template v-if="validateSelectItemShow(item.key)">
          <template v-for="(item, index) in selectArr" :key="`s-${index}`">
            <option v-if="index !== 0" :value="item">{{ item.text }}</option>
          </template>
        </template>
      </template>
    </select>
  </div>
</template>

<script>
import { onMounted, ref } from 'vue'
export default {
  components: {},
  props: {
    treeData: {
      type: Array
    }
  },
  setup(props) {
    const selectedItem = ref({})

    const cloneTreeData = ref([])
    onMounted(() => {
      cloneTreeData.value = JSON.parse(JSON.stringify(props.treeData))
      cloneTreeData.value.forEach((item, index) => {
        item.no = `${index + 1}`
      })
    })

    const selectArr = ref([])
    const selectArrMapText = ref([])
    const selectItemFunc2 = function () {
      const { no } = selectedItem.value
      // 代表根源，所以要清空
      if (!no.includes('-')) {
        selectArr.value = []
      }
      // 清空後為0，代表是新的根
      // if
      // 1. 放入選到的value
      // 2. 放入其children
      // else
      // 找是否有更深的值，有更深的就砍掉
      // 找到要插入的index，如果有children代表可以往下加入，如果沒有則這就是底了
      if (selectArr.value.length === 0) {
        selectArr.value.push(selectedItem.value)
        selectedItem.value.children.forEach((item, index) => {
          item.no = `${no}-${index + 1}`
          selectArr.value.push(item)
        })
      } else {
        const deepStatus = selectArr.value.some((item) => item.no.length > no.length)
        if (deepStatus) {
          selectArr.value = selectArr.value.filter((item) => item.no.length <= no.length)
        }
        let addIndex = selectArr.value.findIndex((item) => item.no === no)
        if (selectedItem.value.children) {
          selectedItem.value.children.forEach((item, index) => {
            item.no = `${no}-${index + 1}`
            selectArr.value.splice(addIndex + 1, 0, item)
            addIndex++
          })
        }
      }
      selectArrMapText.value = selectArr.value.map((item) => item.key)
    }
    // 驗證資料這次資料是否有被選中，如果沒被選中，則不要顯示子層資料
    const validateSelectItemShow = function (key) {
      if (selectArr.value.length === 0) {
        return true
      }
      return selectArrMapText.value.includes(key)
    }
    return {
      data: cloneTreeData,
      selectedItem,
      selectArr,
      validateSelectItemShow,
      selectItemFunc2
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
