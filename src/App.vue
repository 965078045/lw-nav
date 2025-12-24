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
    <div class="main-container">
      <router-view />
    </div>
  </div>
</template>

<style scoped>
/* 你的样式保持不变 */
header {
  line-height: 1.5;
  max-height: 100vh;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}
</style>
