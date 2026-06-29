<template>
  <div class="origami-chat">
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

    <!-- 魔王风格叠加层（蓝黑裂纹 + 扫描线） -->
    <div class="bg-overlay">
      <div class="crack-lines">
        <span class="crack"></span><span class="crack"></span
        ><span class="crack"></span>
      </div>
      <div class="scanlines"></div>
    </div>

    <div class="chat-container">
      <!-- 统计面板 -->
      <div class="stats-panel">
        <div class="stats-header">
          <span class="stats-icon">👑</span>
          <span class="stats-title">档案记录</span>
        </div>
        <div class="stats-grid">
          <div class="stat-item">
            <span class="stat-label">总对话</span>
            <span class="stat-value">{{ stats.totalChats }}</span>
          </div>
          <div class="stat-item">
            <span class="stat-label">首次对话</span>
            <span class="stat-value">{{
              new Date(stats.firstTimestamp).toISOString().slice(0, 10)
            }}</span>
          </div>
          <div class="stat-item">
            <span class="stat-label">活跃天数</span>
            <span class="stat-value">{{ stats.activeDates.length }}</span>
          </div>
          <div class="stat-item">
            <span class="stat-label">今日对话</span>
            <span class="stat-value">{{
              stats.dailyChats[new Date().toISOString().slice(0, 10)] || 0
            }}</span>
          </div>
        </div>
        <div class="panel-buttons">
          <button class="detail-btn" @click="showModal = true">
            <span class="btn-icon">📋</span> 详细档案
          </button>
          <button class="settings-btn" @click="showSettingsModal = true">
            <span class="btn-icon">⚙️</span> 设定
          </button>
        </div>
      </div>

      <!-- 对话消息区 -->
      <div class="messages" ref="msgList">
        <transition-group name="msg" tag="div">
          <div
            v-for="msg in chatLog"
            :key="msg.id"
            :class="['message', msg.role, { error: msg.isError }]"
          >
            <div class="avatar" :class="msg.role"></div>
            <div class="bubble">
              <div class="content" v-html="msg.text"></div>
            </div>
          </div>
          <div v-if="loading" class="message bot" key="loading">
            <div class="avatar bot"></div>
            <div class="bubble loading">
              <span class="loading-text">黑暗中凝聚...</span>
              <span class="dots">
                <span class="dot">.</span><span class="dot">.</span
                ><span class="dot">.</span>
              </span>
            </div>
          </div>
        </transition-group>
      </div>

      <!-- 输入区 -->
      <form class="input-area" @submit.prevent="sendMessage">
        <textarea
          v-model="input"
          placeholder="向折纸倾诉..."
          :disabled="loading"
          @keydown="handleKeydown"
          rows="1"
        ></textarea>
        <div class="input-actions">
          <button
            type="button"
            class="clear-btn"
            @click="clearChat"
            :disabled="loading"
          >
            🗑️ 清空
          </button>
          <button
            type="submit"
            class="send-btn"
            :disabled="!input.trim() || loading"
          >
            {{ loading ? "发送中" : "发送" }}
          </button>
        </div>
      </form>
    </div>

    <!-- 详细统计弹窗 -->
    <div v-if="showModal" class="modal-overlay" @click.self="showModal = false">
      <div class="modal-content">
        <div class="modal-header">
          <span class="modal-icon">📁</span>
          <h3>档案详情</h3>
        </div>
        <ul class="detail-list">
          <li>
            <span>总对话次数</span><strong>{{ stats.totalChats }}</strong>
          </li>
          <li>
            <span>首次对话</span
            ><strong>{{
              new Date(stats.firstTimestamp).toISOString().slice(0, 10)
            }}</strong>
          </li>
          <li>
            <span>活跃天数</span
            ><strong>{{ stats.activeDates.length }} 天</strong>
          </li>
          <li>
            <span>今日对话</span
            ><strong
              >{{
                stats.dailyChats[new Date().toISOString().slice(0, 10)] || 0
              }}
              次</strong
            >
          </li>
          <li>
            <span>总对话时长</span
            ><strong>{{ formatDuration(stats.totalTime) }}</strong>
          </li>
          <li>
            <span>当前连续活跃</span
            ><strong>{{ stats.currentStreak }} 天</strong>
          </li>
          <li>
            <span>最长连续活跃</span
            ><strong>{{ stats.longestStreak }} 天</strong>
          </li>
          <li>
            <span>最活跃日</span
            ><strong
              >{{ mostActiveDayComputed }} ({{
                stats.dailyChats[mostActiveDayComputed] || 0
              }}次)</strong
            >
          </li>
        </ul>
        <button class="close-btn" @click="showModal = false">确认归档</button>
      </div>
    </div>

    <!-- 设定弹窗 -->
    <div
      v-if="showSettingsModal"
      class="modal-overlay"
      @click.self="showSettingsModal = false"
    >
      <div class="modal-content settings-modal">
        <div class="modal-header">
          <span class="modal-icon">⚙️</span>
          <h3>对话设定</h3>
        </div>
        <div class="settings-form">
          <div class="setting-item">
            <label>额外指令 <span class="optional">（临时覆盖）</span></label>
            <textarea
              v-model="tempAdditionalPrompt"
              placeholder="例如：与你对话的人是XXX、你和ta的关系是XXX……"
              rows="3"
              maxlength="200"
            ></textarea>
            <div class="char-counter">
              {{ tempAdditionalPrompt.length }} / 200 字符
            </div>
            <div class="hint">
              💡 追加在角色设定之后，优先级更高，可临时改变AI风格。
            </div>
          </div>
          <div class="setting-item">
            <label>温度 <span class="optional">（创造性）</span></label>
            <div class="temperature-control">
              <input
                type="range"
                v-model.number="tempTemperature"
                min="0.1"
                max="1.9"
                step="0.05"
              />
              <span class="temp-value">{{ tempTemperature.toFixed(2) }}</span>
            </div>
            <div class="hint">
              🌡️ 温度越高（接近2），回答越富有创造性和随机性；<br />🌡️
              温度越低（接近0），回答越确定、保守。建议范围0.5~1.2。
            </div>
          </div>
        </div>
        <div class="modal-actions">
          <button class="cancel-btn" @click="showSettingsModal = false">
            取消
          </button>
          <button class="save-btn" @click="saveSettings">保存设定</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  reactive,
  ref,
  computed,
  onMounted,
  nextTick,
  watch,
  onBeforeUnmount,
} from "vue";
import { sendMessageToHui } from "@/api/deepseekApi";

