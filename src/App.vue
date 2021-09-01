<template>
  <div id="app">
    <HintScreen v-if="page === 0" @buttonClick="buttonClick" />
    <AR v-if="page === 1" @playSound="playSound" />
  </div>
</template>

<script>
import { Component, Vue } from "vue-property-decorator";
import AR from "./components/AR.vue";
import HintScreen from "./components/HintScreen.vue";
import { useSound } from "@vueuse/sound";

@Component({
  components: {
    AR,
    HintScreen,
  },
  setup() {
    //const { play } = useSound("/audio/0.mp3");
    // const play0 = useSound("/audio/0.mp3");
    const audios = [
      useSound("./audio/0.mp3"),
      useSound("./audio/1.mp3"),
      useSound("./audio/2.mp3"),
      useSound("./audio/3.mp3"),
      useSound("./audio/4.mp3"),
    ];
    const handlePlay = (num) => {
      audios.forEach((a) => {
        a.stop();
      });
      audios[num].play();
    };
    return {
      handlePlay,
    };
  },
})
export default class App extends Vue {
  page = 0;
  mounted() {
    //console.log("mounted");
    //this.setup();
  }

  buttonClick() {
    this.page = 1;
    //this.handlePlay(0);
  }
  playSound(num) {
    this.handlePlay(num);
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
