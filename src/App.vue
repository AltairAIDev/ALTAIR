<script setup>
import { ref, onMounted, onUnmounted } from "vue";
import gsap from "gsap";
import stormAudio from "./assets/storm.mp3";
import lostAudioFile from "./assets/lost.mp3";

const canvasRef = ref(null);
const audio = new Audio(stormAudio);
audio.loop = true;
const lostAudio = new Audio(lostAudioFile);
lostAudio.loop = true;
let animationId;
let shakeTween;
let snowflakes = [];
const isStarted = ref(false);

// Easter egg state
let easterEggTriggered = false;
let isAuroraMode = false;
let heat = 0;
let lastMouse = { x: null, y: null };
let startTime = 0;
let frostLevel = 0;

const initSnow = () => {
  const canvas = canvasRef.value;
  if (!canvas) return;
  const ctx = canvas.getContext("2d");
  
  let width = window.innerWidth;
  let height = window.innerHeight;

  const resize = () => {
    width = window.innerWidth;
    height = window.innerHeight;
    canvas.width = width;
    canvas.height = height;
  };
  resize();
  window.addEventListener("resize", resize);

  let numFlakes = width < 768 ? 100 : 250;
  snowflakes = [];
  for (let i = 0; i < numFlakes; i++) {
    snowflakes.push({
      x: Math.random() * width,
      y: Math.random() * height,
      radius: Math.random() * 2.5 + 0.5,
      speedY: Math.random() * 6 + 4, // real winter falling speed
      speedX: Math.random() * 7 + 5, // steady wind blowing right
      opacity: Math.random() * 0.8 + 0.2
    });
  }

  const render = () => {
    ctx.clearRect(0, 0, width, height);

    if (isAuroraMode) {
      ctx.fillStyle = "rgba(150, 255, 200, 0.9)"; // magical glowing dust
    } else {
      ctx.fillStyle = "rgba(130, 150, 170, 0.85)";
    }
    
    snowflakes.forEach(flake => {
      ctx.beginPath();
      ctx.arc(flake.x, flake.y, flake.radius, 0, Math.PI * 2);
      ctx.globalAlpha = flake.opacity;
      ctx.fill();

      if (isAuroraMode) {
        // Float gently upwards
        flake.y -= flake.speedY * 0.15;
        flake.x += Math.sin(flake.y * 0.02) * 1.5;
        
        if (flake.y < -10) {
          flake.y = height + 10;
          flake.x = Math.random() * width;
        }
      } else {
        flake.x += flake.speedX;
        flake.y += flake.speedY;

        if (flake.y > height) {
          flake.y = -10;
          flake.x = Math.random() * width * 1.5 - width * 0.5;
        }
        if (flake.x > width) {
          flake.x = -10;
          flake.y = Math.random() * height * 1.5 - height * 0.5;
        }
      }
    });
    ctx.globalAlpha = 1.0;

    // Heat and Frost logic for Easter Egg
    if (!easterEggTriggered && isStarted.value) {
      heat *= 0.95;
      
      // The hint: Start freezing the screen after 20 seconds
      if (Date.now() - startTime > 20000) {
        let oldFrost = frostLevel;
        if (heat < 50) {
          frostLevel += 0.003; // Slowly freeze (approx 5-6 seconds to full frost)
        } else {
          frostLevel -= 0.05; // Rapidly melt when rubbed
        }
        frostLevel = Math.max(0, Math.min(1, frostLevel));
        
        // Only trigger heavy DOM updates if the value actually changed
        if (frostLevel !== oldFrost) {
          gsap.set(".frost-overlay", { opacity: frostLevel });
        }
      }

      if (heat > 1500) { // Threshold reached by vigorously scrubbing the mouse
        triggerEasterEgg();
      }
    }

    animationId = requestAnimationFrame(render);
  };

  render();
};

const cameraShake = () => {
  if (window.innerWidth < 768) return; // Disable shake on mobile for better performance
  shakeTween = gsap.to(".root", {
    x: Math.random() * 10 - 5,
    y: Math.random() * 10 - 5,
    rotation: Math.random() * 0.5 - 0.25,
    duration: Math.random() * 1.5 + 0.5,
    ease: "sine.inOut",
    onComplete: cameraShake
  });
};

