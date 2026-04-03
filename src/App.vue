<template>
  <div class="app">
    <Sidebar :current-doc="currentDoc" @select-doc="navigateToDoc" />
    
    <main class="main-content">
      <div v-if="loading" class="loading">
        Loading document...
      </div>
      <div v-else-if="error" class="error">
        {{ error }}
      </div>
      <AsciiDocRenderer v-else :content="adocContent" />
    </main>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue'
import Sidebar from './components/Sidebar.vue'
import AsciiDocRenderer from './components/AsciiDocRenderer.vue'

const currentDoc = ref('getting-started/intro')
const adocContent = ref('')
const loading = ref(false)
const error = ref(null)

const loadDocument = async (path) => {
  if (!path) return
  
  loading.value = true
  error.value = null
  
  try {
    const response = await fetch(`/content/${path}.adoc`)
    
    if (!response.ok) {
      throw new Error(`Document "${path}" not found`)
    }
    
    adocContent.value = await response.text()
    
    localStorage.setItem('lastDocument', path)
  } catch (err) {
    error.value = err.message
    adocContent.value = ''
  } finally {
    loading.value = false
  }
}

const navigateToDoc = (path) => {
  if (currentDoc.value === path) return
  currentDoc.value = path
  loadDocument(path)
  
  const url = new URL(window.location)
  url.searchParams.set('doc', path)
  window.history.pushState({}, '', url)
}

onMounted(() => {
  const urlParams = new URLSearchParams(window.location.search)
  const docParam = urlParams.get('doc')
  
  if (docParam) {
    currentDoc.value = docParam
    loadDocument(docParam)
  } else {
    const lastDoc = localStorage.getItem('lastDocument')
    if (lastDoc) {
      currentDoc.value = lastDoc
      loadDocument(lastDoc)
    } else {
      loadDocument(currentDoc.value)
    }
  }
})

window.addEventListener('popstate', () => {
  const urlParams = new URLSearchParams(window.location.search)
  const docParam = urlParams.get('doc')
  if (docParam && docParam !== currentDoc.value) {
    currentDoc.value = docParam
    loadDocument(docParam)
  }
})
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: #fafafa;
}

.app {
  display: flex;
  min-height: 100vh;
}

.main-content {
  flex: 1;
  margin-left: 280px;
  padding: 30px 40px;
  max-width: calc(100% - 280px);
}

.content-header {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 20px;
}

.copy-link-btn {
  padding: 8px 16px;
  background: white;
  border: 1px solid #ddd;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.2s;
}

.copy-link-btn:hover {
  background: #f0f0f0;
  border-color: #42b983;
}

.loading {
  text-align: center;
  padding: 60px;
  color: #666;
  font-size: 1.1rem;
}

.error {
  color: #d32f2f;
  padding: 20px;
  background: #ffebee;
  border-radius: 8px;
  border-left: 4px solid #d32f2f;
}

@media (max-width: 768px) {
  .main-content {
    margin-left: 0;
    padding: 20px;
    max-width: 100%;
  }
}
</style>