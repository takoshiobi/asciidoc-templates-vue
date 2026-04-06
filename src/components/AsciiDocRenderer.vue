<template>
  <div class="adoc-content" v-html="renderedHtml"></div>
</template>

<script setup>
import { ref, watch, nextTick } from 'vue';
import hljs from 'highlight.js';
import 'highlight.js/styles/atom-one-light.css'; 

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
    await nextTick();
    const blocks = document.querySelectorAll('.adoc-content pre code');
    blocks.forEach(block => hljs.highlightElement(block));
  } catch (error) {
    console.error('Error while rendering AsciiDoc:', error);
    renderedHtml.value = `<pre>${props.content}</pre>`;
  }
};

watch(() => props.content, renderAsciiDoc, { immediate: true });
</script>

<style scoped>
.adoc-content {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Inter', Roboto, sans-serif;
  font-size: 16px;
  color: #1a1e24;
  background: #ffffff;
}

.adoc-content :deep(h1) {
  font-size: 2.5rem;
  font-weight: 700;
  letter-spacing: -0.02em;
  margin: 2rem 0 1rem 0;
  padding-bottom: 0.75rem;
  border-bottom: 3px solid #1976d2; 
  color: #0a0c10;
}

.adoc-content :deep(h2) {
  font-size: 1.75rem;
  font-weight: 600;
  letter-spacing: -0.01em;
  margin: 2rem 0 1rem 0;
  padding-bottom: 0.5rem;
  border-bottom: 1px solid #e4e7ec;
  color: #1e293b;
}

.adoc-content :deep(h3) {
  font-size: 1.35rem;
  font-weight: 600;
  margin: 2.1rem 0 0.75rem 0;
  color: #2d3a4a;
}

.adoc-content :deep(h4) {
  font-size: 1.1rem;
  font-weight: 600;
  margin: 1.25rem 0 0.5rem 0;
  color: #4a5a6e;
}

.adoc-content :deep(p) {
  margin: 1rem 0;
  color: #2c3e50;
}

.adoc-content :deep(strong) {
  color: #475569;
  font-weight: 600;
}

.adoc-content :deep(a) {
  color: #2c3e50;
  text-decoration: none;
  border-bottom: 1px solid transparent;
  transition: border-color 0.2s;
}

.adoc-content :deep(a:hover) {
  border-bottom-color: #6366f1;
}

.adoc-content :deep(pre) {
  background: #f8fafc;
  border: 1px solid #e4e7ec;
  border-radius: 12px;
  padding: 1rem;
  overflow-x: auto;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.03);
}

.adoc-content :deep(code) {
  font-family: 'SF Mono', 'Fira Code', 'Cascadia Code', monospace;
  font-size: 0.85rem;
  line-height: 1.5;
}

/* Inline code */
.adoc-content :deep(code:not(pre code)) {
  background: #f1f5f9;
  color: #e11d48;
  padding: 0.2rem 0.4rem;
  border-radius: 6px;
  font-size: 0.85rem;
  font-weight: 500;
}

.adoc-content :deep(table) {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.9rem;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
}

.adoc-content :deep(th) {
  background: #f8fafc;
  padding: 12px 16px;
  text-align: left;
  font-weight: 600;
  color: #1a1e24;
  border-bottom: 2px solid #e4e7ec;
}

.adoc-content :deep(td) {
  padding: 10px 16px;
  border-bottom: 1px solid #eef2f6;
  color: #2c3e50;
}

.adoc-content :deep(tr:last-child td) {
  border-bottom: none;
}

.adoc-content :deep(tr:hover td) {
  background: #fafcff;
}

.adoc-content :deep(ul), 
.adoc-content :deep(ol) {
  margin: 1rem 0;
  padding-left: 1.75rem;
}

.adoc-content :deep(li) {
  margin: 0.5rem 0;
  line-height: 1.6;
}

.adoc-content :deep(ul > li::marker) {
  color: #2c3e50;
}

.adoc-content :deep(ol > li::marker) {
  color: #2c3e50;
  font-weight: 500;
}

.adoc-content :deep(blockquote) {
  margin: 1.5rem 0;
  padding: 0.75rem 1.5rem;
  border-left: 4px solid #2c3e50;
  background: #fafcff;
  border-radius: 8px;
  color: #4a5a6e;
  font-style: normal;
}

.adoc-content :deep(.admonitionblock) {
  margin: 1.5rem 0;
  padding: 1rem 1.25rem;
  border-radius: 12px;
  border-left: 4px solid;
}

.adoc-content :deep(.admonitionblock.note) {
  background: #f0f9ff;
  border-left-color: #2c3e50;
}

.adoc-content :deep(.admonitionblock.note .title) {
  color: #1e40af;
}

.adoc-content :deep(.admonitionblock.warning) {
  background: #fffbeb;
  border-left-color: #f59e0b;
}

.adoc-content :deep(.admonitionblock.warning .title) {
  color: #b45309;
}

.adoc-content :deep(.admonitionblock.important) {
  background: #fef2f2;
  border-left-color: #ef4444;
}

.adoc-content :deep(.admonitionblock.important .title) {
  color: #b91c1c;
}

.adoc-content :deep(.admonitionblock.tip) {
  background: #ecfdf5;
  border-left-color: #10b981;
}

.adoc-content :deep(.admonitionblock.tip .title) {
  color: #065f46;
}

.adoc-content :deep(.admonitionblock .title) {
  font-weight: 700;
  margin-bottom: 0.5rem;
  text-transform: uppercase;
  font-size: 0.75rem;
  letter-spacing: 0.05em;
}

.adoc-content :deep(hr) {
  margin: 2rem 0;
  border: none;
  height: 1px;
  background: linear-gradient(90deg, #e4e7ec, #2c3e50, #e4e7ec);
}

.adoc-content :deep(img) {
  max-width: 100%;
  border-radius: 12px;
  margin: 1rem 0;
}

@media (max-width: 768px) {
  .adoc-content {
    font-size: 15px;
  }
  
  .adoc-content :deep(h1) {
    font-size: 2rem;
  }
  
  .adoc-content :deep(h2) {
    font-size: 1.5rem;
  }
  
  .adoc-content :deep(pre) {
    font-size: 0.8rem;
  }
}
</style>