<template>
  <div class="origami-resources">
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
    <!-- 降低遮罩透明度，让背景更清晰 -->
    <div class="bg-overlay"></div>

    <header class="hero">
      <div class="hero-inner">
        <span class="sigil">✦</span>
        <h1>资源分享</h1>
        <p class="subtitle">如绝灭天使之光 · 分享你所珍视的资源</p>
      </div>
    </header>

    <main class="container">
      <!-- 上传区（桌面侧边 / 移动顶部折叠） -->
      <aside class="uploader" :class="{ collapsed: uploaderCollapsed }">
        <div class="uploader-head" @click="toggleUploader">
          <span class="toggle-icon">{{ uploaderCollapsed ? "✧" : "✧" }}</span>
          <span>{{ uploaderCollapsed ? "展开上传" : "收起上传" }}</span>
        </div>

        <form
          @submit.prevent="addResource"
          class="upload-form"
          :aria-hidden="uploaderCollapsed"
        >
          <div class="field">
            <input
              v-model="form.title"
              type="text"
              placeholder="标题（必填，可注明解压码等）"
              aria-label="标题"
            />
          </div>
          <div class="field">
            <input
              v-model="form.type"
              type="text"
              placeholder="链接类型（网页、网盘、视频等）"
              aria-label="来源"
            />
          </div>
          <div class="field">
            <input
              v-model="form.uploader"
              type="text"
              placeholder="上传人（可选）"
              aria-label="上传人"
            />
          </div>
          <div class="field">
            <input
              v-model="form.link"
              type="url"
              placeholder="链接地址（https:// 开头）"
              aria-label="链接"
            />
          </div>
          <div class="actions">
            <button type="submit" class="btn primary">奉献资源</button>
          </div>
        </form>
      </aside>

      <!-- 资源列表 -->
      <section class="list">
        <div class="list-header">
          <h2>资源列表（{{ resources.length }}）</h2>
          <div class="sort">
            <label>
              排序：
              <select v-model="sortBy">
                <option value="time">按时间（新→旧）</option>
                <option value="likes">按点赞（高→低）</option>
              </select>
            </label>
          </div>
        </div>

        <ul class="items">
          <li v-for="item in sortedResources" :key="item.id" class="item">
            <a
              :href="item.link"
              target="_blank"
              rel="noopener noreferrer"
              class="title"
              >{{ item.title }}</a
            >

            <div class="meta">
              <div class="left">
                <span class="uploader-tag">{{ item.uploader || "匿名" }}</span>
                <span class="dot">•</span>
                <time :datetime="item.time">{{ formatTime(item.time) }}</time>
              </div>

              <div class="right">
                <button
                  @click.prevent="handleLike(item)"
                  :aria-pressed="likedIds.has(String(item.id))"
                  class="like-btn"
                  :class="{ active: likedIds.has(String(item.id)) }"
                >
                  <img
                    :src="
                      likedIds.has(String(item.id))
                        ? '/icons/heart-red-filled.svg'
                        : '/icons/heart-red-outline.svg'
                    "
                    class="heart-icon"
                    alt="heart"
                  />
                  <span class="count">{{ item.likes }}</span>
                </button>

                <span class="badge">{{ item.type }}</span>
              </div>
            </div>
          </li>
        </ul>

        <p v-if="resources.length === 0" class="empty">
          尚无资源，成为第一位奉献者吧 ✦
        </p>
      </section>
    </main>

    <footer class="foot">✦ 点击标题直接跳转 ✦</footer>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount } from "vue";
import {
  getResourceList,
  createResource,
  likeResource,
} from "@/api/modules/resource";
import { ElMessage } from "element-plus";

interface Resource {
  id: number | string;
  title: string;
  uploader?: string;
  time: string;
  likes: number;
  link: string;
  type: string;
  role_key?: string;
}

const STORAGE_KEY = "fll_resources_v1";
const DEFAULT_ROLE = "origami";

const form = ref({
  title: "",
  uploader: "",
  link: "",
  type: "",
});

const resources = ref<Resource[]>([]);
const likedIds = ref(new Set<string>());
const sortBy = ref<"time" | "likes">("time");
const uploaderCollapsed = ref(false);

