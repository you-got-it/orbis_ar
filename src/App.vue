<template>
  <div id="app">
    <router-view />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
//import AR from "./components/AR.vue";

@Component({
  components: {
    //  AR,
  },
})
export default class App extends Vue {
  mounted() {
    //console.log("mounted");
  }
}
</script>

<style lang="scss" scoped>
@import "~@/assets/scss/common/mixins";
@import "~@/assets/scss/common/variables";
#app {
  text-align: center;
  color: #2c3e50;
  margin: 0;
  padding: 0;
  border: 0;
}
.content {
  width: 100%;
  max-width: 375px;
  margin: 0 auto;

  @include desktop-up-layout {
    max-width: 746px;
  }
}
</style>
