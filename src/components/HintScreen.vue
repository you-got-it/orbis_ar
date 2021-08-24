<template>
  <div class="content">
    <img class="logo" src="@/assets/images/logo.svg" />
    <canvas
      ref="canvas"
      class="content__canvas"
      width="440"
      height="260"
      :class="{ 'button--visible': canvasVisible, transparent: transparent }"
    />
    <div class="text-center">
      <!-- <h2
        class="content__text"
        ref="hitText"
        :class="{ 'button--visible': buttonVisible, transparent: transparent }"
      >
        <span
          >Hold your phone over the square box <br class="content__break" />on
          the card</span
        >
      </h2> -->

      <router-link
        to="/ar"
        class="btn btn__primary content__button"
        :class="{ 'button--visible': buttonVisible, transparent: transparent }"
        >START THE STORY</router-link
      >
    </div>
    <a href="www.orbis.org">www.orbis.org</a>
  </div>
</template>

<style lang="scss" scoped>
.logo {
  width: 150px;
}
.content {
  height: 100%;
  display: flex;
  justify-content: center;
  flex-direction: column;
  align-items: center;
  &__break {
    @media (orientation: landscape) {
      display: none;
    }
  }
  &__canvas {
    @media (orientation: portrait) {
      width: 100%;
      max-width: 440px;
      margin-left: auto;
      margin-right: auto;
    }
    @media (orientation: landscape) {
      height: 100%;
      max-height: 300px;
    }
    transition: opacity 0.6s ease-in-out;
    opacity: 0;
    margin-bottom: 22px;
    object-fit: contain;
  }
  &__button {
    visibility: hidden;
    opacity: 0;
    transition: opacity 0.6s ease-in-out;
    min-width: 207px;
    margin-bottom: 16px;
    @media (orientation: portrait) {
      margin-bottom: 20%;
    }
    @media (max-aspect-ratio: 16/10) and (orientation: landscape) {
      margin-bottom: 16%;
    }
    font-family: sans-serif;
    font-weight: 800;
    color: #fff;
    border: none;
    background: #00a3e0;
    padding: 10px;
    font-size: 18px;
    letter-spacing: inherit;
    line-height: 22px;
  }
  &__text {
    visibility: hidden;
    opacity: 0;
    transition: opacity 0.6s ease-in-out;
    margin: 0px 20px 22px;
  }
}

.button--visible {
  opacity: 1;
  visibility: visible;
}
</style>

<script>
import { Component, Vue } from "vue-property-decorator";

import { gsap } from "gsap";
import { trueAlpha, randomSign } from "../assets/js/utils";

const images = {
  hint: require("../assets/images/hint.jpg"),
  mobile: require("../assets/images/mobile.png"),
};

@Component
export default class HintScreen extends Vue {
  buttonVisible = false;

  canvasVisible = false;

  transparent = false;

  mounted() {
    gsap.delayedCall(2.0, () => {
      this.buttonVisible = true;
    });
    gsap.delayedCall(1.2, () => {
      this.canvasVisible = true;
    });
    this.draw_event = this.draw.bind(this);

    this.images = {};
    this.imagesToLoad = {
      hint: images.hint,
      mobile: images.mobile,
    };

    this.loadImages(this.imagesToLoad, (images) => {
      this.images = images;
      this.setupObjects();
    });
  }

  beforeDestroy() {
    gsap.ticker.remove(this.draw_event);
  }

  // eslint-disable-next-line class-methods-use-this
  loadImages(imagesToBeLoaded, drawCallback) {
    let loadedImages = 0;
    const images = {};
    Object.keys(imagesToBeLoaded).forEach((name) => {
      images[name] = new window.Image();
      images[name].src = imagesToBeLoaded[name];
      images[name].onload = () => {
        if (++loadedImages >= Object.keys(imagesToBeLoaded).length) {
          drawCallback(images);
        }
      };
    });
  }

