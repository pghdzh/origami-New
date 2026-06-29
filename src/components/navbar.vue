<template>
  <!-- 移动端左上角汉堡按钮 (仅窄屏且抽屉未打开时显示) -->
  <button
    v-if="isMobile && !mobileDrawerOpen"
    class="mobile-hamburger"
    @click="toggleMobileDrawer"
    aria-label="打开档案柜"
  >
    <span class="hamburger-line"></span>
    <span class="hamburger-line"></span>
    <span class="hamburger-line"></span>
  </button>

  <!-- 侧边档案柜主体 (AST深色档案柜风格) -->
  <aside
    class="ast-archive"
    :class="{
      collapsed: isCollapsed && !isMobile,
      'drawer-open': mobileDrawerOpen,
    }"
  >
    <!-- 动态背景层：扫描线 + 粒子 + 光晕 + 网格 -->
    <div class="bg-scanlines"></div>
    <div class="particles-bg">
      <span
        v-for="i in 30"
        :key="'p' + i"
        class="particle"
        :class="i % 3 === 0 ? 'gold' : 'ice'"
        :style="particleStyle(i)"
      ></span>
    </div>
    <div class="bg-glow"></div>
    <div class="bg-grid"></div>

    <!-- 移动端关闭按钮 (右上角，仅抽屉打开时显示) -->
    <button
      v-if="isMobile && mobileDrawerOpen"
      class="mobile-close-btn"
      @click="toggleMobileDrawer"
      aria-label="关闭档案柜"
    >
      ✕
    </button>

    <!-- 档案标题区 -->
    <div class="archive-header" v-show="!isCollapsed || isMobile">
      <h1 class="archive-title">折纸电子设定集</h1>
      <div class="online-badge" v-if="onlineCount !== null">
        <span class="badge-dot"></span>
        <span class="online-text"
          >在线：<span class="count-num">{{ onlineCount }}</span></span
        >
      </div>
      <p class="archive-sub">AST 机密档案 · 鸢一折纸</p>
    </div>

    <!-- 导航标签列表 -->
    <nav class="archive-nav">
      <RouterLink
        v-for="(item, index) in navItems"
        :key="item.name"
        :to="item.path"
        class="nav-tab"
        active-class="tab-active"
        :style="{ transitionDelay: `${index * 0.03}s` }"
        @click="closeDrawer"
      >
        <span class="tab-text">{{ item.name }}</span>
        <span class="tab-glow"></span>
      </RouterLink>

      <a
        href="https://slty.site/#/redirector"
        target="_blank"
        rel="noopener"
        class="nav-tab"
        @click="closeDrawer"
      >
        <span class="tab-text">霜落映界</span>
        <span class="tab-glow"></span>
      </a>
    </nav>

    <!-- 底部页脚 (冰蓝脉冲装饰) -->
    <footer class="archive-footer" v-show="!isCollapsed || isMobile">
      <div class="footer-deco">✦ 霜落天亦 ✦</div>
      <p class="footer-text">© {{ currentYear }} 鸢一折纸电子设定集</p>
      <div class="footer-line"></div>
      <p class="footer-quote">“我不需要感情。只要能消灭精灵……”</p>
    </footer>

    <!-- 桌面端折叠按钮 (优化的圆形冰蓝按钮) -->
    <button
      class="collapse-btn"
      @click="toggleCollapse"
      :title="isCollapsed ? '展开档案柜' : '收起档案柜'"
      v-if="!isMobile"
    >
      <span class="collapse-icon" :class="{ flipped: isCollapsed }"> ◀ </span>
    </button>
  </aside>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from "vue";
import { io } from "socket.io-client";

// ============ 导航项 ============
const navItems = [
  { name: "首页", path: "/" },
  { name: "角色概览", path: "/overview" },
  { name: "时间线", path: "/timeline" },
  { name: "留言板", path: "/message" },
  { name: "图集", path: "/gallery" },
  { name: "资源分享", path: "/resources" },
  { name: "AI对话", path: "/talk" },
  { name: "文本分享", path: "/wiki" },
];

// ============ 状态 ============
const isCollapsed = ref(false);
const mobileDrawerOpen = ref(false);
const isMobile = ref(false);

// 动态年份
const currentYear = new Date().getFullYear();

function checkMobile() {
  isMobile.value = window.innerWidth <= 768;
  if (!isMobile.value) mobileDrawerOpen.value = false;
}

onMounted(() => {
  checkMobile();
  window.addEventListener("resize", checkMobile);
});

