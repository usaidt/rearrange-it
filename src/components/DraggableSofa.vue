<template>
    <div
      class="draggable-item"
      :style="style"
      @mousedown="onDragStart"
      @mouseup="onDrop"
    >
      🛋️ Sofa
    </div>
  </template>
  
  <script lang="ts">
  import { defineComponent, reactive } from 'vue';
  import type { CSSProperties } from 'vue';
  
  export default defineComponent({
    name: 'DraggableSofa',
    setup(_, { emit }) {
      const style = reactive<CSSProperties>({
        position: 'relative',
        cursor: 'move',
        left: '0px',
        top: '0px',
      });
  
      let dragging = false;
      let startX = 0;
      let startY = 0;
  
      const onDragStart = (event: MouseEvent) => {
        dragging = true;
        startX = event.clientX - parseInt(String(style.left || '0'), 10);
        startY = event.clientY - parseInt(String(style.top || '0'), 10);

        const onDragMove = (e: MouseEvent) => {
          if (dragging) {
            style.left = `${e.clientX - startX}px`;
            style.top = `${e.clientY - startY}px`;
          }
        };
  
        const onDragEnd = () => {
          dragging = false;
          document.removeEventListener('mousemove', onDragMove);
          document.removeEventListener('mouseup', onDragEnd);
        };
  
        document.addEventListener('mousemove', onDragMove);
        document.addEventListener('mouseup', onDragEnd);
      };
  
      const onDrop = (event: MouseEvent) => {
        const canvas = document.getElementById('canvas') as HTMLCanvasElement;
        if (canvas) {
          const canvasRect = canvas.getBoundingClientRect();
          const position = {
            x: event.clientX - canvasRect.left,
            y: event.clientY - canvasRect.top,
          };
          emit('add-to-canvas', 'sofa', position);
        }
        style.left = '0px';
        style.top = '0px';
      };
  
      return { style, onDragStart, onDrop };
    },
  });
  </script>
  
  <style scoped>
  .draggable-item {
    background-color: #f4f4f4;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
    cursor: move;
    display: inline-block;
    user-select: none;
  }
  </style>
  