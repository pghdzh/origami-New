<template>
  <div class="gallery-container">
    <!-- 上传按钮 -->
    <button class="upload-btn" @click="openUploadModal">上传图片</button>

    <section class="gallery section">
      <!-- 排序控制 -->
      <div class="sort-controls">
        <button @click="toggleSort" class="sort-btn">
          <span v-if="sortBy === 'like_count'">♛ 按点赞排序</span>
          <span v-else>⏳ 按最新排序</span>
        </button>
      </div>

      <!-- 图片网格 -->
      <div class="gallery-grid">
        <div
          v-for="(img, index) in images"
          :key="img.id"
          class="card"
          @click="openLightbox(index)"
        >
          <div class="card-inner">
            <img
              :src="img.src"
              :alt="img.alt || '图集'"
              loading="lazy"
              @load="onImageLoad($event)"
            />
            <div class="overlay">
              <span>查看大图</span>
            </div>
            <button class="like-btn" @click.stop="handleLike(img)">
              <i class="heart" :class="{ liked: img.liked }"></i>
              <span class="like-count">{{ img.likeCount }}</span>
            </button>
          </div>
        </div>
      </div>

      <!-- 无限滚动哨兵 -->
      <div ref="sentinel" class="sentinel"></div>
      <div class="loading" v-if="loading">加载中...</div>
      <div class="finished" v-if="finished">—— 已至深渊尽头 · 暂无更多 ——</div>
    </section>

    <!-- 右侧排行榜 -->
    <aside class="ranking-panel" :class="{ collapsed: !expanded }">
      <div class="panel-header" @click="expanded = !expanded">
        <div class="header-left">
          <span class="crown-icon">👑</span>
          <h3 class="ranking-title">排行榜</h3>
        </div>
        <div class="header-right">
          <span class="total-count">共{{ imgTotal }}张</span>
          <span class="toggle-icon">{{ expanded ? "▾" : "▸" }}</span>
        </div>
      </div>
      <transition name="fade">
        <ul v-if="expanded" class="ranking-list">
          <li
            v-for="(item, idx) in rankingList"
            :key="idx"
            class="ranking-item"
            :class="'rank-' + (idx + 1)"
          >
            <span class="rank">
              <i v-if="idx === 0" class="trophy">🏆</i>
              <i v-else-if="idx === 1" class="medal">🥈</i>
              <i v-else-if="idx === 2" class="medal">🥉</i>
              <span v-else class="rank-num">{{ idx + 1 }}</span>
            </span>
            <span class="name">{{ item.nickname }}</span>
            <span class="count">{{ item.count }} 张</span>
            <div class="glow-bar"></div>
          </li>
        </ul>
      </transition>
    </aside>

    <!-- 灯箱 -->
    <div v-if="lightboxOpen" class="lightbox" @click.self="closeLightbox">
      <span class="close" @click="closeLightbox">✕</span>
      <span class="prev" @click.stop="prevImage">‹</span>
      <img :src="images[currentIndex].src" :alt="images[currentIndex].alt" />
      <span class="next" @click.stop="nextImage">›</span>
    </div>

    <!-- 上传弹窗（略做精简，保留核心功能） -->
    <div
      v-if="uploadModalOpen"
      class="upload-modal-overlay"
      @click.self="closeUploadModal"
    >
      <div class="upload-modal">
        <!-- 顶部标题 -->
        <div class="modal-header">
          <span class="modal-icon">👑</span>
          <h3>上传于魔王</h3>
        </div>

        <!-- 今日上传状态 -->
        <div class="stats-box">
          <span
            >今日已上传 <strong>{{ uploadedToday }}</strong> 张</span
          >
          <span class="divider">|</span>
          <span
            >剩余额度 <strong>{{ remaining }}</strong> 张</span
          >
        </div>

        <!-- 输入区 -->
        <label class="input-label">
          <span class="label-text">上传者代号</span>
          <input v-model="nickname" type="text" placeholder="留下你的名号..." />
        </label>

        <label class="input-label">
          <span class="label-text">选择图片（单日最多 {{ remaining }} 张）</span>
          <div class="file-input-wrapper">
            <input
              ref="fileInput"
              type="file"
              multiple
              accept="image/*"
              @change="handleFileSelect"
            />
          </div>
          <p v-if="selectedFiles.length" class="selected-count">
            已选择 <strong>{{ selectedFiles.length }}</strong> 张图片
          </p>
        </label>

        <!-- 审核提示（完整保留） -->
        <div class="tips-container">
          <ul class="tips-list">
            <li>
              <strong>审核规则：</strong>1. 不要色情倾向（不要露三点，我怕被封）
              2. 要我能认出是折纸。
            </li>
            <li>
              由于没有用户系统，我这边不好做审核反馈，但只要显示上传成功，我这边肯定能收到。
            </li>
            <li>
              如果图片数量较多请在b站私信联系我给我网盘链接，因为我云服务器比较小一次性上传太多图片可能会导致上传不上，感谢理解。
            </li>
            <li>
              因为审核上传一次比较麻烦，所以审核时间不定，最晚一周，感谢谅解。
            </li>
          </ul>
        </div>

        <!-- 操作按钮 -->
        <div class="modal-actions">
          <button class="btn-cancel" @click="closeUploadModal">取消</button>
          <button
            class="btn-submit"
            :disabled="!canSubmit || isUploading"
            @click="submitUpload"
          >
            {{ isUploading ? "上传中..." : "上传" }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed, nextTick, onBeforeUnmount } from "vue";
