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

const loadAsciidoctor = async () => {
  const module = await import('asciidoctor');
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
    console.error('Error while rendering AsciiDoc:', error);
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
  border-bottom: 1px solid #e8e8e8;
  padding-bottom: 10px;
  margin-top: 20px;
}

.adoc-content :deep(h2) {
  color: #34495e;
  margin-top: 20px;
  margin-bottom: 20px;
}

.adoc-content :deep(h3) {
  color: #5d7c9d;
  margin-top: 24px;
}

.adoc-content :deep(h4) {
  margin-top: 12px;
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

.adoc-content :deep(ul) {
  padding-left: 0px;
  list-style-type: none;
}

.adoc-content :deep(ul li) {
  margin: 2px 0;  
  padding-left: 0px;
  position: relative;
}

.adoc-content :deep(ol) {
  padding-left: 16px;
  counter-reset: item;
}

.adoc-content :deep(ol li) {
  margin: 2px 0;
  padding-left: 0px;
  position: relative;
}
</style>