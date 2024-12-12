<template>
  <canvas ref="canvasRef" id="canvas"></canvas>
</template>

<script setup lang="ts">
import { ref, onMounted, reactive } from 'vue';

const canvasRef = ref<HTMLCanvasElement | null>(null);
const context = ref<CanvasRenderingContext2D | null>(null);
const cellSize = 40;

const state = reactive({
  scale: 1,
  offsetX: 0,
  offsetY: 0,
  isDragging: false,
  lastX: 0,
  lastY: 0,
  touchMode: "single" as "single" | "double",
  prevTouch: [null as Touch | null, null as Touch | null],
});

const virtualHeight = (): number => (canvasRef.value?.clientHeight ?? 0) / state.scale;
const virtualWidth = (): number => (canvasRef.value?.clientWidth ?? 0) / state.scale;

const zoom = (amount: number, centerX: number, centerY: number): void => {
  state.scale *= amount;
  
  const zoomRatioX = centerX / (canvasRef.value?.clientWidth ?? 1);
  const zoomRatioY = centerY / (canvasRef.value?.clientHeight ?? 1);

  const unitsZoomedX = virtualWidth() * (1 - amount);
  const unitsZoomedY = virtualHeight() * (1 - amount);

  const unitsAddLeft = unitsZoomedX * zoomRatioX;
  const unitsAddTop = unitsZoomedY * zoomRatioY;

  state.offsetX += unitsAddLeft;
  state.offsetY += unitsAddTop;

  draw();
};

const draw = (): void => {
  if (canvasRef.value && context.value) {
    canvasRef.value.width = document.body.clientWidth;
    canvasRef.value.height = document.body.clientHeight;
    context.value.clearRect(0, 0, canvasRef.value.width, canvasRef.value.height);
    drawGrid();
  }
};

const drawGrid = (): void => {
  if (canvasRef.value && context.value) {
    context.value.strokeStyle = "rgb(229,231,235)";
    context.value.lineWidth = 1;
    context.value.beginPath();

    const width = canvasRef.value.clientWidth;
    const height = canvasRef.value.clientHeight;

    for (
      let x = (state.offsetX % cellSize) * state.scale;
      x <= width;
      x += cellSize * state.scale
    ) {
      const source = x;
      context.value.moveTo(source, 0);
      context.value.lineTo(source, height);
    }

    for (
      let y = (state.offsetY % cellSize) * state.scale;
      y <= height;
      y += cellSize * state.scale
    ) {
      const destination = y;
      context.value.moveTo(0, destination);
      context.value.lineTo(width, destination);
    }
    context.value.stroke();
  }
};

const onMouseDown = (event: MouseEvent): void => {
  state.isDragging = true;
  state.lastX = event.clientX;
  state.lastY = event.clientY;
};

const onMouseMove = (event: MouseEvent): void => {
  if (state.isDragging) {
    const deltaX = event.clientX - state.lastX;
    const deltaY = event.clientY - state.lastY;
    state.offsetX += deltaX / state.scale;
    state.offsetY += deltaY / state.scale;
    state.lastX = event.clientX;
    state.lastY = event.clientY;
    draw();
  }
};

const onMouseUp = (): void => {
  state.isDragging = false;
};

const onWheel = (event: WheelEvent): void => {
  event.preventDefault();
  const zoomFactor = event.deltaY > 0 ? 0.9 : 1.1;
  zoom(zoomFactor, event.clientX, event.clientY);
};

const onTouchStart = (event: TouchEvent): void => {
  const touches = event.touches;
  if (touches.length === 1) {
    state.touchMode = "single";
  } else if (touches.length >= 2) {
    state.touchMode = "double";
  }
  state.prevTouch[0] = touches[0];
  state.prevTouch[1] = touches[1];
};

const onTouchMove = (event: TouchEvent): void => {
  const touches = event.touches;
  if (state.touchMode === "single" && touches.length === 1) {
    const deltaX = touches[0].pageX - (state.prevTouch[0]?.pageX ?? touches[0].pageX);
    const deltaY = touches[0].pageY - (state.prevTouch[0]?.pageY ?? touches[0].pageY);
    state.offsetX += deltaX / state.scale;
    state.offsetY += deltaY / state.scale;
    draw();
  } else if (state.touchMode === "double" && touches.length === 2) {
    const touch0 = touches[0];
    const touch1 = touches[1];
    const prevTouch0 = state.prevTouch[0]!;
    const prevTouch1 = state.prevTouch[1]!;

    const currentDist = Math.hypot(touch0.pageX - touch1.pageX, touch0.pageY - touch1.pageY);
    const prevDist = Math.hypot(prevTouch0.pageX - prevTouch1.pageX, prevTouch0.pageY - prevTouch1.pageY);

    const zoomFactor = currentDist / prevDist;
    const midX = (touch0.pageX + touch1.pageX) / 2;
    const midY = (touch0.pageY + touch1.pageY) / 2;

    zoom(zoomFactor, midX, midY);

    const panX = (touch0.pageX + touch1.pageX - prevTouch0.pageX - prevTouch1.pageX) / 2;
    const panY = (touch0.pageY + touch1.pageY - prevTouch0.pageY - prevTouch1.pageY) / 2;

    state.offsetX += panX / state.scale;
    state.offsetY += panY / state.scale;
  }
  state.prevTouch[0] = touches[0];
  state.prevTouch[1] = touches[1];
};

onMounted(() => {
  if (canvasRef.value) {
    context.value = canvasRef.value.getContext("2d");

    canvasRef.value.addEventListener("mousedown", onMouseDown);
    document.addEventListener("mousemove", onMouseMove);
    document.addEventListener("mouseup", onMouseUp);
    canvasRef.value.addEventListener("wheel", onWheel);
    canvasRef.value.addEventListener("touchstart", onTouchStart);
    canvasRef.value.addEventListener("touchmove", onTouchMove);

    window.addEventListener("resize", () => draw());
    draw();
  }
});
</script>

<style scoped>
canvas {
  width: 100vw;
  height: 100vh;
  position: absolute;
  top: 0;
  left: 0;
}
</style>
