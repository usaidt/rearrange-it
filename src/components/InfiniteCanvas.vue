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
  lastY: 0
});

const toVirtualX = (xReal: number): number => (xReal + state.offsetX) * state.scale;
const toVirtualY = (yReal: number): number => (yReal + state.offsetY) * state.scale;
const toRealX = (xVirtual: number): number => xVirtual / state.scale - state.offsetX;
const toRealY = (yVirtual: number): number => yVirtual / state.scale - state.offsetY;

const virtualHeight = (): number => (canvasRef.value?.clientHeight ?? 0) / state.scale;
const virtualWidth = (): number => (canvasRef.value?.clientWidth ?? 0) / state.scale;

const zoom = (amount: number, centerX: number, centerY: number): void => {
  const oldScale = state.scale;
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

onMounted(() => {
  if (canvasRef.value) {
    context.value = canvasRef.value.getContext('2d');
    
    canvasRef.value.addEventListener("mousedown", onMouseDown);
    document.addEventListener("mousemove", onMouseMove);
    document.addEventListener("mouseup", onMouseUp);
    canvasRef.value.addEventListener("wheel", onWheel);

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

