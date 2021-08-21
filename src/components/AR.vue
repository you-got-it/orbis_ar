<template>
  <div ref="container" id="ar" class="ar">
    <canvas class="ar__canvas" ref="3d" width="10px" height="10px"></canvas>
    <div class="debug" v-html="getDebugString"></div>
  </div>
</template>

<script>
import { Component, Prop, Vue } from "vue-property-decorator";
import {
  Texture,
  AddOperation,
  EquirectangularReflectionMapping,
  BoxGeometry,
  BoxBufferGeometry,
  PerspectiveCamera,
  Vector3,
  Vector2,
  AmbientLight,
  WebGLRenderer,
  Color,
  DirectionalLight,
  DoubleSide,
  sRGBEncoding,
  Scene,
  MeshStandardMaterial,
  Raycaster,
  PMREMGenerator,
  Box3,
  UnsignedByteType,
  MeshPhongMaterial,
  Mesh,
  AnimationMixer,
  Shape,
  ShapePath,
  LineCurve3,
  MirroredRepeatWrapping,
  BufferGeometry,
  BufferAttribute,
  Group,
  MeshBasicMaterial,
  ExtrudeGeometry as ExtrudeGeometryNew,
  PlaneGeometry,
  LoopOnce,
  Plane,
} from "three";
import {
  trueAlpha,
  random,
  randomSign,
  preload,
  PromiseAll,
  preloadImage,
  isTouchDevice,
  getTexture,
  newCanvas,
  radialGradientTexture,
} from "../assets/js/utils";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader";
import { SkeletonUtils } from "three/examples/jsm/utils/SkeletonUtils";

//import { ArToolkitProfile } from "@ar-js-org/ar.js";
// import {
//   ArToolkitProfile,
//   ArToolkitSource,
//   ArToolkitContext,
//   ArMarkerControls,
//   ArSmoothedControls,
// } from "../assets/js/ar-threex-nft.js";
// } from "@ar-js-org/ar.js/three.js/build/ar-threex-nft.js";

//import gsap from "gsap";
import { gsap } from "gsap";
import { CustomEase } from "../assets/js/CustomEase";

gsap.registerPlugin(CustomEase);

import postscribe from "postscribe";
window.THREE = require("three");
//require("@/assets/js/ar-nft.js");
@Component
export default class AR extends Vue {
  isDesktop = false;
  isStarted = false;
  explore = {};
  currentScene;
  groundPetals = [];
  models = {};
  explorePetals = [];
  mixer;
  mixers = [];
  debugString = 'String';

  get getDebugString(){
    return this.debugString;
  }

  setDebugString(text){
    this.debugString = text;
  }

  mounted() {
    //console.log(ArToolkitProfile);
    window.app = this;
    if (!window.THREEx) {
      // eslint-disable-next-line no-useless-escape
      postscribe("#ar", '<script src="./ar-nft.js"><\/script>', {
        done: () => {
          window.app = this;
          this.startAR();
        },
      });
    } else {
      this.startAR();
    }
    // this.startAR();
  }

  onWindowResize() {
    this.width = window.innerWidth;
    this.height = window.innerHeight;
    if (!this.isDesktop) {
      this.threeCamera.updateProjectionMatrix();
      this.renderer.setSize((640 * this.height) / 480, this.height);
      this.renderer.setPixelRatio(1 + (window.devicePixelRatio - 1) * 0.34);
    } else {
      this.renderer.setPixelRatio(1);
      this.threeCamera.aspect = this.width / this.height;
      this.threeCamera.updateProjectionMatrix();
      this.renderer.setSize(this.width, this.height);
    }
    console.log("resize1");
    console.log(this.renderer.domElement.style.height);
  }
  resizeAR() {
    this.arToolkitSource.onResizeElement();
    console.log(this.renderer.domElement.style.height);
    this.arToolkitSource.copyElementSizeTo(this.renderer.domElement);
    // const mat = this.arToolkitContext.getProjectionMatrix();
    // mat.elements[14] *= 200;
    // this.threeCamera.projectionMatrix.copy(mat);
    if (this.arToolkitContext.arController !== null) {
      // this.arToolkitSource.copyElementSizeTo(
      //   this.arToolkitContext.arController.canvas
      // );
    }
    console.log("resize2");
  }

