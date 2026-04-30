<script setup>
import { ref, computed } from 'vue';

// --- 狀態管理 ---
const currentTab = ref('quiz'); 
const showQuiz = ref(false); 
const submitted = ref(false);
const isSettled = ref(false); 

// --- 進階音效函數 ---
const playSound = (type) => {
  const ctx = new (window.AudioContext || window.webkitAudioContext)();
  
  if (type === 'click') {
    const osc = ctx.createOscillator();
    const gain = ctx.createGain();
    osc.connect(gain); gain.connect(ctx.destination);
    osc.frequency.setValueAtTime(400, ctx.currentTime);
    gain.gain.setValueAtTime(0.1, ctx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.1);
    osc.start(); osc.stop(ctx.currentTime + 0.1);
  } 
  else if (type === 'perfect' || type === 'victory') {
    // ✨ 模擬勝利歡呼音效 (多重振盪器疊加)
    const playNote = (freq, start, duration, vol) => {
      const osc = ctx.createOscillator();
      const g = ctx.createGain();
      osc.type = 'triangle';
      osc.connect(g); g.connect(ctx.destination);
      osc.frequency.setValueAtTime(freq, ctx.currentTime + start);
      osc.frequency.exponentialRampToValueAtTime(freq * 1.5, ctx.currentTime + start + duration);
      g.gain.setValueAtTime(0, ctx.currentTime + start);
      g.gain.linearRampToValueAtTime(vol, ctx.currentTime + start + 0.05);
      g.gain.linearRampToValueAtTime(0, ctx.currentTime + start + duration);
      osc.start(ctx.currentTime + start);
      osc.stop(ctx.currentTime + start + duration);
    };
    // 快速連音模擬歡呼聲
    playNote(523.25, 0, 0.5, 0.1); // Do
    playNote(659.25, 0.1, 0.5, 0.1); // Mi
    playNote(783.99, 0.2, 0.5, 0.1); // Sol
    playNote(1046.5, 0.3, 0.8, 0.1); // High Do
  }
};

// --- 1. [教學測驗] 內容鎖定，不變動 ---
const teachingContent = {
  title: "櫛瓜雞肉溫沙拉：健康小教室",
  points: [
    "櫛瓜：低卡路里且含有豐富維生素 C 與鉀。",
    "雞胸肉：提供高質量的蛋白質，幫助肌肉修復。",
    "橄欖油：健康的油脂能增加飽足感並保護心血管。",
    "溫沙拉：稍微加熱蔬菜，可以減少腸胃負擔並提升風味。"
  ]
};
const questions = ref([
  { id: 1, type: 'single', text: '櫛瓜又被稱為什麼？', options: ['小黃瓜', '夏南瓜', '冬瓜'], answer: '夏南瓜', userAnswer: null },
  { id: 2, type: 'single', text: '這道料理主要提供的營養素是？', options: ['醣類', '蛋白質與纖維', '飽和脂肪'], answer: '蛋白質與纖維', userAnswer: null },
  { id: 3, type: 'multiple', text: '櫛瓜雞肉溫沙拉常用的辛香料有哪些？(多選)', options: ['黑胡椒', '蒜末', '義大利香料', '砂糖'], answer: ['黑胡椒', '蒜末', '義大利香料'], userAnswer: [] },
  { id: 4, type: 'single', text: '烹飪櫛瓜時，建議加熱時間？', options: ['久煮至軟爛', '快速翻炒保持脆口', '不需要加熱'], answer: '快速翻炒保持脆口', userAnswer: null },
  { id: 5, type: 'single', text: '關於減脂料理的觀念，何者正確？', options: ['只能吃水煮肉', '完全不能攝取油脂', '挑選原型食物為主'], answer: '挑選原型食物為主', userAnswer: null }
]);
const quizScore = computed(() => {
  let total = 0;
  questions.value.forEach(q => {
    if (q.type === 'single' && q.userAnswer === q.answer) total += 20;
    else if (q.type === 'multiple' && q.userAnswer?.length === q.answer.length && q.userAnswer.every(v => q.answer.includes(v))) total += 20;
  });
  return total;
});

