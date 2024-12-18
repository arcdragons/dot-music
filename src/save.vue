<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const canvas = ref<HTMLCanvasElement | null>(null)
const context = ref<CanvasRenderingContext2D | null>(null)
let audioContext: null | AudioContext = null;
let oscillator: OscillatorNode | null = null;
let amp: GainNode | null = null;

type Point = { x: number; y: number }
type Event = { point: Point, time: number, duration: number }
type Eventc = { point: Point, time: number, duration: number }
type Eventcc = { point: Point, time: number, duration: number }


let width = 1000
let height = 1000
let time = 0
let x = 0
let y = 0
let hz = 0
let mouse = { x: 0, y: 0 }
let ctx: any

let saveplaces: Array<Point> = []
let places: Array<Point> = []
const events: Event[] = []
const eventsc: Eventc[] = []
const eventscc: Eventcc[] = []

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

function easeOutCubic(x: number): number {
return 1 - Math.pow(1 - x, 3);
}

function easeInCubic(x: number): number {
  const c1 = 1.70158;
  const c3 = c1 + 1;

return 1 + c3 * Math.pow(x - 1, 3) + c1 * Math.pow(x - 1, 2);
}

function getDistance(a: Point, b: Point): number {
  return Math.hypot(a.x - b.x, a.y - b.y)
}

function getAngle(a: Point, b: Point): number {
  return Math.atan2(b.y - a.y, b.x - a.x)
}

function cadre(a: number, b: number):
  Point {
  return {
    x: a,
    y: b,
  }
}

function play() {
  audioContext = new AudioContext()
  oscillator = audioContext.createOscillator()
  amp = audioContext.createGain()
  
//   you can use other AudioParams here
//   amp.gain.setValueAtTime(0.9, amp.context.currentTime)
 
  oscillator.connect(amp).connect(audioContext.destination)
  oscillator.frequency.value = hz
  oscillator.frequency.setValueAtTime(0, 1)
  oscillator.type = "triangle"
  oscillator.connect(audioContext.destination)
  oscillator.start()
  oscillator.connect(audioContext.destination)
  oscillator.start()
 
}

function start() {
  if (!context.value) return
  ctx = context.value
  for (let i = 0; i < 100; i++) {
    for (let j = 0; j < 100; j++) {
      x = i * 10 + 5
      y = j * 10 + 5
      places.push(cadre(x, y))
      saveplaces.push(cadre(x, y))
    }
  }
  requestAnimationFrame(draw)
}

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


function draw(timestamp: number) {
  ctx = context.value
  ctx.clearRect(0, 0, width, height)
  time = timestamp

  places.forEach(place => {
    const dot = new Path2D()
    dot.arc(place.x, place.y, 1, 0, 2 * Math.PI)
    ctx.fillStyle = 'white'
    ctx.fill(dot)
  });
  
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
    // ctx.stroke(circle)


    places.forEach(place => {
      if ( getDistance(event.point, place) < radius) {
        // const dot = new Path2D()
        // dot.arc(place.x, place.y, 2, 0, 2 * Math.PI)
        // //console.log(getPointAtAngle(place,getAngle(event.point,place),getDistance(event.point,place)))
        // ctx.fill(dot)
        // ctx.strokeStyle = "blue"
        // ctx.stroke(dot)
        const d = easeOutCubic((getDistance(event.point,place)+0.25)/1000)
        place.x = getPointAtAngle(place,getAngle(event.point,place),d).x
        place.y = getPointAtAngle(place,getAngle(event.point,place),d).y
        // ctx.moveTo(event.point.x,event.point.y)
        // ctx.lineTo(place.x,place.y);
        // ctx.stroke()
      }

    });

    
    
    
  })

  saveplaces.forEach(event => {
    let b = saveplaces.indexOf(event)
    let place = places[b]
      if(getDistance(event,place)!= 0){  
      const d = easeInCubic((-getDistance(event,place)+0.25)/100)
      place.x = getPointAtAngle(place,getAngle(event,place),d).x
      place.y = getPointAtAngle(place,getAngle(event,place),d).y
    } 
  })
  
  eventsc.forEach((event, index) => {
    const progress = (time - event.time) / event.duration
        
    if (0.11 > progress && progress > 0.1) {
        eventscc.push({ point: { x: event.point.x, y: event.point.y }, time, duration: 2000 })
        }
    if (progress > 1) {
      eventsc.splice(index, 1)
      return
    }
    const circle = new Path2D()
    const radius = easeOutSine(progress) * 1000
    circle.arc(event.point.x, event.point.y, radius, 0, 2 * Math.PI, true)
    ctx.strokeStyle = "rgba(125,200,255," + easeOutBack(progress) + ")"
    ctx.stroke(circle)
    


    places.forEach(place => {
      if ( getDistance(event.point, place) < radius) {
        const d = easeOutCubic((getDistance(event.point,place)+0.25)/1000)
        place.x = getPointAtAngle(place,getAngle(event.point,place),d).x
        place.y = getPointAtAngle(place,getAngle(event.point,place),d).y

      }

    });



  })

  

  requestAnimationFrame(draw)
}
function click(ev: MouseEvent) {
  // console.log('click')
  //events.push({ point: { x: mouse.x, y: mouse.y }, time, duration: 2000 })
  play()
}

