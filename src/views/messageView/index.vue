<template>
  <div class="origami-message-board" aria-live="polite">
    <!-- 背景轮播（保留原有图片） -->
    <div class="carousel carousel1" aria-hidden="true">
      <img
        v-for="(src, idx) in randomFive"
        :key="idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
      />
    </div>
    <div class="carousel carousel2" aria-hidden="true">
      <img
        v-for="(src, idx) in randomFive2"
        :key="idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
      />
    </div>
    <!-- 柔和叠加层，让背景融入天使氛围 -->
    <div class="bg-overlay"></div>

    <!-- 顶部标题区（档案标签风格） -->
    <header class="board-header" role="banner">
      <div class="header-tab">
        <span class="tab-icon">✦</span>
        <h1>留言板</h1>
        <span class="title-count">共 {{ totalCount }} 条</span>
      </div>
      <p class="subtitle">绝灭天使之光 · 每一句留言皆为永恒</p>
    </header>

    <!-- 留言展示区（双列错落布局） -->
    <section class="message-list">
      <div class="message-list-inner">
        <transition-group name="msg" tag="div" class="msg-grid">
          <div
            v-if="loading && messages.length === 0"
            class="skeleton-wrap"
            key="skeleton"
          >
            <div class="skeleton" v-for="i in 3" :key="i">
              <div class="sk-avatar"></div>
              <div class="sk-lines">
                <div class="sk-line short"></div>
                <div class="sk-line"></div>
              </div>
            </div>
          </div>
          <div
            v-for="(msg, idx) in messages"
            :key="msg.id || msg._tempId || idx"
            class="message-card"
            :class="{ 'card-right': idx % 2 === 1 }"
            :data-index="idx"
            tabindex="0"
            role="article"
            :aria-label="`留言来自 ${msg.name || '匿名'}，内容：${msg.content}`"
          >
            <div class="card-inner">
              <div class="message-meta">
                <div class="name-avatar" :title="msg.name || '匿名'">
                  {{ getInitial(msg.name) }}
                </div>
                <div class="meta-texts">
                  <div class="message-name">{{ msg.name || "匿名" }}</div>
                  <div class="message-time">
                    {{ formatTime(msg.created_at) }}
                  </div>
                </div>
              </div>
              <p class="message-content">{{ msg.content }}</p>
              <div class="card-feather"></div>
            </div>
          </div>
        </transition-group>
        <!-- 无限滚动哨兵 -->
        <div ref="sentinel" class="sentinel"></div>
        <!-- 加载更多提示 -->
        <div v-if="loadingMore" class="loading-more">
          <span class="angel-symbol">✧</span> 光羽凝聚中...
          <span class="angel-symbol">✧</span>
        </div>
        <div v-if="!hasMore && messages.length > 0" class="end-message">
          —— 已达绝灭尽头 · 暂无更多留言 ——
        </div>
      </div>
    </section>

    <!-- 底部发送区（悬浮信笺风格） -->
    <section class="message-form" aria-label="留下你的光芒箴言">
      <div class="form-inner">
        <label class="sr-only" for="mb-name">你的昵称</label>
        <input
          id="mb-name"
          v-model="name"
          type="text"
          placeholder="光之羽翼下，留下你的名号"
          @keydown.enter.prevent
        />
        <label class="sr-only" for="mb-content">留言内容</label>
        <textarea
          id="mb-content"
          v-model="content"
          placeholder="有什么想对折纸大师留言的吗…"
          @keydown.ctrl.enter.prevent="submitMessage"
          @input="autoGrow"
          ref="textareaRef"
        />
        <div class="form-row">
          <div class="hint">
            <span class="angel-symbol">✧</span>
            <kbd>Ctrl</kbd> + <kbd>Enter</kbd>快速发送
            <span class="angel-symbol">✧</span>
          </div>
          <button
            @click="submitMessage"
            :disabled="isSending || !content.trim()"
          >
            <span v-if="!isSending">铭刻留言</span>
            <span v-else>光羽凝集…</span>
          </button>
        </div>
      </div>
    </section>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, nextTick } from "vue";
import { getMessageList, createMessage } from "@/api/modules/message";

const messages = ref<any[]>([]);
const totalCount = ref(0);
const currentPage = ref(1);
const pageSize = 20;
const hasMore = ref(true);
const loading = ref(true);
const loadingMore = ref(false);
let observer: IntersectionObserver | null = null;

const name = ref(localStorage.getItem("message_name") || "");
const content = ref("");
const isSending = ref(false);
const textareaRef = ref<HTMLTextAreaElement | null>(null);
const sentinel = ref<HTMLElement | null>(null);