// --- 2. [小組計分] 維持自由取名、勝利動畫、新增歡呼聲 ---
const teams = ref([]); 
const newGroupName = ref(''); 
const addTeam = () => {
  if (newGroupName.value.trim()) {
    teams.value.push({ id: Date.now(), name: newGroupName.value, score: 0 });
    newGroupName.value = ''; playSound('click');
  }
};
const winnerTeamId = computed(() => {
  if (!isSettled.value || teams.value.length === 0) return null;
  const max = Math.max(...teams.value.map(t => t.score));
  const winners = teams.value.filter(t => t.score === max);
  return winners.length === 1 ? winners[0].id : 'tie';
});

const handleSettle = () => {
  isSettled.value = true;
  if (winnerTeamId.value && winnerTeamId.value !== 'tie') {
    playSound('victory'); // 觸發勝利歡呼
  }
};

// --- 3. [隨機抽籤] 維持自由名單 ---
const drawInput = ref(''); 
const drawList = ref([]); 
const winner = ref('');
const isDrawing = ref(false);
const addToDrawList = () => {
  if (drawInput.value.trim()) {
    drawList.value.push(drawInput.value.trim());
    drawInput.value = ''; playSound('click');
  }
};
const startDraw = () => {
  if (drawList.value.length < 2 || isDrawing.value) return;
  isDrawing.value = true; playSound('click');
  let count = 0;
  const timer = setInterval(() => {
    winner.value = drawList.value[Math.floor(Math.random() * drawList.value.length)];
    if (++count > 20) { clearInterval(timer); isDrawing.value = false; playSound('perfect'); }
  }, 80);
};
</script>

<template>
  <div class="cooking-app">
    <header class="main-header">
      <h1 class="page-title">🥗 {{ teachingContent.title }}</h1>
    </header>

    <nav class="nav-bar">
      <button :class="{ active: currentTab === 'quiz' }" @click="currentTab = 'quiz'; showQuiz = false">📖 教學測驗</button>
      <button :class="{ active: currentTab === 'score' }" @click="currentTab = 'score'">🏆 小組計分</button>
      <button :class="{ active: currentTab === 'draw' }" @click="currentTab = 'draw'">🎲 隨機抽籤</button>
    </nav>

    <div class="main-content">
      <section v-if="currentTab === 'quiz'" class="card">
        <div v-if="!showQuiz">
          <h3>課程重點筆記</h3>
          <ul style="text-align: left; line-height: 2;"><li v-for="p in teachingContent.points" :key="p">{{ p }}</li></ul>
          <button @click="showQuiz = true" class="action-btn">開始挑戰測驗</button>
        </div>
        <div v-else>
          <div v-for="(q, i) in questions" :key="q.id" class="q-card">
            <p>{{ i+1 }}. {{ q.text }}</p>
            <label v-for="o in q.options" :key="o" class="opt-label">
              <input :type="q.type === 'single' ? 'radio' : 'checkbox'" v-model="q.userAnswer" :value="o" @change="playSound('click')" :disabled="submitted"> {{ o }}
            </label>
            <div v-if="submitted" class="feedback">
              <span v-if="JSON.stringify(q.userAnswer) === JSON.stringify(q.answer)" style="color: green;">✅ 答對囉！</span>
              <span v-else style="color: red;">❌ 正解：{{ q.answer }}</span>
            </div>
          </div>
          <button v-if="!submitted" @click="submitted = true; if(quizScore === 100) playSound('perfect')" class="action-btn">提交作答</button>
          <div v-else class="result-summary"><h2>得分：{{ quizScore }}</h2><button @click="showQuiz=false; submitted=false" class="action-btn outline">返回教學</button></div>
        </div>
      </section>

      <section v-if="currentTab === 'score'" class="card">
        <h3>小組競賽評分</h3>
        <div v-if="!isSettled" class="input-row">
          <input v-model="newGroupName" @keyup.enter="addTeam" placeholder="輸入小組名..." class="styled-input">
          <button @click="addTeam" class="add-btn">新增</button>
        </div>
        <div class="team-list">
          <div v-for="t in teams" :key="t.id" class="team-item" :class="{ 'victory-anim': winnerTeamId === t.id }">
            <div class="team-info"><span v-if="winnerTeamId === t.id" class="win-badge">👑 WINNER</span>{{ t.name }}</div>
            <div class="counter">
              <button @click="t.score--" :disabled="isSettled" class="c-btn">-</button>
              <transition name="pop" mode="out-in"><span :key="t.score" class="score-val">{{ t.score }}</span></transition>
              <button @click="t.score++; playSound('click')" :disabled="isSettled" class="c-btn">+</button>
            </div>
          </div>
        </div>
        <div v-if="teams.length > 0" class="settle-area">
          <p v-if="winnerTeamId === 'tie'" class="tie-msg">🤝 平手！再接再厲</p>
          <button v-if="!isSettled" @click="handleSettle" class="action-btn">🏆 結算比賽</button>
          <button v-else @click="isSettled = false; teams = []" class="reset-link">重新比賽</button>
        </div>
      </section>

      <section v-if="currentTab === 'draw'" class="card">
        <h3>🎲 隨機抽籤機</h3>
        <div v-if="!isDrawing" class="input-row">
          <input v-model="drawInput" @keyup.enter="addToDrawList" placeholder="輸入名字..." class="styled-input">
          <button @click="addToDrawList" class="add-btn">加入</button>
        </div>
        <div class="name-tags"><span v-for="(n, idx) in drawList" :key="idx" class="tag">{{ n }}</span></div>
        <div class="winner-display">{{ winner || (drawList.length >= 2 ? '點擊開始' : '請先加入名單') }}</div>
        <button @click="startDraw" class="action-btn" :disabled="isDrawing || drawList.length < 2">開始抽籤</button>
      </section>
    </div>
  </div>
