<template>
  <div class="adoc-content" v-html="renderedHtml"></div>
</template>

<script setup>
import { ref, watch } from 'vue';

const props = defineProps({
  content: {
    type: String,
    default: ''
  }
});

const renderedHtml = ref('');

// Загружаем библиотеку глобально, но без CDN
const loadAsciidoctor = async () => {
  // Используем асинхронный импорт для загрузки из node_modules
  const module = await import('asciidoctor');
  // Для Vite часто требуется обратиться к default.default
  return module.default.default ? module.default.default() : module.default();
};

const renderAsciiDoc = async () => {
  if (!props.content) return;

  try {
    const asciidoctor = await loadAsciidoctor();
    const html = asciidoctor.convert(props.content, {
      attributes: { showtitle: true, icons: 'font' }
    });
    renderedHtml.value = html;
  } catch (error) {
    console.error('Ошибка рендеринга AsciiDoc:', error);
    renderedHtml.value = `<pre>${props.content}</pre>`;
  }
};

watch(() => props.content, renderAsciiDoc, { immediate: true });
</script>

<style scoped>
.adoc-content {
  line-height: 1.6;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

.adoc-content :deep(h1) {
  color: #2c3e50;
  border-bottom: 2px solid #42b983;
  padding-bottom: 10px;
  margin-top: 20px;
}

.adoc-content :deep(h2) {
  color: #34495e;
  margin-top: 30px;
}

.adoc-content :deep(pre) {
  background: #f4f4f4;
  padding: 1rem;
  border-radius: 4px;
  overflow-x: auto;
}

.adoc-content :deep(code) {
  background: #f4f4f4;
  padding: 2px 4px;
  border-radius: 3px;
}

.adoc-content :deep(table) {
  border-collapse: collapse;
  width: 100%;
  margin: 20px 0;
}

.adoc-content :deep(th),
.adoc-content :deep(td) {
  border: 1px solid #ddd;
  padding: 10px;
  text-align: left;
}

.adoc-content :deep(th) {
  background: #f2f2f2;
  font-weight: bold;
}

.adoc-content :deep(.admonitionblock) {
  margin: 20px 0;
  padding: 10px 15px;
  border-left: 4px solid #3498db;
  background: #e3f2fd;
}
</style>