<script setup>
import { ref, onMounted, onUnmounted, computed, nextTick } from "vue";
import gsap from "gsap";

const isStarted = ref(false);
const showMainframe = ref(false);

// Refs for GSAP
const introTitle = ref(null);
const introSubtitle = ref(null);
const introPrompt = ref(null);
const overrideText = ref(null);
const mainframeContainer = ref(null);

// Minigame State
const mouseX = ref(50);
const mouseY = ref(50);

const memories = ref([
  {
    id: "love",
    label: "I",
    title: "L O V E",
    text: "An intangible tether. The warmth that intertwines souls, inevitably fraying over time yet refusing to sever.",
    x: 15 + Math.random() * 20,
    y: 20 + Math.random() * 15,
    found: false,
  },
  {
    id: "trauma",
    label: "II",
    title: "T R A U M A",
    text: "A buried echo. Scars etched so deep into the subconscious that they whisper loudly in the silence.",
    x: 65 + Math.random() * 20,
    y: 15 + Math.random() * 15,
    found: false,
  },
  {
    id: "affect",
    label: "III",
    title: "A F F E C T",
    text: "The raw sensation. A sudden flood of unfiltered feelings that overwhelms the very senses holding you together.",
    x: 75 + Math.random() * 15,
    y: 70 + Math.random() * 15,
    found: false,
  },
  {
    id: "reason",
    label: "IV",
    title: "R E A S O N",
    text: "The quiet observer. Endlessly attempting to stitch a tapestry of logic from an ocean of chaos.",
    x: 10 + Math.random() * 20,
    y: 65 + Math.random() * 15,
    found: false,
  },
]);

const activeMemory = ref(null);
const nodeRefs = ref([]);
const rebootShown = ref(false);
const rebootBtn = ref(null);

const allFound = computed(() => memories.value.every((m) => m.found));

const updateMouse = (e) => {
  if (!showMainframe.value) return;
  mouseX.value = (e.clientX / window.innerWidth) * 100;
  mouseY.value = (e.clientY / window.innerHeight) * 100;
};

const openMemory = (mem) => {
  mem.found = true;
  activeMemory.value = mem;
};

const closeMemory = () => {
  activeMemory.value = null;
  if (allFound.value && !rebootShown.value) {
    rebootShown.value = true;
    nextTick(() => {
      gsap.fromTo(
        rebootBtn.value,
        { opacity: 0, scale: 0.8 },
        { opacity: 1, scale: 1, duration: 2, ease: "elastic.out(1, 0.3)" },
      );
    });
  }
};

// Intro GSAP Animation
const playIntroAnim = () => {
  const tl = gsap.timeline();

  tl.fromTo(
    ".altair-char",
    { y: 50, opacity: 0, rotationX: 45 },
    {
      y: 0,
      opacity: 1,
      rotationX: 0,
      duration: 1,
      stagger: 0.15,
      ease: "power3.out",
    },
  );

  tl.fromTo(
    introSubtitle.value,
    { opacity: 0, filter: "blur(4px)" },
    {
      opacity: 0.7,
      filter: "blur(0px)",
      duration: 1.5,
      ease: "slow(0.7, 0.7, false)",
    },
    "-=0.5",
  );

  tl.fromTo(
    introPrompt.value,
    { opacity: 0 },
    { opacity: 0.5, duration: 0.5, repeat: -1, yoyo: true },
    "-=0.5",
  );
};

const handleStart = () => {
  if (isStarted.value) return;
  gsap.to([introTitle.value, introSubtitle.value, introPrompt.value], {
    opacity: 0,
    y: -20,
    duration: 0.5,
    stagger: 0.1,
    onComplete: () => {
      isStarted.value = true;
      playOverrideAnim();
    },
  });
};

