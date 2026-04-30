<template>
  <div class="card">
    <h2>📝 數位測驗系統</h2>
    <div v-for="(q, idx) in questions" :key="idx" class="q-block">
      <p><strong>{{ idx + 1 }}. {{ q.title }}</strong></p>
      <button v-for="opt in q.options" :key="opt" 
              @click="answers[idx] = opt"
              :class="{ active: answers[idx] === opt }" class="opt-btn">
        {{ opt }}
      </button>
    </div>
    <button @click="showScore = true" class="submit-btn">交卷</button>
    <h3 v-if="showScore">得分：{{ score }}</h3>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
const questions = [
  { title: 'Vue.js 是哪種框架？', options: ['前端', '後端'], ans: '前端' },
  { title: 'Vite 的作用是？', options: ['建構工具', '資料庫'], ans: '建構工具' }
];
const answers = ref([]);
const showScore = ref(false);
const score = computed(() => {
  let s = 0;
  questions.forEach((q, i) => { if (answers.value[i] === q.ans) s += 50; });
  return s;
});
</script>

<style scoped>
.q-block { margin-bottom: 15px; text-align: left; }
.opt-btn { margin: 5px; padding: 5px 10px; cursor: pointer; border: 1px solid #ccc; background: white; }
.active { background: #35495e; color: white; }
.submit-btn { width: 100%; background: #35495e; color: white; padding: 10px; border: none; }
</style>