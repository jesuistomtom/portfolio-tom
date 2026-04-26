<template>
  <div class="temp-background">
    <img
      v-for="shape in SHAPES"
      :key="shape.name"
      :src="shape.src"
      alt=""
      class="shape"
      :class="shape.name"
      :style="getShapeStyle(shape)"
    />
  </div>
</template>

<script lang="ts" setup>
import { onMounted, onUnmounted, ref, computed } from "vue";
import triangleImg from "~/assets/images/triangle.webp";
import circleImg from "~/assets/images/circle.webp";
import squareImg from "~/assets/images/square.webp";

const windowSize = ref({ width: 0, height: 0 });
const targetMouse = ref({ x: 0, y: 0 });
const mouse = ref({ x: 0, y: 0 });
const time = ref(0);
let rafId: number;

const SHAPES = [
  {
    name: "circle",
    src: circleImg,
    parallax: { x: -0.5, y: 0.4 },
    position: { top: "-10vh", left: "45vw" },
    mobilePosition: { top: "10vh", left: "60vw" },
    rotation: -30,
    color: "#33FF57",
    offset: Math.random() * 10000,
    speed: 0.8,
    rotationIntensity: 0.5,
    floatIntensity: 0.8,
  },
  {
    name: "triangle",
    src: triangleImg,
    parallax: { x: 0.3, y: 0.2 },
    position: { top: "35vh", right: "-5vw" },
    mobilePosition: { top: "60vh", right: "-15vw" },
    rotation: 7.93,
    color: "#FF5733",
    offset: Math.random() * 10000,
    speed: 1,
    rotationIntensity: 1,
    floatIntensity: 1,
  },
  {
    name: "square",
    src: squareImg,
    parallax: { x: 0.2, y: -0.3 },
    position: { bottom: "-15vh", left: "15vw" },
    mobilePosition: { bottom: "-10vh", left: "-10vw" },
    rotation: -15,
    color: "#FF33FF",
    offset: Math.random() * 10000,
    speed: 1.2,
    rotationIntensity: 1.5,
    floatIntensity: 1,
  },
];

const shapeScale = computed(() => {
  return 1;
});

const getShapeStyle = (shape: (typeof SHAPES)[0]) => {
  const isMobile = windowSize.value.width < 768;
  const currentPosition = isMobile && shape.mobilePosition ? shape.mobilePosition : shape.position;

  const centerX = windowSize.value.width / 2;
  const centerY = windowSize.value.height / 2;

  const moveX = (mouse.value.x - centerX) * shape.parallax.x * 0.02;
  const moveY = (mouse.value.y - centerY) * shape.parallax.y * 0.02;

  // Float logic inspired by @react-three/drei
  const t = (shape.offset + time.value) * shape.speed;
  const rotX = (Math.cos(t / 4) / 8) * shape.rotationIntensity;
  const rotY = (Math.sin(t / 4) / 8) * shape.rotationIntensity;

  let floatY = Math.sin(t / 4) / 10;
  // Map -0.1 to 0.1 to a pixel range (e.g., -20px to 20px)
  floatY = floatY * 200 * shape.floatIntensity;

  return {
    ...currentPosition,
    transform: `
      translate(${moveX}px, ${moveY + floatY}px)
      rotate(${shape.rotation}deg)
      rotateX(${rotX}rad)
      rotateY(${rotY}rad)
      scale(${shapeScale.value})
    `,
    transformStyle: "preserve-3d",
  };
};

const handleMouseMove = (e: MouseEvent) => {
  targetMouse.value = { x: e.clientX, y: e.clientY };
};

const handleResize = () => {
  windowSize.value = { width: window.innerWidth, height: window.innerHeight };
};

const update = () => {
  // Smoothly interpolate mouse position (Lerp)
  // Adjust 0.05 to change smoothness (lower = smoother/slower)
  const lerpFactor = 0.05;
  mouse.value.x += (targetMouse.value.x - mouse.value.x) * lerpFactor;
  mouse.value.y += (targetMouse.value.y - mouse.value.y) * lerpFactor;

  time.value = performance.now() / 1000;
  rafId = requestAnimationFrame(update);
};

onMounted(() => {
  handleResize();

  // Initialize mouse at center
  const centerX = window.innerWidth / 2;
  const centerY = window.innerHeight / 2;
  mouse.value = { x: centerX, y: centerY };
  targetMouse.value = { x: centerX, y: centerY };

  window.addEventListener("mousemove", handleMouseMove);
  window.addEventListener("resize", handleResize);
  rafId = requestAnimationFrame(update);
});

onUnmounted(() => {
  window.removeEventListener("mousemove", handleMouseMove);
  window.removeEventListener("resize", handleResize);
  cancelAnimationFrame(rafId);
});
</script>

<style lang="scss" scoped>
.temp-background {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100dvh;
  z-index: -1;
  overflow: hidden;
  perspective: 1000px;
  transform-style: preserve-3d;

  .shape {
    position: absolute;
    pointer-events: none;
    width: clamp(200px, 20vw, 550px);
    height: auto;
    transform-origin: center center;
    will-change: transform;

    @media screen and (max-width: 768px) {
      width: clamp(180px, 50vw, 400px);
    }
  }
}
</style>