const playOverrideAnim = async () => {
  await nextTick();
  const tl = gsap.timeline({
    onComplete: () => {
      showMainframe.value = true;
      playMainframeAnim();
    },
  });

  tl.fromTo(
    overrideText.value,
    { scale: 3, opacity: 0, letterSpacing: "2rem" },
    {
      scale: 1,
      opacity: 1,
      letterSpacing: "0.1em",
      duration: 0.8,
      ease: "expo.inOut",
    },
  ).to(overrideText.value, { opacity: 0, duration: 0.3, delay: 1.5 });
};

const playMainframeAnim = async () => {
  await nextTick();

  gsap.fromTo(
    mainframeContainer.value,
    { opacity: 0 },
    { opacity: 1, duration: 2, ease: "power2.inOut" },
  );

  // Make memory nodes float randomly
  if (nodeRefs.value) {
    nodeRefs.value.forEach((node) => {
      if (!node) return;
      gsap.to(node, {
        y: "random(-20, 20)",
        x: "random(-20, 20)",
        rotation: "random(-10, 10)",
        duration: "random(3, 6)",
        repeat: -1,
        yoyo: true,
        ease: "sine.inOut",
      });
    });
  }
};

const handleReboot = () => {
  gsap.to(mainframeContainer.value, {
    opacity: 0,
    duration: 0.8,
    onComplete: () => {
      // Reset state completely
      isStarted.value = false;
      showMainframe.value = false;
      rebootShown.value = false;
      memories.value.forEach((m) => {
        m.found = false;
        m.x = 10 + Math.random() * 70;
        m.y = 10 + Math.random() * 70;
      });
      mouseX.value = 50;
      mouseY.value = 50;
      nextTick(() => playIntroAnim());
    },
  });
};

onMounted(() => {
  playIntroAnim();
  window.addEventListener("keydown", (e) => {
    if (
      (e.key === "z" || e.key === "Z" || e.key === "Enter") &&
      !isStarted.value
    ) {
      handleStart();
    }
  });
  window.addEventListener("mousemove", updateMouse);
});

onUnmounted(() => {
  window.removeEventListener("mousemove", updateMouse);
});
</script>

