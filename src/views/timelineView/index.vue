<template>
  <div class="timeline-page">
    <!-- 反转体风格背景 -->
    <div class="bg-layer">
      <!-- 破碎的黑色王冠光环 -->
      <div class="broken-crown-ring"></div>
      <div class="broken-crown-ring ring2"></div>
      <!-- 下沉的蓝黑光粒（数量增加、尺寸加大） -->
      <span
        v-for="p in particles"
        :key="p.id"
        class="particle"
        :style="p.style"
      ></span>
      <!-- 暗色羽翼投影 -->
      <div class="wing-shadow"></div>
      <!-- 缓慢扩散的黑色涟漪 -->
      <div class="ripple"></div>
      <div class="ripple ripple2"></div>
      <!-- 裂痕 -->
      <div class="crack-lines">
        <span
          class="crack"
          v-for="i in 6"
          :key="'c' + i"
          :style="crackStyle(i)"
        ></span>
      </div>
      <!-- 扫描线 -->
      <div class="scanlines"></div>
      <!-- 深色光晕 -->
      <div class="deep-gloom"></div>
    </div>

    <main class="content">
      <header class="hero">
        <h1 class="title">角色经历</h1>
        <p class="subtitle">鸢一折纸 · Tobiichi Origami</p>
        <div class="mode-badge">救世魔王 · Satan</div>
      </header>

      <div class="timeline">
        <!-- 所有事件节点硬编码，保持独特风格 -->
        <div class="tl-node left">
          <div class="tl-card">
            <div class="card-header">
              <h3 class="card-title">幼年时期</h3>
              <span class="card-episode">起源</span>
            </div>
            <p>
              幼年的折纸拥有一个美好的童年，在家庭的包容中快乐地长大。然而<strong>十一岁时</strong>，因“天使”（精灵）的攻击而令双亲丧命。在绝望之际遇到了来自五年后的士道，被他保护安慰（使用了七罪的赝造魔女而变小），于是将自己的<strong>微笑和眼泪托付给了士道</strong>，将仇恨和愤恨留给自己。
            </p>
            <p>
              从此以后极端地憎恨着“精灵”，为了不会再让别人遇上和自己一样的惨剧并且为父母报仇，她选择加入了<strong>陆上自卫队对精灵部队AST</strong>，以消灭“精灵”为目标而奋斗。直到再次遇到士道之前，阶级已经是<strong>上士</strong>。
            </p>
            <div class="axis-marker"></div>
          </div>
        </div>

        <div class="tl-node right">
          <div class="tl-card">
            <img :src="origamiVsTohka" alt="折纸与十香" class="card-img" />
            <div class="card-header">
              <h3 class="card-title">十香时期</h3>
              <span class="card-episode">第一季 第1-6话</span>
            </div>
            <p>
              二年级分班与五河士道同班并坐在他旁边，此时的折纸早已是学校中的名人。十香出现引发空间震后，她立刻更换装备前往迎战。在与十香对峙中发现了士道，为了保护士道前去作战，使得〈Ratatoskr〉司令官五河琴里顺利回收士道。
            </p>
            <p>
              次日折纸在放学时找到士道，询问他为何出现在那里，并告知自己为何加入AST及父母被精灵所害的事。之后在走廊中遇到正在练习约会技巧的士道，突然被士道表白并<strong>毫不犹豫地答应</strong>。十香出现在来禅高中后，她因过度担心士道而独自冲上前对战，误伤士道导致其“死亡”与十香暴走。事后向士道道歉并得到原谅。
            </p>
            <div class="axis-marker"></div>
          </div>
        </div>

        <div class="tl-node left">
          <div class="tl-card">
            <img :src="maidOutfit" alt="女仆装" class="card-img" />
            <div class="card-header">
              <h3 class="card-title">四糸乃时期</h3>
              <span class="card-episode">第一季 第7-9话</span>
            </div>
            <p>
              与十香处处作对，从士道与殿町宏人的交谈中记下士道“喜欢女仆”一事。四糸乃出现后参与追击，将四糸乃的手偶<strong>四糸奈</strong>带回家中。士道为夺回手偶登门拜访，折纸穿着<strong>女仆装</strong>迎接，并献上混合精力饮料的水。她以骑乘姿势坐在士道身上，以<strong>用名字称呼自己</strong>为条件才肯下来，达成目的后离开洗澡，给了士道找到手偶的机会。
            </p>
            <div class="axis-marker"></div>
          </div>
        </div>

        <div class="tl-node right">
          <div class="tl-card">
            <img :src="vsKurumi" alt="被狂三压制" class="card-img" />
            <div class="card-header">
              <h3 class="card-title">狂三时期</h3>
              <span class="card-episode">第一季 第10-12话</span>
            </div>
            <p>
              真那被调到AST后，折纸与真那因训练损坏装备被燎子训斥。发现真那的哥哥就是士道后，<strong>自称士道的恋人</strong>，令真那称呼自己为“大嫂”。时崎狂三出现并自称为精灵，折纸当晚参与抹杀行动。狂三死而复生后，折纸被狂三压制，狂三称她“非常美味”。
            </p>
            <p>
              为保护士道，她决定在周末与士道约会，致使士道陷入“三重约会”境界。狂三袭击学院时，折纸使用临时作战装备与狂三分身战斗，在天台正面交锋时受伤昏倒，被琴里所救。
            </p>
            <div class="axis-marker"></div>
          </div>
        </div>

        <div class="tl-node left">
          <div class="tl-card">
            <img :src="hospitalReturn" alt="回病房" class="card-img" />
            <div class="card-header">
              <h3 class="card-title">琴里时期</h3>
              <span class="card-episode">第二季 第1-3话</span>
            </div>
            <p>
              因狂三战斗受伤住院，恰遇前来找真那的士道，要求他背自己回病房并削苹果喂食。出院后观看录像发现<strong>琴里就是炎之精灵</strong>。误认琴里为杀害双亲的凶手后，不顾一切盗走实验型重装CR-Unit〈White
              Lycoris〉，攻击正在游乐园与士道约会的琴里。最后在十香与四糸乃的阻拦及士道舍命劝说下，超过活动极限而昏倒，被AST成员逮捕。
            </p>
            <div class="axis-marker"></div>
          </div>
        </div>

        <div class="tl-node right">
          <div class="tl-card">
            <div class="card-header">
              <h3 class="card-title">八舞时期</h3>
              <span class="card-episode">第二季 第4-6话</span>
            </div>
            <p>
              琴里事件中行为违反纪律，AST上层决定将她开除，但DEM总裁维斯考特突然出现说情，改为<strong>停职两个月</strong>。维斯考特在背地里称折纸为“一万个人中才有一个的天才”。修学旅行中与士道一组，识破殿町宏人的偷窥计划，并担任<strong>八舞夕弦的恋爱指导员</strong>，被夕弦尊称为“折纸大师”。遇险时被令音所救，表达了想变强的愿望。
            </p>
            <div class="axis-marker"></div>
          </div>
        </div>

        <div class="tl-node left">
          <div class="tl-card">
            <img :src="protectShido" alt="保护士道" class="card-img" />
            <div class="card-header">
              <h3 class="card-title">美九时期</h3>
              <span class="card-episode">第二季 第7-10话</span>
            </div>
            <p>
              停职结束后与AST一起出动迎击美九，差点被其控制但迅速反击。获悉DEM将在天央祭捉拿士道和十香的消息后，再次无视命令启动〈White
              Lycoris〉保护士道。与DEM社作战时达到活动界限陷入昏迷，被真那所救。苏醒后为救士道，在队友帮助下获得前SSS成员的CR-UNIT，赶往DEM日本分社，<strong>与艾伦一对一交锋</strong>，虽因实力差距被打败，但在〈Ratatoskr〉掩护下成功击伤艾伦迫使其撤退，成为<strong>第二个能伤到艾伦的人</strong>。
            </p>
            <div class="axis-marker"></div>
          </div>
        </div>

        <div class="tl-node right">
          <div class="tl-card">
            <img :src="pointOutCulprit" alt="和十香指出犯人" class="card-img" />
            <div class="card-header">
              <h3 class="card-title">七罪时期</h3>
              <span class="card-episode">第三季 第1-3话</span>
            </div>
            <p>
              七罪变身成士道作恶，折纸通过<strong>身体微妙的差别</strong>轻易识破。在七罪的游戏中被当作嫌犯之一，率先利用自己对士道的全面了解<strong>摆脱了嫌疑</strong>，并帮助士道寻找真正的七罪。七罪被识破后将众人变成小孩，折纸在混乱中自称<strong>“五河千代纸”</strong>——士道与折纸之女。之后与艾伦相遇被诱导，选择<strong>加入DEM社</strong>，身穿CR-Unit成功击毁反维斯考特派扔下天宫市的第三颗人造卫星。
            </p>
            <div class="axis-marker"></div>
          </div>
        </div>

        <div class="tl-node left highlight">
          <div class="tl-card">
            <img :src="vsTohkaAgain" alt="和十香对峙" class="card-img" />
            <div class="card-header">
              <h3 class="card-title">化身精灵</h3>
              <span class="card-episode">第三季 第4-8话</span>
            </div>
            <p>
              与十香等人战斗，起初占据上风将显现限定灵装的精灵们打败，但不敌完全灵装的十香。落败后与<幻影>相遇，<strong>被赋予灵结晶而精灵化</strong>。显现天使<strong>〈绝灭天使〉</strong>与十香对战，逼使十香使用【最后之剑】。士道突然现身后，她不欲让士道看见精灵化的自己而离去。
            </p>
            <p>
              为追寻杀害双亲的<幻影>，请求狂三使用<十二之弹>将自己送回五年前。回到过去后发动【炮冠】企图杀死〈幻影〉，却误杀了自己的父母——发现<strong>杀死父母的“天使”就是自己</strong>后精神崩溃，<strong>灵结晶反转</strong>，以反转体状态回到现世，显现魔王<strong>〈救世魔王〉</strong>对整个天宫市展开无差别攻击。士道在狂三帮助下两次使用<十二之弹>回到过去，改变了折纸的未来。
            </p>
            <div class="axis-marker"></div>
          </div>
        </div>

        <div class="tl-node right">
          <div class="tl-card">
            <img :src="newWorldOrigami" alt="世界线折纸" class="card-img" />
            <div class="card-header">
              <h3 class="card-title">新世界线</h3>
              <span class="card-episode">第三季 第9-12话</span>
            </div>
            <p>
              世界线改变后，留起长发的折纸以<strong>转学生身份</strong>来到来禅高中，与士道再次会面。因十二之弹的作用，折纸仍保留原世界线的<strong>精灵身份与双重人格</strong>——在发觉精灵或灵力时原本人格会以反转精灵状态苏醒。与士道约会时，士道凭借对折纸的了解使其好感急速增加。士道受伤展现灵力时折纸反转显现<strong>〈救世魔王〉</strong>攻击。
            </p>
            <p>
              士道的呼唤使两个人格融合，折纸在精神崩坏时被挽回，<strong>取回了微笑</strong>。士道主动亲吻成功封印灵力。返校后折纸剪回短发，对精灵的态度柔和了很多，也开始露出微笑了。
            </p>
            <div class="axis-marker"></div>
          </div>
        </div>

        <div class="tl-node left">
          <div class="tl-card">
            <img :src="eveningDress" alt="晚礼服" class="card-img" />
            <div class="card-header">
              <h3 class="card-title">暴走士道时期</h3>
              <span class="card-episode">第三季 第12话 / 第四季 第1话</span>
            </div>
            <p>
              士道吸收过多灵力暴走，需精灵进行逆攻略。折纸建议全体进行泳装攻势，被DEM打断后改为<strong>化妆舞会攻势</strong>。与士道共饮汽水时想下药却被另一人格阻止，误将药撒到自己杯中，陷入混沌状态向士道直接发出<strong>爱的告白</strong>，歪打正着成功攻略士道。
            </p>
            <p>
              琴里受命用轨道武器抹杀失控的士道时，折纸用手枪阻止。在<幻影>协助下显现限定灵装和天使，最终利用瞬移到士道怀里完成封印，士道转危为安。
            </p>
            <div class="axis-marker"></div>
          </div>
        </div>

        <div class="tl-node right">
          <div class="tl-card">
            <img :src="createDoujin" alt="制作同人本" class="card-img" />
            <div class="card-header">
              <h3 class="card-title">二亚时期</h3>
              <span class="card-episode">第四季 第2-5话</span>
            </div>
            <p>
              为攻略二亚，折纸在七罪的提示下提出<strong>制作士道的同人本</strong>的提案。与七罪、八舞姐妹、士道不分日夜工作完成了同人本。在Comiket上遇到AST前队长燎子和队友美纪惠、米琪，向她们销售同人本。二亚看了同人本后被深深感动。二亚反转后，折纸显现天使协助士道突破异形重围，并在二亚苏醒后提示她可能可以显现限定灵装。
            </p>
            <div class="axis-marker"></div>
          </div>
        </div>

        <div class="tl-node left">
          <div class="tl-card">
            <img :src="shrineVisit" alt="神社参拜" class="card-img" />
            <div class="card-header">
              <h3 class="card-title">六喰时期</h3>
              <span class="card-episode">第四季 第6-12话</span>
            </div>
            <p>
              六喰向地球扔陨石警告DEM社，来禅高中也涉入其中。随士道登上<strong>〈佛拉克西纳斯EX〉</strong>前往宇宙。被维斯考特关入童话世界，扮演<strong>《国王的新装》里的国王</strong>，后被士道救出。琴里赐予折纸CR-Unit〈Brunnhilde〉，身穿装备与反转十香战斗，协助士道阻止六喰。
            </p>
            <p>
              回到地球后被六喰封印对士道的记忆，但因新世界人格恢复记忆。反转十香出现后，折纸提议<strong>争取士道的嘴唇来代替战斗</strong>。在天宫塔与士道一同出现阻止十香与六喰的战斗，身穿〈Brunnhilde〉并显现限定灵装协助士道封印六喰。
            </p>
            <div class="axis-marker"></div>
          </div>
        </div>
      </div>

      <footer class="page-footer">
        <p>救世魔王 · Satan</p>
        <p>© 2026 鸢一折纸电子设定集</p>
      </footer>
    </main>
  </div>
