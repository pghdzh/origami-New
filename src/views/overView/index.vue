<template>
  <div class="character-page">
    <!-- 动态背景 -->
    <div class="bg-layer">
      <div class="halo halo1"></div>
      <div class="halo halo2"></div>
      <div class="halo halo3"></div>
      <span
        v-for="f in feathers"
        :key="'f-' + f.id"
        class="feather"
        :style="f.style"
      ></span>
      <span
        v-for="p in particles"
        :key="'p-' + p.id"
        class="particle"
        :style="p.style"
      ></span>
      <span
        v-for="s in stars"
        :key="'s-' + s.id"
        class="star"
        :style="s.style"
      ></span>
      <div class="soft-glow"></div>
      <div class="scanlines"></div>
    </div>

    <main class="content">
      <!-- 头部档案 -->
      <section class="hero">
        <div class="crown-icon">✦</div>
        <h1 class="name">鸢一折纸</h1>
        <p class="name-sub">Tobiichi Origami</p>
        <div class="identifiers">
          <span class="badge">识别代号：Angel</span>
          <span class="badge">AST 上士</span>
        </div>
        <div class="profile-card">
          <table>
            <tbody>
              <tr>
                <th>生日</th>
                <td>11月11日</td>
                <th>年龄</th>
                <td>16岁</td>
              </tr>
              <tr>
                <th>身高</th>
                <td>152 cm</td>
                <th>血型</th>
                <td>A型</td>
              </tr>
              <tr>
                <th>三围</th>
                <td>75/55/79 cm</td>
                <th>学校</th>
                <td>都立来禅高中 2年4班</td>
              </tr>
              <tr>
                <th>卡巴拉</th>
                <td>王冠 Kether</td>
                <th>逆卡巴拉</th>
                <td>无神论</td>
              </tr>
              <tr>
                <th>喜欢</th>
                <td colspan="3">Calorie Mate、五河士道</td>
              </tr>
              <tr>
                <th>讨厌</th>
                <td colspan="3">精灵</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- 角色形象 -->
      <section class="block">
        <h2 class="section-title"><span class="gold-line"></span>角色形象</h2>
        <div class="image-gallery">
          <div class="img-card floating-img">
            <img :src="normalImg" alt="常服" loading="lazy" /><span>常服</span>
          </div>
          <div class="img-card floating-img delay1">
            <img :src="longHairImg" alt="长发常服" loading="lazy" /><span
              >长发常服</span
            >
          </div>
          <div class="img-card floating-img delay2">
            <img :src="spiritDressImg" alt="灵装" loading="lazy" /><span
              >灵装</span
            >
          </div>
        </div>
      </section>

      <!-- 身份背景 -->
      <section class="block">
        <h2 class="section-title"><span class="gold-line"></span>身份背景</h2>
        <div class="cards-row">
          <div class="card hover-card">
            <h3>AST魔术师</h3>
            <p>
              陆上自卫队对精灵部队AST队员，阶级上士。被称为“万中选一的天才”，但也因轻率行动被队长注意。
            </p>
          </div>
          <div class="card hover-card">
            <h3>精灵·天使</h3>
            <p>
              为获取更强力量接受灵结晶，灵装为〈神威灵装·一番〉，天使为〈绝灭天使（Methratton）〉，代号Angel。
            </p>
          </div>
          <div class="card hover-card">
            <h3>悲剧与救赎</h3>
            <p>
              五年前父母因精灵丧命，极度憎恨精灵。精灵化后得知真相精神崩溃，后被士道拯救并改变过去。
            </p>
          </div>
        </div>
      </section>

      <!-- 能力设定 -->
      <section class="block">
        <h2 class="section-title"><span class="gold-line"></span>能力设定</h2>

        <div class="equipment-section">
          <h3 class="sub-section-title">
            <span class="gold-dot"></span>魔术师装备 / AST 战术显现装置
          </h3>
          <div class="equipment-grid">
            <div
              class="equip-card hover-card"
              v-for="equip in equipments"
              :key="equip.name"
            >
              <img
                :src="equip.img"
                :alt="equip.name"
                class="equip-img"
                loading="lazy"
              />
              <div class="equip-info">
                <h4 class="equip-name">{{ equip.name }}</h4>
                <p class="equip-desc">{{ equip.desc }}</p>
              </div>
            </div>
          </div>
        </div>

        <div class="ability-dual">
          <div class="ability-panel angel-panel">
            <div class="panel-img">
              <img
                :src="angelImg"
                alt="绝灭天使"
                class="spirit-img floating-img"
              />
              <span class="img-label">绝灭天使 (Methratton)</span>
            </div>
            <div class="stat-bars">
              <div class="bar" v-for="stat in angelStats" :key="stat.name">
                <span class="bar-label">{{ stat.name }}</span>
                <span class="bar-value">{{ stat.value }}</span>
                <div class="bar-track">
                  <div
                    class="bar-fill"
                    :style="{ width: stat.percent + '%' }"
                  ></div>
                </div>
              </div>
            </div>
            <div class="danger-tag">
              综合危险度 AAA · 空间震 AA · 灵装 AA · 天使 AAA
            </div>
            <h4 class="skill-title">绝灭天使技能</h4>
            <table class="skill-table">
              <thead>
                <tr>
                  <th>技能</th>
                  <th>解说</th>
                </tr>
              </thead>
              <tbody>
                <tr>
                  <td>日轮 (Shemesh)</td>
                  <td>光弹雨无差别轰炸</td>
                </tr>
                <tr>
                  <td>天翼 (Mal'akh)</td>
                  <td>高速飞行 / 瞬间移动</td>
                </tr>
                <tr>
                  <td>光剑 (Kadour)</td>
                  <td>浮游炮多角度炮击</td>
                </tr>
                <tr>
                  <td>炮冠 (Artelif)</td>
                  <td>王冠炮口最强一击</td>
                </tr>
              </tbody>
            </table>
          </div>
          <div class="ability-panel devil-panel">
            <div class="panel-img">
              <img
                :src="devilImg"
                alt="救世魔王"
                class="spirit-img floating-img delay1"
              />
              <span class="img-label">救世魔王 (Satan)</span>
            </div>
            <div class="stat-bars">
              <div class="bar" v-for="stat in devilStats" :key="stat.name">
                <span class="bar-label">{{ stat.name }}</span>
                <span class="bar-value">{{ stat.value }}</span>
                <div class="bar-track">
                  <div
                    class="bar-fill devil-fill"
                    :style="{ width: stat.percent + '%' }"
                  ></div>
                </div>
              </div>
            </div>
            <div class="danger-tag devil-tag">
              综合危险度 SS · 空间震 AAA · 灵装 AA · 天使 AAA
            </div>
            <p class="devil-desc">
              拥有与绝灭天使相同的能力，破坏力更加强大。会展开无数的<羽毛>保护宿主，可自主攻击。吞噬天空的黑暗，拥有形体的绝望。
            </p>
          </div>
        </div>
      </section>

      <!-- 人际关系 -->
      <section class="block">
        <h2 class="section-title"><span class="gold-line"></span>人际关系</h2>
        <div class="rel-grid">
          <div
            class="rel-card hover-card"
            v-for="rel in relations"
            :key="rel.name"
          >
            <strong>{{ rel.name }}</strong>
            <span class="rel-type">{{ rel.type }}</span>
            <p>{{ rel.desc }}</p>
          </div>
        </div>
      </section>

      <!-- 经典台词 -->
      <section class="block">
        <h2 class="section-title"><span class="gold-line"></span>经典台词</h2>
        <div class="quote-wall">
          <blockquote
            v-for="(q, i) in quotes"
            :key="i"
            :style="{ '--r': q.rotation }"
            class="hover-quote"
          >
            {{ q.text }}
          </blockquote>
        </div>
      </section>

      <footer class="page-footer">
        <p>✦ 绝灭天使 · Methratton ✦</p>
        <p>© {{currentYear}} 鸢一折纸电子设定集</p>
      </footer>
    </main>
  </div>
</template>

<script setup lang="ts">
import normalImg from "./img/常服.webp";
import longHairImg from "./img/长发常服.webp";
import spiritDressImg from "./img/灵装.webp";
import angelImg from "./img/Angel.webp";
import devilImg from "./img/Devil.webp";
import astEquipImg from "./img/AST装备.webp";
import sssEquipImg from "./img/英国对精灵装备.webp";
import aliceImg from "./img/爱丽丝.webp";
import crUnitImg from "./img/CR-Unit.webp";
import crUnit2Img from "./img/CR-Unit2.webp";
import brunnhildeImg from "./img/CR-Unit〈Brunnhilde〉.webp";
const currentYear = new Date().getFullYear();

const equipments = [
  {
    name: "AST制式采用装备",
    img: astEquipImg,
    desc: "日本陆上自卫队AST的正规装备。由于一般物理兵器不足以对抗精灵，队员皆装备CR-Unit，使一般人也具有战斗能力。头部需埋入小型显现装置。是折纸平常使用的对精灵作战装备。",
  },
  {
    name: "SSS——英国对精灵装备",
    img: sssEquipImg,
    desc: "英国对精灵部队SSS的正规装备，服装为蓝色线路服。包括镭射加农、突击步枪、格林机关枪等。折纸突入DEM日本分部时所使用的SSS装备由冈峰美纪惠发现。",
  },
  {
    name: "Ashcroft—〈爱丽丝（Alice）〉",
    img: aliceImg,
    desc: "外传中阿休克罗夫特系列机之一，拥有超强防御与空间闭锁能力。著装者身份认证后机体使用权归一人所有。对魔术师性能优秀，核心使用了阿尔忒弥西亚的脑内情报。",
  },
  {
    name: "CR-Unit（禁忌的武器）",
    img: crUnitImg,
    desc: "DEM的试验用机，魔术师最强讨灭兵装。纯白色机型，与DW-029R为姐妹机，超过活动界限即无法战斗。实验员使用半小时即成废人。折纸曾用其对付琴里及保护士道。",
  },
  {
    name: "CR-Unit〈Mordred〉",
    img: crUnit2Img,
    desc: "DEM最新款CR-Unit，深灰色，菱角分明。X字推进器与巨大兵装构成，能放出威力极大的魔法炮，也可化作近战巨大刀刃。折纸加入DEM后获得，可勉强对峙完全灵装的精灵。",
  },
  {
    name: "CR-Unit〈Brunnhilde〉",
    img: brunnhildeImg,
    desc: "最新款CR-Unit，名字取自女武神“布伦希尔德”。银色，武器为长枪，可组合成钻型机体。琴里将此装备交给折纸，折纸身穿CR-Unit显现限定灵装，实力大增。曾拖住反转十香。",
  },
];

const angelStats = [
  { name: "力量", value: 158 },
  { name: "耐力", value: 152 },
  { name: "灵力", value: 219 },
  { name: "敏捷", value: 136 },
  { name: "智力", value: 243 },
].map((s) => ({ ...s, percent: Math.round((s.value / 243) * 100) }));

const devilStats = [
  { name: "力量", value: 198 },
  { name: "耐力", value: 202 },
  { name: "灵力", value: 242 },
  { name: "敏捷", value: 128 },
  { name: "智力", value: 230 },
].map((s) => ({ ...s, percent: Math.round((s.value / 242) * 100) }));

const relations = [
  {
    name: "五河士道",
    type: "恋人",
    desc: "幼时被救，名义恋人。唯一称呼“士道”。",
  },
  {
    name: "崇宫真那",
    type: "朋友",
    desc: "AST同僚，士道实妹。强迫其叫“大嫂”。",
  },
  { name: "冈峰美纪惠", type: "朋友", desc: "AST时期好友，单恋折纸。" },
  {
    name: "夜刀神十香",
    type: "亦敌亦友",
    desc: "拌嘴冤家，后期和解互称名字。",
  },
  {
    name: "五河琴里",
    type: "亦敌亦友",
    desc: "曾因父母之死怀有敌意，后和解。",
  },
  { name: "时崎狂三", type: "亦敌亦友", desc: "多次交手，被称“很美味”。" },
  { name: "崇宫澪", type: "母亲/老师", desc: "精灵意义上的母亲，副班主任。" },
];

const quotes = [
  { text: "并不是跟在你的身后，只是前进的方向是同样的。", rotation: "2deg" },
  { text: "我将我的泪水寄存给你。把我的笑容托付给你。", rotation: "-3deg" },
  { text: "原来如此……是我自己，杀死了他们……", rotation: "1.5deg" },
  { text: "不要过来……我不想再失去任何人了……", rotation: "-2deg" },
  { text: "女孩子并非只有被人保护才是幸福。", rotation: "3deg" },
  { text: "真正的爱，从现在才要开始。", rotation: "-1deg" },
];

const feathers = Array.from({ length: 15 }, (_, i) => ({
  id: i,
  style: {
    width: `${10 + Math.random() * 20}px`,
    height: `${20 + Math.random() * 30}px`,
    left: `${Math.random() * 100}%`,
    animationDuration: `${12 + Math.random() * 18}s`,
    animationDelay: `${Math.random() * -20}s`,
  },
}));
const particles = Array.from({ length: 60 }, (_, i) => ({
  id: i,
  style: {
    width: `${2 + Math.random() * 4}px`,
    height: `${2 + Math.random() * 4}px`,
    left: `${Math.random() * 100}%`,
    top: `${Math.random() * 100}%`,
    animationDuration: `${6 + Math.random() * 12}s`,
    animationDelay: `${Math.random() * -12}s`,
  },
}));
const stars = Array.from({ length: 30 }, (_, i) => ({
  id: i,
  style: {
    width: `${1 + Math.random() * 3}px`,
    height: `${1 + Math.random() * 3}px`,
    left: `${Math.random() * 100}%`,
    top: `${Math.random() * 100}%`,
    animationDelay: `${Math.random() * 5}s`,
    animationDuration: `${2 + Math.random() * 3}s`,
  },
}));
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

.character-page {
  background: $bg;
  min-height: 100vh;
  overflow-x: hidden;
  font-family: "Noto Sans SC", "Inter", sans-serif;
  color: $text;
  position: relative;
}

/* 动态背景 */
.bg-layer {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 0;
}

.halo {
  position: absolute;
  border-radius: 50%;
  border: 1px solid rgba($gold, 0.2);
  animation: rotate-halo 20s linear infinite;
  &.halo1 {
    width: 550px;
    height: 550px;
    top: -5%;
    left: -5%;
    border-color: rgba($gold, 0.18);
    animation-duration: 25s;
  }
  &.halo2 {
    width: 400px;
    height: 400px;
    bottom: 0;
    right: 0;
    border-color: rgba($gold-light, 0.2);
    animation-direction: reverse;
  }
  &.halo3 {
    width: 700px;
    height: 700px;
    top: 30%;
    left: 50%;
    transform: translate(-50%, -50%);
    border-color: rgba($blue-light, 0.12);
    animation-duration: 35s;
  }
}
@keyframes rotate-halo {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.feather {
  position: absolute;
  top: -10%;
  background: linear-gradient(
    to bottom,
    rgba($gold-light, 0.3),
    rgba(255, 255, 255, 0.9)
  );
  clip-path: polygon(50% 0%, 100% 40%, 80% 100%, 20% 100%, 0% 40%);
  opacity: 0;
  animation: fall-feather linear infinite;
  filter: drop-shadow(0 0 4px $gold-light);
}
@keyframes fall-feather {
  0% {
    transform: translateY(0) rotate(0deg) scale(0.8);
    opacity: 0;
  }
  10% {
    opacity: 0.6;
  }
  90% {
    opacity: 0.4;
  }
  100% {
    transform: translateY(110vh) rotate(360deg) scale(0.5);
    opacity: 0;
  }
}

.particle {
  position: absolute;
  border-radius: 50%;
  background: radial-gradient(circle, $gold-light, $gold);
  box-shadow: 0 0 10px rgba($gold, 0.7);
  animation: rise-particle linear infinite;
}
@keyframes rise-particle {
  0% {
    transform: translateY(0) translateX(0) scale(0.4);
    opacity: 0;
  }
  10% {
    opacity: 1;
  }
  90% {
    opacity: 0.5;
  }
  100% {
    transform: translateY(-110vh) translateX(20px) scale(0.1);
    opacity: 0;
  }
}

.star {
  position: absolute;
  border-radius: 50%;
  background: $white;
  box-shadow: 0 0 6px $blue-light;
  animation: twinkle ease-in-out infinite;
}
@keyframes twinkle {
  0%,
  100% {
    opacity: 0.2;
    transform: scale(0.8);
  }
  50% {
    opacity: 1;
    transform: scale(1.3);
  }
}

.soft-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 600px;
  height: 600px;
  background: radial-gradient(circle, rgba($blue-light, 0.08), transparent 70%);
  transform: translate(-50%, -50%);
  animation: soft-pulse 6s ease-in-out infinite;
}
@keyframes soft-pulse {
  0%,
  100% {
    opacity: 0.4;
    transform: translate(-50%, -50%) scale(0.9);
  }
  50% {
    opacity: 0.7;
    transform: translate(-50%, -50%) scale(1.1);
  }
}

.scanlines {
  position: absolute;
  inset: 0;
  background: repeating-linear-gradient(
    0deg,
    transparent,
    transparent 3px,
    rgba($blue, 0.03) 3px,
    rgba($blue, 0.03) 6px
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

.content {
  position: relative;
  z-index: 1;
  max-width: 1100px;
  margin: 0 auto;
  padding: 60px 20px 40px;
}

.hero {
  text-align: center;
  margin-bottom: 60px;
}
.crown-icon {
  font-size: 48px;
  color: $gold;
  filter: drop-shadow(0 0 12px $gold);
  animation: crown-float 3s infinite;
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
  font-size: 64px;
  font-weight: 800;
  letter-spacing: 10px;
  margin: 10px 0 5px;
  background: linear-gradient(180deg, #1a2634, #3a4b5e);
  background-clip: text;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  font-family: "Noto Serif SC", serif;
}
.name-sub {
  font-size: 18px;
  color: $text-dim;
  letter-spacing: 5px;
  margin-bottom: 20px;
}
.identifiers {
  display: flex;
  justify-content: center;
  gap: 16px;
  margin-bottom: 30px;
}
.badge {
  padding: 8px 24px;
  border: 1px solid $gold;
  border-radius: 24px;
  color: $gold;
  font-size: 14px;
  font-weight: 500;
  background: rgba($gold, 0.05);
  transition: all 0.3s;
  &:hover {
    background: rgba($gold, 0.15);
    box-shadow: 0 0 15px rgba($gold, 0.3);
    transform: translateY(-2px);
  }
}
.profile-card {
  background: $white;
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 8px 30px rgba(44, 58, 74, 0.08);
  table {
    width: 100%;
    border-collapse: collapse;
    th,
    td {
      padding: 12px;
      border-bottom: 1px solid #eaeff5;
      font-size: 15px;
    }
    th {
      color: $blue;
      font-weight: 600;
      width: 20%;
      text-align: right;
      padding-right: 16px;
    }
    td {
      text-align: left;
    }
  }
}

.block {
  margin-top: 60px;
}
.section-title {
  font-size: 28px;
  font-weight: 700;
  color: $text;
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 24px;
  font-family: "Noto Serif SC", serif;
  .gold-line {
    display: inline-block;
    width: 4px;
    height: 28px;
    background: linear-gradient(to bottom, $gold, $gold-light);
    border-radius: 2px;
  }
}

.image-gallery {
  display: flex;
  gap: 24px;
  justify-content: center;
  flex-wrap: wrap;
}
.img-card {
  background: $white;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.08);
  transition: transform 0.3s, box-shadow 0.3s;
  width: 260px;
  &:hover {
    transform: translateY(-6px) scale(1.02);
    box-shadow: 0 12px 28px rgba(0, 0, 0, 0.12);
  }
  img {
    width: 100%;
    height: auto;
    display: block;
  }
  span {
    display: block;
    text-align: center;
    padding: 10px;
    font-size: 14px;
    color: $text-dim;
  }
}
.floating-img {
  animation: float-up-down 5s ease-in-out infinite;
  &.delay1 {
    animation-delay: 1.5s;
  }
  &.delay2 {
    animation-delay: 3s;
  }
}
@keyframes float-up-down {
  0%,
  100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-6px);
  }
}

.cards-row {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
}
.hover-card {
  transition: all 0.3s;
  &:hover {
    transform: translateY(-5px);
    box-shadow: 0 12px 24px rgba(0, 0, 0, 0.1);
  }
}
.card {
  background: $white;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.04);
  h3 {
    color: $gold;
    margin-bottom: 10px;
  }
  p {
    color: $text-dim;
    font-size: 14px;
    line-height: 1.6;
  }
}