import { uploadImages } from "@/api/modules/images";
import { getRankingList } from "@/api/modules/ranking";
import { getImagesLikesList, likeImage } from "@/api/modules/imagesLikes";
import { debounce } from "lodash";

const sortBy = ref<"uploaded_at" | "like_count">("like_count");
const order = ref<"asc" | "desc">("desc");
function toggleSort() {
  sortBy.value = sortBy.value === "uploaded_at" ? "like_count" : "uploaded_at";
  pageImage.value = 1;
  images.value = [];
  finished.value = false;
  loadNextPage();
}

function getLikedIds(): number[] {
  const data = localStorage.getItem("likedImageIds");
  return data ? JSON.parse(data) : [];
}
function setLikedIds(ids: number[]) {
  localStorage.setItem("likedImageIds", JSON.stringify(ids));
}

async function handleLike(img: ImageItem) {
  if (img.liked) return;
  try {
    await likeImage(img.id);
    img.likeCount += 1;
    img.liked = true;
    const likedIds = getLikedIds();
    likedIds.push(img.id);
    setLikedIds(likedIds);
  } catch (error) {
    console.error(error);
  }
}

interface ImageItem {
  src: string;
  alt: string;
  likeCount: number;
  id: number;
  liked: boolean;
}
interface RankingItem {
  nickname: string;
  count: number;
}

const images = ref<ImageItem[]>([]);
const pageImage = ref(1);
const limit = 20;
const loading = ref(false);
const finished = ref(false);
const sentinel = ref<HTMLElement | null>(null);
const rankingList = ref<RankingItem[]>([]);
const expanded = ref(true);
const imgTotal = ref(0);

const fetchRanking = async () => {
  const res = await getRankingList({
    page: 1,
    pageSize: 99,
    character_key: "origami",
  });
  if (res.success) rankingList.value = res.data;
};

const fixImageUrl = (url: string): string => {
  if (url.includes("127.0.0.1") || url.includes("localhost")) {
    return url.replace(
      /https?:\/\/127\.0\.0\.1(:\d+)?/,
      window.location.origin
    );
  }
  return url;
};