  setupObjects() {
    const ctx = this.$refs.canvas.getContext("2d");
    // ctx.fillRect(0, 0, 600, 600);
    // this.card = new createjs.Container();
    // this.card.initShapes = function initShapes() {
    //   this.shape = new createjs.Shape();
    //   this.shape.graphics
    //     .f("#1E1E1E")
    //     .s()
    //     .p(
    //       "Ah+CzICbAXIhUAygAAEgBICOgIIA0hmICKCLIjCBugAlPgbIBxhBQAvgRAhAPIgcgcIBug/QAvgRAfAQIgagbIBBgmICMCOIhAAlIAAAAIilBZIhChCQAPAggjAaIguAaIAjAmIhEAngAA2iMQAAAMgDAJIgGAKQgCABBTAhIhJhKQACAEgBAFgAhzBBIBXgxIBGBJg"
    //     );
    //   this.shape.setTransform(99.5, 78.8);

    //   this.shape_1 = new createjs.Shape();
    //   this.shape_1.graphics
    //     .f("#FD595F")
    //     .s()
    //     .p(
    //       "ABOAzIijgXQgSgCgMgOQgLgMACgQQACgRAOgKQAOgKASADICiAXQASACALAOQALALgBARQgCARgOAKQgLAIgMAAIgIgBg"
    //     );
    //   this.shape_1.setTransform(95, 92.3);

    //   // Layer 1
    //   this.shape_2 = new createjs.Shape();
    //   this.shape_2.graphics
    //     .f("#1E1E1E")
    //     .s()
    //     .p("ApvhLIK8mdIIjI1Iq7GdgAm6gzIGCGOIHzkoImCmNg");
    //   this.shape_2.setTransform(99.3, 79.5);

    //   this.shape_3 = new createjs.Shape();
    //   this.shape_3.graphics
    //     .f("rgba(0,0,0,0.298)")
    //     .s()
    //     .p("AxEpJIAVgNIRdSFIP9pXIAaAbIwSJkg");
    //   this.shape_3.setTransform(109.9, 124);

    //   this.shape_4 = new createjs.Shape();
    //   this.shape_4.graphics
    //     .f("#FEFAE5")
    //     .s()
    //     .p(
    //       "AwqkBIgagcIQSplIR3SfIgVANIv9JZgArUi0IIkI1IK7mbIoho3gAoficIH1knIGAGQInyElg"
    //     );
    //   this.shape_4.setTransform(109.4, 90);
    //   this.addChild(
    //     this.shape_2,
    //     this.shape_3,
    //     this.shape_4,
    //     this.shape,
    //     this.shape_1
    //   );
    // };
    // this.card.initShapes();
    // this.card.cache(0, 0, 219.25, 183.9, 1);

    // this.screenShape = new createjs.Shape();
    // this.screenShape.graphics
    //   .f("#098644")
    //   .s()
    //   .p(
    //     "AESNfQg8gNgngpIydyyQgogoADgsQABgQAGgMQALgUAagNIB2g8IAfAfQAKAKAPADQAPAEALgGIFMinQAJgFACgIQADgEABgGQABgLgLgKIgbgcIBxg5QAsgXA7AOQA7ANAoAoISdSyQAoApgDAsQgBAPgGANQgLAUgaANIpvE6QgcANghAAQgUAAgWgEg"
    //   );
    this.screenShape = {
      x: 0,
      y: 0,
      dx: 0,
      dy: 0,
    };
    this.screenShape.dx = 218.15 / 2 + 0.6;
    this.screenShape.dy = 187.75 / 2 - 3.6;

    // this.phone = new createjs.Container();
    // this.phone.initShapes = function initShapes(screenShape) {
    //   this.shape = new createjs.Shape();
    //   this.shape.graphics
    //     .f("#DADADA")
    //     .s()
    //     .p(
    //       "AEILOQhLgQgxgzIyeyyQgagbgMgeQgLgeAEgcIALg6QgBA0AvAwISeSyQAyAzBKARQBKAQA4gcIJvk6QAlgSAOgfQAGgLADgNIgQBUQgGATgOAQQgOAQgWALIpvE6QgiASgqAAQgaAAgcgHg"
    //     );
    //   this.shape.setTransform(-0.1, 21.4);

    //   this.shape_1 = new createjs.Shape();
    //   this.shape_1.graphics
    //     .f("#FFFFFF")
    //     .s()
    //     .p(
    //       "AENOAQhKgQgygzIyeyyQgvgwABg0IAAgHQACgWAJgSQAOgfAlgSIJvk6QA3gcBLAQQBKARAyAyISeSzQAyAzgEA3IgBALIgBAFQgDANgGALQgOAfglASIpvE7QgjARgrAAQgZAAgbgGgAvnoaQgaANgLAUQgGAMgBAQQgDAsAoAoISdSyQAnApA8ANQA7ANAsgWIJvk6QAagNALgUQAGgNABgPQADgsgogpIydyyQgogog7gNQg7gOgsAXIhxA5IAbAcQALAKgBALQgBAGgDAEQgCAIgJAFIlMCnQgLAGgPgEQgPgDgKgKIgfgfg"
    //     );
    //   this.shape_1.setTransform(0.6, -3.6);
    //   this.addChild(this.shape, this.shape_1, screenShape);
    // };
    // this.phone.initShapes(this.screenShape);
    // this.phone.cache(-218.15 / 2, -187.75 / 2, 218.15, 187.75, 1);
    // this.screenShape.graphics._fill = null;
    // this.screenShape.graphics._dirty = true;
    this.phone = {};
    // this.petals = [];
    // let frame = 0;
    // for (let i = 0; i < 7; i += 1) {
    //   const petal = {};
    //   petal.fx = (frame % 2) * 64;
    //   petal.fy = Math.trunc(frame / 2) * 64;
    //   petal.x = 0;
    //   petal.y = 0;
    //   petal.sy = Math.random() * 140 - 70;
    //   petal.sRadius = 20 + Math.random() * 10;
    //   petal.radius = 20 + Math.random() * 10;
    //   petal.sRadius *= 0.7;
    //   petal.radius *= 0.6;
    //   petal.scale = 1;
    //   petal.sScale = 0.4 + Math.random() * 0.1;
    //   petal.rotation = Math.random() * Math.PI * 2;
    //   petal.dRotation = 0.01 + Math.random() * 0.02;
    //   petal.dRotation *= randomSign();
    //   petal.ang = Math.random() * Math.PI * 2;
    //   petal.dAng = 0.032 + Math.random() * 0.01;
    //   this.petals.push(petal);
    //   frame += 1;
    //   frame %= 4;
    // }
    gsap.ticker.add(this.draw_event);
    this.startAnim();
  }

