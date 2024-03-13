<script>
import { ref, onMounted, onBeforeMount, nextTick, provide } from 'vue'
import defaultData from '@/lib/data.js'
import TreeListLi from '@/components/TreeListLi.vue'
import TreeListSelect from '@/components/TreeListSelect.vue'
import TreeSelfOption from '@/components/TreeSelfOption.vue'
export default {
  components: {
    TreeListLi,
    TreeListSelect,
    TreeSelfOption
  },
  setup() {
    onBeforeMount(() => {
      initDefaultData()
    })
    onMounted(() => {
      if (circlesRef.value) {
        const circles = circlesRef.value
        circles.forEach((circle) => {
          if (!originalCircleLeftPlace) {
            originalCircleLeftPlace = circle.offsetLeft
          }
        })
      }
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
    let animateType = ref('css') // 一開始預設動畫type
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
          circleAnimationToRightUseJs() // js操作動畫
        } else if (clickType === 'canvas') {
          nextTick(() => {
            initHundredCircle() // canvas 操作動畫
          })
        }
      } catch (error) {
        console.error(error)
      }
    }
    // 讓圓圈回到原本地方，因為有可能動畫切到一半，要復原回原點
    const moveCirclesToOrigin = function () {
      const circles = circlesRef.value
      circles.forEach((circle) => {
        moveXPx.value = originalCircleLeftPlace
        circle.style.left = `${originalCircleLeftPlace}px`
      })
    }
    // 讓圓圈往右跑
    const moveCirclesToRight = function () {
      const circles = circlesRef.value
      circles.forEach((circle) => {
        let leftValue = parseInt(circle.offsetLeft) || 0
        leftValue += 2
        circle.style.left = `${leftValue}px`
      })
    }
    // js 動畫的時間interval
    const circleAnimationToRightUseJs = function () {
      timerIntervalToRIght = setInterval(() => {
        moveCirclesToRight()
      }, 50)
      timerIntervalToOrigin = setInterval(() => {
        moveCirclesToOrigin()
      }, 2000)
    }
    // 100顆球的動畫（使用canvas）
    let [startX, startY, endX, endY] = [ref(0), ref(0), ref(100), ref(300)]
    const initHundredCircle = function () {
      try {
        const canvas = document.getElementById('canvasFor100Circle')
        const ctx = canvas.getContext('2d') // 2d 渲染
        const balls = []
        const numBalls = 100
        const ballRadius = 10

        for (let i = 0; i < numBalls; i++) {
          const dx = (endX.value - startX.value) / 100
          const dy = (endY.value - startY.value) / 100

          balls.push({
            x: startX.value,
            y: startY.value,
            endX: endX.value,
            endY: endY.value,
            dx: dx,
            dy: dy,
            radius: ballRadius,
            color: `radial-gradient(circle, rgba(113,81,95,1) 81%, rgba(0,0,0,1) 100%)`
          })
        }
        draw(ctx, canvas, balls)
      } catch (e) {
        console.error(e.message)
      }
    }
    const drawBall = function (ctx, ball) {
      ctx.beginPath()
      ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2) // 繪出圓形的方法
      ctx.fillStyle = ball.color
      ctx.fill() // 填充當前路徑
      ctx.closePath()
    }

    const draw = function (ctx, canvas, balls) {
      ctx.clearRect(0, 0, canvas.width, canvas.height) // 繪圖之前，先清除
      balls.forEach((ball) => {
        drawBall(ctx, ball)
        ball.x += ball.dx
        ball.y += ball.dy
        if (Math.abs(ball.x - ball.endX) < 1 && Math.abs(ball.y - ball.endY) < 1) {
          ball.x = ball.startX
          ball.y = ball.startY
        }
      })
      requestAnimationFrame(() => draw(ctx, canvas, balls))
    }
    // 儲存open的陣列，要記錄到local中
    const data = ref([])
    provide('saveArr', data)
    const updateTreeNewValue = function (nvl) {
      for (let i = 0; i < nvl.length; i++) {
        const currentItem = nvl[i]
        const currentItemNo = currentItem.no
        for (let j = i + 1; j < nvl.length; j++) {
          const nextItem = nvl[j]
          if (nextItem.no.indexOf(currentItemNo) === -1) {
            // 如果下一個元素的編號中不包含當前元素的編號，則刪除該元素
            nvl.splice(j, 1)
            j-- // 因為刪除了元素，所以要調整索引以避免跳過下一個元素
          }
        }
      }
      const result = nvl.map((item) => ({
        text: item.text,
        key: item.key,
        no: item.no
      }))
      localStorage.setItem('userClickDrink', JSON.stringify(result))
    }
    const initDefaultData = function () {
      const clickDrink = JSON.parse(localStorage.getItem('userClickDrink'))
      if (clickDrink) {
        toOriginal(clickDrink)
      }
    }
    const toOriginal = function (clickDrink) {
      const updateIsOpen = (data, saveItem, isLast) => {
        if (data.key === saveItem.key) {
          // 設定此層開啟狀態
          data.isOpen = true
          if (isLast) {
            // 如果是最後一層則高亮
            data.highlight = true
          }
          data.no = saveItem.no
        }
        // 如果有子層，那再繼續往下遍歷
        if (data.children) {
          data.children.forEach((child) => updateIsOpen(child, saveItem, isLast))
        }
      }
      clickDrink.forEach((saveItem, index) => {
        const isLast = index === clickDrink.length - 1
        defaultData.forEach((data) => updateIsOpen(data, saveItem, isLast))
      })
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
      startX,
      startY,
      endX,
      endY,
      updateTreeNewValue
    }
  }
}
</script>

