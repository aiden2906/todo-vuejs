<template>
  <div id="app" class="container">
    <div class="group-col">
      <Container
        @drop="onDrop"
        drag-class="opacity-ghost"
        drop-class="opacity-ghost-drop"
        group-name="col"
      >
        <div class="add-item">
          <input
            class="input"
            type="text"
            v-model="newItem"
            v-on:keyup.enter="addItem()"
            placeholder="Add a to-do..."
          />
          <button v-on:click="addItem()" class="feature-icon">
            <i class="fa fa-plus"></i>
          </button>
        </div>
        <Draggable v-for="item in filterItem" :key="item.id">
          <Item
            :value="item"
            @change-status="(status) => changeItem(status, item.id)"
            @remove-item="(id)=>removeItem(id)"
            :key="item.id"
          ></Item>
        </Draggable>
      </Container>
      <!-- <Container
        @drop="onDrop"
        drag-class="opacity-ghost"
        drop-class="opacity-ghost-drop"
        group-name="col"
      >
        <div class="add-item">
          <input
            class="input"
            type="text"
            v-model="newItem"
            v-on:keyup.enter="addItem()"
            placeholder="Add a to-do..."
          />
          <button v-on:click="addItem()" class="feature-icon">
            <i class="fa fa-plus"></i>
          </button>
        </div>
        <Draggable v-for="item in filterItem" :key="item.id">
          <Item
            :value="item"
            @change-status="(status) => changeItem(status, item.id)"
            @remove-item="(id)=>removeItem(id)"
            :key="item.id"
          ></Item>
        </Draggable>
      </Container> -->
    </div>
    <button v-on:click="changeStatus('all')" class="btn btn-all">All</button>
    <button v-on:click="changeStatus('active')" class="btn btn-active">Active</button>
    <button v-on:click="changeStatus('complete')" class="btn btn-complete">Complete</button>
    <button v-on:click="clearAll" class="btn btn-clear">Clear all</button>
  </div>
</template>

<script>
import Item from "./components/Item";
import { Container, Draggable } from "vue-smooth-dnd";
import { applyDrag } from "./utils/common";

let id = 0;

export default {
  name: "App",
  data() {
    return {
      items: [],
      status: "all",
      newItem: "",
    };
  },
  created() {
    this.getData();
  },
  computed: {
    filterItem() {
      console.log("a");
      if (this.status === "all") {
        return this.items;
      }
      if (this.status === "active") {
        return this.items.filter((i) => i.status === false);
      }
      return this.items.filter((i) => i.status === true);
    },
  },
  methods: {
    onCardDrop(columnId, dropResult) {
      if (dropResult.removedIndex !== null || dropResult.addedIndex !== null) {
        const scene = Object.assign({}, this.scene);
        const column = scene.children.filter((p) => p.id === columnId)[0];
        const columnIndex = scene.children.indexOf(column);

        const newColumn = Object.assign({}, column);
        newColumn.children = applyDrag(newColumn.children, dropResult);
        scene.children.splice(columnIndex, 1, newColumn);

        this.scene = scene;
      }
    },
    onDrop(dropResult) {
      this.items = applyDrag(this.items, dropResult);
      localStorage.setItem("items", JSON.stringify(this.items));
    },
    getData() {
      if (localStorage.getItem("items")) {
        this.items = JSON.parse(localStorage.getItem("items")) || [];
        id = JSON.parse(localStorage.getItem("id"));
      }
    },
    addItem() {
      if (this.newItem === "") return;
      this.items.push({
        name: this.newItem,
        status: false,
        id: id++,
      });
      localStorage.setItem("items", JSON.stringify(this.items));
      localStorage.setItem("id", JSON.stringify(id));
      this.newItem = "";
    },
    changeItem(status, id) {
      const index = this.items.findIndex((i) => i.id === id);
      this.$set(this.items, index, { ...this.items[index], status });
      localStorage.setItem("items", JSON.stringify(this.items));
      localStorage.setItem("id", JSON.stringify(id));
    },
    changeStatus(status) {
      this.status = status;
    },
    removeItem(id) {
      const index = this.items.findIndex((i) => i.id === id);
      this.items.splice(index, 1);
      localStorage.setItem("items", JSON.stringify(this.items));
      localStorage.setItem("id", JSON.stringify(id));
    },
    clearAll() {
      this.items = [];
      localStorage.removeItem("items");
      localStorage.removeItem("id");
    },
    getCardPayload (columnId) {
      return index => {
        return this.scene.children.filter(p => p.id === columnId)[0].children[index]
      }
    },
  },
  components: {
    Item,
    Container,
    Draggable,
  },
};
</script>

