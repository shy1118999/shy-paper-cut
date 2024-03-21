<!--
 * @Author: shaohang-shy
 * @Date: 2022-08-10 18:57:59
 * @LastEditors: shaohang-shy
 * @LastEditTime: 2023-04-06 08:58:46
 * @Description: index page
-->
<script setup lang="ts">
// 一共有8页 先弄4页
const images = [
  '/1.png',
]
function getImageSize(url: string) {
  return new Promise<{ width: number; height: number }>((resolve, reject) => {
    const image = new Image()
    image.onload = () => {
      resolve({ width: image.width, height: image.height })
    }
    image.onerror = (e) => {
      reject(e)
    }
    image.src = url
  })
}
// 实现一个方法实现小数四舍五入
function round(num: number, decimal: number = 0) {
  return Math.round(num * 10 ** decimal) / 10 ** decimal
}
const pages = ref<{ image: string; width: number; height: number }[]>([])
images.forEach(async (img) => {
  const size = await getImageSize(img)
  pages.value.push({
    image: img,
    width: size.width,
    height: size.height,
  })
})

const xLineStr = ref('50,743,397')
const yLineStr = ref('50,1073,562')
const xLine = computed(() => xLineStr.value.split(',').map(i => Number.parseInt(i)))
const yLine = computed(() => yLineStr.value.split(',').map(i => Number.parseInt(i)))

function getPath(width: number, height: number, xs: number[], ys: number[]) {
  const path: string[] = []
  xs.forEach((x) => {
    path.push(`M ${x} 0 L ${x} ${height}`)
  })
  ys.forEach((y) => {
    path.push(`M 0 ${y} L ${width} ${y}`)
  })
  return path.join(' ')
}

interface Point {
  x: number
  y: number
}
interface Position {
  x: number
  y: number
  w: number
  h: number
  no?: number
}

function getPoint(e: MouseEvent | PointerEvent): Point {
  return { x: round(e.offsetX), y: round(e.offsetY) }
}

function getPagePoint(e: PointerEvent): Point {
  return { x: round(e.pageX), y: round(e.pageY) }
}
function getSymbol(a: number) {
  if (a < 0)
    return -1
  return 1
}

function splitNum(a: number) {
  return [Math.abs(a), getSymbol(a)]
}

const gapStr = ref('10')
const gap = computed(() => Number.parseInt(gapStr.value))

const boxes = ref<Position[]>([])

const box = ref<Position | null>(null)
let startBox = { x: 0, y: 0 }
let pageStartBox = { x: 0, y: 0 }

function handleMouseDown(e: PointerEvent) {
  startBox = getPoint(e)
  box.value = { ...startBox, w: 0, h: 0 }
  pageStartBox = getPagePoint(e)
}
function handlePointerMove(e: PointerEvent) {
  if (box.value) {
    const p = getPagePoint(e)
    const [dx, sx] = splitNum(p.x - pageStartBox.x)
    const [dy, sy] = splitNum(p.y - pageStartBox.y)
    let [x1, x2] = [startBox.x, startBox.x + dx * sx].sort((a, b) => a - b)
    let [y1, y2] = [startBox.y, startBox.y + dy * sy].sort((a, b) => a - b)
    x1 = x1 < 0 ? 0 : x1
    y1 = y1 < 0 ? 0 : y1
    // 判断吸附情况 吸附到参考线上 根据xLine yLine gap
    xLine.value.forEach((x) => {
      if (Math.abs(x - x1) < gap.value)
        x1 = x

      if (Math.abs(x - x2) < gap.value)
        x2 = x
    })
    yLine.value.forEach((y) => {
      if (Math.abs(y - y1) < gap.value)
        y1 = y

      if (Math.abs(y - y2) < gap.value)
        y2 = y
    })

    box.value.x = x1
    box.value.y = y1
    box.value.w = x2 - x1
    box.value.h = y2 - y1
  }
}
function handlePointerUp(e: PointerEvent) {
  const p = getPoint(e)
  // eslint-disable-next-line no-console
  console.log(p)
  if (box.value && box.value.w > 10 && box.value.h > 10)
    boxes.value.push({ ...box.value })
  box.value = null
}
onMounted(() => {
  window.addEventListener('pointermove', handlePointerMove, { passive: false })
  window.addEventListener('pointerup', handlePointerUp, { passive: false })
})

onUnmounted(() => {
  window.removeEventListener('pointermove', handlePointerMove)
  window.removeEventListener('pointerup', handlePointerUp)
})
</script>

<template>
  <div>
    <div class="flex items-center justify-center">
      XLine: <input v-model="xLineStr" class="input">
      YLine: <input v-model="yLineStr" class="input">
      Gap: <input v-model="gapStr" class="input">
    </div>
    <div class="select-none py-3">
      <div
        v-for="item, idx in pages" :key="idx"
        :style="{ width: `${item.width}px`, height: `${item.height}px`, backgroundImage: `url(${item.image})` }"
        class="absolute relative ma box-border cursor-crosshair b-1 b-red b-solid bg-cover bg-no-repeat"
        @pointerdown.stop="handleMouseDown"
      >
        <!-- <img :src="item.image" class="h-full w-full"> -->
        <svg :width="item.width" :height="item.height" class="select-none">
          <path
            :d="getPath(item.width, item.height, xLine, yLine)" stroke="#FF0000" stroke-width="0.5"
            stroke-dasharray="1,1"
          />
        </svg>
        <div
          v-if="box" class="absolute select-none bg-red/20"
          :style="{ width: `${box.w}px`, height: `${box.h}px`, left: `${box.x}px`, top: `${box.y}px` }"
        />
        <div
          v-for="sub, i in boxes" :key="i" class="absolute select-none bg-red/20"
          :style="{ width: `${sub.w}px`, height: `${sub.h}px`, left: `${sub.x}px`, top: `${sub.y}px` }"
        >
          {{ sub.no }}
        </div>
      </div>
    </div>
  </div>
</template>