</template>

<style scoped>
/* 基本樣式鎖定 */
.cooking-app { font-family: sans-serif; background: #f4f7f6; min-height: 100vh; padding: 40px 20px; text-align: center; }
.page-title { color: #2E7D32; font-size: 2.5em; }
.nav-bar button { padding: 12px 24px; margin: 5px; cursor: pointer; border-radius: 50px; border: 1px solid #ccc; background: white; font-weight: bold; }
.nav-bar button.active { background: #4CAF50; color: white; border-color: #4CAF50; }
.card { background: white; padding: 40px; border-radius: 24px; max-width: 550px; margin: 0 auto; box-shadow: 0 15px 35px rgba(0,0,0,0.05); border-top: 10px solid #8BC34A; }

.action-btn { background: #4CAF50; color: white; border: none; width: 100%; padding: 16px; border-radius: 12px; cursor: pointer; font-size: 1.1em; font-weight: bold; margin-top: 20px; }
.action-btn.outline { background: white; color: #4CAF50; border: 2px solid #4CAF50; }

/* 勝利動畫 */
.victory-anim { background: #fffde7; border: 2px solid #fbc02d; transform: scale(1.06); animation: bounce 0.6s infinite alternate; }
@keyframes bounce { from { transform: scale(1.06); } to { transform: scale(1.1); } }
.win-badge { display: block; font-size: 12px; color: #f57c00; font-weight: bold; }

/* 縮放感樣式 */
.pop-enter-active { animation: pop-jump 0.3s ease-out; }
@keyframes pop-jump { 0% { transform: scale(1); } 50% { transform: scale(1.6); color: #FF9800; } 100% { transform: scale(1); } }

/* 其餘純淨樣式 */
.q-card { border-bottom: 1px solid #eee; padding: 15px 0; text-align: left; }
.team-item { display: flex; justify-content: space-between; align-items: center; padding: 15px; margin-bottom: 10px; border-radius: 15px; background: #fafafa; }
.score-val { display: inline-block; width: 40px; font-size: 24px; font-weight: bold; color: #4CAF50; }
.winner-display { font-size: 36px; color: #FB8C00; font-weight: bold; margin: 30px 0; }
.input-row { display: flex; gap: 10px; margin-bottom: 20px; }
.styled-input { flex: 1; padding: 12px; border-radius: 10px; border: 1px solid #ddd; }
.tag { background: #e8f5e9; padding: 5px 10px; border-radius: 5px; font-size: 12px; margin: 2px; display: inline-block; }
.reset-link { display: block; margin-top: 15px; color: #999; text-decoration: underline; background: none; border: none; cursor: pointer; width: 100%; }
</style>