  async startAR() {
    document.body.style.backgroundColor = "transparent";
    window.THREEx.parent = this.$el;
    this.initScene();
    this.onWindowResize();
    window.addEventListener("resize", this.onWindowResize.bind(this), false);
    await this.loadResources();
    this.setupObjects();
  }

  async loadResources() {
    const imagesToLoad = {
      layer_0: "images/1/0.jpg",
      layer_1: "images/1/1.png",
      layer_2: "images/1/2.png",
    };
    const modelsToLoad = {
      model: "models/anim_7.glb",
      //model: "models/GroundVehicle.glb",
    };
    this.images = {};
    this.models = {};
    this.animations = {};
    const promises = [];
    Object.keys(imagesToLoad).map((key) =>
      promises.push(
        preloadImage(imagesToLoad[key], key).then((data) => {
          this.images[data.key] = data.image;
        })
      )
    );
    promises.push(
      new Promise((resolve, reject) => {
        new RGBELoader().setDataType(UnsignedByteType).load(
          "./images/env.hdr",
          (texture) => {
            this.pmremGenerator._renderer.toneMappingExposure = 100;
            const envMap =
              this.pmremGenerator.fromEquirectangular(texture).texture;
            this.hdrEnv = envMap;
            resolve({ envMap });
          },
          undefined,
          null
        );
      })
    );
    Object.keys(modelsToLoad).map((key) => {
      promises.push(this.loadModel(key, modelsToLoad[key]));
    });
    //

    await Promise.all(promises).then(() => {
      this.resLoaded = true;
      console.log("promiseAll");
      //
    });
  }

  initScene() {
    this.scene = new Scene();
    this.ambient = new AmbientLight(0xffffff, 0.5);
    this.scene.add(this.ambient);
    this.light = new DirectionalLight(0xffffff, 0.52);
    this.light.position.set(-1, 20, 15);
    this.scene.add(this.light);

    this.threeCamera = new PerspectiveCamera(60, 1, 1, 10000);
    this.threeCamera.position.set(0, 15, 40);
    this.scene.add(this.threeCamera);

    // render
    this.renderer = new WebGLRenderer({
      antialias: false,
      alpha: true,
      powerPreference: "high-performance",
      canvas: this.$refs["3d"],
    });
    this.renderer.setClearColor(0xffeea0, 0);
    this.renderer.outputEncoding = sRGBEncoding;
    this.renderer.autoClear = false;
    //this.renderer.clippingPlanes = [new Plane(new Vector3(0, 1, 0), 0)];

    if (this.isDesktop) {
      this.controls = new OrbitControls(this.threeCamera, this.$refs["3d"]);
      this.controls.target.set(0, -0.2, -0.2);
      this.controls.maxPolarAngle = 1.45;
      this.controls.update();
      this.sceneGroup = new Group();
      this.scene.add(this.sceneGroup);
    } else {
      this.markerRoot = new Group();
      this.sceneGroup = new Group();
      this.scene.add(this.markerRoot);
      this.threeCamera.position.set(0, 0, 0);
    }

    this.pmremGenerator = new PMREMGenerator(this.renderer);
    this.pmremGenerator.compileEquirectangularShader();
  }

  setupObjects() {
    this.scene.environment = this.hdrEnv;
    // this.models.model.scale.set(6, 6, 6);
    // this.sceneGroup.add(this.models.model);
    if (this.isDesktop) {
      this.currentScene = this.scene;
      this.sceneGroup.scale.set(2, 2, 2);
      this.raf = window.requestAnimationFrame(this.animate.bind(this));
      //
    } else {
      this.initAR();
      this.currentScene = this.sceneGroup;
      //this.sceneGroup.scale.set(0.026 * 2, 0.026 * 2, 0.026 * 2);
      this.sceneGroup.scale.set(70, 70, 70);
      this.sceneGroup.position.set(200, 0, -160);
      this.smoothedRoot.add(this.sceneGroup);
    }

    const width = 1024 * 0.01;
    const height = 724 * 0.01;
    this.layerGeometry = new PlaneGeometry(width, height);
    this.layerGeometry.translate(0, height * 0.5, 0);

    this.setupLayer(this.images.layer_0, -3, 0);
    this.setupLayer(this.images.layer_1, -1, 0.8);
    this.setupLayer(this.images.layer_2, 1, 1.6);
  }

