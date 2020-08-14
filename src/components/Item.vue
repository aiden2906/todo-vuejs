<template>
  <div class="item">
    <input type="checkbox" v-model="status" class="cb" />
    <p class="title">{{ value.name }}</p>
    <button class="btn-remove" v-on:click="remove"><i class="fa fa-times"></i></button>
  </div>
</template>

<script>
export default {
  name: "Item",
  data() {
    return {
      name: "",
      status: false,
      id: 0,
    };
  },
  props: {
    value: {
      type: Object,
      required: true,
    },
  },
  mounted() {
    this.status = this.value ? this.value.status : false;
    this.id = this.value ? this.value.id : 0;
  },
  watch: {
    status() {
      this.$emit("change-status", this.status);
    },
    value() {
      this.status = this.value ? this.value.status : false;
    },
  },
  methods: {
    remove() {
      this.$emit("remove-item", this.id);
    },
  },
};
</script>

<style>
.cb {
  grid-column-start: 1;
  grid-column-end: 1;
  margin: auto;
  color: #99c191;
  cursor: pointer;
}
.item {
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
.title {
  grid-column-start: 2;
  grid-column-end: 2;
  display: flex;
  justify-content: flex-start;
  align-items: center;
  font-size: 1.6rem;
}
.btn-remove{
  border: none;
}
</style>


