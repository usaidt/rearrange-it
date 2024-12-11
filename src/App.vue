<template>
  <div id="app">
    <InfiniteCanvas />
    <div id="furniture-bucket">
      <h2>Furniture Bucket</h2>
      <div class="bucket-items">
        <DraggableSofa @add-to-canvas="addToCanvas" />
        <!-- Add more furniture components here -->
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';
import InfiniteCanvas from './components/InfiniteCanvas.vue';
import DraggableSofa from './components/DraggableSofa.vue';

export default defineComponent({
  name: 'App',
  components: { InfiniteCanvas, DraggableSofa },
  setup() {
    const canvasItems = ref<Array<{ type: string, position: { x: number, y: number } }>>([]);

    const addToCanvas = (type: string, position: { x: number, y: number }) => {
      canvasItems.value.push({ type, position });
    };

    return { addToCanvas };
  }
});
</script>

<style>
#app {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
}

#furniture-bucket {
  position: fixed;
  top: 20px;
  left: 20px;
  background-color: rgba(255, 255, 255, 0.8);
  padding: 10px;
  border-radius: 5px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.bucket-items {
  display: flex;
  gap: 10px;
}
</style>