const triggerEasterEgg = () => {
  easterEggTriggered = true;
  
  // Bright flash transitions into the magical night
  gsap.to(".eye-flash", { opacity: 1, duration: 1, ease: "power2.in", onComplete: () => {
    isAuroraMode = true;
    snowflakes.forEach(f => f.radius *= 1.5); // make particles larger
    
    // Reveal Aurora background
    gsap.set(".aurora-bg", { opacity: 1 });
    gsap.set([".overlay-fog", ".frost-overlay"], { display: "none" }); // hide storm fog and frost
    
    // Change Title aesthetics
    gsap.set(".title", { color: "#e0ffff", textShadow: "0 0 30px #00ff88, 0 0 10px #ffffff" });
    
    // Fade flash back out to reveal
    gsap.to(".eye-flash", { opacity: 0, duration: 4, ease: "power2.out" });
  }});
  
  // Gently fade out the aggressive storm audio
  gsap.to(audio, { volume: 0, duration: 2, onComplete: () => audio.pause() });
};

const startExperience = () => {
  if (isStarted.value) return;
  isStarted.value = true;
  startTime = Date.now();
  
  if (audio.paused) {
    audio.play().catch(e => console.error("Audio play failed:", e));
  }
  
  if (lostAudio.paused) {
    lostAudio.volume = 0;
    lostAudio.play().catch(e => console.error("Lost audio play failed:", e));
    gsap.to(lostAudio, { volume: 0.2, duration: 4, ease: "power2.inOut" });
  }
  
  gsap.set(".scene-blur-wrapper", { filter: "blur(15px)" });
  
  const tl = gsap.timeline();
  
  tl.delay(0.5)
    // First blink is white
    .to(".eye-flash", { opacity: 1, duration: 0.1 }, "blink1")
    .to(".eyelid-hole", { height: "35vh", duration: 0.3, ease: "power2.out" }, "blink1")
    // Close blink
    .to(".eyelid-hole", { height: "0vh", duration: 0.3, ease: "power2.in", delay: 0.2 }, "close1")
    // Full open
    .to(".eyelid-hole", { height: "150vh", duration: 0.6, ease: "power3.inOut", delay: 0.3 }, "open2")
    // Flash white right as eyes open wide
    .to(".eye-flash", { opacity: 1, duration: 0.1 }, "open2+=0.3")
    // Focus eyes (remove blur completely for performance)
    .set(".scene-blur-wrapper", { clearProps: "filter" }, "open2+=0.4")
    // Fade out white flash
    .to(".eye-flash", { opacity: 0, duration: 2.5, ease: "power2.out" }, "open2+=0.4")
    // Fog animation sync
    .from(".overlay-fog", { opacity: 0, duration: 4, ease: "power1.inOut" }, "open2+=0.4");
};

onMounted(() => {
  initSnow();
  cameraShake();
  
  window.addEventListener('click', startExperience, { once: true });
  window.addEventListener('touchstart', startExperience, { once: true });
  
  const handleMove = (e) => {
    if (easterEggTriggered || !isStarted.value) return;
    let clientX = e.touches ? e.touches[0].clientX : e.clientX;
    let clientY = e.touches ? e.touches[0].clientY : e.clientY;
    
    if (lastMouse.x !== null) {
      let dx = clientX - lastMouse.x;
      let dy = clientY - lastMouse.y;
      heat += Math.sqrt(dx*dx + dy*dy);
    }
    lastMouse.x = clientX;
    lastMouse.y = clientY;
  };

  window.addEventListener('mousemove', handleMove);
  window.addEventListener('touchmove', handleMove, { passive: true });
});

onUnmounted(() => {
  cancelAnimationFrame(animationId);
  window.removeEventListener("resize", () => {});
  audio.pause();
  lostAudio.pause();
  if (shakeTween) shakeTween.kill();
});
</script>

<template>
  <div class="root">
    <div class="scene-blur-wrapper">
      <!-- Snowy canvas -->
      <canvas ref="canvasRef" class="snow-canvas"></canvas>
      
      <!-- Moving Fog/Blizzard effect -->
      <div class="overlay-fog"></div>
      <div class="vignette"></div>
      
      <!-- Frost hint overlay -->
      <div class="frost-overlay"></div>
      
      <!-- Aurora Easter Egg Background -->
      <div class="aurora-bg">
        <div class="aurora-ribbons"></div>
      </div>

      <!-- Content -->
      <div class="content-wrapper">
        <h1 class="title">ALTAIR</h1>
      </div>
    </div>
    
    <!-- Cinematic Opening -->
    <div class="eyelid-hole"></div>
    <div class="eye-flash"></div>
    
    <!-- Start Prompt -->
    <div v-if="!isStarted" class="start-prompt">
      <p>click anywhere to open your eyes.</p>
    </div>
  </div>
