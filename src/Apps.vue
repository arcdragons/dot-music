<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const canvas = ref<HTMLCanvasElement | null>(null)
const context = ref<CanvasRenderingContext2D | null>(null)

onMounted(() => {
  context.value = canvas.value!.getContext('2d')
  requestAnimationFrame(draw)
})

onUnmounted(() => {
  context.value = null
})


type Point = { x: number; y: number }
function getPointAtAngle(
  center: Point,
  angle: number,
  distance: number
): Point {
  return {
    x: center.x + Math.cos(angle) * distance,
    y: center.y + Math.sin(angle) * distance,
  }
}
let mouse = { x:0,y:0}
let moont = 5
let time = 0

function getAngle(a: Point, b: Point): number {
  return Math.atan2(b.y - a.y, b.x - a.x)
}
function bouge(ev: MouseEvent) {
  console.log(ev.y)
  console.log(ev.offsetX)
  mouse.x=ev.offsetX
  mouse.y=ev.offsetY

}

function draw(timestamp: number) {
  if (!context.value) return
  const ctx = context.value
  ctx.clearRect(0, 0, 1000, 1000)
  
  const easeLinear = (x: number) => x
  time = timestamp /1000
  const positionx = 0 - easeLinear(time)

  const terre = new Path2D()
  const terrep = {
    x : 500,
    y : 500
  }
  const moon = new Path2D()
  const moonp = getPointAtAngle(terrep,getAngle(terrep,mouse), 150)
  const sun = new Path2D()
  const sunp = getPointAtAngle(mouse, time*2,50)
  
  
  // const terre = new Path2D()
  
  // const moonx = Math.sin(positionx)*100+500
  // const moony = Math.cos(positionx)*100+500
  
  
  
  // const suny = 500+Math.cos(positionx)*50
  // const sunx = 500+Math.sin(positionx)*50
  
  terre.arc(terrep.x, terrep.y, moont*1.5, 0, 2 * Math.PI)
  sun.arc(sunp.x, sunp.y , moont*0.25, 0, 2 * Math.PI)
  moon.arc(moonp.x, moonp.y, moont, 0, 2 * Math.PI)


ctx.fillText('souris x :'+ mouse.x + ' y :'+ mouse.y,10,10)
ctx.fillStyle = 'white'
// ctx.stroke(sun)
ctx.fill(sun)
ctx.fillStyle = 'black'
ctx.stroke(moon)
ctx.fill(moon) 
ctx.strokeStyle = 'grey'
ctx.fillStyle = 'grey'
ctx.stroke(terre)
ctx.fill(terre) 

// events.forEach((event, index) => {
  //   const progress = (timestamp - event.time) / event.duration
  //   if (progress > 1) {
    //     events.splice(index, 1)
    //     return
    //   }
    // })
    
    if(timestamp < 3000) {
      moont = time*time+5
  }
  requestAnimationFrame(draw)
}
function click (ev: MouseEvent) {
 
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