async function loadNextPage() {
  if (loading.value || finished.value) return;
  loading.value = true;
  try {
    const res = await getImagesLikesList({
      page: pageImage.value,
      limit,
      sortBy: sortBy.value,
      character_key: "origami",
      order: order.value,
    });
    imgTotal.value = res.total;
    const likedIds = getLikedIds();
    const list: ImageItem[] = (res.images as any[]).map((item) => ({
      src: fixImageUrl(item.url),
      alt: "",
      likeCount: item.like_count,
      id: item.id,
      liked: likedIds.includes(item.id),
    }));
    if (list.length === 0) {
      finished.value = true;
      return;
    }
    const existingIds = new Set(images.value.map((i) => i.id));
    const filtered = list.filter((item) => !existingIds.has(item.id));
    images.value.push(...filtered);
    pageImage.value++;
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
}

const debouncedLoad = debounce(loadNextPage, 200, {
  leading: true,
  trailing: false,
});

const lightboxOpen = ref(false);
const currentIndex = ref(0);
function openLightbox(index: number) {
  currentIndex.value = index;
  lightboxOpen.value = true;
}
function closeLightbox() {
  lightboxOpen.value = false;
}
function prevImage() {
  currentIndex.value =
    (currentIndex.value + images.value.length - 1) % images.value.length;
}
function nextImage() {
  currentIndex.value = (currentIndex.value + 1) % images.value.length;
}

function onImageLoad(e: Event) {
  const img = e.target as HTMLImageElement;
  img.classList.add("loaded");
  // 移除卡片初始的模糊类
  const card = img.closest(".card");
  if (card) card.classList.add("loaded");
}

// 上传相关
const uploadModalOpen = ref(false);
const nickname = ref("");
const fileInput = ref<HTMLInputElement>();
const selectedFiles = ref<File[]>([]);
const uploadedToday = ref(Number(localStorage.getItem(getTodayKey()) || 0));
const remaining = computed(() => Math.max(27 - uploadedToday.value, 0));
const isUploading = ref(false);
const canSubmit = computed(
  () =>
    nickname.value.trim() &&
    selectedFiles.value.length > 0 &&
    selectedFiles.value.length <= remaining.value
);
function getTodayKey() {
  return `uploaded_${new Date().toISOString().slice(0, 10)}`;
}
function clearOldRecords() {
  for (const key of Object.keys(localStorage)) {
    if (!key.startsWith("uploaded_")) continue;
    const diff = (Date.now() - new Date(key.slice(9)).getTime()) / 86400000;
    if (diff > 2) localStorage.removeItem(key);
  }
}
function openUploadModal() {
  clearOldRecords();
  nickname.value = "";
  selectedFiles.value = [];
  if (fileInput.value) fileInput.value.value = "";
  uploadedToday.value = Number(localStorage.getItem(getTodayKey()) || 0);
  uploadModalOpen.value = true;
}
function closeUploadModal() {
  uploadModalOpen.value = false;
}
function handleFileSelect(e: Event) {
  const files = Array.from((e.target as HTMLInputElement).files || []);
  const valid = files.filter((f) => f.size <= 20 * 1024 * 1024);
  if (valid.length < files.length) alert("部分文件过大已过滤");
  if (valid.length === 0) return;
  if (valid.length > remaining.value) {
    alert(`最多还能上传 ${remaining.value} 张，已截取前 ${remaining.value} 张`);
    selectedFiles.value = valid.slice(0, remaining.value);
  } else {
    selectedFiles.value = valid;
  }
}
async function submitUpload() {
  if (!canSubmit.value) return;
  isUploading.value = true;
  try {
    const res = await uploadImages(
      selectedFiles.value,
      nickname.value.trim(),
      "origami"
    );
    uploadedToday.value += res.data.length;
    localStorage.setItem(getTodayKey(), String(uploadedToday.value));
    alert(`成功上传 ${res.data.length} 张`);
    closeUploadModal();
  } catch (err: any) {
    alert(err.message || "上传失败");
  } finally {
    isUploading.value = false;
  }
}

let sentinelObserver: IntersectionObserver;
onMounted(async () => {
  await fetchRanking();
  await loadNextPage();
  sentinelObserver = new IntersectionObserver(
    (entries) => {
      if (entries[0].isIntersecting) debouncedLoad();
    },
    { threshold: 0.1 }
  );
  if (sentinel.value) sentinelObserver.observe(sentinel.value);
});
onBeforeUnmount(() => {
  sentinelObserver?.disconnect();
});
</script>

<style scoped lang="scss">
// 救世魔王配色
$bg: #0b0e14;
$card-bg: #111622;
$blue-black: #1a2330;
$blood: #6b2a4a;
$blood-light: #8e3a5a;
$pale: #b0b8c5;
$dim: #6e7885;
$gold: #8a6e3e;
$silver: #7b8a9a;
$bronze: #a0704a;
.gallery-container {
  background: $bg;
  min-height: 100vh;
  color: $pale;
  font-family: "Noto Sans SC", "Inter", sans-serif;
  position: relative;
  padding: 20px 0 60px;

  &::before {
    content: "";
    position: fixed;
    inset: 0;
    background: radial-gradient(
        circle at 30% 40%,
        rgba($blood, 0.1) 0%,
        transparent 50%
      ),
      radial-gradient(
        circle at 70% 80%,
        rgba($blue-black, 0.5) 0%,
        transparent 40%
      );
    pointer-events: none;
  }

  .upload-btn {
    position: fixed;
    bottom: 28px;
    right: 28px;
    z-index: 100;
    background: $blood;
    border: 1px solid rgba($pale, 0.2);
    border-radius: 24px;
    padding: 10px 24px;
    color: #fff;
    font-weight: 700;
    letter-spacing: 1px;
    cursor: pointer;
    box-shadow: 0 6px 20px rgba(0, 0, 0, 0.6), 0 0 16px rgba($blood, 0.4);
    transition: all 0.3s;
    &:hover {
      background: $blood-light;
      box-shadow: 0 10px 28px rgba(0, 0, 0, 0.8), 0 0 24px rgba($blood, 0.7);
      transform: translateY(-2px);
    }
  }

  .gallery {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 20px;
    position: relative;
    z-index: 1;
  }

  .sort-controls {
    text-align: center;
    margin: 30px 0;
    .sort-btn {
      background: rgba($blue-black, 0.7);
      border: 1px solid rgba($blood, 0.4);
      color: $pale;
      padding: 8px 28px;
      border-radius: 20px;
      font-weight: 600;
      cursor: pointer;
      transition: 0.3s;
      &:hover {
        background: $blood;
        border-color: $blood;
        color: white;
      }
    }
  }

  .gallery-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: 20px;

    .card {
      opacity: 0;
      transform: translateY(30px);
      transition: opacity 0.5s ease, transform 0.5s ease;
      &.loaded {
        opacity: 1;
        transform: translateY(0);
      }
      .card-inner {
        position: relative;
        border-radius: 8px;
        overflow: hidden;
        background: $card-bg;
        border: 1px solid rgba($blood, 0.25);
        box-shadow: 0 4px 15px rgba(0, 0, 0, 0.5);
        transition: transform 0.3s, box-shadow 0.3s;
        &:hover {
          transform: translateY(-5px);
          box-shadow: 0 12px 24px rgba(0, 0, 0, 0.8);
          border-color: $blood;
          img {
            transform: scale(1.03);
          }
        }
        img {
          width: 100%;
          height: 220px;
          object-fit: cover;
          display: block;
          transition: transform 0.4s ease, filter 0.4s ease;
          filter: blur(8px) brightness(0.7);
          &.loaded {
            filter: blur(0) brightness(1);
          }
        }
        .overlay {
          position: absolute;
          bottom: 0;
          left: 0;
          right: 0;
          background: linear-gradient(transparent, rgba(0, 0, 0, 0.7));
          color: white;
          text-align: center;
          padding: 15px;
          opacity: 0;
          transition: opacity 0.3s;
          font-size: 0.9rem;
          letter-spacing: 1px;
          font-weight: 600;
          pointer-events: none;
        }
        &:hover .overlay {
          opacity: 1;
        }
        .like-btn {
          position: absolute;
          bottom: 10px;
          right: 10px;
          background: rgba(0, 0, 0, 0.6);
          border: none;
          border-radius: 20px;
          padding: 4px 12px;
          display: flex;
          align-items: center;
          gap: 6px;
          cursor: pointer;
          color: $pale;
          font-size: 0.9rem;
          backdrop-filter: blur(4px);
          .heart {
            width: 18px;
            height: 18px;
            background: url("/icons/heart-red-outline.svg") no-repeat center /
              contain;
            &.liked {
              background-image: url("/icons/heart-red-filled.svg");
              filter: drop-shadow(0 0 4px red);
            }
          }
        }
      }
    }
  }

  .sentinel {
    height: 20px;
  }
  .loading,
  .finished {
    text-align: center;
    padding: 24px;
    color: $dim;
    font-style: italic;
    letter-spacing: 1px;
  }

  // 排行榜
  .ranking-panel {
    position: fixed;
    top: 80px;
    right: 20px;
    width: 220px;
    background: rgba($blue-black, 0.8);
    backdrop-filter: blur(16px);
    border: 1px solid rgba($blood, 0.4);
    border-radius: 20px;
    padding: 16px 14px;
    z-index: 50;
    color: $pale;
    box-shadow: 0 12px 30px rgba(0, 0, 0, 0.7), 0 0 20px rgba($blood, 0.3),
      inset 0 1px 0 rgba(255, 255, 255, 0.05);
    transition: all 0.3s ease;

    &.collapsed {
      padding-bottom: 12px;
      .panel-header {
        margin-bottom: 0;
      }
    }

    .panel-header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      cursor: pointer;
      margin-bottom: 12px;
      user-select: none;

      .header-left {
        display: flex;
        align-items: center;
        gap: 6px;
        .crown-icon {
          font-size: 1.3rem;
          filter: drop-shadow(0 0 6px $gold);
          line-height: 1;
        }
        .ranking-title {
          font-weight: 800;
          font-size: 1.2rem;
          background: linear-gradient(to right, $pale, $gold);
          -webkit-background-clip: text;
          -webkit-text-fill-color: transparent;
          letter-spacing: 1px;
        }
      }

      .header-right {
        display: flex;
        align-items: center;
        gap: 8px;
        .total-count {
          font-size: 0.7rem;
          color: $dim;
          background: rgba(0, 0, 0, 0.3);
          padding: 2px 8px;
          border-radius: 10px;
        }
        .toggle-icon {
          font-size: 1.2rem;
          color: $blood-light;
          transition: transform 0.3s;
        }
      }

      &:hover {
        .toggle-icon {
          color: $pale;
          transform: scale(1.2);
        }
      }
    }

    .ranking-list {
      margin-top: 8px;
      list-style: none;
      padding: 0;
      max-height: 60vh;
      overflow-y: auto;
      scrollbar-width: thin;
      scrollbar-color: $blood rgba(0, 0, 0, 0.2);

      &::-webkit-scrollbar {
        width: 4px;
      }
      &::-webkit-scrollbar-track {
        background: rgba(0, 0, 0, 0.1);
        border-radius: 4px;
      }
      &::-webkit-scrollbar-thumb {
        background: $blood;
        border-radius: 4px;
      }

      .ranking-item {
        display: flex;
        align-items: center;
        padding: 8px 10px;
        border-radius: 10px;
        margin-bottom: 6px;
        background: rgba(0, 0, 0, 0.25);
        transition: all 0.3s ease;
        position: relative;
        overflow: hidden;
        font-size: 0.85rem;

        &:last-child {
          margin-bottom: 0;
        }

        .glow-bar {
          position: absolute;
          left: 0;
          top: 0;
          height: 100%;
          width: 3px;
          background: transparent;
          transition: background 0.3s;
        }

        &:hover {
          background: rgba($blood, 0.2);
          transform: translateX(-4px);
          .glow-bar {
            background: $blood-light;
          }
        }

        .rank {
          width: 30px;
          font-weight: 800;
          text-align: center;
          flex-shrink: 0;

          .trophy,
          .medal {
            font-size: 1rem;
          }

          .rank-num {
            color: $dim;
          }
        }

        .name {
          flex: 1;
          margin: 0 8px;
          font-weight: 500;
        
          color: $pale;
        }

        .count {
          font-weight: 700;
          color: $dim;
          background: rgba(0, 0, 0, 0.3);
          padding: 2px 8px;
          border-radius: 12px;
          font-size: 0.75rem;
          letter-spacing: 0.5px;
          transition: all 0.3s;
        }

        // 前三名特殊装饰
        &.rank-1 {
          background: linear-gradient(
            135deg,
            rgba($gold, 0.2) 0%,
            rgba(0, 0, 0, 0.3) 100%
          );
          border: 1px solid rgba($gold, 0.4);
          box-shadow: 0 0 12px rgba($gold, 0.2);
          .rank {
            color: $gold;
          }
          .name {
            color: #fff;
            font-weight: 700;
          }
          .count {
            background: rgba($gold, 0.3);
            color: #fff;
          }
        }

        &.rank-2 {
          background: linear-gradient(
            135deg,
            rgba($silver, 0.15) 0%,
            rgba(0, 0, 0, 0.3) 100%
          );
          border: 1px solid rgba($silver, 0.3);
          .rank {
            color: $silver;
          }
          .name {
            font-weight: 600;
          }
          .count {
            background: rgba($silver, 0.2);
          }
        }

        &.rank-3 {
          background: linear-gradient(
            135deg,
            rgba($bronze, 0.15) 0%,
            rgba(0, 0, 0, 0.3) 100%
          );
          border: 1px solid rgba($bronze, 0.3);
          .rank {
            color: $bronze;
          }
          .name {
            font-weight: 600;
          }
          .count {
            background: rgba($bronze, 0.2);
          }
        }
      }
    }
  }

  /* 过渡动画 */
  .fade-enter-active,
  .fade-leave-active {
    transition: opacity 0.3s ease, transform 0.3s ease;
  }
  .fade-enter-from,
  .fade-leave-to {
    opacity: 0;
    transform: translateY(-10px);
  }

  // 灯箱
  .lightbox {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.95);
    z-index: 1000;
    display: flex;
    align-items: center;
    justify-content: center;
    img {
      max-width: 90%;
      max-height: 90%;
      border-radius: 4px;
      box-shadow: 0 0 50px rgba($blood, 0.3);
    }
    .close,
    .prev,
    .next {
      position: absolute;
      color: $pale;
      font-size: 3rem;
      cursor: pointer;
      transition: color 0.2s;
      &:hover {
        color: $blood-light;
      }
    }
    .close {
      top: 20px;
      right: 30px;
    }
    .prev {
      left: 30px;
      top: 50%;
      transform: translateY(-50%);
    }
    .next {
      right: 30px;
      top: 50%;
      transform: translateY(-50%);
    }
  }

  // 上传弹窗
  .upload-modal-overlay {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.82);
    backdrop-filter: blur(12px);
    z-index: 2000;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 20px;
  }

  .upload-modal {
    width: 520px;
    max-width: 100%;
    background: linear-gradient(
      160deg,
      rgba($blue-black, 0.95) 0%,
      rgba(0, 0, 0, 0.95) 100%
    );
    border: 1px solid rgba($blood, 0.5);
    border-radius: 24px;
    padding: 28px;
    color: $pale;
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.8), 0 0 30px rgba($blood, 0.3),
      inset 0 1px 0 rgba(255, 255, 255, 0.05);
    max-height: 85vh;
    overflow-y: auto;
    scrollbar-width: thin;
    scrollbar-color: $blood rgba(0, 0, 0, 0.3);

    &::-webkit-scrollbar {
      width: 4px;
    }
    &::-webkit-scrollbar-track {
      background: rgba(0, 0, 0, 0.2);
      border-radius: 4px;
    }
    &::-webkit-scrollbar-thumb {
      background: $blood;
      border-radius: 4px;
    }

    .modal-header {
      display: flex;
      align-items: center;
      gap: 10px;
      margin-bottom: 20px;
      .modal-icon {
        font-size: 2rem;
        filter: drop-shadow(0 0 6px $gold);
      }
      h3 {
        margin: 0;
        font-size: 1.5rem;
        font-weight: 800;
        background: linear-gradient(to right, $pale, $blood-light);
        -webkit-background-clip: text;
        -webkit-text-fill-color: transparent;
        letter-spacing: 1px;
      }
    }

    .stats-box {
      background: rgba(0, 0, 0, 0.4);
      border-radius: 12px;
      padding: 10px 16px;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 12px;
      font-size: 0.9rem;
      margin-bottom: 20px;
      border: 1px solid rgba($blood, 0.2);
      strong {
        color: $blood-light;
        font-weight: 700;
      }
      .divider {
        color: $dim;
        opacity: 0.5;
      }
    }

    .input-label {
      display: block;
      margin-bottom: 16px;
      .label-text {
        display: block;
        font-size: 0.8rem;
        font-weight: 600;
        color: $dim;
        margin-bottom: 6px;
        letter-spacing: 0.5px;
      }
      input[type="text"],
      input[type="file"] {
        width: 100%;
        background: rgba(0, 0, 0, 0.5);
        border: 1px solid rgba($blood, 0.3);
        border-radius: 10px;
        padding: 10px 14px;
        color: $pale;
        font-size: 0.95rem;
        outline: none;
        transition: border-color 0.3s, box-shadow 0.3s;
        &::placeholder {
          color: rgba($pale, 0.3);
        }
        &:focus {
          border-color: $blood-light;
          box-shadow: 0 0 8px rgba($blood-light, 0.3);
        }
      }
      .file-input-wrapper {
        position: relative;
        overflow: hidden;
        cursor: pointer;
        &::before {
          content: "点击选择文件";
          position: absolute;
          top: 0;
          left: 0;
          width: 100%;
          height: 100%;
          background: linear-gradient(
            135deg,
            rgba($blood, 0.8),
            rgba($blood-light, 0.8)
          );
          color: white;
          display: flex;
          align-items: center;
          justify-content: center;
          border-radius: 10px;
          font-weight: 600;
          pointer-events: none;
        }
        input[type="file"] {
          opacity: 0;
          height: 44px;
          cursor: pointer;
        }
        &:hover::before {
          background: linear-gradient(135deg, $blood-light, $blood);
        }
      }
      .selected-count {
        margin-top: 8px;
        font-size: 0.85rem;
        color: $pale;
        strong {
          color: $blood-light;
        }
      }
    }

    .tips-container {
      background: rgba(0, 0, 0, 0.35);
      border-left: 3px solid $blood;
      border-radius: 8px;
      padding: 14px 16px;
      margin: 18px 0 24px;
      .tips-list {
        margin: 0;
        padding-left: 18px;
        list-style: none;
        li {
          font-size: 0.82rem;
          color: $dim;
          line-height: 1.6;
          margin-bottom: 8px;
          position: relative;
          &::before {
            content: "⚡";
            position: absolute;
            left: -18px;
            color: $blood-light;
            font-size: 0.7rem;
            top: 2px;
          }
          strong {
            color: $blood-light;
            font-weight: 600;
          }
        }
      }
    }

    .modal-actions {
      display: flex;
      justify-content: flex-end;
      gap: 12px;
      margin-top: 8px;
      button {
        padding: 10px 28px;
        border-radius: 12px;
        font-weight: 700;
        font-size: 0.95rem;
        cursor: pointer;
        transition: all 0.3s;
        letter-spacing: 0.5px;
      }
      .btn-cancel {
        background: transparent;
        border: 1px solid $dim;
        color: $dim;
        &:hover {
          border-color: $pale;
          color: $pale;
        }
      }
      .btn-submit {
        background: linear-gradient(135deg, $blood, $blood-light);
        border: none;
        color: white;
        box-shadow: 0 4px 12px rgba($blood, 0.5);
        &:hover:not(:disabled) {
          transform: translateY(-2px);
          box-shadow: 0 8px 20px rgba($blood, 0.7);
        }
        &:disabled {
          opacity: 0.5;
          cursor: not-allowed;
          box-shadow: none;
        }
      }
    }
  }
}
</style>
