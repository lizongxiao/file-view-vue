<template>
  <div id="blobIframe"></div>
</template>

<script setup lang="ts">
import { onMounted, defineProps, computed } from 'vue'

/**
 * 定义组件接收的属性，符合 FileData 接口要求
 */
interface FileData {
  originalData: any // blob 数据
  fileName: string // 文件名称
  url: string // 文件 URL
  fileType: string // 文件类型
}

const props = defineProps<{ fileData: FileData }>()

// 定义 file-viewer 的地址配置，支持生产环境下可能的路径前缀问题
const context = {
  origin: location.origin + '/file-viewer/index.html',
  frame: null as HTMLIFrameElement | null
}

// 动态计算 iframe 的 src 参数：若存在 blob 数据，则通过传递文件名参数来加载预览器；否则直接传 URL
const iframeSrc = computed(() => {
  if (props.fileData.originalData) {
    return (
      context.origin +
      '?name=' +
      encodeURIComponent(props.fileData.fileName) +
      '&from=' +
      encodeURIComponent(location.origin)
    )
  } else {
    return context.origin + '?fileUrl=' + encodeURIComponent(props.fileData.url)
  }
})

// 动态创建并插入 iframe
function appendFrame(src: string): HTMLIFrameElement {
  const blobIframe = document.getElementById('blobIframe')
  if (!blobIframe) {
    throw new Error('找不到 id 为 blobIframe 的容器')
  }
  if (context.frame) {
    blobIframe.removeChild(context.frame)
  }
  const frame = document.createElement('iframe')
  context.frame = frame
  frame.src = src
  frame.style.cssText = 'border:0;height: 100%;width:100%'
  return blobIframe.appendChild(frame) as HTMLIFrameElement
}

// 加载 blob 数据
function loadBlobData() {
  const frame = appendFrame(iframeSrc.value)
  frame.onload = () => {
    frame.contentWindow?.postMessage(props.fileData.originalData, location.origin)
  }
}

onMounted(() => {
  if (props.fileData.originalData) {
    loadBlobData()
  } else {
    appendFrame(iframeSrc.value)
  }
})

// window.addEventListener('message', (event) => {
//   if (event.origin !== location.origin) return
//   if (event.data.type === 'fileLoaded') {
//   }
// })
</script>

<style scoped lang="scss">
#blobIframe {
  width: 100%;
  height: 500px;
  padding: 10px;
}
</style>
