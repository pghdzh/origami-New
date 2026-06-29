<template>
  <div class="origami-wiki">
    <!-- 背景轮播（保留原有图片） -->
    <div class="carousel" aria-hidden="true">
      <img
        v-for="(src, idx) in randomFive"
        :key="idx"
        :src="src"
        class="carousel-image"
        :class="{ active: idx === currentIndex }"
      />
    </div>
    <!-- 天使叠加层：柔和白色渐变 + 光点纹理 -->
    <div class="bg-overlay">
      <div class="sparkle-dots"></div>
    </div>

    <!-- 头部标题区 -->
    <header class="wiki-header">
      <div class="title-area">
        <span class="sigil">✦</span>
        <h1 class="bigTitle">文本分享</h1>
        <p class="subtitle">真正的爱，从现在才要开始。</p>
      </div>
      <div class="actions">
        <input v-model="search" class="search" placeholder="搜索标题或标签…" />
        <button class="btn btn-new" @click="openCreate">✦ 新建词条</button>
      </div>
    </header>

    <!-- 词条列表 -->
    <main class="wiki-body">
      <div v-if="filteredEntries.length === 0" class="empty">
        没有找到匹配的词条 ✦
      </div>
      <ul class="entry-list">
        <li v-for="entry in filteredEntries" :key="entry.id" class="entry-card">
          <div class="card-left">
            <div class="entry-meta" @click="openDetail(entry)">
              <h2 class="entry-title">{{ entry.title }}</h2>
              <div class="entry-tags">
                <span class="badge">#{{ entry.slug || "未设置" }}</span>
                <span class="author-badge">✎ {{ entry.author }}</span>
                <span class="time-badge"
                  >⏳ {{ formatTime(entry.updatedAt) }}</span
                >
              </div>
            </div>
          </div>
          <div class="card-right">
            <button
              class="like-btn"
              :class="{ liked: isLiked(entry.id) }"
              @click.stop="toggleLike(entry.id)"
            >
              <img
                :src="
                  isLiked(entry.id)
                    ? '/icons/heart-red-filled.svg'
                    : '/icons/heart-red-outline.svg'
                "
                alt="like"
              />
              <span>{{ entry.likes }}</span>
            </button>
            <div class="edit-delete" v-if="canEdit(entry.id)">
              <button
                class="icon-btn"
                @click.stop="openEdit(entry)"
                title="编辑"
              >
                ✎
              </button>
              <button
                class="icon-btn danger"
                @click.stop="remove(entry.id)"
                title="删除"
              >
                ✕
              </button>
            </div>
          </div>
        </li>
      </ul>
    </main>

    <!-- 新建/编辑弹窗 -->
    <transition name="modal-fade">
      <div class="modal-overlay" v-if="showModal" @click.self="closeModal">
        <div class="modal-content">
          <div class="modal-header">
            <h3>{{ editing ? "编辑词条" : "新建词条" }}</h3>
            <button class="close-btn" @click="closeModal">✕</button>
          </div>
          <div class="modal-body">
            <label>
              标题
              <input v-model="form.title" placeholder="请输入标题" />
            </label>
            <label>
              词条（短标签）
              <input v-model="form.slug" placeholder="例如：二创小说、AI人设，AI对话记录" />
            </label>
            <label>
              作者
              <input v-model="form.author" placeholder="作者昵称" />
            </label>
            <label>
              内容
              <textarea
                v-model="form.content"
                rows="6"
                placeholder="请输入词条内容"
              ></textarea>
            </label>
          </div>
          <div class="modal-footer">
            <button class="btn cancel-btn" @click="closeModal">取消</button>
            <button class="btn save-btn" @click="submit">
              {{ editing ? "保存" : "创建" }}
            </button>
          </div>
        </div>
      </div>
    </transition>

    <!-- 详情弹窗 -->
    <transition name="modal-fade">
      <div
        class="modal-overlay"
        v-if="detailEntry"
        @click.self="detailEntry = null"
      >
        <div class="modal-content detail-modal">
          <div class="modal-header">
            <h3>{{ detailEntry.title }}</h3>
            <button class="close-btn" @click="detailEntry = null">✕</button>
          </div>
          <div class="modal-body">
            <div class="detail-text">{{ detailEntry.content }}</div>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed, onMounted, onUnmounted } from "vue";
