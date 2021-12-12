<template>
  <droppable class="canvas" @drop="onDrop">
    <draggable :style="draggableStyle">
      <div class="marker" :style="markerStyle"></div>
    </draggable>
    <div class="panel">
      {{ position.x.toFixed(0) }} x {{ position.y.toFixed(0) }}
    </div>
  </droppable>
</template>
<script>
import draggable from "./draggable.vue";
import droppable from "./droppable.vue";
export default {
  components: {
    draggable,
    droppable
  },
  data() {
    return {
      width: 0,
      height: 0,
      markerSize: 50,
      position: {
        x: 0,
        y: 0,
      },
    };
  },
  computed: {
    draggableStyle() {
      return {
        cursor: "move",
        position: "absolute",
        left: `${this.position.x - this.markerSize / 2}px`,
        top: `${this.position.y - this.markerSize / 2}px`,
      };
    },
    markerStyle() {
      return {
        width: `${this.markerSize}px`,
        height: `${this.markerSize}px`,
      };
    },
  },
  mounted() {
    const { width, height } = this.$el.getBoundingClientRect();
    this.position.x = width / 2;
    this.position.y = height / 2;
    this.width = width;
    this.height = height;
  },
  methods: {
    startDrag(event) {
      event.dataTransfer.dropEffect = "move";
      event.dataTransfer.effectAllowed = "move";
      const deltaX = this.position.x - event.clientX;
      const deltaY = this.position.y - event.clientY;
      event.dataTransfer.setData("deltaX", deltaX);
      event.dataTransfer.setData("deltaY", deltaY);
    },
    onDrop(event) {
      const deltaX = this.markerSize / 2 - event.dataTransfer.getData("deltaX");
      const deltaY = this.markerSize / 2 - event.dataTransfer.getData("deltaY");
      console.log({ deltaX, deltaY });
      // const newPositionX = event.offsetX + deltaX;
      // const newPositionY = event.offsetY + deltaY;
      // if (
      //   newPositionX > this.markerSize / 2 &&
      //   newPositionX < this.width - this.markerSize / 2 &&
      //   newPositionY > this.markerSize / 2 &&
      //   newPositionY < this.height - this.markerSize / 2
      // ) {
      //   this.position.x = newPositionX;
      //   this.position.y = newPositionY;
      // }
      this.position.x = event.offsetX + deltaX;
      this.position.y = event.offsetY + deltaY;
    },
  },
};
</script>
<style lang="scss">
* {
  margin: 0;
  padding: 0;
  font-size: 16px;
  box-sizing: border-box;
}

.canvas {
  width: 50vw;
  height: 50vh;
  margin-left: 25vw;
  margin-top: 25vh;
  position: relative;
  background-color: #ccc;
  .panel {
    position: absolute;
    width: 160px;
    height: 32px;
    border: 1px solid #333;
    background-color: #fff;
    display: flex;
    align-items: center;
    justify-content: center;
    top: -32px;
    left: calc(25vw - 80px);
  }
  .marker {
    position: absolute;
    background-color: #333;
    border-radius: 100%;
    outline: none;
    transform: translate(0, 0); // <== black magic from
    // https://github.com/react-dnd/react-dnd/issues/788#issuecomment-367300464
    //transition: all 200ms ease;
  }
}
</style>