<template>
  <div @click="handleClickOutside">
    <div :class="$style.rightDrawerBtn">
      <div :class="$style.animateTypeClass">
        <div>
          <button @click="changeAnimateType('css')">CSS Animate</button>
          <button @click="changeAnimateType('js')">JS Animate</button>
          <button @click="changeAnimateType('css-to-center')">CSS to Center</button>
        </div>
        <div>
          <button @click="changeAnimateType('canvas')">100 circles Canvas</button>
          <div><input v-model.number="startX" /><span>x軸起始點</span></div>
          <div><input v-model.number="startY" /><span>y軸起始點</span></div>
          <div><input v-model.number="endX" /><span>x軸終點</span></div>
          <div><input v-model.number="endY" /><span>y軸終點</span></div>
        </div>
      </div>
      <button @click.self.stop="drawerStatus = !drawerStatus">Show Drawer</button>
    </div>
    <div
      ref="rightDrawer"
      :class="[$style.mainDrawer, { [$style.mainDrawer__open]: drawerStatus }]"
    >
      <TreeListLi :tree-data="treeData" :root="true" @updateTreeNewValue="updateTreeNewValue" />
      <TreeListSelect :tree-data="treeData" />
      <TreeSelfOption :tree-data="treeData" />
    </div>
    <div :class="$style.nineSquare">
      <div :class="$style.nineSquare__mainDiv">
        <div v-for="index in 9" :key="index" :class="$style.nineSquare__repeatDiv">
          <!-- 圓圈 -->
          <div
            v-if="circleShowIndex.includes(index) && animateType !== 'canvas'"
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
          <div v-if="animateType === 'canvas'" :class="$style.canvasPosition">
            <canvas id="canvasFor100Circle" width="500" height="500"></canvas>
          </div>
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

<style lang="scss" module>
$drawer-btn-height: 50px;

.rightDrawerBtn {
  width: 100%;
  height: $drawer-btn-height;
  background-color: white;
  display: flex;
  justify-content: end;
  align-items: center;
  button,
  input {
    margin: 10px;
  }
}
// 抽屜打開來的樣子
.mainDrawer {
  color: white;
  width: 50vw;
  height: 100vh;
  position: fixed;
  top: 0;
  right: -50vw;
  z-index: 9999;
  background-color: black;
  transition: right 1s ease;
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
  overflow: hidden;
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
// canvas 相關設定
.canvasPosition {
  border: 1px solid white;
  z-index: 1;
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  > canvas {
    width: 500px;
    height: 500px;
  }
}
// 切換動畫的形式
.animateTypeClass {
  position: fixed;
  display: flex;
  flex-direction: column;
  top: $drawer-btn-height;
  left: 0;
  z-index: 10;
}
// 方塊的動畫
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
// 往中間的動畫
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