import { ElMessage } from "element-plus";
import {
  getWikiList,
  createWiki,
  updateWiki,
  deleteWiki,
  likeWiki,
} from "@/api/modules/wiki";

// 本地存储自己创建的词条ID
const LS_MY_WIKI_IDS = "origami:wiki:my_ids";
const myWikiIds: string[] = JSON.parse(
  localStorage.getItem(LS_MY_WIKI_IDS) || "[]"
);
const markAsMine = (id: string | number) => {
  if (!myWikiIds.includes(String(id))) {
    myWikiIds.push(String(id));
    localStorage.setItem(LS_MY_WIKI_IDS, JSON.stringify(myWikiIds));
  }
};
const canEdit = (id: string | number) => myWikiIds.includes(String(id));

const entries = ref<any[]>([]);
const LS_LIKED_IDS = "origami:wiki:liked_ids";
const likedIds = ref<string[]>(
  JSON.parse(localStorage.getItem(LS_LIKED_IDS) || "[]")
);

const showModal = ref(false);
const editing = ref(false);
const editingId = ref<string | number | null>(null);
const detailEntry = ref<any>(null);
const form = reactive({ title: "", slug: "", author: "", content: "" });
const search = ref("");

function formatTime(ts: string | number | null | undefined) {
  if (!ts) return "未知时间";
  const date = new Date(ts);
  if (isNaN(date.getTime())) return "未知时间";
  return `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(
    2,
    "0"
  )}-${String(date.getDate()).padStart(2, "0")}`;
}

async function loadEntries() {
  try {
    const res: any = await getWikiList();
    entries.value = res.data.map((e: any) => ({
      ...e,
      createdAt: e.createdAt || e.created_at,
      updatedAt: e.updatedAt || e.updated_at,
    }));
  } catch (err) {
    console.error(err);
    ElMessage.error("加载词条失败");
  }
}

function openCreate() {
  editing.value = false;
  editingId.value = null;
  form.title = "";
  form.slug = "";
  form.author = "";
  form.content = "";
  showModal.value = true;
}
function openEdit(entry: any) {
  if (!canEdit(entry.id)) {
    ElMessage.warning("只有创建者可以编辑");
    return;
  }
  editing.value = true;
  editingId.value = entry.id;
  form.title = entry.title;
  form.slug = entry.slug;
  form.author = entry.author;
  form.content = entry.content;
  showModal.value = true;
}
function closeModal() {
  showModal.value = false;
}
const canSubmit = computed(() => form.title.trim() && form.content.trim());

async function submit() {
  if (!canSubmit.value) {
    ElMessage.warning("请填写标题和内容");
    return;
  }
  const payload: any = {
    title: form.title.trim(),
    author: form.author.trim() || "匿名",
    content: form.content.trim(),
  };
  if (form.slug.trim()) payload.slug = form.slug.trim();
  try {
    if (editing.value && editingId.value) {
      await updateWiki(editingId.value, payload);
      ElMessage.success("编辑成功");
    } else {
      const res: any = await createWiki(payload);
      markAsMine(res.data.id);
      editingId.value = res.data.id;
      ElMessage.success("创建成功");
    }
    showModal.value = false;
    loadEntries();
  } catch (err) {
    console.error(err);
    ElMessage.error("提交失败");
  }
}

