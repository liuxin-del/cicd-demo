<template>
  <div>
    <canvas
      ref="canvasRef"
      :width="400"
      :height="200"
      style="margin: 20px; box-shadow: 0px 0px 4px #aaa; cursor: url(./pan.cur), auto;"
    ></canvas>
   
    <button @click="handleSubmit">提交签名</button>
    <button @click="handleClear">清除签名</button>
    <h1>哈哈哈哈</h1>
    <h2>咔咔咔咔</h2>
    <h3>啦啦啦啦</h3>
    <h4>lalalall</h4>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import axios from 'axios';
// import { useStore } from 'vuex';

axios.defaults.baseURL = 'http://localhost:3000';

const canvasRef = ref(null);
const ctxRef = ref(null);

const initCanvas = () => {
  const canvas = canvasRef.value;
  const ctx = canvas.getContext('2d');
   ctx.lineWidth = 2;  // 设置线条宽度
  ctx.strokeStyle = '#000';  // 设置线条颜色
  ctx.lineCap = 'round';  // 设置线条末端样式
  canvasRef.value = ctx;
};

const handleMouseDown = (e) => {
  const ctx = ctxRef.value;
  if (!ctx) return;  // 确保 ctx 存在
  const rect = canvasRef.value.getBoundingClientRect();
  const x = e.clientX - rect.left;
  const y = e.clientY - rect.top;
  ctx.beginPath();
  ctx.moveTo(x, y);
  const handleMouseMove = (ev) => {
    const moveX = ev.clientX - rect.left;
    const moveY = ev.clientY - rect.top;
    ctx.lineTo(moveX, moveY);
    ctx.stroke();
  };
  // 监听鼠标移动
  document.addEventListener('mousemove', handleMouseMove);
  // 鼠标松开后移除事件监听
  const handleMouseUp = () => {
    document.removeEventListener('mousemove', handleMouseMove);
    document.removeEventListener('mouseup', handleMouseUp);
  };
  document.addEventListener('mouseup', handleMouseUp);
};

const handleClear = () => {
  const ctx = ctxRef.value;
  const canvas = canvasRef.value;
  ctx.clearRect(0, 0, canvas.width, canvas.height);
};

const handleSubmit = async () => {
  const canvas = canvasRef.value;
  const dataUrl = canvas.toDataURL('image/png');
  console.log('Signature Path', dataUrl);
  const { data: { code } } = await axios.post('/list', { dataUrl: dataUrl });
  if (code === 200) {
    alert('Signature successful');
    // 下载图片功能
    const a = document.createElement('a');
    a.href = dataUrl;
    a.download = 'signature';
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
  } else {
    alert('Signature failed');
  }
};

onMounted(() => {
  initCanvas();
  const canvas = canvasRef.value;
  // 绑定原生鼠标事件
  canvas.addEventListener('mousedown', handleMouseDown);
});

onBeforeUnmount(() => {
  // 清理事件监听器
  canvasRef.value.removeEventListener('mousedown', handleMouseDown);
});
</script>

<style scoped>
/* Your styles here */
</style>