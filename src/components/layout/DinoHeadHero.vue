<template>
  <permission-modal
    v-if="showPermissionModal"
    @allow="requestAccess"
    @cancel="rotateDinoHead"
  ></permission-modal>
  <section class="o-dino-header-hero">
    <div class="o-dino-header-hero__text-wrapper">
      <p
        class="o-dino-header-hero__text-wrapper__text o-dino-header-hero__text-wrapper__text--first"
      >
        READY FOR
      </p>
      <p
        class="o-dino-header-hero__text-wrapper__text o-dino-header-hero__text-wrapper__text--second"
      >
        IMPACT
      </p>
    </div>
    <Renderer
      ref="rendererC"
      antialias
      :orbit-ctrl="{ enabled: false }"
      :alpha="true"
      resize="window"
      class="renderer"
    >
      <Camera ref="cameraC" :far="1000000" :position="{ x: 0, y: 0, z: 30 }" />
      <Scene ref="sceneC">
        <DirectionalLight color="#008400" :position="{ x: 0, y: 0, z: 10 }" />
        <AmbientLight></AmbientLight>
        <GltfModel
          :rotation="{ y: 5.4 }"
          :position="{ x: 0.6, y: -5, z: 0 }"
          src="models/glftfDinoHeadNoComp.glb"
          @load="onReady"
        />
      </Scene>
    </Renderer>
    <scroll-down class="o-dino-header-hero__scroll-down"></scroll-down>
  </section>
</template>

<script setup>
import { ref, onMounted } from "vue";
import PermissionModal from "@/components/modal/PermissionModal.vue";
import ScrollDown from "@/components/btn/ScrollDown.vue";
import {
  Camera,
  GltfModel,
  AmbientLight,
  Renderer,
  Scene,
  DirectionalLight,
} from "troisjs";
import * as THREE from "three";
import { isMobile } from "mobile-device-detect";

//defining the refs
const rendererC = ref();
const sceneC = ref();
const cameraC = ref();
const showPermissionModal = ref(false);

/*
 * Loaders
 */

// texture loader
const cubeTextureLoader = new THREE.CubeTextureLoader();

/*
 * Textures
 */

// environment map
const environmentMapTexture = cubeTextureLoader.load([
  "/environmentMaps/0/px.webp",
  "/environmentMaps/0/nx.webp",
  "/environmentMaps/0/py.webp",
  "/environmentMaps/0/ny.webp",
  "/environmentMaps/0/pz.webp",
  "/environmentMaps/0/nz.webp",
]);

/*
// Material
 */
const dinoMetalMaterial = new THREE.MeshStandardMaterial({
  color: 0xffffff,
  roughness: 0,
  metalness: 1,
  envMap: environmentMapTexture,
});

/*
// Loading Mesh
 */
const onReady = (model) => {
  model.scene.traverse((child) => {
    if (child.isMesh) {
      /*
       * Applying the texture to the material
       */
      child.material = dinoMetalMaterial;
    }
  });
};
onMounted(() => {
  const renderer = rendererC.value;
  const camera = cameraC.value;
  //checking if the device is mobile
  if (!isMobile) {
    renderer.onBeforeRender(() => {
      document.addEventListener("mousemove", (event) => {
        //getting the mouse position where the axis is the center of the screen
        const mouse = {
          x: (event.clientX / window.innerWidth) * 2 - 1,
          y: -(event.clientY / window.innerHeight) * 2 + 1,
        };

        camera.camera.position.x = -mouse.x * 20;
        camera.camera.position.y = -mouse.y * 20;
      });
    });
  } else {
    if (
      typeof DeviceOrientationEvent !== "undefined" &&
      typeof DeviceOrientationEvent.requestPermission === "function"
    ) {
      //show modal
      showPermissionModal.value = true;
    } else {
      // handle regular non iOS 13+ devices
      window.addEventListener("deviceorientation", handleOrientation);
    }
  }

  //capturing the canvas element and enabeling scroll for mobile
  renderer.renderer.domElement.style.touchAction = "auto";
  renderer.renderer.domElement.style.width = "100%";
});

const requestAccess = () => {
  DeviceOrientationEvent.requestPermission()
    .then((response) => {
      if (response === "granted") {
        //hide modal
        showPermissionModal.value = false;
        window.addEventListener("deviceorientation", handleOrientation);
      } else {
        //hide modal
        showPermissionModal.value = false;
        //rotate dino head
        rotateDinoHead();
      }
    })
    .catch(console.error);
};

const handleOrientation = (e) => {
  const renderer = rendererC.value;
  const camera = cameraC.value;
  renderer.onBeforeRender(() => {
    const beta = e.beta;
    const gamma = e.gamma;

    camera.camera.position.x = gamma * 0.22;
    camera.camera.position.y = (beta - 90) * 0.22;
  });
};

const rotateDinoHead = () => {
  const renderer = rendererC.value;
  const camera = cameraC.value;
  //hide modal
  showPermissionModal.value = false;
  renderer.onBeforeRender(() => {
    //rotate camera in circle
    camera.camera.position.x = Math.sin(Date.now() * 0.0005) * 20;
  });
};
</script>