  setupLayer(image, zOffset, delay) {
    const width = 1024 * 0.01;
    const height = 724 * 0.01;
    const material = new MeshBasicMaterial({
      color: 0xffffff,
      map: getTexture(image),
      side: DoubleSide,
      transparent: true,
    });
    //this.plane = new Mesh(this.layerGeometry, material);
    //this.plane = this.models.model.clone();
    this.plane = SkeletonUtils.clone(this.models.model);
    this.plane.material = material;
    this.plane.position.y = -0.64;
    this.plane.position.z = zOffset;
    //this.plane.material = material;
    this.plane.children[0].children[1].material = material;
    // this.setClips(this.clips, this.plane);
    const mixer = new AnimationMixer(this.plane);
    gsap.delayedCall(delay, () => {
      const action = mixer.clipAction(this.clips[0]).play();
      action.setLoop(LoopOnce);
      action.clampWhenFinished = true;
    });

    this.mixers.push(mixer);

    // this.plane.rotation.x = -Math.PI / 2;
    this.sceneGroup.add(this.plane);
    this.startAnim(this.plane, zOffset, delay);
  }

  startAnim(plane, zOffset, delay) {
    const params = {
      x: plane.position.x,
      y: plane.position.y,
      z: plane.position.z,
      rx: plane.rotation.x,
      ry: plane.rotation.y,
      rz: plane.rotation.z,
    };
    gsap.to(params, {
      duration: 0.6,
      delay: delay,
      y: 0,
      ease: "power1.out",
      onUpdate: () => {
        plane.position.set(params.x, params.y, params.z);
        plane.rotation.set(params.rx, params.ry, params.rz);
      },
    });
    /*
    const ease = CustomEase.create(
      "custom",
      "M0,0 C0.366,0.138 0.257,1.016 0.512,1.09 0.672,1.136 0.838,1 1,1 "
    );

    gsap.to(params, {
      delay: delay + 0.4,
      duration: 1.0,
      rx: 0,
      ease: ease,
      onUpdate: () => {
        plane.position.set(params.x, params.y, params.z);
        plane.rotation.set(params.rx, params.ry, params.rz);
      },
    });*/
  }