async function remove(id: string | number) {
  if (!canEdit(id)) {
    ElMessage.warning("只有创建者可以删除");
    return;
  }
  if (!confirm("确认删除该词条？此操作不可撤销")) return;
  try {
    await deleteWiki(id);
    const index = myWikiIds.indexOf(String(id));
    if (index !== -1) myWikiIds.splice(index, 1);
    localStorage.setItem(LS_MY_WIKI_IDS, JSON.stringify(myWikiIds));
    ElMessage.success("删除成功");
    loadEntries();
  } catch (err) {
    console.error(err);
    ElMessage.error("删除失败");
  }
}

function persistLikedIds() {
  localStorage.setItem(LS_LIKED_IDS, JSON.stringify(likedIds.value));
}
function isLiked(id: string | number) {
  return likedIds.value.includes(String(id));
}

async function toggleLike(id: string | number) {
  const entry = entries.value.find((e) => e.id === id);
  if (!entry) return;
  const idStr = String(id);
  const wasLiked = likedIds.value.includes(idStr);
  if (wasLiked) {
    entry.likes = Math.max(0, (entry.likes || 0) - 1);
    likedIds.value = likedIds.value.filter((x) => x !== idStr);
  } else {
    entry.likes = (entry.likes || 0) + 1;
    likedIds.value.push(idStr);
  }
  persistLikedIds();
  try {
    const action = wasLiked ? "unlike" : "like";
    await likeWiki(id, action);
  } catch (err) {
    if (wasLiked) {
      entry.likes = (entry.likes || 0) + 1;
      if (!likedIds.value.includes(idStr)) likedIds.value.push(idStr);
    } else {
      entry.likes = Math.max(0, (entry.likes || 0) - 1);
      likedIds.value = likedIds.value.filter((x) => x !== idStr);
    }
    persistLikedIds();
    ElMessage.error("点赞失败，请稍后重试");
  }
}

function openDetail(entry: any) {
  detailEntry.value = entry;
}

const filteredEntries = computed(() => {
  const q = String(search.value || "")
    .trim()
    .toLowerCase();
  let list = entries.value;
  if (q) {
    list = list.filter(
      (e) =>
        (e.title || "").toLowerCase().includes(q) ||
        (e.slug || "").toLowerCase().includes(q)
    );
  }
  return [...list].sort((a, b) => (b.likes || 0) - (a.likes || 0));
});

// 背景轮播
const pcModules = import.meta.glob("@/assets/images1/*.{jpg,png,jpeg,webp}", {
  eager: true,
});
const mobileModules = import.meta.glob(
  "@/assets/images2/*.{jpg,png,jpeg,webp}",
  { eager: true }
);
const pcSrcs: string[] = Object.values(pcModules).map((m: any) => m.default);
const mobileSrcs: string[] = Object.values(mobileModules).map(
  (m: any) => m.default
);
function shuffle<T>(arr: T[]): T[] {
  const a = arr.slice();
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}
const randomFive = ref<string[]>([]);
const currentIndex = ref(0);
let timer: number;
function pickImages() {
  const isMobile = window.innerWidth < 768;
  const all = isMobile ? mobileSrcs : pcSrcs;
  randomFive.value = shuffle(all).slice(0, 5);
}

onMounted(() => {
  loadEntries();
  pickImages();
  window.addEventListener("resize", pickImages);
  timer = window.setInterval(() => {
    if (randomFive.value.length > 0)
      currentIndex.value = (currentIndex.value + 1) % randomFive.value.length;
  }, 5000);
});
onUnmounted(() => {
  clearInterval(timer);
  window.removeEventListener("resize", pickImages);
});
</script>

<style scoped lang="scss">
// 天使配色
$bg: #f5f8fc;
$white: #ffffff;
$gold: #c5a059;
$gold-light: #e8cd7a;
$blue: #5b9bd5;
$blue-light: #8ec5fc;
$text: #333333;
$text-dim: #5a6b7e;
$shadow-soft: 0 8px 30px rgba(44, 58, 74, 0.08);