function mapServerToLocal(row: any): Resource {
  return {
    id: row.id,
    title: row.title,
    uploader: row.uploader || "匿名",
    time: row.created_at || row.time || new Date().toISOString(),
    likes: row.likes ?? 0,
    link: row.link,
    type: row.storage_type || row.type || "其他",
    role_key: row.role_key,
  };
}

async function loadResources() {
  try {
    const res: any = await getResourceList({
      role_key: DEFAULT_ROLE,
      page: 1,
      pageSize: 100,
    });
    if (res && res.success && Array.isArray(res.data)) {
      resources.value = res.data.map(mapServerToLocal);
      const raw = localStorage.getItem(STORAGE_KEY);
      if (raw) {
        try {
          const parsed = JSON.parse(raw);
          if (parsed.liked && Array.isArray(parsed.liked)) {
            parsed.liked.forEach((id: string) => likedIds.value.add(id));
          }
        } catch (e) {}
      }
      return;
    }
  } catch (err) {
    console.warn("拉取资源失败", err);
  }
}

function saveLocalCache() {
  try {
    const liked = Array.from(likedIds.value);
    localStorage.setItem(STORAGE_KEY, JSON.stringify({ liked }));
  } catch (e) {}
}

// 背景图片导入与轮播（保留原有逻辑）
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

onMounted(() => {
  loadResources();
  Imgtimer = window.setInterval(() => {
    currentIndex.value =
      (currentIndex.value + 1) % Math.max(1, randomFive.value.length);
  }, 5200);
  // 移动端默认折叠上传区
  uploaderCollapsed.value = window.innerWidth <= 640;
});

function toggleUploader() {
  uploaderCollapsed.value = !uploaderCollapsed.value;
}

onBeforeUnmount(() => {
  if (Imgtimer) clearInterval(Imgtimer);
});

