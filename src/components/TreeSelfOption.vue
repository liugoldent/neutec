<template>
  <div :class="$style.treeDisplay">
    <h1>100層children資料</h1>

    <select v-model="rootHasSelectedModel[0]" @change="handleItemChange(0)">
      <option value="">請選擇</option>
      <option v-for="item in props.treeData" :value="item.key" :key="item.key">
        {{ item.text }}
      </option>
    </select>
    <template v-for="(selectedItem, index) in childCanSelectedItems">
      {{ index }}
      <template v-if="true">
        <select
          v-model="childHasSelectedModel[index]"
          @change="handleItemChangeForChild(index, selectedItem.children)"
          :key="index"
        >
          <option value="">請選擇</option>
          <option
            v-for="childItem in selectedItem.children"
            :value="childItem.key"
            :key="childItem.key"
          >
            {{ childItem.text }}
          </option>
        </select>
      </template>
    </template>
  </div>
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
    const rootHasSelectedModel = ref([]) // 父層選到的內容
    const childHasSelectedModel = ref([]) // 父層選到的內容
    const childCanSelectedItems = ref([]) // 子層可以選的內容
    const handleItemChange = function (index) {
      if (index === 0) {
        const item = props.treeData.find((item) => item.key === rootHasSelectedModel.value[0])
        rootHasSelectedModel.value = [item.key, null]
        childCanSelectedItems.value = []
        childCanSelectedItems.value.push(item)
      }
    }
    const handleItemChangeForChild = function (index, selectedItems) {
      // 代表是新選擇
      if (index + 1 === childCanSelectedItems.value.length) {
        const item = childCanSelectedItems.value[index].children.find((item) => {
          return item.key === childHasSelectedModel.value[index]
        })
        // 這邊可以去打API，如果有children再Loading
        if (item.children) {
          childCanSelectedItems.value.push(item)
        }
      } else if (index + 1 <= childCanSelectedItems.value.length) {
        // 代表改變子層級，改變層級先把之後的資料砍掉，然後再改變children
        childCanSelectedItems.value.splice(index + 1)
        childHasSelectedModel.value.splice(index + 1)
        handleItemChangeForChild(index, selectedItems)
      }
    }
    return {
      props,
      rootHasSelectedModel,
      childCanSelectedItems,
      childHasSelectedModel,
      handleItemChange,
      handleItemChangeForChild
    }
  }
}
</script>
<style lang='scss' module>
$distanceOfGapMargin: 20px;
$itemDistance: 10px;
.treeDisplay {
  display: flex;
  flex-direction: column;
  margin: $itemDistance;
  border: 1px solid white;
  padding: $itemDistance;
  > h1 {
    font-size: 30px;
    margin-bottom: $itemDistance;
  }
  &__selected {
    margin-bottom: $distanceOfGapMargin;
    > p {
      margin-bottom: $itemDistance;
    }
    &__li {
      margin-bottom: $itemDistance;
    }
  }
  &__canSelected {
    > p,
    div {
      margin-bottom: $itemDistance;
    }
  }
}
</style>



