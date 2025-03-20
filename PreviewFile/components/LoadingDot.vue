<script lang="ts" setup>
import { computed, unref } from 'vue'
import { merge } from 'lodash'

const props = withDefaults(
  defineProps<{
    width?: number
    num?: number
    dotWidth?: number
    color?: string
    loadingText?: object
    loadingSpeed?: number
  }>(),
  {
    width: 20,
    num: 8,
    color: '#165BFE',
    dotWidth: 4,
    loadingText: () => {
      return {
        color: '#165BFE',
        text: '加载中...',
        show: false
      }
    },
    loadingSpeed: 1.5
  }
)

let defaultloadingOptions = {
  color: '#165BFE',
  text: '加载中...',
  show: false
}

const getLoadingOpions = computed(() => {
  let ops = merge(defaultloadingOptions, props.loadingText)
  return ops
})

const dotStyle = computed(() => ({
  position: 'absolute',
  left: '50%',
  top: '50%',
  width: `${props.dotWidth}px`,
  height: `${props.dotWidth}px`,
  backgroundColor: props.color
}))

// 计算样式
const getDotStyle = (index: number) => {
  const angle = (index / props.num) * 360 // 角度
  const distance = props.width / 2 // 圆环的半径，距离调整为宽度的一半减去小圆点的半径
  const x = distance * Math.cos((angle * Math.PI) / 180)
  const y = distance * Math.sin((angle * Math.PI) / 180)

  return {
    ...unref(dotStyle),
    transform: `translate(-50%, -50%) translate(${x}px, ${y}px)`
  }
}
</script>
<template>
  <div
    class="LoadingDot"
    :style="{
      '--loadingSpeed': loadingSpeed + 's'
    }"
  >
    <div :style="{ width: width + 'px', height: width + 'px' }" class="loading-wrapper">
      <div class="loading-spinner">
        <div v-for="i in num" :key="i" :style="getDotStyle(i)" class="dot"></div>
      </div>
    </div>
    <div
      v-if="getLoadingOpions?.show"
      :style="{ color: getLoadingOpions?.color }"
      class="loading-text"
    >
      {{ getLoadingOpions?.text }}
    </div>
  </div>
</template>

<style scoped>
.LoadingDot {
  display: flex;
  flex-direction: column;
  align-items: center;
  .loading-wrapper {
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    .loading-spinner {
      position: relative;
      border-radius: 50%;
      animation: rotate var(--loadingSpeed) linear infinite;
      .dot {
        border-radius: 50%;
        background-color: #fff;
        position: absolute;
        &:last-child {
          background-color: #fff !important;
        }
      }
    }
  }
  .loading-text {
    margin-top: 10px;
  }
}

@keyframes rotate {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