onBeforeUnmount(() => {
  window.removeEventListener("resize", checkMobile);
});

function toggleCollapse() {
  isCollapsed.value = !isCollapsed.value;
}

function toggleMobileDrawer() {
  mobileDrawerOpen.value = !mobileDrawerOpen.value;
}

function closeDrawer() {
  if (isMobile.value) mobileDrawerOpen.value = false;
}

// ============ 在线人数 ============
const siteId = "origami";
const onlineCount = ref<number | null>(null);
const socket: any = io(import.meta.env.VITE_API_BASE_URL, {
  query: { siteId },
});

onMounted(() => {
  socket.on("onlineCount", (count: number) => {
    onlineCount.value = count;
  });
});

onBeforeUnmount(() => {
  socket.disconnect();
});

// ============ 粒子随机样式 ============
function particleStyle(i: number) {
  const size = 2 + Math.random() * 5;
  const x = Math.random() * 100;
  const y = Math.random() * 100;
  const duration = 8 + Math.random() * 12;
  const delay = Math.random() * -10;
  return {
    width: `${size}px`,
    height: `${size}px`,
    left: `${x}%`,
    top: `${y}%`,
    animationDuration: `${duration}s`,
    animationDelay: `${delay}s`,
  };
}
</script>

<style scoped lang="scss">
/* ====================================
   鸢一折纸 · AST 深色档案柜
   配色：深枪铁灰 / 冰晶蓝 / 哑光金
   ==================================== */
$archive-width: 260px;
$archive-collapsed-width: 50px;

// 核心角色配色（完全硬编码，无任何动态函数）
$gunmetal: #2c3a4a;
$gunmetal-hover: #3e4f5f; // 替代 lighten($gunmetal, 5%)
$ice-blue: #5b9bd5;
$ice-blue-light: #8ec5fc;
$ice-blue-dark: #3a7ab5;
$gold: #c5a059;
$gold-light: #e8cd7a;
$white: #ffffff;
$text-dim: #8a9aac;
$bg-dark: #1a2634;
$border-subtle: rgba($ice-blue, 0.15);

// 移动端汉堡按钮
.mobile-hamburger {
  display: none;
  position: fixed;
  top: 16px;
  left: 16px;
  z-index: 1200;
  width: 42px;
  height: 36px;
  background: $gunmetal;
  border: 1px solid $ice-blue;
  border-radius: 6px;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 5px;
  cursor: pointer;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.5);

  .hamburger-line {
    display: block;
    width: 22px;
    height: 2px;
    background: $white;
    border-radius: 1px;
  }

  @media (max-width: 768px) {
    display: flex;
  }
}

// 主侧边栏
.ast-archive {
  position: fixed;
  left: 0;
  top: 0;
  width: $archive-width;
  height: 100vh;
  background: $gunmetal;
  border-right: 1px solid rgba($ice-blue, 0.2);
  box-shadow: 4px 0 30px rgba(0, 0, 0, 0.5);
  display: flex;
  flex-direction: column;
  padding: 30px 0 20px;
  z-index: 1100;
  overflow: hidden;
  transition: width 0.35s cubic-bezier(0.4, 0, 0.2, 1),
    transform 0.35s cubic-bezier(0.4, 0, 0.2, 1);
  font-family: "Noto Sans SC", "Inter", sans-serif;

  // 金属拉丝纹理
  &::before {
    content: "";
    position: absolute;
    inset: 0;
    background: repeating-linear-gradient(
      90deg,
      transparent,
      transparent 2px,
      rgba(255, 255, 255, 0.01) 2px,
      rgba(255, 255, 255, 0.01) 4px
    );
    pointer-events: none;
  }

  &.collapsed {
    width: $archive-collapsed-width;

    .archive-header,
    .archive-footer,
    .tab-text {
      opacity: 0;
      visibility: hidden;
    }

    .nav-tab {
      justify-content: center;
      padding: 12px 0;
    }

    // 完全隐藏选中效果
    .nav-tab.tab-active {
      background: transparent;
      border-left: none;
      box-shadow: none;
      .tab-glow {
        transform: scaleY(0);
      }
      .tab-text {
        color: transparent;
      }
    }
  }

  @media (max-width: 768px) {
    transform: translateX(-100%);
    width: $archive-width !important;

    &.drawer-open {
      transform: translateX(0);
    }
  }
}

// 移动端关闭按钮
.mobile-close-btn {
  position: absolute;
  top: 16px;
  right: 16px;
  width: 36px;
  height: 36px;
  background: transparent;
  border: none;
  font-size: 24px;
  color: $white;
  cursor: pointer;
  z-index: 20;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: color 0.2s;

  &:hover {
    color: $ice-blue-light;
  }
}

