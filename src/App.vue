<script>
import { ref, onMounted } from 'vue'
import defaultData from './assets/data.js'
import TreeList from './components/TreeList.vue'
export default {
  components: {
    TreeList
  },
  setup() {
    onMounted(() => {
      const circles = circlesRef.value
      circles.forEach((circle) => {
        if (!originalCircleLeftPlace) {
          originalCircleLeftPlace = circle.offsetLeft
        }
      })
    })
    const rightDrawer = ref(null)
    const drawerStatus = ref(false)
    // 如果點擊目標在drawer之外，則將drawer關閉
    const handleClickOutside = function (event) {
      if (!rightDrawer.value.contains(event.target)) {
        drawerStatus.value = false
      }
    }
    // 動畫類組
    // 參數
    let animateType = ref('css-to-center') // 一開始預設動畫type
    let originalCircleLeftPlace = null // 原始動畫left位子
    let timerIntervalToRIght = null // 往右邊的時間間隔
    let timerIntervalToOrigin = null // 回中間的時間間隔
    let moveXPx = ref(null)
    const circlesRef = ref(null)
    const changeAnimateType = function (clickType) {
      try {
        animateType.value = clickType
        clearInterval(timerIntervalToRIght)
        clearInterval(timerIntervalToOrigin)
        if (clickType === 'js') {
          circleAnimationToRightUseJs()
        }
      } catch (error) {
        console.error(error)
      }
    }
    const moveCirclesToOrigin = function () {
      const circles = circlesRef.value
      circles.forEach((circle) => {
        moveXPx.value = originalCircleLeftPlace
        circle.style.left = `${originalCircleLeftPlace}px`
      })
    }
    const moveCirclesToRight = function () {
      const circles = circlesRef.value
      circles.forEach((circle) => {
        let leftValue = parseInt(circle.offsetLeft) || 0
        leftValue += 2
        circle.style.left = `${leftValue}px`
      })
    }
    const circleAnimationToRightUseJs = function () {
      timerIntervalToRIght = setInterval(() => {
        moveCirclesToRight()
      }, 50)
      timerIntervalToOrigin = setInterval(() => {
        moveCirclesToOrigin()
      }, 2000)
    }
    return {
      circleShowIndex: [1, 3, 7, 9], // 哪些方格要顯示圓圈
      fadeInOutShowIndex: [3, 5, 9], // 哪些方格要淡入淡出
      drawerStatus,
      rightDrawer,
      treeData: defaultData,
      handleClickOutside,
      animateType,
      circlesRef,
      changeAnimateType,
    }
  }
}
</script>

<template>
  <div @click="handleClickOutside">
    <div :class="$style.rightDrawerBtn">
      <div>
        <button @click="changeAnimateType('css')">CSS Animate</button>
        <button @click="changeAnimateType('js')">JS Animate</button>
        <button @click="changeAnimateType('css-to-center')">CSS to Center</button>
      </div>
      <button @click.self.stop="drawerStatus = !drawerStatus">Show Drawer</button>
    </div>
    <div
      ref="rightDrawer"
      :class="[$style.mainDrawer, { [$style.mainDrawer__open]: drawerStatus }]"
    >
      <TreeList :tree-data="treeData" />
    </div>
    <div :class="$style.nineSquare">
      <div :class="$style.nineSquare__mainDiv">
        <div v-for="index in 9" :key="index" :class="$style.nineSquare__repeatDiv">
          <!-- 圓圈 -->
          <div
            v-if="circleShowIndex.includes(index)"
            ref="circlesRef"
            :class="[
              $style.nineSquare__circle,
              { [$style.nineSquare__cssAnimate]: animateType === 'css' },
              {
                [$style.nineSquare__cssToCenterAnimate__1]:
                  animateType === 'css-to-center' && index === 1
              },
              {
                [$style.nineSquare__cssToCenterAnimate__3]:
                  animateType === 'css-to-center' && index === 3
              },
              {
                [$style.nineSquare__cssToCenterAnimate__7]:
                  animateType === 'css-to-center' && index === 7
              },
              {
                [$style.nineSquare__cssToCenterAnimate__9]:
                  animateType === 'css-to-center' && index === 9
              }
            ]"
          ></div>
          <!-- 方塊 -->
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

.rightDrawerBtn {
  width: 100%;
  height: $drawer-btn-height;
  background-color: white;
  display: flex;
  justify-content: space-between;
  align-items: center;
  button {
    margin: 10px;
  }
}
.mainDrawer {
  width: 50vw;
  height: 100vh;
  position: fixed;
  top: 0;
  right: -50vw;
  z-index: 9999;
  background-color: red;
  transition: right 1s ease; /* 添加过渡效果 */
  padding: 10px;
  box-sizing: border-box;
  &__open {
    right: 0;
  }
}

.nineSquare {
  width: 100%;
  height: calc(100vh - $drawer-btn-height);
  background-color: gray;
  display: flex;
  align-items: center;
  overflow: hidden; /* 添加溢出隐藏样式 */
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
  }
  &__cssAnimate {
    animation: circleMoveToRight 2s infinite;
  }
  @for $i from 1 through 9 {
    &__cssToCenterAnimate__#{$i} {
      animation: moveCenter_#{$i} 2s infinite;
    }
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
@for $i from 1 through 9 {
  $top: 0;
  $left: 0;
  @if $i == 1 {
    $top: calc(150% + 10px);
    $left: calc(150% + 10px);
  }
  @if $i == 3 {
    $top: calc(150% + 10px);
    $left: calc(-50% - 10px);
  }
  @if $i == 7 {
    $top: calc(-50% - 10px);
    $left: calc(150% + 10px);
  }
  @if $i == 9 {
    $top: calc(-50% - 10px);
    $left: calc(-50% - 10px);
  }

  @keyframes moveCenter_#{$i} {
    0% {
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
    }
    100% {
      top: $top;
      left: $left;
      transform: translate(-50%, -50%);
    }
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