  startAnim() {
    this.tl = new gsap.timeline({ repeat: -1, repeatDelay: 0.3 });
    this.phone.x = 160;
    this.phone.y = 74;
    this.phone.alpha = 0;

    this.tl
      .to(
        this.phone,
        0.9,
        {
          x: 120 - 30 - 30,
          y: 74,
          alpha: 1,
          ease: "power2.inOut",
        },
        "s"
      )
      .to(this.phone, 0.8, { x: 120, y: 100, ease: "sine.inOut" }, "s+=1.0")
      .to(
        this.phone,
        0.8,
        { x: 120 - 30 - 30, y: 74, ease: "sine.inOut" },
        "s+=2.0"
      )
      .to(
        this.phone,
        1.1,
        {
          x: this.phone.x,
          y: this.phone.y,
          alpha: 0,
          ease: "power2.inOut",
        },
        "s+=3.2"
      );
  }

  draw() {
    const ctx = this.$refs.canvas.getContext("2d");
    const w = this.$refs.canvas.width;
    const h = this.$refs.canvas.height;
    ctx.fillStyle = "#fff";
    ctx.fillRect(0, 0, w, h);
    ctx.drawImage(this.images.hint, 0, 20);
    this.drawPhone(ctx);
  }

  drawScreen(ctx) {
    ctx.save();
    ctx.translate(-140, -120);
    ctx.scale(1.4, 1.4);
    ctx.drawImage(this.images.hint, 0, 0);
    ctx.restore();
  }
  drawScreenShape(ctx) {
    ctx.save();
    //ctx.translate(104.5221, 87.361696);
    ctx.beginPath();
    ctx.moveTo(27.3, 86.8);
    ctx.quadraticCurveTo(21.4, 85.6, 17.4, 81.5);
    ctx.lineTo(-100.8, -39.3);
    ctx.quadraticCurveTo(-104.8, -43.4, -104.5, -47.8);
    ctx.quadraticCurveTo(-104.2, -52.2, -99.8, -54.4);
    ctx.lineTo(-87.9, -60.5);
    ctx.lineTo(-85, -57.4);
    ctx.quadraticCurveTo(-83.9, -56.4, -82.4, -56);
    ctx.quadraticCurveTo(-81, -55.7, -79.8, -56.2);
    ctx.lineTo(-46.7, -73.1);
    ctx.quadraticCurveTo(-45.5, -73.6, -45.4, -74.7);
    ctx.quadraticCurveTo(-45.4, -75.9, -46.4, -76.9);
    ctx.lineTo(-49.4, -80);
    ctx.lineTo(-37.6, -86);
    ctx.quadraticCurveTo(-33.1, -88.2, -27.2, -86.9);
    ctx.quadraticCurveTo(-21.3, -85.6, -17.4, -81.6);
    ctx.lineTo(100.8, 39.2);
    ctx.quadraticCurveTo(104.8, 43.4, 104.5, 47.8);
    ctx.quadraticCurveTo(104.2, 52.2, 99.9, 54.4);
    ctx.lineTo(37.5, 85.9);
    ctx.quadraticCurveTo(34.8, 87.3, 31.5, 87.3);
    ctx.quadraticCurveTo(29.5, 87.3, 27.3, 86.8);
    ctx.closePath();
    //ctx.fill();
    ctx.translate(-104.5221, -87.361696);
    ctx.restore();
  }

  sortByScale(a, b) {
    if (a.scale < b.scale) {
      return -1;
    }
    if (a.scale > b.scale) {
      return 1;
    }
    return 0;
  }

  drawPhone(ctx) {
    ctx.save();
    ctx.translate(this.phone.x + 110, this.phone.y - 70);
    ctx.scale(0.75, 0.75);
    ctx.globalAlpha = trueAlpha(this.phone.alpha);
    ctx.drawImage(this.images.mobile, 0, 0);
    ctx.save();
    ctx.beginPath();
    ctx.translate(this.screenShape.dx, this.screenShape.dy);
    this.drawScreenShape(ctx);
    ctx.clip();
    ctx.setTransform(1, 0, 0, 1, 0, 0);
    this.drawScreen(ctx);
    ctx.restore();
    ctx.restore();
  }
}
</script>