const STORAGE_KEY = "origami_chat_log";
const STORAGE_STATS_KEY = "origami_chat_stats";
const STORAGE_SETTINGS_KEY = "origami_chat_settings";

interface Stats {
  firstTimestamp: number;
  totalChats: number;
  activeDates: string[];
  dailyChats: Record<string, number>;
  currentStreak: number;
  longestStreak: number;
  totalTime: number;
}

const defaultStats: Stats = {
  firstTimestamp: Date.now(),
  totalChats: 0,
  activeDates: [],
  dailyChats: {},
  currentStreak: 0,
  longestStreak: 0,
  totalTime: 0,
};

function loadStats(): Stats {
  const saved = localStorage.getItem(STORAGE_STATS_KEY);
  if (saved) {
    try {
      return { ...defaultStats, ...JSON.parse(saved) };
    } catch {}
  }
  return { ...defaultStats };
}
function saveStats() {
  localStorage.setItem(STORAGE_STATS_KEY, JSON.stringify(stats));
}

function updateActive(date: string) {
  if (!stats.activeDates.includes(date)) {
    stats.activeDates.push(date);
    updateStreak();
    saveStats();
  }
}
function updateStreak() {
  const dates = [...stats.activeDates].sort();
  let curr = 0,
    max = stats.longestStreak,
    prevTs = 0;
  const todayStr = new Date().toISOString().slice(0, 10);
  dates.forEach((d) => {
    const ts = new Date(d).getTime();
    if (prevTs && ts - prevTs === 86400000) curr++;
    else curr = 1;
    max = Math.max(max, curr);
    prevTs = ts;
  });
  stats.currentStreak = dates[dates.length - 1] === todayStr ? curr : 0;
  stats.longestStreak = max;
  saveStats();
}
function updateDaily(date: string) {
  stats.dailyChats[date] = (stats.dailyChats[date] || 0) + 1;
  saveStats();
}

const mostActiveDayComputed = computed(() => {
  let day = "",
    max = 0;
  for (const [d, c] of Object.entries(stats.dailyChats)) {
    if (c > max) {
      max = c;
      day = d;
    }
  }
  return day || new Date().toISOString().slice(0, 10);
});
function formatDuration(ms: number): string {
  const min = Math.floor(ms / 60000);
  const h = Math.floor(min / 60);
  const m = min % 60;
  return h ? `${h} 小时 ${m} 分钟` : `${m} 分钟`;
}