<style>
#app {
  font-family: cursive, Helvetica, sans-serif;
  font-size: 1rem;
  color: #333;
  width: 100%;
  max-width: 480px;
}

body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  background: linear-gradient(to bottom right, #99c191, #c3d98d);
}
.container {
  padding: 10px;
  width: calc(100% - 20px);
  min-height: calc(100% - 20px);
}

/* .group-col {
} */

.container .add-item input {
  font-family: cursive, Helvetica, sans-serif;
  grid-column-start: 2;
  grid-column-end: 2;
  background: none;
  border: none;
  width: 100%;
  height: 60px;
  color: #fff;
  font-size: 1.6rem;
}
.container .add-item input::placeholder {
  color: #fff;
}
.container .add-item .feature-icon {
  grid-column-start: 3;
  grid-column-end: 3;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 1.6rem;
  margin: 0 15px;
  cursor: pointer;
  background-color: transparent;
  border: none;
}
.container .add-item {
  display: grid;
  grid-template-columns: 70px auto 70px;
  grid-template-rows: 60px;
  width: 100%;
  height: 60px;
  background: rgba(0, 0, 0, 0.3);
  border-radius: 5px;
  overflow: hidden;
}
.container .todo-list {
  padding: 0;
}
.container .todo-list .item {
  display: grid;
  grid-template-columns: 70px auto 70px;
  grid-template-rows: 60px;
  width: 100%;
  height: 60px;
  margin-bottom: 2px;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 5px;
  overflow: hidden;
  transition: all 0.5s linear;
}
.container .todo-list .item .item-checkbox {
  grid-column-start: 1;
  grid-column-end: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 1.6rem;
  color: #99c191;
  cursor: pointer;
}
.container .todo-list .item .item-title {
  grid-column-start: 2;
  grid-column-end: 2;
  display: flex;
  justify-content: flex-start;
  align-items: center;
  font-size: 1.6rem;
}
.container .todo-list .item .feature-icon {
  grid-column-start: 3;
  grid-column-end: 3;
  display: flex;
  justify-content: center;
  align-items: center;
  color: #333;
  font-size: 1.6rem;
  margin: 0 15px;
  cursor: pointer;
}
.btn {
  font-family: cursive, Helvetica, sans-serif;
  background-color: #4caf50; /* Green */
  border: none;
  color: white;
  padding: 16px 30px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin: 4px 2px;
  transition-duration: 0.4s;
  cursor: pointer;
}
.btn-all {
  background-color: white;
  color: black;
  border: 2px solid #008cba;
}
.btn-all:hover {
  background-color: #008cba;
  color: white;
}
.btn-active {
  background-color: white;
  color: black;
  border: 2px solid #f44336;
}
.btn-active:hover {
  background-color: #f44336;
  color: white;
}
.btn-complete {
  background-color: white;
  color: black;
  border: 2px solid #4caf50;
}
.btn-complete:hover {
  background-color: #4caf50;
  color: white;
}
.btn-clear {
  background-color: white;
  color: black;
  border: 2px solid #555555;
}
.btn-clear:hover {
  background-color: #555555;
  color: white;
}
.opacity-ghost {
  transition: all 0.18s ease;
  opacity: 0.8;
  /* transform: rotateZ(5deg); */
  background-color: cornflowerblue;
  box-shadow: 3px 3px 10px 3px rgba(0, 0, 0, 0.3);
}

.opacity-ghost-drop {
  opacity: 1;
  /* transform: rotateZ(0deg); */
  background-color: white;
  box-shadow: 3px 3px 10px 3px rgba(0, 0, 0, 0);
}
.draggable-item {
  height: 50px;
  line-height: 50px;
  text-align: center;
  display: block;
  background-color: #fff;
  outline: 0;
  border: 1px solid rgba(0, 0, 0, 0.125);
  margin-bottom: 2px;
  margin-top: 2px;
  cursor: default;
  user-select: none;
}

.draggable-item-horizontal {
  height: 300px;
  padding: 10px;
  line-height: 100px;
  text-align: center;
  /* width : 200px; */
  display: block;
  background-color: #fff;
  outline: 0;
  border: 1px solid rgba(0, 0, 0, 0.125);
  margin-right: 4px;
  cursor: default;
}

.dragging {
  background-color: yellow;
}
</style>
