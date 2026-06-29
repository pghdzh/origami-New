<template>
  <div class="origami-hero" :class="{ reversed: isReversed }">
    <!-- ==================== 动态背景层 ==================== -->
    <div class="bg-layer">
      <!-- 天使：上升金色光点 + 飘落羽毛 -->
      <template v-if="!isReversed">
        <span
          v-for="item in angelParticles"
          :key="'ap-' + item.id"
          class="angel-particle"
          :style="item.style"
        ></span>
        <span
          v-for="item in angelFeathers"
          :key="'af-' + item.id"
          class="angel-feather"
          :style="item.style"
        ></span>
        <div class="angel-glow"></div>
      </template>
      <!-- 恶魔：下沉暗紫粒子 + 旋转裂痕光环 -->
      <template v-else>
        <span
          v-for="item in devilParticles"
          :key="'dp-' + item.id"
          class="devil-particle"
          :style="item.style"
        ></span>
        <div class="devil-vortex"></div>
        <div class="devil-cracks">
          <span class="crack"></span>
          <span class="crack"></span>
          <span class="crack"></span>
        </div>
      </template>
      <!-- 通用扫描线 -->
      <div class="scanlines"></div>
    </div>

    <!-- ==================== 中心内容 ==================== -->
    <div class="hero-content">
      <button class="crown-btn" @click="toggleMode" :title="modeHint">
        <span class="crown">{{ isReversed ? "👑" : "✦" }}</span>
      </button>
      <h1 class="name">
        鸢一折纸
        <span class="name-sub">Tobiichi Origami</span>
      </h1>
      <div class="title-line">
        <span class="line"></span>
        <span class="code">{{
          isReversed ? "识别代号：Devil" : "识别代号：Angel"
        }}</span>
        <span class="line"></span>
      </div>
      <div class="typewriter-box">
        <span class="typing-text">{{ displayedText }}</span>
        <span class="cursor">|</span>
      </div>
      <div class="ast-badge">
        {{ isReversed ? "反转体 · 救世魔王" : "AST 对精灵部队 · 上士" }}
      </div>
    </div>

    <!-- ==================== 底部 & 页脚 ==================== -->
    <div class="bottom-area">
      <div class="bottom-light"></div>
      <footer class="page-footer">
        <div class="footer-deco">
          {{ isReversed ? "救世魔王 · Satan" : "✦ 灭绝天使 · Methratton ✦" }}
        </div>
        <div class="footer-mode-hint">
          {{ isReversed ? "点击王冠可返回天使形态" : "点击星星可切换恶魔形态" }}
        </div>
        <p class="footer-text">© {{ currentYear }} 鸢一折纸电子设定集</p>
      </footer>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount, watch } from "vue";

const currentYear = new Date().getFullYear();

// ========== 形态切换 ==========
const isReversed = ref(false);
const modeHint = computed(() =>
  isReversed.value ? "返回天使形态" : "切换恶魔形态"
);

function toggleMode() {
  isReversed.value = !isReversed.value;
}

// ========== 打字机台词库 ==========
// ========== 天使形态（Angel）台词库 ==========
// 共 10 句，均出自官方小说/语录集
const angelQuotes: string[] = [
  "至今为止我对士道所抱有的感情并不是爱，而是单纯的依存心，真正的爱，从现在才要开始。",
  "这是我最后一次哭泣，同时这也是我最后一次露出笑容。",
  "我将我的泪水寄存给你。把我的笑容托付给你。我的喜悦、我的快乐、全部都交付给你。",
  "并不是跟在你的身后，只是前进的方向是同样的。",
  "那种地方，我最重要的人就在那里。所以……我非去不可。",
  "但是、只有这份愤怒是我自己的。这份丑陋的感情是我的。——我要、杀死它。杀死那个——天使。",
  "把我的父亲……以及母亲杀掉，居然还说要实现愿望？别开玩笑了……我要让你一事无成地死去。",
  "我确实已经不再将精灵作为绝对的歼灭对象看待了。但是，十香。我是不会把士道交给你的。",
  "没有问题。虽然不知道是什么原因，不过只要士道在那里，我就要去帮他。",
  "我——就施展这个力量，打倒精灵吧。",
];

// ========== 反转形态（Devil）台词库 ==========
// 共 8 句，主要出自小说第十一卷及动画第三季第11话
const devilQuotes: string[] = [
  "原来如此……是我自己，杀死了他们……",
  "不要过来……我不想再失去任何人了……",
  "既然一切都是假的，那就全部毁掉吧。",
  "这个世界，已经不需要我了。",
  "反转吧，灵结晶。",
  "我恨你……但我也……爱你。",
  "救世魔王，Satan。这就是我的力量。",
  "为什么……为什么你要救我……我已经……",
];

