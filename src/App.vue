<template>
  <div id="app">
    <AR />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import AR from "./components/AR.vue";

@Component({
  components: {
    AR,
  },
})
export default class App extends Vue {
  mounted() {
    //console.log("mounted");
  }
}
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