const stats = reactive<Stats>(loadStats());
const sessionStart = Date.now();

interface ChatMsg {
  id: number;
  role: "user" | "bot";
  text: string;
  isError?: boolean;
}
const chatLog = ref<ChatMsg[]>(loadChatLog());
const input = ref("");
const loading = ref(false);
const msgList = ref<HTMLElement>();

const showModal = ref(false);
const showSettingsModal = ref(false);
const tempAdditionalPrompt = ref("");
const tempTemperature = ref(0.7);
const currentAdditionalPrompt = ref("");
const currentTemperature = ref(0.7);

function loadSettings() {
  const saved = localStorage.getItem(STORAGE_SETTINGS_KEY);
  if (saved) {
    try {
      const s = JSON.parse(saved);
      currentAdditionalPrompt.value = s.additionalPrompt || "";
      currentTemperature.value = s.temperature ?? 0.7;
      tempAdditionalPrompt.value = currentAdditionalPrompt.value;
      tempTemperature.value = currentTemperature.value;
    } catch {}
  }
}

function saveSettings() {
  if (tempAdditionalPrompt.value.length > 200) {
    alert("额外指令不能超过200个字符");
    return;
  }
  currentAdditionalPrompt.value = tempAdditionalPrompt.value;
  currentTemperature.value = tempTemperature.value;
  localStorage.setItem(
    STORAGE_SETTINGS_KEY,
    JSON.stringify({
      additionalPrompt: currentAdditionalPrompt.value,
      temperature: currentTemperature.value,
    })
  );
  showSettingsModal.value = false;
}

async function sendMessage() {
  if (!input.value.trim()) return;
  if (stats.totalChats === 0 && !localStorage.getItem(STORAGE_STATS_KEY)) {
    stats.firstTimestamp = Date.now();
    saveStats();
  }
  const date = new Date().toISOString().slice(0, 10);
  stats.totalChats++;
  updateActive(date);
  updateDaily(date);
  saveStats();

  const userText = input.value;
  chatLog.value.push({ id: Date.now(), role: "user", text: userText });
  input.value = "";
  loading.value = true;

  try {
    const history = chatLog.value.filter((msg) => !msg.isError);
    const options: any = { character: "origami" };
    if (currentAdditionalPrompt.value.trim()) {
      options.additionalPrompt = currentAdditionalPrompt.value.trim();
    }
    if (currentTemperature.value != null) {
      options.temperature = currentTemperature.value;
    }
    const botReply = await sendMessageToHui(userText, history, options);
    if (botReply === "error") {
      chatLog.value.push({
        id: Date.now() + 2,
        role: "bot",
        text: "出错啦，清空对话再试试如果还不行就在b站私信我吧",
        isError: true,
      });
    } else {
      chatLog.value.push({ id: Date.now() + 1, role: "bot", text: botReply });
    }
  } catch (e) {
    console.error(e);
  } finally {
    loading.value = false;
    await scrollToBottom();
  }
}

function handleKeydown(e: KeyboardEvent) {
  if (e.key === "Enter") sendMessage();
}

function clearChat() {
  if (confirm("确定要清空全部对话吗？")) {
    chatLog.value = [
      {
        id: Date.now(),
        role: "bot",
        text: "…你的话语，我在倾听",
      },
    ];
    localStorage.removeItem(STORAGE_KEY);
  }
}

function loadChatLog(): ChatMsg[] {
  const saved = localStorage.getItem(STORAGE_KEY);
  if (saved) {
    try {
      return JSON.parse(saved);
    } catch {}
  }
  return [
    {
      id: Date.now(),
      role: "bot",
      text: "…你的话语，我在倾听",
    },
  ];
}

async function scrollToBottom() {
  await nextTick();
  if (msgList.value) msgList.value.scrollTop = msgList.value.scrollHeight;
}

watch(
  chatLog,
  async () => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(chatLog.value));
    await scrollToBottom();
  },
  { deep: true }
);

function handleBeforeUnload() {
  stats.totalTime += Date.now() - sessionStart;
  saveStats();
}

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
  scrollToBottom();
  window.addEventListener("beforeunload", handleBeforeUnload);
  Imgtimer = window.setInterval(() => {
    currentIndex.value =
      (currentIndex.value + 1) % Math.max(1, randomFive.value.length);
  }, 5200);
  loadSettings();
});

onBeforeUnmount(() => {
  window.removeEventListener("beforeunload", handleBeforeUnload);
  if (Imgtimer) clearInterval(Imgtimer);
});
</script>