const currentQuotes = computed(() =>
  isReversed.value ? devilQuotes : angelQuotes
);

const displayedText = ref("");
let currentQuoteIdx = 0;
let charIndex = 0;
let isDeleting = false;
let typeTimer: ReturnType<typeof setTimeout> | null = null;

function type() {
  const quotes = currentQuotes.value;
  const fullText = quotes[currentQuoteIdx] || "";

  if (!isDeleting) {
    displayedText.value = fullText.substring(0, charIndex + 1);
    charIndex++;
    if (charIndex === fullText.length) {
      typeTimer = setTimeout(() => {
        isDeleting = true;
        type();
      }, 2500);
      return;
    }
  } else {
    displayedText.value = fullText.substring(0, charIndex - 1);
    charIndex--;
    if (charIndex === 0) {
      isDeleting = false;
      currentQuoteIdx = (currentQuoteIdx + 1) % quotes.length;
      typeTimer = setTimeout(type, 600);
      return;
    }
  }

  typeTimer = setTimeout(type, isDeleting ? 45 : 90);
}

function resetTypewriter() {
  if (typeTimer) clearTimeout(typeTimer);
  currentQuoteIdx = 0;
  charIndex = 0;
  isDeleting = false;
  displayedText.value = "";
  type();
}

watch(isReversed, () => {
  resetTypewriter();
});

onMounted(() => {
  type();
});

onBeforeUnmount(() => {
  if (typeTimer) clearTimeout(typeTimer);
});

// ========== 静态背景元素（预生成，避免重新渲染） ==========
interface StaticElement {
  id: number;
  style: Record<string, string>;
}

// 天使粒子：上升的金色光点
const angelParticles: StaticElement[] = Array.from({ length: 50 }, (_, i) => {
  const size = 2 + Math.random() * 4;
  const x = Math.random() * 100;
  const y = Math.random() * 100;
  const duration = 6 + Math.random() * 10;
  const delay = Math.random() * -12;
  return {
    id: i,
    style: {
      width: `${size}px`,
      height: `${size}px`,
      left: `${x}%`,
      top: `${y}%`,
      animationDuration: `${duration}s`,
      animationDelay: `${delay}s`,
      opacity: (0.2 + Math.random() * 0.6).toString(),
    },
  };
});

// 天使羽毛：缓缓飘落的白色光羽
const angelFeathers: StaticElement[] = Array.from({ length: 12 }, (_, i) => {
  const size = 10 + Math.random() * 20;
  const x = Math.random() * 100;
  const duration = 12 + Math.random() * 15;
  const delay = Math.random() * -15;
  return {
    id: i,
    style: {
      width: `${size}px`,
      height: `${size * 2}px`,
      left: `${x}%`,
      animationDuration: `${duration}s`,
      animationDelay: `${delay}s`,
    },
  };
});

// 恶魔粒子：下沉的暗紫光点
const devilParticles: StaticElement[] = Array.from({ length: 60 }, (_, i) => {
  const size = 2 + Math.random() * 15;
  const x = Math.random() * 100;
  const y = Math.random() * 100;
  const duration = 8 + Math.random() * 12;
  const delay = Math.random() * -15;
  return {
    id: i,
    style: {
      width: `${size}px`,
      height: `${size}px`,
      left: `${x}%`,
      top: `${y}%`,
      animationDuration: `${duration}s`,
      animationDelay: `${delay}s`,
      opacity: (0.3 + Math.random() * 0.5).toString(),
    },
  };
});
</script>

<style scoped lang="scss">
/* ====================================
   鸢一折纸 · 双形态动态印象首页
   天使：白/金，上升光羽
   恶魔：黑/深蓝，下沉漩涡裂痕
   ==================================== */

