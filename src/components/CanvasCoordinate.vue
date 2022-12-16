<template>
  <div id="box">
    X 좌표 :<input type="number" v-model="xVal">
    Y 좌표 :<input type="number" v-model="yVal">
    <p>
      <button type="button" @click="coordinateEnter">좌표 입력</button>
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

let ctx = ref(null);
let xVal = ref(0);
let yVal = ref(0);
let useMouse = ref(false);
const maxHeight = ref(750);
const maxWidth = ref(750);
const r = 5;
const c = "rgb(29, 219, 22)";

onMounted(() => {
  const canvasObj = document.getElementById("canvas");
  ctx = canvasObj.getContext('2d');
})

const coordinateEnter = () => {
  if (checkMaxXAndMaxY(xVal.value, yVal.value)) {
    dotDrawing(ctx, xVal.value, yVal.value, r, c);
  }
}

let create_dot_arr = ref([]);
const draw = (e) => {
  if(!useMouse.value){
    return false;
  }
  const x = e.offsetX;
  const y = e.offsetY;
  if (checkMaxXAndMaxY(x, y)) {
    xVal.value = x;
    yVal.value = y;
    dotDrawing(ctx, x, y, r, c);
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

const dotDrawing = (ctx, x, y, r, color) => {
  if (ctx != null) {
    ctx.save();
    ctx.beginPath();
    ctx.fillStyle = color;
    ctx.arc(x, y, r, 0, Math.PI * 2, true);
    ctx.fill();
    ctx.restore();
  }

  if (create_dot_arr.value.length) {
    const beforeDot = create_dot_arr.value[create_dot_arr.value.length - 1];
    const beforeX = beforeDot.x;
    const beforeY = beforeDot.y;
    lineDrawing(ctx, beforeX, beforeY, x, y, 'red');
    arrowDrawing(ctx, beforeX, beforeY, x, y, 'red');
  }
  const obj = {
    'color': color,
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

const arrowDrawing = (ctx, sx, sy, ex, ey, color) => {
  if (ctx != null) {
    var aWidth = 5;
    var aLength = 12;
    var dx = ex - sx;
    var dy = ey - sy;
    var angle = Math.atan2(dy, dx);
    var length = Math.sqrt(dx * dx + dy * dy);

    //두점 선긋기
    ctx.translate(sx, sy);
    ctx.rotate(angle);
    ctx.fillStyle = color;
    ctx.beginPath();

    ctx.fill();
    ctx.setTransform(1, 0, 0, 1, 0, 0);
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