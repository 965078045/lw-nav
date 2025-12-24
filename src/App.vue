<script setup>
import { RouterView } from 'vue-router'
import { onMounted } from 'vue'
import { useNavigation } from './apis/useNavigation'

const { categories, fetchCategories } = useNavigation()

onMounted(() => {
  fetchCategories()
})
</script>

<template>
  <div class="app-layout">
    <aside class="glass-sidebar">
      <div class="sidebar-header">
        <h2 class="logo-text">猫猫导航</h2>
      </div>
      
      <nav class="sidebar-nav">
        <a 
          v-for="cat in (categories.categories || categories)" 
          :key="cat.id" 
          :href="'#' + cat.id"
          class="sidebar-item"
        >
          <span class="sidebar-icon">{{ cat.icon || '📂' }}</span>
          <span class="sidebar-name">{{ cat.name }}</span>
        </a>
      </nav>
    </aside>

    <main class="main-body">
      <RouterView />
    </main>
  </div>
</template>

<style>
/* 侧边栏专属样式 */
.app-layout {
  display: flex;
  min-height: 100vh;
}

.glass-sidebar {
  width: 240px;
  height: 100vh;
  position: fixed;
  left: 0;
  top: 0;
  background: rgba(0, 0, 0, 0.3);
  backdrop-filter: blur(20px);
  border-right: 1px solid rgba(255, 255, 255, 0.1);
  display: flex;
  flex-direction: column;
  z-index: 100;
}

.sidebar-header {
  padding: 30px 20px;
  text-align: center;
}

.logo-text {
  font-size: 1.5rem;
  color: #42b883;
  margin: 0;
}

.sidebar-nav {
  flex: 1;
  padding: 10px;
  overflow-y: auto;
}

.sidebar-item {
  display: flex;
  align-items: center;
  padding: 12px 15px;
  margin-bottom: 5px;
  border-radius: 10px;
  color: rgba(255, 255, 255, 0.8);
  text-decoration: none;
  transition: all 0.3s;
}

.sidebar-item:hover {
  background: rgba(255, 255, 255, 0.1);
  color: #42b883;
  transform: translateX(5px);
}

.sidebar-icon {
  margin-right: 12px;
  font-size: 1.2rem;
}

/* 右侧内容偏移 */
.main-body {
  flex: 1;
  margin-left: 240px; /* 避开固定侧边栏 */
  padding: 20px;
}

@media (max-width: 768px) {
  .glass-sidebar { display: none; }
  .main-body { margin-left: 0; }
}
</style>