/* 装备子区域 */
.sub-section-title {
  font-size: 22px;
  font-weight: 700;
  color: $text;
  margin-bottom: 20px;
  display: flex;
  align-items: center;
  gap: 8px;
  .gold-dot {
    display: inline-block;
    width: 8px;
    height: 8px;
    background: $gold;
    border-radius: 50%;
  }
}
.equipment-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 20px;
  margin-bottom: 40px;
}
.equip-card {
  background: $white;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.04);
  overflow: hidden;
  display: flex;
  flex-direction: column;
  transition: all 0.3s;
  &:hover {
    transform: translateY(-5px);
    box-shadow: 0 12px 24px rgba(0, 0, 0, 0.1);
  }
  .equip-img {
    width: 100%;
    max-height: 220px; /* 限制最大高度，防止竖图过高 */
    object-fit: contain; /* 完整显示图片，不裁剪 */
    background: #f0f3f7; /* 浅灰背景填充空白 */
    padding: 8px;
    box-sizing: border-box;
  }
  .equip-info {
    padding: 16px;
    flex: 1;
  }
  .equip-name {
    font-size: 16px;
    font-weight: 700;
    color: $gold;
    margin: 0 0 8px;
  }
  .equip-desc {
    font-size: 13px;
    color: $text-dim;
    line-height: 1.5;
    margin: 0;
  }
}