/* ========== 动态背景层 ========== */
.bg-scanlines {
  position: absolute;
  inset: 0;
  background: repeating-linear-gradient(
    0deg,
    transparent,
    transparent 3px,
    rgba(91, 155, 213, 0.04) 3px,
    rgba(91, 155, 213, 0.04) 6px
  );
  pointer-events: none;
  animation: scan-move 10s linear infinite;
}

@keyframes scan-move {
  from {
    background-position: 0 0;
  }
  to {
    background-position: 0 20px;
  }
}

.particles-bg {
  position: absolute;
  inset: 0;
  overflow: hidden;
  pointer-events: none;

  .particle {
    position: absolute;
    border-radius: 50%;
    animation: float-up linear infinite;

    &.gold {
      background: radial-gradient(
        circle,
        rgba($gold-light, 0.9),
        transparent 70%
      );
      box-shadow: 0 0 6px $gold-light;
    }
    &.ice {
      background: radial-gradient(
        circle,
        rgba($ice-blue-light, 0.8),
        transparent 70%
      );
      box-shadow: 0 0 6px $ice-blue-light;
    }
  }
}

@keyframes float-up {
  0% {
    transform: translateY(0) translateX(0) scale(0.6);
    opacity: 0;
  }
  10% {
    opacity: 0.9;
  }
  90% {
    opacity: 0.4;
  }
  100% {
    transform: translateY(-100vh) translateX(15px) scale(0.1);
    opacity: 0;
  }
}

.bg-glow {
  position: absolute;
  top: 15%;
  right: -40px;
  width: 200px;
  height: 200px;
  background: radial-gradient(circle, rgba($ice-blue, 0.12), transparent 70%);
  border-radius: 50%;
  filter: blur(50px);
  animation: drift 12s ease-in-out infinite;
  pointer-events: none;
}

.bg-grid {
  position: absolute;
  inset: 0;
  background-image: linear-gradient(rgba($ice-blue, 0.05) 1px, transparent 1px),
    linear-gradient(90deg, rgba($ice-blue, 0.05) 1px, transparent 1px);
  background-size: 40px 40px;
  pointer-events: none;
}

@keyframes drift {
  0%,
  100% {
    transform: translate(0, 0);
  }
  50% {
    transform: translate(-20px, -10px);
  }
}

/* ========== 标题区 ========== */
.archive-header {
  padding: 0 24px 20px;
  border-bottom: 1px solid $border-subtle;
  margin-bottom: 16px;
  position: relative;
  z-index: 1;
  transition: opacity 0.25s, visibility 0.25s;

  .archive-title {
    font-size: 24px;
    font-weight: 700;
    margin: 0 0 12px;
    background: linear-gradient(
      130deg,
      $ice-blue-light,
      $ice-blue,
      $ice-blue-dark
    );
    background-clip: text;
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    letter-spacing: 1.5px;
    white-space: nowrap;
    position: relative;

    &::after {
      content: "";
      position: absolute;
      top: 0;
      left: -100%;
      width: 60%;
      height: 100%;
      background: linear-gradient(
        90deg,
        transparent,
        rgba(255, 255, 255, 0.15),
        transparent
      );
      transform: skewX(-15deg);
      animation: shine 3s infinite;
    }
  }

  .online-badge {
    display: inline-flex;
    align-items: center;
    background: rgba(0, 0, 0, 0.3);
    backdrop-filter: blur(8px);
    border: 1px solid rgba($gold, 0.3);
    border-radius: 14px;
    padding: 4px 14px;
    font-size: 0.85rem;
    color: $white;
    margin-bottom: 10px;

    .badge-dot {
      width: 7px;
      height: 7px;
      border-radius: 50%;
      background: #4caf50;
      margin-right: 6px;
      box-shadow: 0 0 10px #4caf50;
      animation: pulse-dot 2s infinite;
    }

    .count-num {
      font-weight: 700;
      background: linear-gradient(180deg, $gold-light, $gold);
      background-clip: text;
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      margin-left: 4px;
    }
  }

  .archive-sub {
    font-size: 12px;
    color: $text-dim;
    margin: 4px 0 0;
    letter-spacing: 2px;
    font-style: italic;
    text-transform: uppercase;
  }
}

@keyframes shine {
  0% {
    left: -100%;
  }
  40% {
    left: 100%;
  }
  100% {
    left: 100%;
  }
}

