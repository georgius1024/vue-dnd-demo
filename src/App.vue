<template>
  <div class="layout">
    <aside class="picker">
      <DemoBlock
        v-for="(item, index) in pickerItems"
        :id="item.id"
        :x="100 - 3"
        :y="50 + index * 120"
        :size="100"
        :color="item.color"
        :content="item.content"
        :key="item.id"
      />
    </aside>
    <main
      class="canvas"
      ref="canvas"
      @drop="onDrop"
      @dragover.prevent
      @dragenter.prevent
    >
      <DemoBlock
        v-for="item in scene"
        :id="item.id"
        :x="item.x"
        :y="item.y"
        :size="100"
        :color="item.color"
        :content="item.content"
        :key="item.id"
      />
    </main>
  </div>
</template>
<script>
import { nanoid } from "nanoid";
import draggable from "./draggable.vue";
import droppable from "./droppable.vue";
import DemoBlock from "./components/DemoBlock.vue";

export default {
  components: {
    draggable,
    droppable,
    DemoBlock,
  },
  data() {
    return {
      scene: [],
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
    pickerItems() {
      return [
        {
          id: 101,
          color: "red",
          content: "🐵",
        },
        {
          id: 102,
          color: "navy",
          content: "🦑",
        },
        {
          id: 103,
          color: "yellow",
          content: "🐶",
        },
        {
          id: 104,
          color: "darkblue",
          content: "🐱",
        },
      ];
    },
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
    const { width, height } = this.$refs.canvas.getBoundingClientRect();
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
      const id = event.dataTransfer.getData("id");
      const shiftX = +event.dataTransfer.getData("shiftX");
      const shiftY = +event.dataTransfer.getData("shiftY");
      if (+id) {
        const pickerItem = this.pickerItems.find((e) => e.id === +id);
        if (pickerItem) {
          this.scene.push({
            ...pickerItem,
            id: nanoid(),
            x: event.offsetX + +shiftX,
            y: event.offsetY + +shiftY,
          });
        }
      } else {
        this.scene = this.scene.map((e) => {
          if (e.id === id) {
            return {
              ...e,
              x: event.offsetX + +shiftX,
              y: event.offsetY + +shiftY,
            };
          }
          return e;
        });
      }

      //this.pickedItems
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

.layout {
  display: grid;
  grid-template-columns: 200px 1fr;
  grid-template-rows: 1fr;
  grid-template-areas: "sidebar main";
  width: 100vw;
  height: 100vh;
}
.picker {
  grid-area: sidebar;
  background-color: aqua;
  border-right: 6px solid #333;
  cursor: default;
  position: relative;
}

.canvas {
  grid-area: main;
  position: relative;
  background-color: #ccc;
  display: flex;

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
