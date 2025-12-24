<script setup>
import { RouterView } from 'vue-router'
import { onMounted } from 'vue'

// ==============================================
// 🐶 标题看门狗 (Title Watchdog)
// 作用：监控 document.title 的变化，一旦发现不对劲，立刻强制改回环境变量
// ==============================================
const startTitleWatchdog = () => {
  const envTitle = import.meta.env.VITE_SITE_TITLE
  
  // 如果没配置环境变量，就不启动看门狗，避免副作用
  if (!envTitle || envTitle.trim() === '') return

  // 1. 立即强制执行一次
  document.title = envTitle

  // 2. 使用 MutationObserver 监听 <title> 标签的变化
  const target = document.querySelector('title')
  if (target) {
    const observer = new MutationObserver(() => {
      if (document.title !== envTitle) {
        console.log(`[Title Watchdog] 检测到标题被篡改为 "${document.title}"，正在强制恢复为 "${envTitle}"`)
        document.title = envTitle
      }
    })
    
    observer.observe(target, { childList: true, subtree: true, characterData: true })
  }
  
  // 3. 定时器兜底 (防止 MutationObserver 还没生效时的极短间隙)
  setInterval(() => {
    if (document.title !== envTitle && document.title === '') {
      // 只有当标题变成空（显示域名）或者不正确时才修正
      document.title = envTitle
    }
  }, 500)
}

onMounted(() => {
  startTitleWatchdog()
})
</script>

<template>
  <div id="app">
    <!-- 背景层 -->
    <div class="bg-layer"></div>

    <!-- 页面内容 -->
    <div class="main-container">
      <router-view />
    </div>
  </div>
</template>
<style>
/* 背景层，固定在页面底部，永远显示 */
.bg-layer {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: -999; /* 永远在最底层 */
  background-image: url('https://pixabay.com/zh/images/download/x-10003814_1920.jpg');
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  background-attachment: fixed;
}
</style>