  initAR() {
    window.THREEx.ArToolkitContext.baseURL = "./";
    //const config = { video: { width: 320 /* 320-640-1280 */ } };
    /* const v = document.createElement('video');
  const start = () => navigator.mediaDevices.getUserMedia(config)
    .then(stream => v.srcObject = stream)
    .then(() => new Promise(resolve => v.onloadedmetadata = resolve))
    .then(() => log(`Success: ${v.videoWidth}x${v.videoHeight}`))
    .catch(log);
  start();
  var log = msg => this.$refs.log.innerHTML += `<p>${msg}</p>`; */
    this.arToolkitSource = new window.THREEx.ArToolkitSource({
      // to read from the webcam
      sourceType: "webcam",
     
      // sourceWidth: (window.innerWidth > window.innerHeight ? 640 : 480)*1,
      // sourceHeight: (window.innerWidth > window.innerHeight ? 480 : 640)*1,      
      sourceWidth: (window.innerWidth > window.innerHeight ? 640 : 480)*1,
      sourceHeight: (window.innerWidth > window.innerHeight ? 480 : 640)*1,      
      displayWidth: (window.innerWidth > window.innerHeight ? 640 : 480),
      displayHeight: (window.innerWidth > window.innerHeight ? 480 : 640),
    });

    this.arToolkitSource.init(() => {
      setTimeout(() => {
        this.resizeAR();
      }, 1000);
      window.addEventListener("resize", this.resizeAR.bind(this), false);
    });
    this.arToolkitContext = new window.THREEx.ArToolkitContext(
      {
        cameraParametersUrl: "./data/camera_para.dat",
        detectionMode: "mono",
        //patternRatio: 0.75,
        maxDetectionRate: 6,       
        // canvasWidth: (window.innerWidth > window.innerHeight ? 640 : 480)*0.25,
        //  canvasHeight: (window.innerWidth > window.innerHeight ? 480 : 640)*0.25,
        canvasWidth: 640*0.25,
        canvasHeight: 480*0.25,
        //imageSmoothingEnabled: false,
        // sourceWidth: window.innerWidth > window.innerHeight ? 640 : 480,
        // sourceHeight: window.innerWidth > window.innerHeight ? 480 : 640,
      }
      // ,
      // {
      //   sourceWidth: (window.innerWidth > window.innerHeight ? 640 : 480)*0.1,
      //   sourceHeight: (window.innerWidth > window.innerHeight ? 480 : 640)*0.1, 
      // }
    );

    this.onRenderFcts = [];
    this.onRenderFcts.push(() => {
      if (!this.arToolkitSource.ready) return;
      this.arToolkitContext.update(this.arToolkitSource.domElement);
      // update scene.visible if the marker is seen
    });

    this.markerControls = new window.THREEx.ArMarkerControls(
      this.arToolkitContext,
      this.markerRoot,
      {
        type: "nft",
        descriptorsUrl: "data/1/marker",

        // patternUrl: './data/patt.hiro',
        // as we controls the camera, set changeMatrixMode: 'cameraTransformMatrix'
        // changeMatrixMode: 'cameraTransformMatrix',
        // turn on/off camera smoothing
        // smooth: true,
        // number of matrices to smooth tracking over, more = smoother but slower follow
        // smoothCount: 5,
        // distance tolerance for smoothing, if smoothThreshold # of matrices are under tolerance, tracking will stay still
        // smoothTolerance: 0.01,
        // threshold for smoothing, will keep still unless enough matrices are over tolerance
        // smoothThreshold: 2,
      }
    );
    // this.markerControls2 = new ArMarkerControls(
    //   this.arToolkitContext,
    //   this.markerRoot,
    //   {
    //     type: "nft",
    //     descriptorsUrl: "data/1/marker",

    //     // patternUrl: './data/patt.hiro',
    //     // as we controls the camera, set changeMatrixMode: 'cameraTransformMatrix'
    //     // changeMatrixMode: 'cameraTransformMatrix',
    //     // turn on/off camera smoothing
    //     // smooth: true,
    //     // number of matrices to smooth tracking over, more = smoother but slower follow
    //     // smoothCount: 5,
    //     // distance tolerance for smoothing, if smoothThreshold # of matrices are under tolerance, tracking will stay still
    //     // smoothTolerance: 0.01,
    //     // threshold for smoothing, will keep still unless enough matrices are over tolerance
    //     // smoothThreshold: 2,
    //   }
    // );
    this.markerControls.addEventListener("markerFound", (evt) => {
      // console.log("onMarkerFound!!1");
    });

    // initialize it
    this.arToolkitContext.init(() => {
      // copy projection matrix to camera
      // const mat = this.arToolkitContext.getProjectionMatrix();
      // mat.elements[14] *= 200;
      // this.threeCamera.projectionMatrix.copy(mat);
      const canvas = this.arToolkitContext.arController.canvas;
      this.$el.appendChild(canvas);
      canvas.style.position = "fixed";
      canvas.style.zIndex = 55;
      canvas.style.left = 0;
    });
    // this.markerControls2.addEventListener("markerFound", (evt) => {
    //   console.log("onMarkerFound!!1");
    // });

    // build a smoothedControls
    this.smoothedRoot = new Group();
    this.scene.add(this.smoothedRoot);

    this.smoothedControls = new window.THREEx.ArSmoothedControls(
      this.smoothedRoot,
      {
        // lerp coeficient for the position - between [0,1] - default to 1
        lerpPosition: 0.16,
        // lerp coeficient for the quaternion - between [0,1] - default to 1
        lerpQuaternion: 0.16,
        // lerp coeficient for the scale - between [0,1] - default to 1
        lerpScale: 0.16,
        // delay for lerp fixed steps - in seconds - default to 1/120
        lerpStepDelay: 1 / 50,
        // minimum delay the sub-control must be visible before this controls become visible - default to 0 seconds
        minVisibleDelay: 0.0,
        // minimum delay the sub-control must be unvisible before this controls become unvisible - default to 0 seconds
        minUnvisibleDelay: 10,
      }
    );

    this.onRenderFcts.push(() => {
      this.smoothedControls.update(this.markerRoot);
    });

    // as we do changeMatrixMode: 'cameraTransformMatrix', start with invisible scene
    this.onRenderFcts.push(() => {
      const ratio = this.renderer.getPixelRatio();
      const left = (this.renderer.domElement.width / ratio - this.width) / 2;
      // const { width } = this;
      this.renderer.setScissorTest(true);
      this.renderer.setScissor(left, 0, this.width + 5, this.height);
      // this.threeCamera.position.x = 0;
      // this.threeCamera.position.y = 0;
      // this.threeCamera.position.z = 5;
      // this.markerRoot.position.set(0, 0, 0);
      // this.threeCamera.lookAt(this.markerRoot);
      //this.markerRoot.position.set(0, 0, 0);
      // this.markerRoot.scale.set(1, 1, 1);
      //console.log(this.markerRoot.scale.x);
      //this.smoothedRoot.position.y = -20;
      //this.markerRoot.visible = true;
      this.renderer.render(this.scene, this.threeCamera);
    });

    this.lastTimeMsec = null;
    this.raf = window.requestAnimationFrame(this.animate.bind(this));
    //console.log(this.arToolkitContext);
  }

