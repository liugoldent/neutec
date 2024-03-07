<script>
export default {
  props: {
    componentType: {}
  },
  components: {},
  setup(props) {
    return {
      circleShowIndex: [1, 3, 7, 9], // 哪些方格要顯示圓圈
      fadeInOutShowIndex: [3, 5, 9] // 哪些方格要淡入淡出
    }
  }
}
</script>

<template>
  <div>
    <div :class="$style.rightDrawer"></div>
    <div :class="$style.nineSquare">
      <div :class="$style.nineSquare__mainDiv">
        <div v-for="index in 9" :key="index" :class="$style.nineSquare__repeatDiv">
          <div v-if="circleShowIndex.includes(index)" :class="$style.nineSquare__circle"></div>
          <div
            :class="[
              $style.nineSquare__eachSquare,
              { [$style.nineSquare__eachSquareAnimate]: fadeInOutShowIndex.includes(index) }
            ]"
          ></div>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang='scss' module>
$drawer-btn-height: 50px;

.rightDrawer {
  width: 100%;
  height: $drawer-btn-height;
  background-color: white;
}
.nineSquare {
  width: 100%;
  height: calc(100vh - $drawer-btn-height);
  background-color: gray;
  display: flex;
  align-items: center;

  &__mainDiv {
    flex-grow: 1;
    display: grid;
    grid-template-rows: repeat(3, 1fr);
    grid-template-columns: repeat(3, 1fr);
    grid-gap: 10px;
    grid-auto-flow: row;
    padding-left: 10px;
    padding-right: 10px;
  }
  &__repeatDiv {
    position: relative;
  }
  &__circle {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 30px;
    height: 30px;
    background-color: black;
    border-radius: 50%;
    z-index: 1;
    animation: circleMoveToRight 2s infinite;
  }
  &__eachSquare {
    height: 100px;
    border: black solid 2px;
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    background: radial-gradient(circle, rgba(113, 81, 95, 1) 81%, rgba(0, 0, 0, 1) 100%);
  }
  &__eachSquareAnimate {
    animation: squareFadeInOut 2s infinite;
  }
}

@keyframes squareFadeInOut {
  0% {
    opacity: 1;
  }
  50% {
    opacity: 0.6;
  }
  100% {
    opacity: 1;
  }
}

@keyframes circleMoveToRight {
  0% {
    left: 50%;
  }
  100% {
    left: 200%;
  }
}
</style>
