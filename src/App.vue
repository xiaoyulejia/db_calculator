<template>
  <div id="app">
    <Calculator />
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import Calculator from "./components/Calculator.vue";

export default defineComponent({
  name: "App",
  components: {
    Calculator,
  },
});
</script>

<style>
#app {
  text-align: center;
  margin-top: 2rem;
}
</style>