async function addResource() {
  const t = form.value.title.trim();
  const l = form.value.link.trim();
  if (!t || !l) return ElMessage.warning("请填写完整信息");
  if (!/^https?:\/\//i.test(l))
    return ElMessage.error("请输入正确的链接(https开头)");

  try {
    const payload = {
      title: t,
      uploader: form.value.uploader.trim() || "匿名",
      link: l,
      storage_type: form.value.type,
      role_key: DEFAULT_ROLE,
    };
    const res: any = await createResource(payload);
    if (res && res.success && res.data) {
      const added = mapServerToLocal(res.data);
      resources.value.unshift(added);
      saveLocalCache();
      form.value = { title: "", uploader: "", link: "", type: "" };
      ElMessage.success("上传成功");
      return;
    }
    ElMessage.error("上传失败");
  } catch (err) {
    console.warn("创建资源失败", err);
  }
}

async function handleLike(item: Resource) {
  const id = item.id;
  const wasLiked = likedIds.value.has(String(id));
  if (wasLiked) {
    likedIds.value.delete(String(id));
    item.likes = Math.max(0, item.likes - 1);
  } else {
    likedIds.value.add(String(id));
    item.likes++;
  }
  saveLocalCache();

  try {
    const action = wasLiked ? "unlike" : "like";
    const res: any = await likeResource(id, action);
    if (
      res &&
      res.success &&
      res.data &&
      typeof res.data.likes !== "undefined"
    ) {
      item.likes = res.data.likes;
    }
  } catch (err) {
    console.warn("点赞接口失败，回滚", err);
    if (wasLiked) {
      likedIds.value.add(String(id));
      item.likes++;
    } else {
      likedIds.value.delete(String(id));
      item.likes = Math.max(0, item.likes - 1);
    }
    saveLocalCache();
  }
}

const sortedResources = computed(() => {
  const arr = [...resources.value];
  if (sortBy.value === "time") {
    arr.sort((a, b) => +new Date(b.time) - +new Date(a.time));
  } else {
    arr.sort((a, b) => b.likes - a.likes);
  }
  return arr;
});

function formatTime(iso: string) {
  try {
    const d = new Date(iso);
    return new Intl.DateTimeFormat("zh-CN", {
      year: "numeric",
      month: "2-digit",
      day: "2-digit",
      hour: "2-digit",
      minute: "2-digit",
    }).format(d);
  } catch (e) {
    return iso;
  }
}
</script>

<style scoped lang="scss">
/* 天使配色 */
$bg: #f5f8fc;
$white: #ffffff;
$gold: #c5a059;
$gold-light: #e8cd7a;
$blue: #5b9bd5;
$blue-light: #8ec5fc;
$text: #333333;
$text-dim: #5a6b7e;

.origami-resources {
  position: relative;
  min-height: 100vh;
  background: $bg;
  font-family: "Noto Sans SC", "Inter", sans-serif;
  color: $text;
  overflow-x: hidden;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-bottom: 40px;

  /* 背景轮播（保留图片） */
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
        opacity: 1; /* 提高透明度 */
      }
    }
  }
  .carousel2 {
    display: none;
  }

  /* 叠加层变浅，让背景可见 */
  .bg-overlay {
    position: fixed;
    inset: 0;
    background: linear-gradient(
      180deg,
      rgba($bg, 0.13),
      rgba($bg, 0.15)
    ); /* 降低不透明度 */
    z-index: 1;
    pointer-events: none;
  }

  /* 头部 */
  .hero {
    position: relative;
    z-index: 10;
    width: 100%;
    text-align: center;
    padding: 40px 20px 30px;
    .hero-inner {
      .sigil {
        font-size: 36px;
        color: $gold;
        display: inline-block;
        filter: drop-shadow(0 0 8px $gold-light);
        margin-bottom: 10px;
      }
      h1 {
        margin: 0;
        font-size: 2.2rem;
        font-weight: 800;
        color: $text;
        letter-spacing: 2px;
        font-family: "Noto Serif SC", serif;
      }
      .subtitle {
        margin-top: 8px;
        font-size: 1rem;
        color: $gold;
        font-style: italic;
        letter-spacing: 1px;
      }
    }
  }

  /* 主容器 */
  .container {
    position: relative;
    z-index: 10;
    width: 100%;
    max-width: 1100px;
    margin: 0 auto;
    padding: 0 20px;
    display: flex;
    gap: 30px;
    align-items: flex-start;
  }

  /* 上传区（桌面侧边 / 移动顶部折叠） */
  .uploader {
    flex: 0 0 280px;
    background: rgba($white, 0.25);
    backdrop-filter: blur(2px);
    border-radius: 20px;
    border: 1px solid rgba($gold, 0.3);
    box-shadow: 0 8px 30px rgba(44, 58, 74, 0.06);
    padding: 20px;
    transition: all 0.3s;
    position: sticky;
    top: 20px;

    .uploader-head {
      cursor: pointer;
      display: flex;
      align-items: center;
      gap: 8px;
      font-weight: 700;
      color: $gold;
      user-select: none;
      .toggle-icon {
        font-size: 1.2rem;
        transition: transform 0.3s;
      }
    }

    &.collapsed {
      .upload-form {
        max-height: 0;
        overflow: hidden;
        opacity: 0;
        margin-top: 0;
        padding: 0;
      }
      .uploader-head .toggle-icon {
        transform: rotate(180deg);
      }
    }

    .upload-form {
      margin-top: 15px;
      display: flex;
      flex-direction: column;
      gap: 12px;
      transition: max-height 0.35s ease, opacity 0.3s, margin 0.3s;
      max-height: 500px;
      opacity: 1;

      .field {
        input {
          width: 100%;
          padding: 10px 14px;
          border-radius: 12px;
          border: 1px solid rgba($gold, 0.3);
          background: rgba($white, 0.8);
          font-size: 0.9rem;
          color: $text;
          outline: none;
          transition: 0.2s;
          &::placeholder {
            color: rgba($text-dim, 0.6);
          }
          &:focus {
            border-color: $gold;
            box-shadow: 0 0 8px rgba($gold, 0.2);
          }
        }
      }

      .actions {
        margin-top: 8px;
        .btn.primary {
          width: 100%;
          padding: 12px;
          border-radius: 30px;
          background: linear-gradient(135deg, $gold, $gold-light);
          border: none;
          color: white;
          font-weight: 700;
          font-size: 1rem;
          letter-spacing: 1px;
          cursor: pointer;
          box-shadow: 0 4px 12px rgba($gold, 0.3);
          transition: transform 0.2s, box-shadow 0.2s;
          &:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 20px rgba($gold, 0.5);
          }
        }
      }
    }
  }

  /* 资源列表 */
  .list {
    flex: 1;
    min-width: 0;

    .list-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 20px;
      h2 {
        margin: 0;
        font-size: 1.5rem;
        font-weight: 700;
        color: $text;
        font-family: "Noto Serif SC", serif;
      }
      .sort select {
        padding: 8px 12px;
        border-radius: 8px;
        border: 1px solid rgba($gold, 0.3);
        background: rgba($white, 0.8);
        color: $text;
      }
    }

    .items {
      list-style: none;
      padding: 0;
      margin: 0;
      display: flex;
      flex-direction: column;
      gap: 16px;

      .item {
        background: rgba($white, 0.75);
        backdrop-filter: blur(1px);
        border-radius: 16px;
        padding: 18px 20px;
        border: 1px solid rgba($gold, 0.2);
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.03);
        transition: transform 0.3s, box-shadow 0.3s;
        border-left: 4px solid transparent;

        &:hover {
          transform: translateY(-3px);
          box-shadow: 0 8px 24px rgba(0, 0, 0, 0.08);
          border-left-color: $gold;
        }

        .title {
          display: block;
          color: $text;
          font-weight: 700;
          font-size: 1.1rem;
          text-decoration: none;
          margin-bottom: 10px;
          word-break: break-word;
          &:hover {
            color: $gold;
          }
        }

        .meta {
          display: flex;
          justify-content: space-between;
          align-items: center;
          font-size: 0.85rem;

          .left {
            display: flex;
            align-items: center;
            gap: 8px;
            .uploader-tag {
              font-size: 0.8rem; /* 缩小上传者字号 */
              font-weight: 600;
              color: $blue;
              background: rgba($blue, 0.1);
              padding: 2px 8px;
              border-radius: 12px;
            }
            .dot {
              color: $text-dim;
              font-size: 0.8rem;
            }
            time {
              color: $text-dim;
              font-size: 0.8rem;
            }
          }

          .right {
            display: flex;
            align-items: center;
            gap: 12px;

            .like-btn {
              background: none;
              border: none;
              cursor: pointer;
              display: flex;
              align-items: center;
              gap: 4px;
              padding: 4px 8px;
              border-radius: 20px;
              transition: 0.2s;
              color: $text-dim;

              .heart-icon {
                width: 18px;
                height: 18px;
                filter: grayscale(0.3);
              }

              &.active {
                .heart-icon {
                  filter: none;
                }
              }

              &:hover {
                background: rgba($gold, 0.1);
              }
            }

            .badge {
              background: rgba($gold, 0.15);
              color: $gold;
              padding: 4px 12px;
              border-radius: 20px;
              font-weight: 600;
              font-size: 0.8rem;
              border: 1px solid rgba($gold, 0.3);
            }
          }
        }
      }
    }

    .empty {
      text-align: center;
      color: $text-dim;
      padding: 60px 20px;
      font-style: italic;
    }
  }

  /* 页脚 */
  .foot {
    position: relative;
    z-index: 10;
    margin-top: 40px;
    color: $gold;
    font-size: 0.9rem;
    letter-spacing: 1px;
  }

  /* 响应式：移动端布局调整 */
  @media (max-width: 640px) {
    .carousel1 {
      display: none;
    }
    .carousel2 {
      display: block;
    }

    .container {
      flex-direction: column;
      gap: 20px;
    }
    .uploader {
      position: static; /* 取消粘性定位 */
      width: 100%;
      flex: none;
      margin-bottom: 10px;
    }
    .list .items .item .meta {
      flex-direction: column;
      align-items: flex-start;
      gap: 8px;
    }
    .uploader-tag {
      font-size: 0.75rem; /* 移动端更小 */
    }
  }
}
</style>
