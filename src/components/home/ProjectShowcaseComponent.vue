<script setup>
import { onMounted, onBeforeUnmount, ref } from "vue";
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
import { LinkComponent, ProjectShowcaseModel } from "@/components";
const projectsWrapper = ref(null);
onMounted(() => {
  gsap.registerPlugin(ScrollTrigger);
  if (ScrollTrigger.getAll().length === 0) {
    gsap.to(".o-project-showcase-component__projects-wrapper", {
      xPercent: -117,
      x: () => innerWidth,
      ease: "power4.inOut",
      scrollTrigger: {
        trigger: ".o-project-showcase-component",
        end: () => projectsWrapper.value.innerWidth * 50,
        scrub: true,
        pin: true,
        // markers: true,
      },
    });
  }
  ScrollTrigger.refresh();
});
onBeforeUnmount(() => {
  ScrollTrigger.getAll().forEach((trigger) => trigger.kill());
});
defineProps({
  projectShowcase: {
    type: Object,
    required: true,
  },
});
</script>
<template>
  <div class="o-project-showcase-component">
    <div class="u-layout-grid">
      <div
        ref="projectsWrapper"
        class="o-project-showcase-component__projects-wrapper"
      >
        <project-showcase-model
          v-for="project in projectShowcase"
          :key="project.slug"
          :title="project.title"
          :slug="project.slug"
          :image="project.logo"
        ></project-showcase-model>
      </div>
      <link-component
        title="All Projects"
        to="/work"
        link-type="internal"
        class="o-project-showcase-component__link"
      >
        ALL PROJECTS &nbsp; <img src="/images/icons/arrow-right.svg" />
      </link-component>
    </div>
    <span class="o-project-showcase-component__title">Work</span>
  </div>
</template>
