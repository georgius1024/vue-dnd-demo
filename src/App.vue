<template>
  <div class="layout">
    <aside class="picker">
      <DemoBlock
        v-for="(item, index) in pickerItems"
        :id="item.id"
        :x="100 - 3"
        :y="50 + index * 120"
        :size="80"
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
    <header class="header">
      <button @click="undo" :disabled="!undoable">
        <svg style="width: 24px; height: 24px" viewBox="0 0 24 24">
          <path
            fill="currentColor"
            d="M12.5,8C9.85,8 7.45,9 5.6,10.6L2,7V16H11L7.38,12.38C8.77,11.22 10.54,10.5 12.5,10.5C16.04,10.5 19.05,12.81 20.1,16L22.47,15.22C21.08,11.03 17.15,8 12.5,8Z"
          />
        </svg>
      </button>
      <button @click="redo" :disabled="!redoable">
        <svg style="width: 24px; height: 24px" viewBox="0 0 24 24">
          <path
            fill="currentColor"
            d="M18.4,10.6C16.55,9 14.15,8 11.5,8C6.85,8 2.92,11.03 1.54,15.22L3.9,16C4.95,12.81 7.95,10.5 11.5,10.5C13.45,10.5 15.23,11.22 16.62,12.38L13,16H22V7L18.4,10.6Z"
          />
        </svg>
      </button>
    </header>
    <main
      class="canvas"
      ref="canvas"
      @mousedown="spotCoords"
      @drop="onDrop"
      @dragover.prevent
      @dragenter.prevent
    >
      <span
        v-for="step in 20"
        :key="step"
        :style="{
          position: 'absolute',
          left: `${(step - 1) * 100 + 100}px`,
          borderLeft: '1px solid black',
        }"
      >
        {{ step }}
      </span>
      <span
        v-for="step in 10"
        :key="step"
        :style="{
          position: 'absolute',
          top: `${(step - 1) * 100 + 100}px`,
          transform: 'rotate(-90deg)',
          borderRight: '1px solid black',
          paddingTop: '6px',
        }"
      >
        {{ step }}
      </span>
      <template v-if="history">
        <DemoBlock
          v-for="item in flatScene"
          :id="item.id"
          :x="gridToCanvas(item.x, item.y).x"
          :y="gridToCanvas(item.x, item.y).y"
          :size="80"
          :color="item.color"
          :content="item.content"
          :key="item.id"
        />
      </template>
    </main>
  </div>
</template>
<script>
import { nanoid } from "nanoid";
import draggable from "./draggable.vue";
import droppable from "./droppable.vue";
import DemoBlock from "./components/DemoBlock.vue";
import {
  initialize,
  addState,
  getCurrent,
  undoable,
  redoable,
  undo,
  redo,
} from "./utils/history";