.origami-wiki {
  position: relative;
  min-height: 100vh;
  padding: 60px 140px;
  background: $bg;
  color: $text;
  font-family: "Noto Sans SC", "Inter", sans-serif;
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
      filter: saturate(1) brightness(0.7);
      &.active {
        opacity: 1;
      }
    }
  }



  @keyframes float-dots {
    from {
      background-position: 0 0, 0 0;
    }
    to {
      background-position: 100px 100px, -50px 50px;
    }
  }

  .wiki-header {
    position: relative;
    z-index: 10;
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: 20px;
    margin-bottom: 40px;
    .title-area {
      text-align: left;
      .sigil {
        font-size: 32px;
        color: $gold;
        filter: drop-shadow(0 0 6px rgba($gold, 0.4));
        display: inline-block;
        margin-bottom: 6px;
      }
      h1 {
        margin: 0;
        font-size: 2rem;
        font-weight: 800;
        color: $text;
        font-family: "Noto Serif SC", serif;
        letter-spacing: 2px;
        color: #fff;
      }
      .subtitle {
        margin: 8px 0 0;
        font-size: 0.95rem;
        color: $gold-light;
        font-style: italic;
        letter-spacing: 1px;
      }
    }
    .actions {
      display: flex;
      gap: 10px;
      align-items: center;
      .search {
        padding: 10px 18px;
        border-radius: 30px;
        border: 1px solid rgba($gold, 0.3);
        background: rgba($white, 0.7);
        backdrop-filter: blur(2px);
        color: $text;
        font-size: 0.95rem;
        outline: none;
        transition: 0.2s;
        &::placeholder {
          color: rgba($text-dim, 0.5);
        }
        &:focus {
          border-color: $gold;
          box-shadow: 0 0 8px rgba($gold, 0.2);
        }
      }
      .btn-new {
        background: linear-gradient(135deg, $gold, $gold-light);
        border: none;
        border-radius: 30px;
        padding: 10px 24px;
        font-weight: 700;
        color: $white;
        cursor: pointer;
        box-shadow: 0 4px 12px rgba($gold, 0.3);
        transition: all 0.2s;
        font-size: 0.95rem;
        &:hover {
          transform: translateY(-2px);
          box-shadow: 0 6px 18px rgba($gold, 0.5);
        }
      }
    }
  }

  .wiki-body {
    position: relative;
    z-index: 10;
    .empty {
      text-align: center;
      padding: 80px 20px;
      color: $gold-light;
      font-style: italic;
    }
    .entry-list {
      list-style: none;
      padding: 0;
      display: flex;
      flex-direction: column;
      gap: 16px;
    }

    .entry-card {
      display: flex;
      align-items: center;
      justify-content: space-between;
      background: rgba($white, 0.75);
      backdrop-filter: blur(2px);
      border-radius: 20px;
      padding: 18px 24px;
      border: 1px solid rgba($gold, 0.15);
      box-shadow: $shadow-soft;
      transition: transform 0.3s, box-shadow 0.3s, border-color 0.3s;
      border-left: 5px solid transparent;
      &:hover {
        transform: translateY(-4px);
        box-shadow: 0 12px 30px rgba(44, 58, 74, 0.1);
        border-left-color: $gold;
      }

      .card-left {
        flex: 1;
        cursor: pointer;
        .entry-title {
          font-size: 1.2rem;
          font-weight: 700;
          color: $text;
          margin: 0 0 8px;
        }
        .entry-tags {
          display: flex;
          flex-wrap: wrap;
          gap: 8px;
          .badge {
            background: rgba($gold, 0.15);
            color: $gold;
            padding: 3px 10px;
            border-radius: 12px;
            font-size: 0.8rem;
            font-weight: 600;
          }
          .author-badge,
          .time-badge {
            color: $text-dim;
            font-size: 0.8rem;
            display: flex;
            align-items: center;
            gap: 4px;
          }
        }
      }

      .card-right {
        display: flex;
        align-items: center;
        gap: 14px;
        .like-btn {
          display: flex;
          align-items: center;
          gap: 6px;
          background: none;
          border: none;
          cursor: pointer;
          color: $text-dim;
          font-weight: 600;
          padding: 6px 12px;
          border-radius: 20px;
          transition: 0.2s;
          img {
            width: 20px;
            height: 20px;
          }
          &.liked {
            color: #c94a3a;
          }
          &:hover {
            background: rgba($gold, 0.1);
          }
        }
        .icon-btn {
          background: none;
          border: 1px solid rgba($gold, 0.2);
          border-radius: 8px;
          width: 32px;
          height: 32px;
          display: flex;
          align-items: center;
          justify-content: center;
          cursor: pointer;
          color: $text-dim;
          transition: 0.2s;
          &:hover {
            border-color: $gold;
            color: $gold;
          }
          &.danger:hover {
            border-color: #c94a3a;
            color: #c94a3a;
          }
        }
      }
    }
  }

  // 弹窗
  .modal-overlay {
    position: fixed;
    inset: 0;
    z-index: 1000;
    background: rgba(0, 0, 0, 0.2);
    backdrop-filter: blur(6px);
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 20px;
  }
  .modal-content {
    width: 600px;
    max-width: 100%;
    background: rgba(255, 255, 255, 0.9);
    backdrop-filter: blur(20px);
    border-radius: 24px;
    border: 1px solid rgba($gold, 0.3);
    box-shadow: 0 20px 50px rgba(0, 0, 0, 0.1);
    padding: 28px;
    .modal-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 20px;
      h3 {
        margin: 0;
        color: $text;
        font-family: "Noto Serif SC", serif;
      }
      .close-btn {
        background: none;
        border: none;
        font-size: 1.5rem;
        color: $text-dim;
        cursor: pointer;
        &:hover {
          color: $gold;
        }
      }
    }
    .modal-body {
      .detail-text{
        white-space: pre-wrap;
      }
      label {
        display: block;
        font-weight: 600;
        color: $text-dim;
        margin-bottom: 14px;
        input,
        textarea {
          display: block;
          width: 100%;
          margin-top: 6px;
          padding: 10px 14px;
          border-radius: 12px;
          border: 1px solid rgba($gold, 0.3);
          background: rgba($white, 0.8);
          color: $text;
          font-size: 0.95rem;
          outline: none;
          transition: 0.2s;
          &:focus {
            border-color: $gold;
            box-shadow: 0 0 6px rgba($gold, 0.2);
          }
        }
      }
    }
    .modal-footer {
      display: flex;
      justify-content: flex-end;
      gap: 12px;
      margin-top: 20px;
      .btn {
        padding: 10px 22px;
        border-radius: 30px;
        font-weight: 700;
        cursor: pointer;
        border: none;
        transition: 0.2s;
        &.cancel-btn {
          background: rgba($text-dim, 0.1);
          color: $text-dim;
          &:hover {
            background: rgba($text-dim, 0.2);
          }
        }
        &.save-btn {
          background: linear-gradient(135deg, $gold, $gold-light);
          color: $white;
          box-shadow: 0 4px 12px rgba($gold, 0.3);
          &:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 18px rgba($gold, 0.5);
          }
        }
      }
    }
  }

  .modal-fade-enter-active,
  .modal-fade-leave-active {
    transition: opacity 0.3s ease, transform 0.3s ease;
  }
  .modal-fade-enter-from,
  .modal-fade-leave-to {
    opacity: 0;
    transform: scale(0.95);
  }

  @media (max-width: 768px) {
    padding: 40px 16px;
    .wiki-header {
      flex-direction: column;
      align-items: flex-start;
    }
    .entry-card {
      flex-direction: column;
      align-items: flex-start !important;
      .card-right {
        margin-top: 12px;
        width: 100%;
        justify-content: space-between;
      }
    }
  }
}
</style>
