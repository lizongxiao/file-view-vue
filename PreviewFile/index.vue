<template>
  <div class="file-preview">
    <LoadingDot v-if="loading" :loading-text="{ show: true }" />
    <img v-else-if="isImage" :src="fileData.url" class="png" />
    <div v-else-if="isDocx" id="DocxContainer" class="preview-content"></div>
    <PptxViewer v-else-if="isPpt" :fileData="fileData" class="preview-content" />
    <div id="ExcelPreview" v-else-if="isExcel" class="preview-content"></div>
    <VuePdfEmbed v-else-if="isPdf" :width="1100" :source="fileData.url" />
    <video v-else-if="isVideo" :src="fileData.url" class="preview-content video"></video>
    <div v-else class="unsupported">不支持的文件类型</div>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, onUnmounted, onUpdated, ref } from 'vue'
import VuePdfEmbed from 'vue-pdf-embed'
import { renderAsync } from 'docx-preview'
import LoadingDot from './components/LoadingDot.vue'
import * as XLSX from 'xlsx'
import PptxViewer from './components/PptxViewer.vue'

interface FileData {
  url: string
  fileName: string
  fileType: string
  originalData: Blob | null
}

interface Props {
  fileData: FileData
}

const props = withDefaults(defineProps<Props>(), {
  fileData: () => ({
    url: '',
    fileName: '',
    fileType: '',
    originalData: null
  })
})

const loading = ref(true)

// 判断是否为图片类型
const isImage = computed(() => {
  return ['jpg', 'jpeg', 'png', 'gif', 'bmp', 'webp', 'svg'].includes(props.fileData.fileType)
})

// TODO:判断是否为ppt类型
const isPpt = computed(() => {
  return ['pptx', 'ppt'].includes(props.fileData.fileType)
})

// 判断是否为PDF类型
const isPdf = computed(() => {
  let types = ['pdf']
  return types.includes(props.fileData.fileType)
})

// 判断是否为docx类型
const isDocx = computed(() => {
  let types = ['docx', 'doc']
  return types.includes(props.fileData.fileType)
})

// 判断是否为excel类型
const isExcel = computed(() => {
  let types = ['xlsx', 'xls', 'csv']
  return types.includes(props.fileData.fileType)
})

// 判断是否为视频类型
const isVideo = computed(() => {
  return ['mp4', 'webm', 'ogg'].includes(props.fileData.fileType)
})

const initDocx = async () => {
  const container = document.getElementById('DocxContainer')
  if (!container) return

  await renderAsync(props.fileData.originalData, container, undefined, {
    inWrapper: false,
    ignoreWidth: true,
    ignoreHeight: true
  })
}
const initExcel = async () => {
  if (!props.fileData.originalData) return
  const reader = new FileReader()
  reader.readAsArrayBuffer(props.fileData.originalData)
  reader.onload = function (e) {
    if (!e.target) return
    const data = e.target.result
    const workbook = XLSX.read(data, { type: 'array' })
    const container = document.getElementById('ExcelPreview')
    if (!container) return
    container.innerHTML = '' // 清空容器，避免重复添加

    workbook.SheetNames.forEach((sheetName) => {
      // 为每个 sheet 创建一个独立容器
      const sheetContainer = document.createElement('div')
      sheetContainer.classList.add('sheet-container')

      // 添加 sheet 标题
      const header = document.createElement('h2')
      header.textContent = sheetName
      sheetContainer.appendChild(header)

      // 将 Excel sheet 转换成 HTML 表格
      const htmlString = XLSX.utils.sheet_to_html(workbook.Sheets[sheetName])
      const tempDiv = document.createElement('div')
      tempDiv.innerHTML = htmlString
      const tableElement = tempDiv.querySelector('table')
      if (tableElement) {
        tableElement.classList.add('excel-table') // 添加自定义样式类
        sheetContainer.appendChild(tableElement)
      }
      container.appendChild(sheetContainer)
    })
  }
}

onUpdated(() => {
  if (isDocx.value) {
    initDocx()
  }

  if (isExcel.value) {
    initExcel()
  }

  setTimeout(() => {
    loading.value = false
  }, 300)
})

const disableRightClick = (e: MouseEvent) => {
  e.preventDefault()
}

const disableCopy = (e: any) => {
  e.preventDefault()
}

onMounted(async () => {
  // 禁用右键
  document.addEventListener('contextmenu', disableRightClick)
  // 禁止选中
  document.addEventListener('selectstart', disableCopy)
})

onUnmounted(() => {
  document.removeEventListener('contextmenu', disableRightClick)
  document.removeEventListener('selectstart', disableCopy)
})
</script>

<style scoped>
.file-preview {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  min-height: 400px;
}

.unsupported {
  color: #ff4444;
  padding: 20px;
  border: 1px solid #ff4444;
  text-align: center;
}

.preview-content {
  width: 100%;
  overflow: hidden;
  background: white;
  max-width: 100%;
  height: 100%;
  object-fit: contain;
}

.png {
  object-fit: cover;
  max-width: 500px;
  max-height: 500px;
}

.pdf {
  width: 100%;
  height: 100%;
}

.video {
  width: 100%;
  height: 100%;
}

:deep(#ExcelPreview) {
  height: 100%;
  width: 100%;
  margin: 20px;
  .sheet-container {
    margin-bottom: 40px;
    width: 100%;
    height: 100%;
    overflow: auto;
  }

  .sheet-container h2 {
    font-size: 1.5em;
    margin-bottom: 10px;
  }

  .excel-table {
    width: 100%;
    border-collapse: collapse;
  }

  .excel-table th,
  .excel-table td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
    white-space: nowrap;
  }

  .excel-table tr:nth-child(even) {
    background-color: #f9f9f9;
  }
}
</style>
