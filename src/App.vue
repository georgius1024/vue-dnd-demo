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
      <div class="separator"></div>
      <div
        class="trash-can"
        @drop="onDelete"
        @dragover.prevent
        @dragenter.prevent
        v-text="'🗑️'"
      />
    </aside>
    <main
      class="canvas"
      ref="canvas"
      @drop="onDrop"
      @dragover.prevent
      @dragenter.prevent
    >
      <template v-for="(row, rowIndex) in scene">
        <template v-for="(item, colIndex) in row">
          <DemoBlock
            v-if="item"
            :id="item.id"
            :x="gridToCanvas(colIndex, rowIndex).x"
            :y="gridToCanvas(colIndex, rowIndex).y"
            :size="100"
            :color="item.color"
            :content="item.content"
            :key="item.id"
          />
        </template>
      </template>
    </main>
  </div>
</template>
<script>
import { nanoid } from "nanoid";
import draggable from "./draggable.vue";
import droppable from "./droppable.vue";
import DemoBlock from "./components/DemoBlock.vue";
const GRID_STEP = 150;
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
    this.width = width;
    this.height = height;
    const maxCol = Math.floor(this.width / GRID_STEP);
    const maxRow = Math.floor(this.height / GRID_STEP);

    for (let row = 0; row < maxRow; row++) {
      this.scene[row] = [];
      for (let col = 0; col < maxCol; col++) {
        this.scene[row][col] = null;
      }
    }
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
    gridToCanvas(col, row) {
      const x = col * GRID_STEP + GRID_STEP / 2;
      const y = row * GRID_STEP + GRID_STEP / 2;
      return { x, y };
    },
    snapToGrid(x, y) {
      const col = Math.round(
        Math.min(x, this.width - GRID_STEP / 2) / GRID_STEP - 0.5
      );
      const row = Math.round(
        Math.min(y, this.height - GRID_STEP / 2) / GRID_STEP - 0.5
      );
      return { col, row };
    },
    validMoves(row, col) {
      const moves = [];
      //if (this.scene[row+1])
      return moves;
    },
    onDrop(event) {
      const { left, top } = this.$refs.canvas.getBoundingClientRect();
      const id = event.dataTransfer.getData("id");
      const shiftX = +event.dataTransfer.getData("shiftX");
      const shiftY = +event.dataTransfer.getData("shiftY");
      const coords = {
        x: event.pageX - left + shiftX,
        y: event.pageY - top + shiftY,
      };
      const { row, col } = this.snapToGrid(coords.x, coords.y);
      if (
        col < 0 ||
        col > this.scene[0].length - 1 ||
        row < 0 ||
        row > this.scene.length - 1
      ) {
        return;
      }

      if (+id) {
        const pickerItem = this.pickerItems.find((e) => e.id === +id);
        if (pickerItem) {
          this.scene[row][col] = {
            ...pickerItem,
            id: nanoid(),
            x: col,
            y: row,
          };
        }
      } else {
        const sceneItem = this.scene.flat().find((e) => e && e.id === id);
        this.scene[sceneItem.y][sceneItem.x] = false;
        this.scene[row][col] = {
          ...sceneItem,
          x: col,
          y: row,
        };
      }
    },
    onDelete(event) {
      const id = event.dataTransfer.getData("id");
      if (+id) {
        return;
      }
      const sceneItem = this.scene.flat().find((e) => e && e.id === id);
      this.scene[sceneItem.y][sceneItem.x] = false;
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
  display: flex;
  flex-direction: column;
  .trash-can {
    height: 200;
    font-size: 75px;
    display: flex;
    align-items: center;
    justify-content: center;
  }
}
.separator {
  flex-grow: 1;
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