@keyframes pulse-dot {
  0%,
  100% {
    box-shadow: 0 0 8px #4caf50;
  }
  50% {
    box-shadow: 0 0 18px #4caf50, 0 0 8px #4caf50;
  }
}

/* ========== 导航项 ========== */
.archive-nav {
  flex: 1;
  padding: 0 12px;
  display: flex;
  flex-direction: column;
  gap: 4px;
  overflow-y: auto;
  position: relative;
  z-index: 1;

  &::-webkit-scrollbar {
    width: 4px;
  }
  &::-webkit-scrollbar-thumb {
    background: rgba($ice-blue, 0.3);
    border-radius: 4px;
  }
}

.nav-tab {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 16px;
  border-radius: 8px;
  text-decoration: none;
  font-size: 15px;
  font-weight: 500;
  color: $white;
  background: transparent;
  position: relative;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  cursor: pointer;
  overflow: hidden;
  border: 1px solid transparent;

  .tab-text {
    position: relative;
    z-index: 2;
    transition: transform 0.3s, color 0.3s;
    white-space: nowrap;
  }

  .tab-glow {
    position: absolute;
    left: 0;
    top: 0;
    height: 100%;
    width: 4px;
    background: $ice-blue;
    transform: scaleY(0);
    transition: transform 0.35s cubic-bezier(0.4, 0, 0.2, 1);
    border-radius: 0 4px 4px 0;
    box-shadow: 0 0 10px $ice-blue-light;
  }

  &:hover {
    background: rgba($ice-blue, 0.1);
    border-color: rgba($ice-blue, 0.3);
    box-shadow: 0 0 15px rgba($ice-blue, 0.2);
    color: $ice-blue-light;

    .tab-glow {
      transform: scaleY(1);
    }
    .tab-text {
      transform: translateX(6px);
    }
  }

  &.tab-active {
    background: rgba($ice-blue, 0.15);
    border-left: 3px solid $ice-blue-light;
    font-weight: 700;
    color: $ice-blue-light;
    box-shadow: 0 4px 15px rgba($ice-blue, 0.3);

    .tab-glow {
      transform: scaleY(1);
      background: $ice-blue-light;
    }
    .tab-text {
      transform: translateX(4px);
    }
  }
}

/* ========== 页脚 ========== */
.archive-footer {
  padding: 20px 24px 10px;
  border-top: 1px solid $border-subtle;
  text-align: center;
  font-size: 12px;
  color: $text-dim;
  position: relative;
  z-index: 1;
  transition: opacity 0.25s, visibility 0.25s;

  .footer-deco {
    color: $gold;
    font-size: 13px;
    font-weight: 600;
    letter-spacing: 2px;
    margin-bottom: 8px;
    animation: text-glow 4s ease-in-out infinite;
  }

  .footer-text {
    line-height: 1.6;
    margin: 4px 0;
  }

  .footer-line {
    width: 50px;
    height: 1px;
    background: $ice-blue;
    margin: 12px auto;
    opacity: 0.6;
    animation: line-pulse 2s infinite;
  }

  .footer-quote {
    font-style: italic;
    opacity: 0.8;
    margin-top: 6px;
  }
}

@keyframes text-glow {
  0%,
  100% {
    text-shadow: 0 0 4px rgba($gold, 0.3);
  }
  50% {
    text-shadow: 0 0 15px rgba($gold-light, 0.6);
  }
}

@keyframes line-pulse {
  0%,
  100% {
    opacity: 0.4;
    transform: scaleX(1);
  }
  50% {
    opacity: 1;
    transform: scaleX(1.2);
  }
}

/* ========== 折叠按钮 ========== */
.collapse-btn {
  position: absolute;
  right: -18px;
  top: 50%;
  transform: translateY(-50%);
  width: 36px;
  height: 36px;
  background: $gunmetal;
  border: 2px solid $ice-blue;
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 10;
  box-shadow: 0 0 12px rgba($ice-blue, 0.5);
  transition: background 0.3s, box-shadow 0.3s, border-color 0.3s;
  padding: 0;

  &:hover {
    background: $gunmetal-hover; // 硬编码的提亮色
    border-color: $ice-blue-light;
    box-shadow: 0 0 20px rgba($ice-blue-light, 0.7);
  }

  .collapse-icon {
    font-size: 16px;
    color: $ice-blue-light;
    display: block;
    transition: transform 0.3s;

    &.flipped {
      transform: rotate(180deg); // 折叠时向右箭头
    }
  }
}
</style>
