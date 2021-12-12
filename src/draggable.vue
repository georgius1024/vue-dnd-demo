<template>
  <div draggable="true" @dragstart="startDrag">
    <slot></slot>
  </div>
</template>
<script>
export default {
  methods: {
    startDrag(event) {
      event.dataTransfer.dropEffect = "move";
      event.dataTransfer.effectAllowed = "move";
      const deltaX = event.offsetX;
      const deltaY = event.offsetY;
      event.dataTransfer.setData("deltaX", deltaX);
      event.dataTransfer.setData("deltaY", deltaY);
    },
  },
};
</script>
