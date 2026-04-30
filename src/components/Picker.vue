<template>
  <div class="card">
    <h2>🎯 班級抽籤機</h2>
    <div ref="p5Container"></div>
    <button @click="startDraw" :disabled="isRolling" class="btn">
      {{ isRolling ? '抽選中...' : '開始抽籤' }}
    </button>
    <p v-if="winner">本次中獎：{{ winner }}</p>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import p5 from 'p5';

const p5Container = ref(null);
const students = ['王小明', '李小華', '張大強', '陳美美', '林小新'];
const winner = ref('');
const isRolling = ref(false);

const startDraw = () => {
  isRolling.value = true;
  let count = 0;
  const timer = setInterval(() => {
    winner.value = students[Math.floor(Math.random() * students.length)];
    count++;
    if (count > 15) {
      clearInterval(timer);
      isRolling.value = false;
    }
  }, 100);
};

onMounted(() => {
  new p5((p) => {
    p.setup = () => {
      p.createCanvas(300, 100).parent(p5Container.value);
      p.textAlign(p.CENTER, p.CENTER);
    };
    p.draw = () => {
      p.background(255);
      p.textSize(32);
      p.fill(isRolling.value ? 150 : '#42b883');
      p.text(winner.value || '準備抽籤', p.width/2, p.height/2);
    };
  });
});
</script>

<style scoped>
.btn { padding: 10px 20px; background: #42b883; color: white; border: none; border-radius: 5px; cursor: pointer; }
.card { border: 1px solid #ddd; padding: 20px; border-radius: 10px; margin: 10px; }
</style>