<style scoped lang="scss">
/* 蓝黑丧服配色（完全移除暗红紫） */
$bg-deep: #0a0d14;
$panel-bg: rgba(15, 18, 26, 0.9);
$blue-black: #1a2330;
$ice-blue: #5b7b99; // 冰蓝强调
$ice-light: #7a9bb5; // 冰蓝亮光
$pale-blue: #c4d2e0; // 浅蓝灰文字
$dim-blue: #8e9daf; // 辅助文字
$gold: #b89b5e; // 暗金
$shadow-heavy: 0 10px 30px rgba(0, 0, 0, 0.7);

.origami-chat {
  position: relative;
  min-height: 100vh;
  padding-top: 64px;
  font-family: "Noto Sans SC", "Inter", sans-serif;
  color: $pale-blue;
  display: flex;
  flex-direction: column;
  background: $bg-deep;
  overflow-x: hidden;

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
      filter: brightness(0.4) saturate(0.6);
      &.active {
        opacity: 0.65;
      }
    }
  }
  .carousel2 {
    display: none;
  }

  .bg-overlay {
    position: fixed;
    inset: 0;
    z-index: 1;
    pointer-events: none;
    background: radial-gradient(
      circle at 30% 50%,
      rgba($ice-blue, 0.1),
      transparent 60%
    );
    .crack-lines {
      .crack {
        position: absolute;
        height: 1px;
        background: linear-gradient(
          90deg,
          transparent,
          $ice-light,
          transparent
        );
        opacity: 0;
        animation: crack-flicker 4s infinite;
      }
      .crack:nth-child(1) {
        top: 25%;
        left: 5%;
        width: 320px;
        transform: rotate(8deg);
      }
      .crack:nth-child(2) {
        top: 65%;
        left: 55%;
        width: 240px;
        transform: rotate(-10deg);
        animation-delay: 1.2s;
      }
      .crack:nth-child(3) {
        top: 85%;
        left: 15%;
        width: 280px;
        transform: rotate(4deg);
        animation-delay: 2.4s;
      }
    }
    .scanlines {
      position: absolute;
      inset: 0;
      background: repeating-linear-gradient(
        0deg,
        transparent,
        transparent 3px,
        rgba($ice-blue, 0.04) 3px,
        rgba($ice-blue, 0.04) 6px
      );
      animation: scan 8s linear infinite;
    }
  }

  @keyframes crack-flicker {
    0%,
    100% {
      opacity: 0.1;
    }
    50% {
      opacity: 0.55;
      box-shadow: 0 0 12px $ice-light;
    }
  }
  @keyframes scan {
    from {
      background-position: 0 0;
    }
    to {
      background-position: 0 12px;
    }
  }

  .chat-container {
    position: relative;
    z-index: 10;
    flex: 1;
    width: 920px;
    max-width: calc(100% - 32px);
    margin: 0 auto;
    padding: 24px;
    display: flex;
    flex-direction: column;
    gap: 20px;
  }

  .stats-panel {
    background: $panel-bg;
    backdrop-filter: blur(12px);
    border-radius: 16px;
    padding: 18px 24px;
    border: 1px solid rgba($ice-blue, 0.3);
    box-shadow: $shadow-heavy;
    display: flex;
    align-items: center;
    justify-content: space-between;

    .stats-header {
      display: flex;
      align-items: center;
      gap: 8px;
      .stats-icon {
        font-size: 22px;
      }
      .stats-title {
        font-weight: 700;
        color: $ice-light;
        letter-spacing: 2px;
        font-size: 1.1rem;
      }
    }
    .stats-grid {
      display: flex;
      gap: 28px;
      .stat-item {
        text-align: center;
        .stat-label {
          display: block;
          font-size: 0.8rem;
          color: $dim-blue;
        }
        .stat-value {
          display: block;
          font-size: 1.3rem;
          font-weight: 700;
          color: $gold;
        }
      }
    }
    .panel-buttons {
      display: flex;
      gap: 12px;
      .detail-btn,
      .settings-btn {
        display: flex;
        align-items: center;
        gap: 6px;
        background: rgba($ice-blue, 0.15);
        border: 1px solid rgba($ice-blue, 0.4);
        border-radius: 30px;
        padding: 8px 18px;
        color: $pale-blue;
        font-weight: 600;
        font-size: 0.9rem;
        cursor: pointer;
        transition: all 0.2s;
        .btn-icon {
          font-size: 16px;
        }
        &:hover {
          background: rgba($ice-blue, 0.3);
          border-color: $ice-light;
          transform: translateY(-2px);
        }
      }
    }
  }

  .messages {
    flex: 1;
    overflow-y: auto;
    padding: 12px 0 20px;
    max-height: calc(100vh - 320px);
    display: flex;
    flex-direction: column;
    gap: 20px;
  }

  .message {
    display: flex;
    align-items: flex-start;
    gap: 12px;
    &.user {
      flex-direction: row-reverse;
    }

    .avatar {
      width: 48px;
      height: 48px;
      border-radius: 14px;
      background-size: cover;
      background-position: center;
      flex-shrink: 0;
      border: 2px solid rgba($ice-blue, 0.3);
      box-shadow: 0 8px 20px rgba(0, 0, 0, 0.4);

      &.bot {
        background-image: url("@/assets/avatar/avatar (1).webp");
        border-color: $gold;
      }
      &.user {
        background: rgba($ice-blue, 0.2);
        border-color: $ice-light;
        display: flex;
        align-items: center;
        justify-content: center;
        font-weight: bold;
        color: $pale-blue;
        font-size: 1.1rem;
        &::after {
          content: "你";
        }
      }
    }

    .bubble {
      max-width: 75%;
      padding: 16px 20px;
      background: $panel-bg;
      backdrop-filter: blur(8px);
      border: 1px solid rgba($ice-blue, 0.3);
      border-radius: 18px;
      line-height: 1.7;
      word-break: break-word;
      box-shadow: 0 8px 24px rgba(0, 0, 0, 0.4);
      position: relative;

      .message.user & {
        border-right: 3px solid $ice-light;
        background: linear-gradient(135deg, rgba($ice-blue, 0.1), $panel-bg);
        border-radius: 18px 6px 18px 18px;
      }
      .message.bot & {
        border-left: 3px solid $gold;
        background: linear-gradient(135deg, $panel-bg, rgba($gold, 0.05));
        border-radius: 6px 18px 18px 18px;
      }
      &.error {
        border-color: $ice-light;
        background: rgba($ice-blue, 0.1);
      }
      &.loading {
        .loading-text {
          color: $dim-blue;
          font-size: 1rem;
        }
        .dots .dot {
          animation: dotPulse 1.4s infinite;
          color: $ice-light;
          &:nth-child(2) {
            animation-delay: 0.2s;
          }
          &:nth-child(3) {
            animation-delay: 0.4s;
          }
        }
      }
      .content {
        color: $pale-blue;
        font-size: 1.05rem;
      }
    }
  }

  .input-area {
    background: $panel-bg;
    backdrop-filter: blur(12px);
    border-radius: 20px;
    padding: 16px 20px;
    border: 1px solid rgba($ice-blue, 0.3);
    box-shadow: $shadow-heavy;
    display: flex;
    align-items: flex-end;
    gap: 16px;

    textarea {
      flex: 1;
      background: rgba(0, 0, 0, 0.5);
      border: 1px solid rgba($ice-blue, 0.3);
      border-radius: 16px;
      padding: 14px 18px;
      color: $pale-blue;
      font-size: 1rem;
      line-height: 1.5;
      outline: none;
      resize: none;
      min-height: 56px;
      max-height: 150px;
      font-family: inherit;
      &::placeholder {
        color: rgba($dim-blue, 0.7);
      }
      &:focus {
        border-color: $ice-light;
      }
    }

    .input-actions {
      display: flex;
      align-items: center;
      gap: 12px;

      .clear-btn {
        background: transparent;
        border: 1px solid rgba($ice-blue, 0.3);
        border-radius: 14px;
        width: 44px;
        height: 44px;
        display: flex;
        align-items: center;
        justify-content: center;
        cursor: pointer;
        color: $dim-blue;
        transition: all 0.2s;
        &:hover {
          border-color: $ice-light;
          color: $ice-light;
        }
      }

      .send-btn {
        padding: 0 28px;
        height: 44px;
        background: linear-gradient(135deg, $ice-blue, $ice-light);
        border: none;
        border-radius: 14px;
        font-weight: 700;
        color: white;
        font-size: 1rem;
        cursor: pointer;
        transition: all 0.2s;
        &:hover:not(:disabled) {
          transform: translateY(-2px);
          box-shadow: 0 8px 20px rgba($ice-blue, 0.5);
        }
        &:disabled {
          opacity: 0.5;
          cursor: not-allowed;
        }
      }
    }
  }

  /* 弹窗 */
  .modal-overlay {
    position: fixed;
    inset: 0;
    z-index: 1000;
    background: rgba(0, 0, 0, 0.8);
    backdrop-filter: blur(8px);
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 20px;
    .modal-content {
      width: 480px;
      max-width: 100%;
      background: $panel-bg;
      border-radius: 24px;
      border: 1px solid rgba($ice-blue, 0.4);
      box-shadow: $shadow-heavy;
      padding: 28px;
      .modal-header {
        display: flex;
        align-items: center;
        gap: 10px;
        margin-bottom: 24px;
        padding-bottom: 16px;
        border-bottom: 1px solid rgba($ice-blue, 0.3);
        .modal-icon {
          font-size: 24px;
        }
        h3 {
          margin: 0;
          color: $ice-light;
          font-weight: 700;
          font-size: 1.2rem;
        }
      }
    }
  }

  .detail-list {
    list-style: none;
    margin: 0 0 24px;
    padding: 0;
    li {
      display: flex;
      justify-content: space-between;
      padding: 10px 0;
      border-bottom: 1px dashed rgba($ice-blue, 0.2);
      font-size: 1rem;
      span {
        color: $dim-blue;
      }
      strong {
        color: $gold;
      }
    }
  }

  .settings-modal {
    .settings-form {
      margin-bottom: 28px;
      .setting-item {
        margin-bottom: 28px;
        label {
          display: block;
          font-weight: 600;
          color: $ice-light;
          margin-bottom: 8px;
          font-size: 1rem;
          .optional {
            font-size: 0.85rem;
            color: $dim-blue;
            font-weight: normal;
          }
        }
        textarea {
          width: 100%;
          background: rgba(0, 0, 0, 0.5);
          border: 1px solid rgba($ice-blue, 0.3);
          border-radius: 12px;
          padding: 12px;
          color: $pale-blue;
          font-size: 0.95rem;
          resize: vertical;
        }
        .char-counter {
          text-align: right;
          font-size: 0.8rem;
          color: $dim-blue;
          margin-top: 4px;
        }
        .temperature-control {
          display: flex;
          align-items: center;
          gap: 16px;
          input[type="range"] {
            flex: 1;
            height: 4px;
            background: $dim-blue;
            border-radius: 2px;
          }
          .temp-value {
            color: $gold;
            font-weight: 700;
            min-width: 40px;
          }
        }
      }
    }
    .modal-actions {
      display: flex;
      justify-content: flex-end;
      gap: 12px;
      button {
        padding: 10px 24px;
        border-radius: 30px;
        font-weight: 600;
        cursor: pointer;
        font-size: 0.95rem;
        &.cancel-btn {
          background: rgba($ice-blue, 0.15);
          color: $dim-blue;
          &:hover {
            background: rgba($ice-blue, 0.3);
            color: $pale-blue;
          }
        }
        &.save-btn {
          background: linear-gradient(135deg, $ice-blue, $ice-light);
          color: white;
          &:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 12px rgba($ice-blue, 0.5);
          }
        }
      }
    }
  }

  .close-btn {
    width: 100%;
    padding: 14px;
    background: linear-gradient(135deg, $ice-blue, $ice-light);
    border: none;
    border-radius: 14px;
    font-weight: 700;
    color: white;
    font-size: 1rem;
    cursor: pointer;
    transition: all 0.2s;
    &:hover {
      transform: translateY(-2px);
      box-shadow: 0 8px 20px rgba($ice-blue, 0.5);
    }
  }

  @keyframes dotPulse {
    0%,
    100% {
      opacity: 0.2;
    }
    50% {
      opacity: 1;
    }
  }
  .msg-enter-active,
  .msg-leave-active {
    transition: all 0.3s ease;
  }
  .msg-enter-from {
    opacity: 0;
    transform: translateY(10px);
  }
  .msg-leave-to {
    opacity: 0;
    transform: translateY(-10px);
  }

  @media (max-width: 768px) {
    .carousel1 {
      display: none;
    }
    .carousel2 {
      display: block;
    }
    .stats-panel .stats-grid {
      display: none;
    }
    .detail-btn span:last-child,
    .settings-btn span:last-child {
      display: none;
    }
    .input-area {
      flex-direction: column;
    }
    .input-actions {
      width: 100%;
      justify-content: space-between;
    }
    .message .bubble {
      max-width: 85%;
    }
  }
}
</style>
