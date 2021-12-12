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
<style lang="scss" scoped>
div {
  transform: translate(0, 0); // <== black magic from
  background-color: transparent;
}
</style>