.origami-hero {
  --bg-base: #0a0e14;
  --crown-color: #c5a059;
  --crown-shadow: rgba(197, 160, 89, 0.7);
  --title-gradient: linear-gradient(180deg, #ffffff 0%, #e8d5a3 100%);
  --title-shadow: rgba(197, 160, 89, 0.4);
  --code-color: #c5a059;
  --typing-color: #f5e7d3;
  --badge-border: rgba(197, 160, 89, 0.5);
  --badge-bg: rgba(0, 0, 0, 0.25);
  --badge-color: #f5e7d3;
  --bottom-light-color: #c5a059;
  --footer-deco-color: #c5a059;
  --text-dim: #d5c8b2;
  --line-color: rgba(197, 160, 89, 0.8);
  --cursor-color: #c5a059;

  position: relative;
  width: 100vw;
  height: 100vh;
  background: var(--bg-base);
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: "Noto Sans SC", "Inter", sans-serif;
  user-select: none;
  transition: background 0.6s ease;

  &.reversed {
    --bg-base: #05070a;
    --crown-color: #5a75a0;
    --crown-shadow: rgba(90, 117, 160, 0.6);
    --title-gradient: linear-gradient(180deg, #d0d8e4 0%, #5a75a0 100%);
    --title-shadow: rgba(90, 117, 160, 0.3);
    --code-color: #5a75a0;
    --typing-color: #8a9bb5;
    --badge-border: rgba(90, 117, 160, 0.5);
    --badge-bg: rgba(0, 0, 0, 0.35);
    --badge-color: #8a9bb5;
    --bottom-light-color: #5a75a0;
    --footer-deco-color: #5a75a0;
    --text-dim: #6b7a90;
    --line-color: rgba(90, 117, 160, 0.8);
    --cursor-color: #5a75a0;
  }
}

/* ---------- 背景层 ---------- */
.bg-layer {
  position: absolute;
  inset: 0;
  pointer-events: none;
  z-index: 0;
}

/* 通用扫描线 */
.scanlines {
  position: absolute;
  inset: 0;
  background: repeating-linear-gradient(
    0deg,
    transparent,
    transparent 3px,
    rgba(255, 255, 255, 0.02) 3px,
    rgba(255, 255, 255, 0.02) 6px
  );
  animation: scan 8s linear infinite;
}

@keyframes scan {
  from {
    background-position: 0 0;
  }
  to {
    background-position: 0 12px;
  }
}

/* ========== 天使形态动态元素 ========== */
.angel-particle {
  position: absolute;
  border-radius: 50%;
  background: radial-gradient(circle, #ffe484, #c5a059);
  box-shadow: 0 0 8px #ffd966;
  animation: rise linear infinite;
}

@keyframes rise {
  0% {
    transform: translateY(0) translateX(0) scale(0.5);
    opacity: 0;
  }
  10% {
    opacity: 1;
  }
  90% {
    opacity: 0.7;
  }
  100% {
    transform: translateY(-110vh) translateX(20px) scale(0.2);
    opacity: 0;
  }
}

.angel-feather {
  position: absolute;
  top: -10%;
  background: linear-gradient(to bottom, transparent, #f5e7d3, #ffffff);
  clip-path: polygon(50% 0%, 100% 40%, 80% 100%, 20% 100%, 0% 40%);
  opacity: 0;
  animation: float-feather linear infinite;
}

@keyframes float-feather {
  0% {
    transform: translateY(0) rotate(0deg) scale(0.8);
    opacity: 0;
  }
  10% {
    opacity: 0.6;
  }
  90% {
    opacity: 0.3;
  }
  100% {
    transform: translateY(110vh) rotate(360deg) scale(0.5);
    opacity: 0;
  }
}

.angel-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 500px;
  height: 500px;
  background: radial-gradient(circle, rgba(197, 160, 89, 0.1), transparent 70%);
  animation: glow-pulse 6s ease-in-out infinite;
}

@keyframes glow-pulse {
  0%,
  100% {
    opacity: 0.3;
    transform: translate(-50%, -50%) scale(0.9);
  }
  50% {
    opacity: 0.6;
    transform: translate(-50%, -50%) scale(1.1);
  }
}

/* ========== 恶魔形态动态元素 ========== */
.devil-particle {
  position: absolute;
  border-radius: 50%;
  background: radial-gradient(circle, #8b4a6a, #3a1e3a);
  box-shadow: 0 0 10px #6b2a4a;
  animation: sink linear infinite;
}

@keyframes sink {
  0% {
    transform: translateY(0) translateX(0) scale(0.6);
    opacity: 0;
  }
  10% {
    opacity: 0.8;
  }
  90% {
    opacity: 0.4;
  }
  100% {
    transform: translateY(110vh) translateX(-15px) scale(0.2);
    opacity: 0;
  }
}

.devil-vortex {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 400px;
  height: 400px;
  border: 1px solid rgba(107, 42, 74, 1);
  border-radius: 50%;
  transform: translate(-50%, -50%);
  background: conic-gradient(
    from 0deg,
    transparent,
    rgba(107, 42, 74, 0.1),
    transparent 60%
  );
  animation: rotate-vortex 20s linear infinite;
  filter: blur(2px);
}

@keyframes rotate-vortex {
  from {
    transform: translate(-50%, -50%) rotate(0deg);
  }
  to {
    transform: translate(-50%, -50%) rotate(360deg);
  }
}

.devil-cracks {
  position: absolute;
  inset: 0;
  .crack {
    position: absolute;
    background: linear-gradient(90deg, transparent, rgba(107, 42, 74, 0.2));
    transform-origin: left;
    animation: crack-flicker 3s ease-in-out infinite;
  }
  .crack:nth-child(1) {
    top: 20%;
    left: 10%;
    width: 200px;
    height: 1px;
    transform: rotate(25deg);
    animation-delay: 0s;
  }
  .crack:nth-child(2) {
    top: 60%;
    left: 30%;
    width: 180px;
    height: 1px;
    transform: rotate(-15deg);
    animation-delay: 0.8s;
  }
  .crack:nth-child(3) {
    top: 40%;
    left: 70%;
    width: 150px;
    height: 1px;
    transform: rotate(45deg);
    animation-delay: 1.6s;
  }
}

@keyframes crack-flicker {
  0%,
  100% {
    opacity: 0.2;
  }
  50% {
    opacity: 0.7;
    box-shadow: 0 0 6px rgba(107, 42, 74, 0.5);
  }
}

/* ---------- 中心内容 ---------- */
.hero-content {
  position: relative;
  z-index: 10;
  text-align: center;
  animation: content-fade-in 1s ease-out;
}

@keyframes content-fade-in {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.crown-btn {
  background: none;
  border: none;
  cursor: pointer;
  margin-bottom: 16px;
  padding: 0;
  .crown {
    font-size: 38px;
    color: var(--crown-color);
    display: inline-block;
    filter: drop-shadow(0 0 12px var(--crown-shadow));
    animation: crown-float 3s ease-in-out infinite;
  }
  &:hover {
    filter: drop-shadow(0 0 20px var(--crown-shadow));
  }
}

@keyframes crown-float {
  0%,
  100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-6px);
  }
}

.name {
  font-size: 56px;
  font-weight: 800;
  letter-spacing: 8px;
  margin: 0 0 12px;
  background: var(--title-gradient);
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  text-shadow: 0 0 30px var(--title-shadow);

  .name-sub {
    display: block;
    font-size: 16px;
    font-weight: 400;
    letter-spacing: 5px;
    color: var(--text-dim);
    margin-top: 6px;
    background: none;
    -webkit-text-fill-color: var(--text-dim);
    text-shadow: none;
  }
}

.title-line {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 16px;
  margin: 18px 0;

  .line {
    width: 100px;
    height: 1px;
    background: linear-gradient(
      90deg,
      transparent,
      var(--line-color),
      transparent
    );
  }

  .code {
    font-size: 16px;
    letter-spacing: 4px;
    color: var(--code-color);
    font-weight: 500;
  }
}

.typewriter-box {
  min-height: 52px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 28px 0 24px;
  font-size: 20px;
  color: var(--typing-color);
  letter-spacing: 3px;
  text-shadow: 0 0 12px rgba(0, 0, 0, 0.5);
}

.typing-text {
  white-space: pre-wrap;
}

.cursor {
  display: inline-block;
  color: var(--cursor-color);
  font-weight: 100;
  margin-left: 2px;
  animation: blink 1s step-end infinite;
}

@keyframes blink {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0;
  }
}

.ast-badge {
  display: inline-block;
  padding: 10px 32px;
  border: 1px solid var(--badge-border);
  border-radius: 24px;
  font-size: 14px;
  color: var(--badge-color);
  letter-spacing: 2px;
  backdrop-filter: blur(8px);
  background: var(--badge-bg);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
}

/* ---------- 底部 & 页脚 ---------- */
.bottom-area {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-bottom: 28px;
  z-index: 5;
}

.bottom-light {
  width: 300px;
  height: 2px;
  background: linear-gradient(
    90deg,
    transparent,
    var(--bottom-light-color),
    transparent
  );
  filter: blur(4px);
  animation: bottom-glow 3s ease-in-out infinite;
  margin-bottom: 20px;
}

@keyframes bottom-glow {
  0%,
  100% {
    opacity: 0.3;
    width: 200px;
  }
  50% {
    opacity: 0.9;
    width: 400px;
  }
}

.page-footer {
  text-align: center;
  font-size: 13px;
  color: var(--text-dim);
  letter-spacing: 1px;
  line-height: 1.8;

  .footer-deco {
    color: var(--footer-deco-color);
    margin: 0 0 8px;
    text-shadow: 0 0 8px var(--crown-shadow);
  }

  .footer-mode-hint {
    font-size: 12px;
    opacity: 0.8;
    margin-bottom: 8px;
  }

  .footer-text {
    opacity: 0.6;
  }
}
</style>
