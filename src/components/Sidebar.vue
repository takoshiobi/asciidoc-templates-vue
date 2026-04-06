<template>
  <aside class="sidebar">
    <div class="sidebar-header">
        <h2>Templates</h2>
    </div>
    
    <nav class="sidebar-nav">
      <div v-if="loading" class="loading-menu">
        Loading menu...
      </div>
      <div v-else-if="error" class="error-menu">
        {{ error }}
      </div>
      <template v-else>
        <div v-for="category in categories" :key="category.title" class="category">
          <div 
            class="category-title"
            :class="{ expanded: expandedCategories[category.title] }"
            @click="toggleCategory(category.title)"
          >
            <span class="arrow">{{ expandedCategories[category.title] ? '▼' : '▶' }}</span>
            <span class="icon">{{ category.icon }}</span>
            <span class="title">{{ category.title }}</span>
          </div>
          
          <ul v-if="expandedCategories[category.title]" class="doc-list">
            <li 
              v-for="item in category.items" 
              :key="item.path"
              @click="selectDoc(item.path)"
              :class="{ active: currentDoc === item.path }"
            >
              <span class="doc-title">{{ item.title }}</span>
            </li>
          </ul>
        </div>
      </template>
    </nav>
  </aside>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue'

const props = defineProps({
  currentDoc: {
    type: String,
    default: ''
  }
})

const emit = defineEmits(['select-doc'])

const categories = ref([])
const loading = ref(true)
const error = ref(null)
const expandedCategories = reactive({})

const toggleCategory = (title) => {
  expandedCategories[title] = !expandedCategories[title]
  localStorage.setItem(`expanded_${title}`, expandedCategories[title])
}

const selectDoc = (path) => {
  emit('select-doc', path)
}

onMounted(async () => {
  try {
    console.log('Loading menu...')
    const response = await fetch('/index.json')
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`)
    }
    
    const data = await response.json()
    console.log('Data loaded:', data)
    
    if (data && Array.isArray(data.categories)) {
      categories.value = data.categories
    } else if (Array.isArray(data)) {
      categories.value = data
    } else {
      console.error('Incorrect data structure:', data)
      throw new Error('Incorrect data structure of index.json. Expecting categories array')
    }
    
    categories.value.forEach(cat => {
      const saved = localStorage.getItem(`expanded_${cat.title}`)
      const isExpanded = saved === 'true' || (saved === null && cat.title === categories.value[0]?.title)
      expandedCategories[cat.title] = isExpanded
    })
    
    loading.value = false
  } catch (err) {
    console.error('Menu loading error:', err)
    error.value = `Unable to load menu: ${err.message}`
    loading.value = false
    
    categories.value = [
      {
        title: "Getting started",
        icon: "🚀",
        items: [
          { title: "Introduction", path: "getting-started/intro" }
        ]
      }
    ]
  }
})
</script>

<style scoped>
.sidebar {
  width: 280px;
  background: #f8f9fa;
  border-right: 1px solid #e9ecef;
  height: 100vh;
  position: fixed;
  left: 0;
  top: 0;
  overflow-y: auto;
  z-index: 100;
}

.sidebar-header {
  padding: 20px;
  border-bottom: 1px solid #e9ecef;
  background: #f8f9fa;
  position: sticky;
  top: 0;
  z-index: 10;
}

.sidebar-header h2 {
  margin: 0;
  font-size: 1.2rem;
  color: #2c3e50;
}

.sidebar-nav {
  padding: 10px 0;
}

.loading-menu, .error-menu {
  padding: 20px;
  text-align: center;
  color: #666;
}

.error-menu {
  color: #d32f2f;
  background: #ffebee;
  margin: 10px;
  border-radius: 4px;
}

.category {
  margin-bottom: 5px;
}

.category-title {
  padding: 12px 20px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 500;
  color: #495057;
  transition: all 0.2s;
  user-select: none;
}

.category-title:hover {
  background: #e9ecef;
}

.category-title.expanded {
  color: #2c3e50;
}

.arrow {
  font-size: 10px;
  width: 16px;
  transition: transform 0.2s;
}

.icon {
  font-size: 1.1rem;
}

.title {
  flex: 1;
}

.doc-list {
  list-style: none;
  padding-left: 44px;
  margin: 5px 0;
}

.doc-list li {
  padding: 8px 12px;
  cursor: pointer;
  border-radius: 6px;
  margin: 2px 8px 2px 0;
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 14px;
  color: #6c757d;
  transition: all 0.2s;
}

.doc-list li:hover {
  background: #e9ecef;
  color: #2c3e50;
}

.doc-list li.active {
  background: #e3f2fd;
  color: #1976d2;
  font-weight: 500;
}

.doc-list li.active .doc-icon {
  color: #1976d2;
}

.doc-icon {
  font-size: 12px;
  opacity: 0.7;
}

.doc-title {
  flex: 1;
}

.sidebar::-webkit-scrollbar {
  width: 6px;
}

.sidebar::-webkit-scrollbar-track {
  background: #f1f1f1;
}

.sidebar::-webkit-scrollbar-thumb {
  background: #c1c1c1;
  border-radius: 3px;
}

.sidebar::-webkit-scrollbar-thumb:hover {
  background: #a8a8a8;
}
</style>