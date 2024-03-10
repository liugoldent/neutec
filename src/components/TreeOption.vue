<template>
  <!-- <template v-for="item in props.items" :key="item.key">
    <option :value="item">{{ item.text }}</option>
    <template v-if="selectedItem.children && item.key === selectedItem.key">
        <tree-option :items="selectedItem.children" :selected-item="props.selectedItem" />
    </template>
  </template> -->
  <template v-for="item in children" :key="item.key">
    <option :value="item">22{{ item.text }}</option>
  </template>
</template>

<script>
export default {
  props: {
    items: {
      type: Object
    },
    selectedItem: {
      type: Object
    }
  },
  setup(props) {
    const { children } = props.items.value
    return {
      props,
      children
    }
  }
}
</script>