const GRID_STEP = 100;
const OFFSET = 100;
export default {
  components: {
    draggable,
    droppable,
    DemoBlock,
  },
  data() {
    return {
      history: null,
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
    scene() {
      return this.history ? getCurrent(this.history) : [];
    },
    flatScene() {
      return this.scene.flat().filter(Boolean);
    },
    undoable() {
      return Boolean(this.history && undoable(this.history));
    },
    redoable() {
      return Boolean(this.history && redoable(this.history));
    },
    hasUnsavedChanges() {
      return this.undoable;
    },
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
    //const { width, height } = this.$refs.canvas.getBoundingClientRect();
    const { scrollWidth, scrollHeight } = this.$refs.canvas;

    this.width = scrollWidth;
    this.height = scrollHeight;
    const maxCol = Math.ceil((this.width - 2 * OFFSET) / GRID_STEP);
    const maxRow = Math.ceil((this.height - 2 * OFFSET) / GRID_STEP);
    const scene = [];
    for (let row = 0; row < maxRow; row++) {
      scene[row] = [];
      for (let col = 0; col < maxCol; col++) {
        scene[row][col] = null;
      }
    }
    this.history = initialize(scene);

    this.keyHandler = (e) => {
      const editing = e.target.getAttribute("contenteditable") === "true";
      if (editing) {
        return;
      }
      const undoPressed =
        (e.code === "KeyZ" && e.ctrlKey) ||
        (e.code === "Backspace" && e.ctrlKey);
      const redoPressed = e.code === "KeyY" && e.ctrlKey;
      if (undoPressed && this.undoable) {
        this.undo();
        e.stopPropagation();
      }
      if (redoPressed && this.redoable) {
        this.redo();
        e.stopPropagation();
      }
    };
    document.addEventListener("keydown", this.keyHandler);
  },
  beforeDestroy() {
    document.removeEventListener("keydown", this.keyHandler);
  },
  methods: {
    updateScene(scene) {
      this.history = addState(this.history, scene);
    },
    undo() {
      this.history = undo(this.history);
    },
    redo() {
      this.history = redo(this.history);
    },
    spotCoords(event) {
      // const { offsetLeft, offsetTop, scrollLeft, scrollTop } =
      //   this.$refs.canvas;
      // console.log("page", event.pageX, event.pageY);
      // console.log(
      //   "fixed",
      //   event.pageX - offsetLeft + scrollLeft,
      //   event.pageY - offsetTop + scrollTop
      // );
      // console.log("offset", { offsetLeft, offsetTop, scrollLeft, scrollTop });
      // console.log("offset", event.offsetX, event.offsetY);
    },
    startDrag(event) {
      event.dataTransfer.dropEffect = "move";
      event.dataTransfer.effectAllowed = "move";
      const deltaX = this.position.x - event.clientX;
      const deltaY = this.position.y - event.clientY;
      event.dataTransfer.setData("deltaX", deltaX);
      event.dataTransfer.setData("deltaY", deltaY);
    },
    gridToCanvas(col, row) {
      const x = col * GRID_STEP + OFFSET;
      const y = row * GRID_STEP + OFFSET;
      return { x, y };
    },
    snapToGrid(x, y) {
      const col = Math.round((x - OFFSET) / GRID_STEP); //Math.round(Math.min(x, this.width - 2 * OFFSET) / GRID_STEP);
      const row = Math.round((y - OFFSET) / GRID_STEP); // Math.round(Math.min(y, this.height - 2 * OFFSET) / GRID_STEP);
      return { col, row };
    },
    placeAt(row, col, item) {
      const scene = this.scene.map((e) => [...e]);
      if (scene[row][col] && scene[row][col].id !== item.id) {
        if (!scene[row][col + 1] && col < scene[row].length - 1) {
          scene[row][col + 1] = scene[row][col];
        } else if (!this.scene[row + 1][col] && row < this.scene.length - 1) {
          scene[row + 1][col] = scene[row][col];
        } else {
          return false;
        }
      }
      scene[item.y][item.x] = false;
      scene[row][col] = { ...item, x: col, y: row };
      this.updateScene(scene);
      return true;
    },
    onDrop(event) {
      const { offsetLeft, offsetTop, scrollLeft, scrollTop } =
        this.$refs.canvas;
      const id = event.dataTransfer.getData("id");
      // const shiftX = +event.dataTransfer.getData("shiftX");
      // const shiftY = +event.dataTransfer.getData("shiftY");
      const coords = {
        x: event.pageX - offsetLeft + scrollLeft,
        y: event.pageY - offsetTop + scrollTop,
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
      console.log(col, row);
      if (+id) {
        const pickerItem = this.pickerItems.find((e) => e.id === +id);
        if (pickerItem) {
          this.placeAt(row, col, {
            ...pickerItem,
            id: nanoid(),
            x: col,
            y: row,
          });
        }
      } else {
        const sceneItem = this.scene.flat().find((e) => e && e.id === id);
        this.placeAt(row, col, sceneItem);
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
.separator {
  flex-grow: 1;
}

.layout {
  display: grid;
  grid-template-columns: 200px 1fr;
  grid-template-rows: 64px 1fr;
  grid-template-areas: "header header" "sidebar main";
  width: 100vw;
  height: 100vh;
}
.header {
  grid-area: header;
  display: flex;
  align-items: center;
  background-color: #ccc7;
  button {
    border-radius: 100%;
    background-color: transparent;
    display: flex;
    align-items: center;
    justify-content: center;
    border: none;
    width: 32px;
    height: 32px;
    margin: 32px;
    color: #333;
    cursor: pointer;
    &:disabled {
      background-color: #ccc7;
      color: #3337;
    }
  }
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

.canvas {
  grid-area: main;
  position: relative;
  background-color: #ccc;
  display: flex;
  overflow-y: hidden;
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
