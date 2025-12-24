<script setup>
import { onMounted, ref, computed } from 'vue'
import { useNavigation } from '../apis/useNavigation'

const { categories, fetchCategories, loading } = useNavigation()
const searchQuery = ref('')

const filteredCategories = computed(() => {
  const list = Array.isArray(categories.value) ? categories.value : (categories.value.categories || [])
  if (!searchQuery.value) return list

  return list.map(cat => ({
    ...cat,
    sites: (cat.sites || cat.links || []).filter(site => 
      site.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      (site.description || '').toLowerCase().includes(searchQuery.value.toLowerCase())
    )
  })).filter(cat => cat.sites.length > 0)
})

onMounted(() => { fetchCategories() })
</script>

<template>
  <div class="content-wrapper">
    <div class="top-search-bar">
      <input v-model="searchQuery" placeholder="输入关键字实时过滤内容..." />
    </div>

    <div v-for="cat in filteredCategories" :key="cat.id" :id="cat.id" class="site-section">
      <h3 class="section-title">
        <span class="icon">{{ cat.icon }}</span> {{ cat.name }}
      </h3>
      
      <div class="site-grid">
        <a 
          v-for="site in cat.sites" 
          :key="site.id" 
          :href="site.url" 
          target="_blank" 
          class="site-card"
        >
          <img :src="site.icon || `https://favicon.duckduckgo.com/v2/?url=${site.url}`" class="site-favicon" />
          <div class="site-info">
            <span class="name">{{ site.name }}</span>
            <span class="desc">{{ site.description }}</span>
          </div>
        </a>
      </div>
    </div>
  </div>
</template>

<style scoped>
.content-wrapper {
  max-width: 1400px;
  margin: 0 auto;
}

.top-search-bar {
  margin-bottom: 40px;
}

.top-search-bar input {
  width: 100%;
  max-width: 400px;
  padding: 12px 20px;
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  color: white;
  outline: none;
}

.site-section {
  margin-bottom: 50px;
  scroll-margin-top: 20px; /* 跳转时的顶部留白 */
}

.section-title {
  font-size: 1.4rem;
  margin-bottom: 20px;
  color: #fff;
}

.site-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 15px;
}

.site-card {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  padding: 15px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  gap: 12px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  transition: all 0.3s;
  text-decoration: none;
}

.site-card:hover {
  background: rgba(66, 184, 131, 0.2);
  transform: translateY(-3px);
  border-color: #42b883;
}

.site-favicon {
  width: 36px;
  height: 36px;
  border-radius: 6px;
}

.site-info {
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.name { color: #fff; font-weight: bold; font-size: 0.95rem; }
.desc { color: rgba(255, 255, 255, 0.6); font-size: 0.75rem; white-space: nowrap; text-overflow: ellipsis; overflow: hidden; }
</style>