.ability-dual {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 24px;
}
@media (max-width: 768px) {
  .ability-dual {
    grid-template-columns: 1fr;
  }
}
.ability-panel {
  background: $white;
  border-radius: 12px;
  padding: 24px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.04);
}
.panel-img {
  text-align: center;
  margin-bottom: 20px;
  .spirit-img {
    max-width: 200px;
    border-radius: 12px;
  }
  .img-label {
    display: block;
    margin-top: 8px;
    font-size: 14px;
    color: $gold;
    font-weight: 600;
  }
}
.stat-bars {
  margin: 16px 0;
}
.bar {
  display: flex;
  align-items: center;
  margin: 10px 0;
  gap: 8px;
}
.bar-label {
  width: 50px;
  font-weight: 600;
  color: $blue;
}
.bar-value {
  width: 40px;
  font-weight: 700;
  color: $gold;
  font-family: "Inter", monospace;
}
.bar-track {
  flex: 1;
  height: 8px;
  background: #eaf0f6;
  border-radius: 4px;
  overflow: hidden;
}
.bar-fill {
  height: 100%;
  background: linear-gradient(90deg, $gold-light, $blue-light);
  border-radius: 4px;
  transition: width 1s ease;
}
.devil-fill {
  background: linear-gradient(90deg, #c5a059, #6b5b3a);
}
.danger-tag {
  text-align: center;
  font-weight: 500;
  margin: 8px 0;
  color: #b88a3b;
}
.devil-tag {
  color: #8a4a5a;
}
.skill-title {
  font-size: 18px;
  font-weight: 700;
  color: $gold;
  margin: 20px 0 10px;
}
.skill-table {
  width: 100%;
  border-collapse: collapse;
  th,
  td {
    padding: 10px;
    border-bottom: 1px solid #eaeff5;
    font-size: 13px;
  }
  th {
    background: rgba($gold, 0.08);
    color: $gold;
  }
}
.devil-desc {
  color: $text-dim;
  font-size: 14px;
  line-height: 1.6;
  margin-top: 12px;
}

.rel-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 16px;
}
.rel-card {
  background: $white;
  border-radius: 12px;
  padding: 16px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.04);
  transition: all 0.3s;
  strong {
    color: $blue;
  }
  .rel-type {
    display: block;
    font-size: 12px;
    color: $gold;
    margin: 4px 0 8px;
    font-weight: 600;
  }
  p {
    font-size: 14px;
    color: $text-dim;
    margin: 0;
  }
}

.quote-wall {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
  justify-content: center;
}
.hover-quote {
  background: $white;
  border-left: 4px solid $gold;
  padding: 20px;
  border-radius: 0 12px 12px 0;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.04);
  font-style: italic;
  color: $text-dim;
  width: calc(50% - 16px);
  min-width: 250px;
  transition: all 0.3s;
  transform: rotate(var(--r, 0deg));
  margin: 0;
  position: relative;
  &::before {
    content: "“";
    font-size: 40px;
    color: $gold;
    position: absolute;
    top: -10px;
    left: 10px;
    opacity: 0.2;
  }
  &:hover {
    transform: rotate(0deg) scale(1.03);
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.1);
  }
}

.page-footer {
  text-align: center;
  margin-top: 60px;
  padding: 30px 0 20px;
  border-top: 1px solid rgba($gold, 0.2);
  color: $text-dim;
  font-size: 13px;
  p {
    margin: 4px 0;
  }
}
</style>