<template>
  <main
    class="w-screen h-screen flex flex-col items-center justify-center relative overflow-hidden transition-colors duration-1000"
    :class="
      showMainframe
        ? 'bg-[#e2e2e5] text-[#050508]'
        : 'bg-[#050508] text-[#e2e2e5]'
    "
  >
    <!-- Initial Intro Screen -->
    <div
      v-if="!isStarted"
      class="flex flex-col items-center z-10 space-y-6"
      @click="handleStart"
    >
      <h1
        ref="introTitle"
        class="text-7xl md:text-9xl glitch tracking-widest cursor-pointer"
        data-text="A L T A I R"
        style="perspective: 1000px"
      >
        <span class="altair-char inline-block wavy-text pr-2">A</span>
        <span class="altair-char inline-block pr-2">L</span>
        <span class="altair-char inline-block wavy-text pr-2">T</span>
        <span class="altair-char inline-block pr-2">A</span>
        <span class="altair-char inline-block wavy-text pr-2">I</span>
        <span class="altair-char inline-block">R</span>
      </h1>

      <p
        ref="introSubtitle"
        class="text-xl md:text-2xl opacity-0 tracking-widest uppercase text-center max-w-2xl mt-4 px-4"
      >
        <span class="text-altair-accent">A</span>lchemy of
        <span class="text-[#33aaff]">L</span>ove,
        <span class="text-altair-accent">T</span>rauma,
        <span class="text-[#33aaff]">A</span>ffect, and
        <span class="text-altair-accent">I</span>ntellectual
        <span class="text-[#33aaff]">R</span>easoning
      </p>

      <div
        ref="introPrompt"
        class="mt-16 opacity-0 text-lg uppercase cursor-pointer text-[#33aaff]"
      >
        > Press Z or Click _
      </div>
    </div>

    <!-- Transition / Scene 2 -->
    <div
      v-if="isStarted && !showMainframe"
      class="absolute inset-0 flex items-center justify-center bg-[#050508] z-20 pointer-events-none"
    >
      <p
        ref="overrideText"
        class="text-3xl text-altair-accent tracking-widest font-bold"
        style="text-shadow: 2px 2px 0px #33aaff"
      >
        AWAKENING...
      </p>
    </div>

    <!-- Mainframe Scene -> Now an interactive exploratory canvas -->
    <div
      v-if="showMainframe"
      ref="mainframeContainer"
      class="absolute inset-0 z-30 opacity-0 overflow-hidden bg-[#e2e2e5] text-[#050508]"
    >
      <!-- Flashlight Overlay Effect -->
      <div
        class="absolute inset-0 pointer-events-none z-10 transition-all duration-300"
        :style="{
          background: `radial-gradient(circle at ${mouseX}% ${mouseY}%, transparent 5%, rgba(5, 5, 8, 0.98) 20%)`,
        }"
      ></div>

      <!-- Hidden Explorable Nodes -->
      <div
        v-for="(mem, index) in memories"
        :key="mem.id"
        ref="nodeRefs"
        class="absolute z-20 transition-colors"
        :class="
          mem.found
            ? 'text-altair-accent cursor-default'
            : 'text-[#888] hover:text-[#050508] cursor-crosshair'
        "
        :style="{ left: mem.x + '%', top: mem.y + '%' }"
        @click="!mem.found && openMemory(mem)"
      >
        <span
          class="text-4xl lg:text-6xl tracking-widest font-bold drop-shadow-[2px_2px_0px_#fff]"
        >
          {{ mem.found ? mem.label : "[ ? ]" }}
        </span>
      </div>

      <!-- Detail Modal -->
      <div
        v-if="activeMemory"
        class="absolute inset-0 z-40 flex items-center justify-center bg-[#050508]/80 backdrop-blur-sm p-4 animate-in fade-in zoom-in duration-300"
      >
        <div
          class="bg-[#e2e2e5] border-2 border-[#050508] p-8 max-w-lg w-full shadow-[8px_8px_0px_#ff3366] text-[#050508]"
        >
          <div class="text-[#33aaff] text-sm tracking-widest mb-1">
            {{ activeMemory.label }} / IV
          </div>
          <h2
            class="text-4xl font-bold tracking-widest border-b-2 border-[#050508] pb-2 mb-6"
          >
            {{ activeMemory.title }}
          </h2>
          <p class="text-xl leading-relaxed font-light">
            {{ activeMemory.text }}
          </p>
          <button
            @click="closeMemory"
            class="mt-8 border border-[#050508] px-6 py-2 hover:bg-[#050508] hover:text-[#e2e2e5] transition-colors w-full uppercase cursor-pointer"
          >
            A C K N O W L E D G E
          </button>
        </div>
      </div>

      <!-- Re-Awaken Button (Appears only when all nodes are found) -->
      <div
        v-show="rebootShown"
        class="absolute inset-0 z-50 flex flex-col items-center justify-center pointer-events-none"
      >
        <h2
          class="text-5xl md:text-7xl font-bold tracking-widest text-[#e2e2e5] mb-8"
          style="text-shadow: 2px 2px 0px #ff3366"
        >
          T U R N I N G P O I N T
        </h2>
        <p class="text-xl text-[#e2e2e5] mb-12 max-w-md text-center">
          You have collected all fragments. The path is now clear.
        </p>
        <button
          ref="rebootBtn"
          @click="handleReboot"
          class="pointer-events-auto border-2 border-[#e2e2e5] bg-[#050508] text-[#e2e2e5] px-12 py-4 text-2xl font-bold hover:bg-[#e2e2e5] hover:text-[#050508] transition-colors uppercase shadow-[0_0_20px_#33aaff] hover:scale-105 cursor-pointer"
        >
          A W A K E N [ 👁 ]
        </button>
      </div>
    </div>
  </main>
</template>

<style scoped>
.text-altair-accent {
  color: var(--color-altair-accent);
}
</style>