</template>

<style scoped>
.root {
  position: fixed;
  inset: -20px; /* Stretch past viewport to hide edges during shake */
  background-image: url('./assets/snow.webp');
  background-size: cover;
  background-position: center;
  overflow: hidden;
  user-select: none;
}

.scene-blur-wrapper {
  position: absolute;
  inset: 0;
}

.eyelid-hole {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 150vw;
  height: 0vh; /* fully closed */
  border-radius: 50%;
  border: 150vh solid #000000;
  box-shadow: inset 0 0 100px #000000; /* Feathered, blurry edge for realistic vision */
  box-sizing: content-box;
  background: transparent;
  z-index: 1000;
  pointer-events: none;
}

.eye-flash {
  position: absolute;
  inset: 0;
  background: #ffffff;
  opacity: 0;
  z-index: 999;
  pointer-events: none;
}

.start-prompt {
  position: fixed;
  inset: 0;
  z-index: 2000;
  display: flex;
  align-items: center;
  justify-content: center;
  pointer-events: none;
}
.start-prompt p {
  font-family: "IBM Plex Mono", monospace;
  font-size: 14px;
  letter-spacing: 0.2em;
  color: rgba(255, 255, 255, 0.5);
  animation: pulse-prompt 2s infinite alternate ease-in-out;
}
@keyframes pulse-prompt {
  0% { opacity: 0.2; }
  100% { opacity: 1; }
}

.snow-canvas {
  position: absolute;
  inset: 0;
  z-index: 10;
  pointer-events: none;
  filter: blur(0.5px);
}

.aurora-bg {
  position: absolute;
  inset: 0;
  background: linear-gradient(180deg, #020611 0%, #0a2120 50%, #104231 100%);
  opacity: 0; /* Hidden until triggered */
  z-index: 5;
  pointer-events: none;
}
.aurora-ribbons {
  position: absolute;
  inset: -50%;
  width: 200%; height: 200%;
  background: repeating-linear-gradient(45deg, transparent, rgba(0, 255, 150, 0.15) 10%, transparent 20%);
  filter: blur(40px);
  animation: aurora-wave 30s linear infinite;
  mix-blend-mode: screen;
}
@keyframes aurora-wave {
  0% { transform: translateX(0) rotate(0deg); }
  100% { transform: translateX(-20%) rotate(5deg); }
}

.overlay-fog {
  position: absolute;
  inset: -50%;
  background: radial-gradient(circle at center, rgba(255,255,255,0.08) 0%, rgba(200,200,210,0.15) 50%, transparent 100%);
  background-size: 50% 50%;
  animation: fog-drift 20s linear infinite;
  pointer-events: none;
  z-index: 20;
  will-change: transform;
}
@keyframes fog-drift {
  0% { transform: translate(0, 0); }
  100% { transform: translate(10%, 10%); }
}

.frost-overlay {
  position: absolute;
  inset: 0;
  background: radial-gradient(ellipse at center, transparent 20%, rgba(255,255,255,0.85) 70%, rgba(255,255,255,1) 100%);
  opacity: 0;
  z-index: 25;
  pointer-events: none;
}

.vignette {
  position: absolute;
  inset: 0;
  z-index: 20;
  pointer-events: none;
  background: radial-gradient(ellipse at center, transparent 20%, rgba(255, 255, 255, 0.85) 100%);
}

.content-wrapper {
  position: absolute;
  inset: 0;
  z-index: 30;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.title {
  font-family: "Cormorant Garamond", serif;
  font-size: 100px;
  font-weight: 300;
  letter-spacing: 0.2em;
  color: #1a202c;
  margin-bottom: 20px;
  text-shadow: 0 0 30px rgba(255, 255, 255, 0.8), 0 0 15px rgba(255, 255, 255, 0.9);
}

@media (max-width: 768px) {
  .title {
    font-size: 50px;
    letter-spacing: 0.1em;
  }
  .start-prompt p {
    font-size: 12px;
    text-align: center;
    padding: 0 20px;
  }
}
</style>