</template>

<script setup lang="ts">
import { onMounted } from "vue";
import origamiVsTohka from "./img/折纸与十香.webp";
import maidOutfit from "./img/女仆装.webp";
import vsKurumi from "./img/被狂三压制.webp";
import hospitalReturn from "./img/回病房.webp";
import protectShido from "./img/保护士道.webp";
import pointOutCulprit from "./img/和十香指出犯人.webp";
import vsTohkaAgain from "./img/和十香对峙.webp";
import newWorldOrigami from "./img/世界线折纸.webp";
import eveningDress from "./img/晚礼服.webp";
import createDoujin from "./img/制作同人本.webp";
import shrineVisit from "./img/神社参拜.webp";

// 粒子数量增加至80，尺寸加大（3~9px）
const particles = Array.from({ length: 80 }, (_, i) => ({
  id: i,
  style: {
    width: `${3 + Math.random() * 6}px`,
    height: `${3 + Math.random() * 6}px`,
    left: `${Math.random() * 100}%`,
    top: `${Math.random() * 100}%`,
    animationDuration: `${10 + Math.random() * 15}s`,
    animationDelay: `${Math.random() * -20}s`,
  },
}));

const crackStyle = (i: number) => ({
  animationDelay: `${i * 0.6}s`,
});

onMounted(() => {
  const observer = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) entry.target.classList.add("visible");
      });
    },
    { threshold: 0.15 }
  );
  document.querySelectorAll(".tl-card").forEach((el) => observer.observe(el));
});
</script>

