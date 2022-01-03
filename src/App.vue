<template>
  <div id="app">
    <Vuedate />
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import Vuedate from "@/components/VueDate.vue";

export default Vue.extend({
  name: "App",
  components: {
    Vuedate,
  },
});
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
