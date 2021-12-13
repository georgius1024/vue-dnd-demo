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
        <span style="font-size: 75%">{{ (step - 1) * 100 + 100 }}</span>
      </span>
      <span
        v-for="step in 10"
        :key="step"
        :style="{
          position: 'absolute',
          top: `${(step - 1) * 100 + 100}px`,
          borderTop: '1px solid black',
        }"
      >
        {{ step }}
        <span style="font-size: 75%">{{ (step - 1) * 100 + 100 }}</span>
      </span>

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