<style scoped lang="scss">
/* 反转体专用配色（增强对比度） */
$bg-deep: #0a0d14;
$card-bg: rgba(20, 24, 38, 0.92); /* 卡片背景微调，更利于文字衬托 */
$blue-black: #1e2a3a; /* 灵装主色：蓝黑 */
$dark-accents: #0f151e;
$pale-blue: #b0c4d8; /* 正文颜色大幅提亮 */
$pale-dim: #8a9daf; /* 辅助文字提亮 */
$strong-blue: #7ea3c8; /* 强调色提亮 */
$border-subtle: rgba(#b0c4d8, 0.2);

.timeline-page {
  background: $bg-deep;
  min-height: 100vh;
  font-family: "Noto Sans SC", "Inter", sans-serif;
  color: $pale-blue;
  position: relative;
  overflow-x: hidden;
}

/* 背景样式保持不变（光环、羽翼、涟漪、裂痕、扫描线等） */
.bg-layer {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 0;
}

.broken-crown-ring {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 500px;
  height: 500px;
  transform: translate(-50%, -50%);
  border: 2px solid transparent;
  border-top-color: rgba(#b0c4d8, 0.2);
  border-right-color: rgba(#b0c4d8, 0.15);
  border-radius: 50%;
  animation: rotate-ring 25s linear infinite;
  box-shadow: 0 0 30px rgba(#1e2a3a, 0.3);
  &.ring2 {
    width: 400px;
    height: 400px;
    border-top-color: transparent;
    border-bottom-color: rgba(#b0c4d8, 0.25);
    border-left-color: rgba(#b0c4d8, 0.1);
    animation-direction: reverse;
    animation-duration: 20s;
  }
}

@keyframes rotate-ring {
  from {
    transform: translate(-50%, -50%) rotate(0deg);
  }
  to {
    transform: translate(-50%, -50%) rotate(360deg);
  }
}

.particle {
  position: absolute;
  border-radius: 50%;
  background: radial-gradient(circle, $pale-blue, $blue-black);
  box-shadow: 0 0 8px rgba(#b0c4d8, 0.6);
  animation: sink-particle linear infinite;
}

@keyframes sink-particle {
  0% {
    transform: translateY(0) translateX(0) scale(0.5);
    opacity: 0;
  }
  10% {
    opacity: 0.9;
  }
  90% {
    opacity: 0.4;
  }
  100% {
    transform: translateY(110vh) translateX(-10px) scale(0.2);
    opacity: 0;
  }
}

.wing-shadow {
  position: absolute;
  top: -10%;
  left: -20%;
  width: 140%;
  height: 140%;
  background: radial-gradient(
      ellipse at 30% 50%,
      rgba(#1e2a3a, 0.3),
      transparent 60%
    ),
    radial-gradient(ellipse at 70% 50%, rgba(#0f151e, 0.3), transparent 60%);
  animation: wing-breathe 8s ease-in-out infinite;
}

@keyframes wing-breathe {
  0%,
  100% {
    opacity: 0.6;
    transform: scale(1);
  }
  50% {
    opacity: 0.9;
    transform: scale(1.05);
  }
}

.ripple {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 300px;
  height: 300px;
  transform: translate(-50%, -50%);
  border: 1px solid rgba(#b0c4d8, 0.15);
  border-radius: 50%;
  animation: ripple-expand 6s ease-out infinite;
  &.ripple2 {
    animation-delay: 3s;
  }
}

@keyframes ripple-expand {
  0% {
    width: 100px;
    height: 100px;
    opacity: 0.4;
  }
  100% {
    width: 600px;
    height: 600px;
    opacity: 0;
  }
}

.crack-lines {
  .crack {
    position: absolute;
    height: 1px;
    background: linear-gradient(90deg, transparent, $pale-dim, transparent);
    animation: crack-flicker 5s infinite;
  }
  .crack:nth-child(1) {
    top: 15%;
    left: 5%;
    width: 350px;
    transform: rotate(10deg);
  }
  .crack:nth-child(2) {
    top: 50%;
    left: 40%;
    width: 250px;
    transform: rotate(-8deg);
  }
  .crack:nth-child(3) {
    top: 75%;
    left: 60%;
    width: 300px;
    transform: rotate(20deg);
  }
  .crack:nth-child(4) {
    top: 30%;
    left: 70%;
    width: 200px;
    transform: rotate(-15deg);
  }
  .crack:nth-child(5) {
    top: 8%;
    left: 55%;
    width: 180px;
    transform: rotate(5deg);
  }
  .crack:nth-child(6) {
    top: 85%;
    left: 15%;
    width: 280px;
    transform: rotate(-3deg);
  }
}

@keyframes crack-flicker {
  0%,
  100% {
    opacity: 0.1;
  }
  50% {
    opacity: 0.5;
    box-shadow: 0 0 15px rgba(#b0c4d8, 0.3);
  }
}

.scanlines {
  position: absolute;
  inset: 0;
  background: repeating-linear-gradient(
    0deg,
    transparent,
    transparent 3px,
    rgba(#b0c4d8, 0.03) 3px,
    rgba(#b0c4d8, 0.03) 6px
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

.deep-gloom {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 600px;
  height: 600px;
  background: radial-gradient(circle, rgba(#1e2a3a, 0.4), transparent 70%);
  transform: translate(-50%, -50%);
  filter: blur(40px);
}

/* ========== 内容区（优化可读性） ========== */
.content {
  position: relative;
  z-index: 1;
  max-width: 1100px;
  margin: 0 auto;
  padding: 50px 20px 30px;
}

.hero {
  text-align: center;
  margin-bottom: 50px;
}
.title {
  font-size: 58px;
  font-weight: 800;
  letter-spacing: 8px;
  margin: 10px 0 5px;
  color: #d0e0f0;
  text-shadow: 0 0 25px rgba(#7ea3c8, 0.4);
  font-family: "Noto Serif SC", serif;
}
.subtitle {
  font-size: 18px;
  color: $pale-dim;
  letter-spacing: 4px;
  margin-bottom: 12px;
}
.mode-badge {
  display: inline-block;
  padding: 5px 24px;
  border: 1px solid $border-subtle;
  border-radius: 20px;
  color: $pale-blue;
  font-size: 14px;
  background: rgba(#1e2a3a, 0.5);
  letter-spacing: 2px;
}

/* 时间线主轴 */
.timeline {
  position: relative;
}
.timeline::before {
  content: "";
  position: absolute;
  left: 50%;
  top: 0;
  bottom: 0;
  width: 2px;
  background: linear-gradient(
    to bottom,
    transparent,
    $blue-black,
    $pale-dim,
    $blue-black,
    transparent
  );
  transform: translateX(-50%);
}

.tl-node {
  display: flex;
  margin-bottom: 28px;
  align-items: flex-start;
}
.tl-node.left {
  justify-content: flex-start;
  .tl-card {
    margin-right: auto;
  }
  .axis-marker {
    right: -22px;
  }
}
.tl-node.right {
  justify-content: flex-end;
  .tl-card {
    margin-left: auto;
  }
  .axis-marker {
    left: -22px;
  }
}
.tl-node.highlight .tl-card {
  border-color: $pale-blue;
  box-shadow: 0 0 20px rgba(#7ea3c8, 0.4);
}

.tl-card {
  width: calc(50% - 30px);
  background: $card-bg;
  backdrop-filter: blur(10px);
  border: 1px solid $border-subtle;
  border-radius: 12px;
  padding: 18px 20px;
  transition: all 0.35s ease;
  opacity: 0;
  transform: translateY(20px);
  &.visible {
    opacity: 1;
    transform: translateY(0);
  }
  &:hover {
    transform: translateY(-4px) scale(1.02);
    border-color: $pale-blue;
    box-shadow: 0 8px 25px rgba(0, 0, 0, 0.6);
  }
  .card-img {
    width: 100%;
    height: auto;
    max-height: 240px;
    object-fit: contain;
    background: rgba(0, 0, 0, 0.4);
    border-radius: 8px;
    margin-bottom: 12px;
    border: 1px solid $border-subtle;
  }
  .card-header {
    display: flex;
    align-items: baseline;
    justify-content: space-between;
    margin-bottom: 10px;
  }
  .card-title {
    font-size: 22px;
    font-weight: 700;
    color: $strong-blue;
    font-family: "Noto Serif SC", serif;
    margin: 0;
  }
  .card-episode {
    font-size: 13px;
    color: $pale-dim;
    font-weight: 600;
    background: rgba(#1e2a3a, 0.5);
    padding: 3px 10px;
    border-radius: 12px;
    border: 1px solid $border-subtle;
    font-family: "Inter", monospace;
    white-space: nowrap;
  }
  p {
    font-size: 15px;
    line-height: 1.7;
    color: $pale-dim;
    margin: 0 0 8px;
    strong {
      color: $pale-blue;
    }
  }
  .axis-marker {
    position: absolute;
    top: 24px;
    width: 14px;
    height: 14px;
    background: $blue-black;
    border: 2px solid $pale-dim;
    border-radius: 50%;
    transition: all 0.3s;
    z-index: 2;
  }
}

@media (max-width: 768px) {
  .timeline::before {
    left: 20px;
  }
  .tl-node {
    justify-content: flex-start !important;
    margin-bottom: 24px;
  }
  .tl-node .tl-card {
    width: calc(100% - 45px);
    margin-left: 45px !important;
    margin-right: 0 !important;
  }
  .tl-node .axis-marker {
    left: -28px !important;
    right: auto !important;
  }
  .title {
    font-size: 42px;
  }
}

.page-footer {
  text-align: center;
  margin-top: 50px;
  padding: 25px 0 15px;
  border-top: 1px solid $border-subtle;
  color: $pale-dim;
  font-size: 13px;
  p {
    margin: 3px 0;
  }
}
</style>