  loadModel(key, url) {
    return new Promise((resolve, reject) => {
      new GLTFLoader().load(
        // resource URL
        url,
        // called when the resource is loaded
        (model) => {
          const scene = model.scene;
          //scene.children[0].name = key;
          this.models[key] = scene;
          // this.animations[key] = model.animations || [];

          this.clips = model.animations || [];

          // this.clips.forEach((clip, clipIndex) => {
          //   // Autoplay the first clip.
          //   let action;
          //   if (clipIndex === 0) {
          //     action = this.mixer.clipAction(clip);
          //     action.play();
          //   }
          // });
          resolve({ scene });
        },
        // called when loading is in progresses
        (xhr) => {
          //  console.log(`${xhr.loaded / xhr.total * 100}% loaded`);
        },
        // called when loading has errors
        (error) => {
          // console.log('An error happened');
        }
      );
    });
  }

  setClips(clips, model) {
    console.log("wtf");
    if (this.mixer) {
      this.mixer.stopAllAction();
      this.mixer.uncacheRoot(this.mixer.getRoot());
      this.mixer = null;
    }
    console.log("wtf2");
    this.clips = clips;
    if (!clips.length) return;

    this.mixer = new AnimationMixer(model);
    console.log("asdasd");
  }

  animate(nowMsec) {
    this.lastTimeMsec = this.lastTimeMsec || nowMsec - 1000 / 60;
    const deltaMsec = Math.min(200, nowMsec - this.lastTimeMsec);
    this.lastTimeMsec = nowMsec;
    this.mixers.forEach((mixer) => {
      mixer.update(deltaMsec / 1000);
    });

    //this.mixer && this.mixer.update(deltaMsec / 1000);
    if (this.isDesktop) {
      this.renderer.render(this.scene, this.threeCamera);
    } else {
      this.onRenderFcts.forEach((onRenderFct) => {
        onRenderFct(deltaMsec / 1000, nowMsec / 1000);
      });
    }
    this.raf = window.requestAnimationFrame(this.animate.bind(this));
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
.ar {
  overscroll-behavior: none;
  &__canvas {
    position: fixed;
    left: 0;
    top: 0;
  }
}
.debug{
  font-size: 16px;
  font-weight: 400;
  position: fixed;
  bottom: 0;
  left: 0;
  text-align: left;
  background: rgba(0,0,0,0.3);
  color: #fff;
  padding: 10px;
  z-index: 100;
}
</style>
