<template>
  <pre style="border: solid 1px; background-color: #f7f5f5;">
    <h4>패치노트</h4>
    <span>재생버튼 추가함 (오루 있을수 있음~)</span>
  </pre>
  <div id="box">
    X 좌표 :<input type="number" v-model="xVal">
    Y 좌표 :<input type="number" v-model="yVal">
    <p>
      <button type="button" @click="coordinateEnter">좌표 입력</button>
      <button type="button" @click="play">재생</button>
    </p>
    <p>
      <input type="checkbox" v-model="useMouse" id="useMouse">
      <label for="useMouse">마우스 클릭 사용해버리기</label>
    </p>
    <canvas id="canvas" ref="canvas" :width="maxHeight" :height="maxWidth" @click="draw($event)"></canvas>
  </div>
</template>

<script setup>
import {onMounted, ref} from "vue";

const maxHeight = ref(750);
const maxWidth = ref(750);
const r = 5;
const c = "rgb(29, 219, 22)";

let ctx = ref(null);
let xVal = ref(0);
let yVal = ref(0);
let useMouse = ref(false);
let create_dot_arr = ref([]);

onMounted(() => {
  const canvasObj = document.querySelector("#canvas");
  ctx = canvasObj.getContext('2d');
})

const coordinateEnter = () => {
  if (checkMaxXAndMaxY(xVal.value, yVal.value)) {
    dotDrawing(ctx, xVal.value, yVal.value, r);
  }
}


const draw = (e) => {
  if (!useMouse.value) {
    return false;
  }
  const x = e.offsetX;
  const y = e.offsetY;
  if (checkMaxXAndMaxY(x, y)) {
    xVal.value = x;
    yVal.value = y;
    dotDrawing(ctx, x, y, r);
  }
}

const checkMaxXAndMaxY = (x, y) => {
  if (!(x <= maxWidth.value && y <= maxHeight.value)) {
    alert(`최대 x좌표: ${maxWidth.value} \n최대 y좌표: ${maxHeight.value}\n제발 지키셈`)
    return false;
  } else {
    return true;
  }
}

const dotDrawing = (ctx, x, y, r) => {
  let color;
  if (create_dot_arr.value.length) {
    color = c;
    const beforeDot = create_dot_arr.value[create_dot_arr.value.length - 1];
    const beforeX = beforeDot.x;
    const beforeY = beforeDot.y;
    lineDrawing(ctx, beforeX, beforeY, x, y, 'red');
  } else {
    color = "rgb(22,48,219)";
  }
  if (ctx != null) {
    ctx.save();
    ctx.beginPath();
    ctx.fillStyle = color;
    ctx.arc(x, y, r, 0, Math.PI * 2, true);
    ctx.fill();
    ctx.restore();
  }
  const obj = {
    'color': c.value,
    'x': x,
    'y': y,
    'r': r
  };
  create_dot_arr.value.push(obj);
}

const lineDrawing = (ctx, sx, sy, ex, ey, color) => {
  if (ctx != null) {
    ctx.save();
    ctx.beginPath();
    ctx.strokeStyle = color;
    ctx.moveTo(sx, sy);
    ctx.lineTo(ex, ey);
    ctx.stroke();
    ctx.restore();
  }
}

let moveIndex = 0;
let xBeforeLineDot = 0;
let yBeforeLineDot = 0;
const play = () => {
  const nextLineDot = () => {
    const ax = create_dot_arr.value[moveIndex].x;
    const ay = create_dot_arr.value[moveIndex].y;
    if (moveIndex === 0) {
      xBeforeLineDot = ax;
      yBeforeLineDot = ay;
      moveIndex++;
      return {'x': ax, 'y': ay};
    }
    const yDistance = (create_dot_arr.value[moveIndex - 1].y - ay)
    const xDistance = (create_dot_arr.value[moveIndex - 1].x - ax)
    let inclination = yDistance / xDistance;
    if (Infinity === Math.abs(inclination)) {
      inclination = 999999999;
    }
    const yIntercept = ay - (inclination * ax);
    let x;
    let y;
    if (xBeforeLineDot < ax) {
      x = ++xBeforeLineDot;
      y = inclination * x + yIntercept;
    } else if (xBeforeLineDot > ax) {
      x = --xBeforeLineDot;
      y = inclination * x + yIntercept;
    } else {
      x = xBeforeLineDot;
      moveIndex++;
    }
    if ((xBeforeLineDot < ax && ax < x) || (xBeforeLineDot > ax && ax > x)) {
      moveIndex++;
    }
    return {'x': x, 'y': y};
  }

  if (moveIndex < create_dot_arr.value.length) {
    ctx.clearRect(0, 0, maxWidth.value, maxHeight.value); // 전체 지우기
    ctx.beginPath(); //새로 그리기~
    const nextDot = nextLineDot();
    // console.log(nextDot)
    ctx.arc(nextDot.x, nextDot.y, 5, 0, Math.PI * 2);
    ctx.fill();
    requestAnimationFrame(play); //루프를 통해 매번 새로그림
  } else {
    create_dot_arr.value = [];
    moveIndex = 0;
    xBeforeLineDot = 0;
    yBeforeLineDot = 0;
  }
}


</script>

<style scoped>

canvas {
  border: 1px solid black;
}

#box {
  width: 100%;
  height: 100%;
  position: relative;
}
</style>