<template>
  <div :class="$style.list">
    <ul :class="$style.list__ul">
      <li v-for="item in data" :key="item.key" :class="$style.list__li">
        <span
          @click="toggleTree(item)"
          :class="[$style.list__span, { [$style.highLight]: item.highlight }]"
          >{{ item.text }}</span
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
  </div>
</template>

<script>
import { ref, onMounted, inject } from 'vue'
export default {
  components: {},
  props: {
    treeData: {
      type: Array
    },
    recurIndex: {
      type: String,
      default: `0`
    },
    root: {
      type: Boolean,
      default: false
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
      // 把root的加到前面（讓第0個是根源）(因為是遞迴，所以越子層越後面)
      data.value.unshift(newObj)
      // 如果是root && 長度>1，要去找是否有其他不屬於這層的進來
      if (props.root && data.value.length > 1) {
        const { no } = data.value[0]
        data.value = data.value.filter((item) => item.no.includes(no))
      }
    }
    // if 有新物件，則加入
    // 沒有新物件，則視為關閉
    const updateTreeNewValue = function () {
      const newObj = cloneTreeData.value.find((item) => item.isOpen)
      console.log('newObj: ', newObj);
      if (newObj) {
        findThisTimeCloseIndex(newObj)
      } else {
        data.value = data.value.filter(
          (item) => item.key !== clickItem.value.key && item.no.length < clickItem.value.no.length
        )
      }
      emit('updateTreeNewValue', data.value)
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
.list {
  margin: 10px 10px 0;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  > h1 {
    font-size: 30px;
    margin-bottom: 10px;
  }
  &__ul {
    list-style-type: none;
  }
  &__span {
    display: inline-block;
    margin-bottom: 10px;
  }
  &__li {
    margin-bottom: 10px;
    color: white;
  }
}
// 點選到的項目，要highLight
.highLight {
  color: yellow;
}
</style>
