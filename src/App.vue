<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const canvas = ref<HTMLCanvasElement | null>(null)
const context = ref<CanvasRenderingContext2D | null>(null)

type Point = { x: number; y: number }
type Event = { point: Point, time: number, duration: number }


let time = 0
let x = 0
let y = 0
let mouse = { x: 0, y: 0 }
let ctx: any

let places: Array<Point> = []
const events: Event[] = []

onMounted(() => {
  context.value = canvas.value!.getContext('2d')
  start()
})

onUnmounted(() => {
  context.value = null
})

function easeOutSine(x: number): number {
  return Math.sin((x * Math.PI) / 2);
}

function easeOutBack(x: number): number {
  return Math.cos((x * Math.PI) / 2);
}

function getDistance(a: Point, b: Point): number {
  return Math.hypot(a.x - b.x, a.y - b.y)
}

function cadre(a: number, b: number):
  Point {
  return {
    x: a,
    y: b,
  }
}

function start() {
  if (!context.value) return
  ctx = context.value
  for (let i = 0; i < 10; i++) {
    for (let j = 0; j < 10; j++) {
      console.log(i, j)
      x = i * 100 + 50
      y = j * 100 + 50
      places.push(cadre(x, y))
    }
  }
  console.log(places)


  requestAnimationFrame(draw)
}


function isInCircle(center: Point, dot: Point, radius: number): boolean {
  const distance = getDistance(center, dot)
  return distance < radius
}

function draw(timestamp: number) {
  ctx = context.value
  ctx.clearRect(0, 0, 1000, 1000)
  time = timestamp

  places.forEach(place => {
    const dot = new Path2D()
    dot.arc(place.x, place.y, 2, 0, 2 * Math.PI)
    ctx.fill(dot)
  });
  const sun = new Path2D()
  sun.arc(mouse.x, mouse.y, 1, 0, 2 * Math.PI)
  ctx.fillStyle = 'white'
  ctx.fill(sun)

  events.forEach((event, index) => {
    const progress = (time - event.time) / event.duration
    if (progress > 1) {
      events.splice(index, 1)
      return
    }
    const circle = new Path2D()
    const radius = easeOutSine(progress) * 100
    circle.arc(event.point.x, event.point.y, radius, 0, 2 * Math.PI, true)
    ctx.strokeStyle = "rgba(125,200,255," + easeOutBack(progress) + ")"
    ctx.stroke(circle)


    places.forEach(place => {
      if (isInCircle(event.point, place, radius)) {
        const dot = new Path2D()
        dot.arc(place.x, place.y, 2, 0, 2 * Math.PI)
        ctx.fill(dot)
        ctx.strokeStyle = "blue"
        ctx.stroke(dot)
      }

    });



  })

  requestAnimationFrame(draw)
}
function click(ev: MouseEvent) {
  console.log('click')
  events.push({ point: { x: mouse.x, y: mouse.y }, time, duration: 2000 })
}

function bouge(ev: MouseEvent) {
  mouse.x = ev.offsetX
  mouse.y = ev.offsetY
  //events.push({ point: { x: mouse.x, y: mouse.y }, time, duration: 2000 })
}

</script>

<template>
  <canvas width="1000" height="1000" ref="canvas" @click="click" @mousemove="bouge"></canvas>
</template>

<style lang="css" scoped>
canvas {
  border: 1px solid;
  border-color: brown;
}
</style>