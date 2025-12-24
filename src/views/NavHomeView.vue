<script setup>
import { onMounted, ref, computed } from 'vue'
import { useNavigation } from '../apis/useNavigation'

// 1. 获取数据
const { categories, fetchCategories, loading } = useNavigation()
const searchQuery = ref('')
const activeEngine = ref('bing')

// 2. 搜索引擎配置
const engines = {
  bing: { name: '必应', url: 'https://cn.bing.com/search?q=' },
  google: { name: '谷歌', url: 'https://www.google.com/search?q=' },
  baidu: { name: '百度', url: 'https://www.baidu.com/s?wd=' }
}

const handleSearch = () => {
  if (!searchQuery.value) return
  window.open(engines[activeEngine.value].url + encodeURIComponent(searchQuery.value), '_blank')
}

// 3. 核心修复：这里改为识别 'sites' 字段
const filteredCategories = computed(() => {
  if (!categories.value) return []
  // 如果原始数据结构是 { categories: [...] }，这里做个兼容
  const list = Array.isArray(categories.value) ? categories.value : (categories.value.categories || [])

  if (!searchQuery.value) return list

  return list.map(cat => {
    // 关键点：优先找 cat.sites，其次找 cat.links
    const rawSites = cat.sites || cat.links || []
    
    const filteredSites = rawSites.filter(site => {
      const name = site.name || ''
      const desc = site.description || site.desc || '' // 兼容 description 和 desc
      return name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
             desc.toLowerCase().includes(searchQuery.value.toLowerCase())
    })

    return {
      ...cat,
      sites: filteredSites // 统一赋值给 sites
    }
  }).filter(cat => cat.sites && cat.sites.length > 0)
})

onMounted(() => {
  fetchCategories()
})
</script>

<template>
  <div class="container">
    
    <header class="hero-section">
      <div class="logo-area">
        <h1>猫猫导航</h1>
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
            :placeholder="`在 ${engines[activeEngine].name} 中搜索...`"
          />
          <button @click="handleSearch" class="search-btn">🔍</button>
        </div>
      </div>
    </header>

    <div v-if="loading" class="loading">
      <div class="spinner"></div>
      <p>正在加载数据...</p>
    </div>

    <main v-else class="content-grid">
      <div v-if="filteredCategories.length === 0" class="empty-state">
        没有找到相关内容
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
            v-for="site in (cat.sites || cat.links || [])" 
            :key="site.id || site.url"
            :href="site.url"
            target="_blank"
            class="link-item"
            :title="site.description"
          >
            <img 
              :src="site.icon || `https://favicon.duckduckgo.com/v2/?url=${site.url}`" 
              class="favicon"
              @error="(e) => e.target.style.opacity = 0" 
              alt=""
            />
            <div class="link-info">
              <span class="link-name">{{ site.name }}</span>
              <span v-if="site.description" class="link-desc">{{ site.description }}</span>
            </div>
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
/* 保持高级感样式 */
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 40px 20px;
}

.hero-section {
  text-align: center;
  margin-bottom: 50px;
  animation: fadeIn 0.8s ease;
}

.logo-area h1 {
  font-size: 2.2rem;
  margin-bottom: 5px;
  text-shadow: 0 4px 12px rgba(0,0,0,0.4);
}

.logo-area p {
  opacity: 0.8;
  margin-top: 0;
}

/* 搜索框 */
.search-box-wrapper {
  max-width: 500px;
  margin: 30px auto 0;
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(12px);
  padding: 15px;
  border-radius: 24px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: 0 8px 32px rgba(0,0,0,0.1);
}

.engine-tabs {
  display: flex;
  gap: 15px;
  margin-bottom: 10px;
  padding-left: 5px;
  font-size: 0.9rem;
}

.engine-tabs span {
  cursor: pointer;
  opacity: 0.6;
  padding-bottom: 2px;
}

.engine-tabs span.active {
  opacity: 1;
  font-weight: bold;
  border-bottom: 2px solid #42b883;
}

.input-group {
  display: flex;
  background: rgba(0,0,0,0.2);
  border-radius: 12px;
  overflow: hidden;
}

.input-group input {
  flex: 1;
  background: transparent;
  border: none;
  color: white;
  padding: 12px 15px;
  outline: none;
}

.input-group input::placeholder { color: rgba(255,255,255,0.5); }

.search-btn {
  background: #42b883;
  border: none;
  width: 50px;
  cursor: pointer;
  font-size: 1.2rem;
  transition: 0.2s;
}
.search-btn:hover { background: #3aa876; }

/* 瀑布流布局 */
.content-grid {
  display: columns;
  column-count: 3; /* 3列瀑布流 */
  column-gap: 20px;
}

@media (max-width: 900px) { .content-grid { column-count: 2; } }
@media (max-width: 600px) { .content-grid { column-count: 1; } }

.category-card {
  break-inside: avoid; /* 防止卡片被拆断 */
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.15);
  border-radius: 16px;
  padding: 20px;
  margin-bottom: 20px;
  transition: transform 0.3s ease;
}

.category-card:hover {
  transform: translateY(-5px);
  background: rgba(255, 255, 255, 0.15);
}

.cat-title {
  font-size: 1.1rem;
  margin-bottom: 15px;
  display: flex;
  align-items: center;
  gap: 8px;
  border-bottom: 1px solid rgba(255,255,255,0.1);
  padding-bottom: 8px;
  margin-top: 0;
}

/* 链接列表 */
.links-wrapper {
  display: flex;
  flex-direction: column; /* 列表式垂直排列 */
  gap: 8px;
}

.link-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px;
  border-radius: 8px;
  transition: 0.2s;
  background: rgba(255,255,255,0.05);
}

.link-item:hover {
  background: rgba(66, 184, 131, 0.2);
  transform: translateX(5px);
}

.favicon {
  width: 24px;
  height: 24px;
  border-radius: 4px;
}

.link-info {
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.link-name {
  font-weight: 500;
  font-size: 0.95rem;
  color: white;
}

.link-desc {
  font-size: 0.75rem;
  color: rgba(255,255,255,0.6);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 200px;
}

.loading, .empty-state { text-align: center; margin-top: 50px; opacity: 0.7; }
footer { text-align: center; margin-top: 60px; opacity: 0.4; font-size: 0.8rem; }
</style>
