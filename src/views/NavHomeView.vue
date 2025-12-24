<script setup>
import { onMounted, ref, computed } from 'vue'
// 确保这个路径是对的，如果你的文件在 src/apis 下
import { useNavigation } from '../apis/useNavigation'

// 1. 获取数据逻辑 (功能保留)
const { categories, fetchCategories, loading } = useNavigation()
const searchQuery = ref('')
const activeEngine = ref('bing') // 默认搜索引擎

// 2. 搜索引擎配置
const engines = {
  bing: { name: '必应', url: 'https://cn.bing.com/search?q=' },
  google: { name: '谷歌', url: 'https://www.google.com/search?q=' },
  baidu: { name: '百度', url: 'https://www.baidu.com/s?wd=' }
}

// 3. 混合搜索逻辑：既搜外网，也过滤站内
const handleSearch = () => {
  if (!searchQuery.value) return
  // 打开外部搜索
  window.open(engines[activeEngine.value].url + encodeURIComponent(searchQuery.value), '_blank')
}

// 4. 实时过滤站内卡片 (功能保留)
const filteredCategories = computed(() => {
  if (!searchQuery.value) return categories.value
  // 深度过滤：只显示包含关键词的链接
  return categories.value.map(cat => ({
    ...cat,
    links: cat.links.filter(link => 
      link.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      (link.desc && link.desc.toLowerCase().includes(searchQuery.value.toLowerCase()))
    )
  })).filter(cat => cat.links && cat.links.length > 0)
})

onMounted(() => {
  fetchCategories()
})
</script>

<template>
  <div class="container">
    
    <header class="hero-section">
      <div class="logo-area">
        <h1>我的导航站</h1>
        <p>探索互联网的入口</p>
      </div>

      <div class="search-box-wrapper">
        <div class="engine-tabs">
          <span 
            v-for="(eng, key) in engines" 
            :key="key"
            :class="{ active: activeEngine === key }"
            @click="activeEngine = key"
          >
            {{ eng.name }}
          </span>
        </div>
        <div class="input-group">
          <input 
            v-model="searchQuery" 
            @keyup.enter="handleSearch"
            type="text" 
            :placeholder="`在 ${engines[activeEngine].name} 中搜索，或查找站内书签...`"
          />
          <button @click="handleSearch" class="search-btn">🔍</button>
        </div>
      </div>
    </header>

    <div v-if="loading" class="loading">
      <div class="spinner"></div>
      <p>正在同步 GitHub 数据...</p>
    </div>

    <main v-else class="content-grid">
      <div v-if="filteredCategories.length === 0" class="empty-state">
        没有找到相关内容，按回车试试全网搜索？
      </div>

      <section 
        v-for="cat in filteredCategories" 
        :key="cat.id" 
        class="category-card"
      >
        <h2 class="cat-title">
          <span class="icon">{{ cat.icon || '📂' }}</span> 
          {{ cat.name }}
        </h2>
        
        <div class="links-wrapper">
          <a 
            v-for="link in cat.links" 
            :key="link.url"
            :href="link.url"
            target="_blank"
            class="link-item"
            :title="link.desc"
          >
            <img 
              :src="`https://favicon.duckduckgo.com/v2/?url=${link.url}`" 
              class="favicon"
              @error="(e) => e.target.style.opacity = 0"
            />
            <span class="link-name">{{ link.name }}</span>
          </a>
        </div>
      </section>
    </main>

    <footer>
      <p>© 2024 Powered by Vue & Cloudflare</p>
    </footer>

  </div>
</template>

<style scoped>
/* 布局容器 */
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 40px 20px;
}

/* 顶部区域 */
.hero-section {
  text-align: center;
  margin-bottom: 60px;
  animation: fadeIn 1s ease;
}

.logo-area h1 {
  font-size: 2.5rem;
  margin-bottom: 10px;
  text-shadow: 0 4px 12px rgba(0,0,0,0.3);
}

/* 搜索框美化 */
.search-box-wrapper {
  max-width: 600px;
  margin: 30px auto 0;
  background: var(--glass-bg);
  backdrop-filter: blur(12px);
  padding: 15px;
  border-radius: 20px;
  border: var(--glass-border);
  box-shadow: 0 8px 32px rgba(0,0,0,0.1);
}

.engine-tabs {
  display: flex;
  gap: 15px;
  margin-bottom: 10px;
  padding-left: 10px;
  font-size: 0.9rem;
}

.engine-tabs span {
  cursor: pointer;
  opacity: 0.6;
  transition: 0.3s;
  padding-bottom: 2px;
}

.engine-tabs span.active {
  opacity: 1;
  font-weight: bold;
  border-bottom: 2px solid var(--accent-color);
}

.input-group {
  display: flex;
  background: rgba(0,0,0,0.2);
  border-radius: 12px;
  padding: 5px;
}

.input-group input {
  flex: 1;
  background: transparent;
  border: none;
  color: white;
  padding: 12px;
  font-size: 1rem;
  outline: none;
}

.input-group input::placeholder { color: rgba(255,255,255,0.4); }

.search-btn {
  background: var(--accent-color);
  border: none;
  border-radius: 8px;
  width: 50px;
  cursor: pointer;
  font-size: 1.2rem;
  transition: 0.2s;
}

.search-btn:hover { filter: brightness(1.1); }

/* 分类卡片区 */
.content-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr)); /* 响应式瀑布流 */
  gap: 25px;
}

.category-card {
  background: var(--glass-bg);
  backdrop-filter: blur(10px);
  border: var(--glass-border);
  border-radius: 16px;
  padding: 20px;
  transition: transform 0.3s ease;
}

.category-card:hover {
  transform: translateY(-5px);
  background: rgba(255,255,255,0.2);
}

.cat-title {
  font-size: 1.2rem;
  margin-bottom: 15px;
  display: flex;
  align-items: center;
  gap: 8px;
  border-bottom: 1px solid rgba(255,255,255,0.1);
  padding-bottom: 10px;
}

/* 链接列表 */
.links-wrapper {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.link-item {
  display: flex;
  align-items: center;
  gap: 8px;
  background: rgba(255,255,255,0.1);
  padding: 8px 12px;
  border-radius: 8px;
  transition: 0.2s;
  font-size: 0.95rem;
}

.link-item:hover {
  background: var(--accent-color);
  color: white;
  box-shadow: 0 4px 12px rgba(66, 184, 131, 0.4);
}

.favicon {
  width: 16px;
  height: 16px;
}

/* 底部 */
footer {
  text-align: center;
  margin-top: 50px;
  font-size: 0.8rem;
  opacity: 0.5;
}

/* 动画 */
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}

/* 移动端适配 */
@media (max-width: 600px) {
  .hero-section h1 { font-size: 1.8rem; }
  .content-grid { grid-template-columns: 1fr; }
}
</style>