const loadMessages = async (page: number, append = false) => {
  if (page === 1) loading.value = true;
  else loadingMore.value = true;
  try {
    const res = await getMessageList({ page, pageSize });
    const newData = res.data || [];
    const pagination = res.pagination;
    if (append) messages.value = [...messages.value, ...newData];
    else messages.value = newData;
    totalCount.value = pagination.total;
    hasMore.value = page < pagination.totalPages;
    currentPage.value = page;
    await nextTick();
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
    loadingMore.value = false;
  }
};

const loadNextPage = () => {
  if (!hasMore.value || loadingMore.value) return;
  loadMessages(currentPage.value + 1, true);
};

const submitMessage = async () => {
  if (!content.value.trim() || isSending.value) return;
  isSending.value = true;
  const payload = { name: name.value || "匿名", content: content.value };
  try {
    localStorage.setItem("message_name", name.value);
    content.value = "";
    await nextTick();
    await createMessage(payload);
    currentPage.value = 1;
    hasMore.value = true;
    await loadMessages(1, false);
    const listEl = document.querySelector(".message-list-inner");
    if (listEl) listEl.scrollTop = 0;
  } catch (err) {
    console.error(err);
  } finally {
    isSending.value = false;
  }
};

const formatTime = (time: string) => {
  if (!time) return "";
  const d = new Date(time);
  const y = d.getFullYear();
  const m = String(d.getMonth() + 1).padStart(2, "0");
  const day = String(d.getDate()).padStart(2, "0");
  const hh = String(d.getHours()).padStart(2, "0");
  const mm = String(d.getMinutes()).padStart(2, "0");
  return `${y}-${m}-${day} ${hh}:${mm}`;
};

const getInitial = (n?: string) => {
  if (!n) return "匿";
  return n.trim().slice(0, 1).toUpperCase();
};

const autoGrow = (e?: Event) => {
  const ta = textareaRef.value;
  if (!ta) return;
  ta.style.height = "auto";
  const h = Math.min(ta.scrollHeight, 220);
  ta.style.height = h + "px";
};

// 背景轮播
const modules = import.meta.glob("@/assets/images1/*.{jpg,png,jpeg,webp}", {
  eager: true,
});
const allSrcs: string[] = Object.values(modules).map((mod: any) => mod.default);
const modules2 = import.meta.glob("@/assets/images2/*.{jpg,png,jpeg,webp}", {
  eager: true,
});
const allSrcs2: string[] = Object.values(modules2).map(
  (mod: any) => mod.default
);

function shuffle<T>(arr: T[]): T[] {
  const a = arr.slice();
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}
const randomFive = ref<string[]>(shuffle(allSrcs).slice(0, 5));
const randomFive2 = ref<string[]>(shuffle(allSrcs2).slice(0, 5));
const currentIndex = ref(0);
let Imgtimer: number | undefined;

onMounted(async () => {
  await loadMessages(1, false);
  observer = new IntersectionObserver(
    (entries) => {
      if (entries[0].isIntersecting && hasMore.value && !loadingMore.value) {
        loadNextPage();
      }
    },
    { threshold: 0.5 }
  );
  if (sentinel.value) observer.observe(sentinel.value);
  Imgtimer = window.setInterval(() => {
    currentIndex.value =
      (currentIndex.value + 1) % Math.max(1, randomFive.value.length);
  }, 5200);
  nextTick(() => autoGrow());
});

onUnmounted(() => {
  if (observer) observer.disconnect();
  if (Imgtimer) clearInterval(Imgtimer);
});
</script>

<style scoped lang="scss">
/* 鸢一折纸 天使形态配色 */
$bg-base: #f5f8fc;
$pure-white: #ffffff;
$gold-primary: #c5a059;
$gold-light: #e8cd7a;
$ice-blue: #5b9bd5;
$ice-blue-light: #8ec5fc;
$text-primary: #2e2e2e;
$text-dim: #6b6b6b;
$card-bg: rgba(255, 255, 255, 0.85);
$card-border: rgba(197, 160, 89, 0.25);
$shadow-soft: 0 4px 20px rgba(44, 58, 74, 0.06);
$shadow-card: 0 2px 12px rgba(44, 58, 74, 0.04);