function bouge(ev: MouseEvent) {
  mouse.x = ev.offsetX
  mouse.y = ev.offsetY
  //events.push({ point: { x: mouse.x, y: mouse.y }, time, duration: 2000 })
}

 document.addEventListener('keydown', (ev)=>{
  //  console.log(ev.code)
      if (ev.defaultPrevented) {
  return; // Ne devrait rien faire si l'événement de la touche était déjà consommé.
    }

    switch (ev.code) {
      case "Numpad2":
        eventsc.push({ point: { x: width/2, y: height }, time, duration: 2000 })
      // Faire quelque chose pour la touche "bas" pressée.
      if(oscillator && amp) {
        oscillator.frequency.value = 98.1
        const gain = 10
        amp.gain.value = gain
        // console.log(gain.volume); // 1
        
        ctx.fillText(`gain ${gain} `,100,100)
        } 
         
        break;
      case "Numpad8":
        eventsc.push({ point: { x: width/2, y: 0 }, time, duration: 2000 })
      // Faire quelque chose pour la touche "haut" pressée.
      if(oscillator && amp) {
        oscillator.frequency.value = 490.5
        const gain = 10
        amp.gain.value = gain
        
        ctx.fillText(`gain ${gain} `,100,100)
        } 
        break;
      case "Numpad4":
        eventsc.push({ point: { x: 0, y: height/2 }, time, duration: 2000 })
      // Faire quelque chose pour la touche "gauche" pressée
      if(oscillator && amp) {
        oscillator.frequency.value = 228.9
        const gain = 10
        amp.gain.value = gain
        // console.log(gain.volume); // 1
        
        ctx.fillText(`gain ${gain} `,100,100)
        }
        break;
      case "Numpad6":
        eventsc.push({ point: { x: width, y: height/2 }, time, duration: 2000 })
      // Faire quelque chose pour la touche "droite" pressée.
      if(oscillator && amp) {
        oscillator.frequency.value = 359.7
        const gain = 10
        amp.gain.value = gain
        // console.log(gain.volume); // 1
        
        ctx.fillText(`gain ${gain} `,100,100)
        } 
        break;
        case "Numpad5":
          eventsc.push({ point: { x: width/2, y: height/2 }, time, duration: 2000 })
      // Faire quelque chose pour la touche "centre" pressée.
      if(oscillator && amp) {
        oscillator.frequency.value = 294.3
        const gain = 10
        amp.gain.value = gain
        // console.log(gain.volume); // 1
        
        ctx.fillText(`gain ${gain} `,100,100)
        } 
          break;
        case "Numpad7":
          eventsc.push({ point: { x: 0, y: 0}, time, duration: 2000 })
        // Faire quelque chose pour la touche "haut gauhe" pressée.
        if(oscillator && amp) {
        oscillator.frequency.value = 425.1
        const gain = 10
        amp.gain.value = gain
        // console.log(gain.volume); // 1
        
        ctx.fillText(`gain ${gain} `,100,100)
        } 
          break;
        case "Numpad9":
          eventsc.push({ point: { x: width, y: 0}, time, duration: 2000 })
        // Faire quelque chose pour la touche "haut droit" pressée.
        if(oscillator && amp) {
        oscillator.frequency.value = 523.2
        const gain = 10
        amp.gain.value = gain
        // console.log(gain.volume); // 1
        
        ctx.fillText(`gain ${gain} `,100,100)
        } 
          break;
        case "Numpad1":
          eventsc.push({ point: { x: 0, y: height}, time, duration: 2000 })
        // Faire quelque chose pour la touche "bas gauche" pressée.
        if(oscillator && amp) {
        oscillator.frequency.value = 32.7
        const gain = 10
        amp.gain.value = gain
        // console.log(gain.volume); // 1
        
        ctx.fillText(`gain ${gain} `,100,100)
        } 
          break;
        case "Numpad3":
          eventsc.push({ point: { x: width, y: height}, time, duration: 2000 })
        // Faire quelque chose pour la touche "bas droit" pressée.
        if(oscillator && amp) {
        oscillator.frequency.value = 163.5
        const gain = 10
        amp.gain.value = gain
        // console.log(gain.volume); // 1
        
        ctx.fillText(`gain ${gain} `,100,100)
        } 
          break;
          case "Numpad0":
          eventsc.push({ point: { x: width, y: height}, time, duration: 2000 })
          eventsc.push({ point: { x: width*0, y: height*0}, time, duration: 2000 })
          eventsc.push({ point: { x: width*0, y: height}, time, duration: 2000 })
          eventsc.push({ point: { x: width, y: height*0}, time, duration: 2000 })
          eventsc.push({ point: { x: width*0.5, y: height*0.5}, time, duration: 2000 })
        // Faire quelque chose pour la touche "bas droit" pressée.
        if(oscillator && amp) {
        oscillator.frequency.value = 163.5
        const gain = 0
        amp.gain.value = gain
        // console.log(gain.volume); // 1
        
        ctx.fillText(`gain ${gain} `,100,100)
        } 
          break;}
})


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