.origami-message-board {
  position: relative;
  min-height: 100vh;
  padding: 90px 0 120px;
  display: flex;
  flex-direction: column;
  align-items: center;
  background: $bg-base;
  font-family: "Noto Sans SC", "Inter", sans-serif;
  color: $text-primary;
  overflow-x: hidden;
  -webkit-font-smoothing: antialiased;

  /* 背景轮播 */
  .carousel {
    position: fixed;
    inset: 0;
    z-index: 0;
    pointer-events: none;
    .carousel-image {
      position: absolute;
      width: 100%;
      height: 100%;
      object-fit: cover;
      opacity: 0;
      transition: opacity 1.5s ease;
      filter: brightness(0.75); /* 增加饱和度和亮度，更清晰 */
      &.active {
        opacity: 1;
      }
    }
  }
  .carousel2 {
    display: none;
  }

  /* 柔和叠加层 */
  .bg-overlay {
    position: fixed;
    inset: 0;
    z-index: 1;
    pointer-events: none;
    background: linear-gradient(
      180deg,
      rgba(245, 248, 252, 0.1) 0%,
      rgba(245, 248, 252, 0.25) 100%
    );
  }

  /* ========== 顶部标题区（档案标签风格） ========== */
  .board-header {
    position: relative;
    z-index: 10;
    text-align: center;
    margin-bottom: 40px;
    .header-tab {
      display: inline-flex;
      align-items: center;
      gap: 10px;
      background: $pure-white;
      padding: 12px 32px;
      border-radius: 0 0 20px 20px;
      box-shadow: $shadow-soft;
      border: 1px solid rgba($gold-primary, 0.2);
      border-top: none;
      .tab-icon {
        font-size: 22px;
        color: $gold-primary;
        filter: drop-shadow(0 0 4px rgba($gold-primary, 0.4));
      }
      h1 {
        margin: 0;
        font-size: 1.5rem;
        font-weight: 700;
        color: $text-primary;
        letter-spacing: 2px;
        font-family: "Noto Serif SC", serif;
      }
      .title-count {
        font-size: 0.85rem;
        color: $text-dim;
        font-weight: 500;
        padding-left: 8px;
        border-left: 1px solid rgba($gold-primary, 0.3);
        margin-left: 4px;
      }
    }
    .subtitle {
      margin: 10px 0 0;
      font-size: 0.85rem;
      color: $text-primary;
      font-style: italic;
      letter-spacing: 2px;
    }
  }

  /* ========== 留言列表（双列错落布局） ========== */
  .message-list {
    position: relative;
    z-index: 10;
    width: 100%;
    max-width: 900px;
    padding: 0 20px;
    padding-bottom: 120px;
    .message-list-inner {
      width: 100%;
    }
    .msg-grid {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 16px;
    }

    /* 骨架屏 */
    .skeleton-wrap {
      width: 100%;
      display: flex;
      flex-direction: column;
      gap: 16px;
      .skeleton {
        display: flex;
        gap: 14px;
        align-items: center;
        padding: 16px;
        background: $card-bg;
        border-radius: 16px;
        border: 1px solid $card-border;
      }
      .sk-avatar {
        width: 44px;
        height: 44px;
        border-radius: 50%;
        background: linear-gradient(135deg, $gold-light, $ice-blue-light);
      }
      .sk-lines {
        flex: 1;
        .sk-line {
          height: 12px;
          border-radius: 6px;
          background: rgba($text-dim, 0.12);
          margin-bottom: 8px;
          &.short {
            width: 40%;
          }
        }
      }
    }

    /* 留言卡片 */
    .message-card {
      width: calc(50% - 10px);
      min-width: 280px;
      transition: all 0.35s cubic-bezier(0.2, 0.9, 0.4, 1);
      &.card-right {
        margin-top: 30px;
      }
      .card-inner {
        background: $card-bg;
        backdrop-filter: blur(2px);
        border-radius: 20px;
        padding: 20px;
        border: 1px solid $card-border;
        box-shadow: $shadow-card;
        position: relative;
        overflow: hidden;
        transition: all 0.3s;
        &:hover {
          border-color: $gold-primary;
          box-shadow: 0 8px 24px rgba(197, 160, 89, 0.12);
          transform: translateY(-2px);
          .card-feather {
            opacity: 0.5;
            transform: translateY(-2px) rotate(5deg);
          }
        }
        .card-feather {
          position: absolute;
          top: -12px;
          right: 16px;
          width: 40px;
          height: 40px;
          background: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 40 40"><path d="M20,4 Q28,18 36,20 Q28,22 20,36 Q12,22 4,20 Q12,18 20,4Z" fill="none" stroke="%23C5A059" stroke-width="1" opacity="0.4"/></svg>')
            no-repeat center;
          background-size: contain;
          opacity: 0.2;
          transition: all 0.4s;
          pointer-events: none;
        }
      }
      .message-meta {
        display: flex;
        align-items: center;
        gap: 12px;
        margin-bottom: 14px;
        .name-avatar {
          width: 44px;
          height: 44px;
          border-radius: 50%;
          background: linear-gradient(135deg, $gold-primary, $gold-light);
          display: flex;
          align-items: center;
          justify-content: center;
          font-weight: 700;
          font-size: 1rem;
          color: $pure-white;
          flex-shrink: 0;
          box-shadow: 0 2px 8px rgba(197, 160, 89, 0.3);
        }
        .meta-texts {
          .message-name {
            font-size: 0.95rem;
            font-weight: 700;
            color: $gold-primary;
            letter-spacing: 0.5px;
          }
          .message-time {
            font-size: 0.7rem;
            color: $text-dim;
            margin-top: 2px;
          }
        }
      }
      .message-content {
        font-size: 0.9rem;
        line-height: 1.65;
        color: $text-primary;
        white-space: pre-wrap;
        word-break: break-word;
        margin: 0;
        padding-left: 12px;
        border-left: 2px solid rgba($gold-primary, 0.4);
      }
    }
  }

  /* 无限滚动哨兵 */
  .sentinel {
    height: 20px;
    width: 100%;
  }

  /* 加载更多 */
  .loading-more,
  .end-message {
    text-align: center;
    padding: 20px;
    color: $text-primary;
    font-size: 0.85rem;
    letter-spacing: 1px;
    width: 100%;
    .angel-symbol {
      color: $gold-primary;
      margin: 0 8px;
    }
  }
  .end-message {
    padding: 10px;
    font-style: italic;
    font-weight: bold;
  }

  /* ========== 底部发送区（悬浮信笺） ========== */
  .message-form {
    position: fixed;
    bottom: 24px;
    left: 50%;
    transform: translateX(-50%);
    width: calc(100% - 32px);
    max-width: 700px;
    z-index: 20;
    .form-inner {
      background: rgba(255, 255, 255, 0.62);
      backdrop-filter: blur(2px);
      border-radius: 24px;
      padding: 18px 20px;
      box-shadow: 0 8px 32px rgba(44, 58, 74, 0.1);
      border: 1px solid rgba($gold-primary, 0.25);
      display: flex;
      flex-direction: column;
      gap: 10px;
      input,
      textarea {
        background: rgba($bg-base, 0.7);
        border: 1px solid rgba($gold-primary, 0.2);
        border-radius: 14px;
        padding: 10px 14px;
        font-size: 0.9rem;
        color: $text-primary;
        outline: none;
        transition: all 0.2s;
        font-family: inherit;
        resize: none;
        &::placeholder {
          color: rgba($text-dim, 0.5);
        }
        &:focus {
          border-color: $gold-primary;
          box-shadow: 0 0 6px rgba(197, 160, 89, 0.2);
          background: $pure-white;
        }
      }
      textarea {
        min-height: 60px;
        max-height: 160px;
      }
      .form-row {
        display: flex;
        align-items: center;
        justify-content: space-between;
        gap: 12px;
        .hint {
          font-size: 0.75rem;
          color: $text-dim;
          display: flex;
          align-items: center;
          gap: 4px;
          .angel-symbol {
            color: $gold-primary;
            font-size: 0.85rem;
          }
          kbd {
            background: rgba(0, 0, 0, 0.04);
            border-radius: 4px;
            padding: 2px 6px;
            font-size: 0.7rem;
            font-family: monospace;
            border: 1px solid rgba($gold-primary, 0.15);
          }
        }
        button {
          background: linear-gradient(135deg, $gold-primary, $gold-light);
          border: none;
          padding: 8px 22px;
          border-radius: 30px;
          font-weight: 700;
          font-size: 0.9rem;
          color: $pure-white;
          cursor: pointer;
          transition: all 0.2s;
          font-family: inherit;
          letter-spacing: 1px;
          box-shadow: 0 2px 10px rgba(197, 160, 89, 0.3);
          &:hover:not(:disabled) {
            transform: translateY(-2px);
            box-shadow: 0 6px 16px rgba(197, 160, 89, 0.5);
          }
          &:active:not(:disabled) {
            transform: translateY(1px);
          }
          &:disabled {
            opacity: 0.5;
            cursor: not-allowed;
          }
        }
      }
    }
  }

  /* 响应式 */
  @media (max-width: 768px) {
    padding: 80px 0 140px;
    .carousel1 {
      display: none;
    }
    .carousel2 {
      display: block;
    }

    .board-header {
      margin-bottom: 24px;
      .header-tab {
        padding: 10px 20px;
        h1 {
          font-size: 1.3rem;
        }
      }
      .subtitle {
        font-size: 0.75rem;
      }
    }

    .message-list {
      .msg-grid {
        flex-direction: column;
        align-items: stretch;
      }
      .message-card {
        width: 100%;
        &.card-right {
          margin-top: 0;
        }
      }
    }

    .message-form {
      bottom: 16px;
      width: calc(100% - 16px);
      .form-inner {
        padding: 14px;
        .form-row .hint {
          display: none;
        }
      }
    }
  }

  .sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border: 0;
  }
}
</style>
