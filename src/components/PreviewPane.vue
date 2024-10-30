<script setup lang="ts">
import { ref, watch, onMounted } from 'vue';
import { createApp, h } from 'vue';
import { parse } from '@vue/compiler-sfc';

const props = defineProps<{
  code: string;
}>();

const previewContainer = ref<HTMLDivElement | null>(null);
let previewApp: any = null;

const compileAndRender = async (code: string) => {
  if (!previewContainer.value) return;

  try {
    // Clear previous content
    previewContainer.value.innerHTML = '<div id="preview-root"></div>';

    // Parse SFC
    const { descriptor } = parse(code);
    
    // Extract template and script content
    const templateContent = descriptor.template?.content || '';
    const scriptContent = descriptor.script?.content || descriptor.scriptSetup?.content || '';

    // Create a component with template and setup function
    const component = {
      template: templateContent,
      setup() {
        // Create a safe context for evaluation
        const ctx = {
          Vue: { ref, computed: ref, reactive: ref }, // Simplified reactive system
          exports: {},
          require: () => ({}), // Mock require
          module: { exports: {} }
        };
        // 匹配 script setup 中定义的变量名称
        const variableNames = Array.from(
          scriptContent.matchAll(/\b(const|let|var|ref|reactive)\s+(\w+)/g),
          match => match[2]
        );
        // Remove import statements and export keywords
        // const processedScript = scriptContent
        //   .replace(/import\s+.*?from\s+['"].*?['"]/g, '')
        //   .replace(/export\s+default\s*{/, 'return {')
        //   .replace(/export\s*{[^}]*}/, '')
        //   .replace(/export\s+const/g, 'const');
        const processedScript = scriptContent
          .replace(/import\s+.*?from\s+['"].*?['"]/g, '')
          .replace(/export\s+default\s*{/, 'return {')
          .replace(/export\s*{[^}]*}/, '')
          .replace(/export\s+const/g, 'const');
        try {
          // Create and execute setup function
          const setupFn = new Function('Vue', `
            with (Vue) {
                ${processedScript}
                return { ${variableNames.join(', ')} }; // 确保返回模板变量
              }
          `);
          
          return setupFn(ctx.Vue);
        } catch (error) {
          console.error('Script evaluation error:', error);
          return {};
        }
      },
      // Add basic error handling
      errorCaptured(err: Error) {
        console.error('Runtime error:', err);
        return false;
      }
    };

    // Destroy previous app if exists
    if (previewApp) {
      previewApp.unmount();
    }

    // Create and mount new app
    previewApp = createApp(component);
    previewApp.mount('#preview-root');

    // Apply styles
    const styles = document.querySelectorAll('style[data-preview]');
    styles.forEach(style => style.remove());

    if (descriptor.styles.length > 0) {
      const styleElement = document.createElement('style');
      styleElement.setAttribute('data-preview', 'true');
      styleElement.textContent = descriptor.styles.map(s => s.content).join('\n');
      document.head.appendChild(styleElement);
    }
  } catch (error: any) {
    console.error('Compilation error:', error);
    previewContainer.value.innerHTML = `
      <div class="error">
        <h3>Compilation Error</h3>
        <pre>${error.message}</pre>
      </div>
    `;
  }
};

watch(() => props.code, (newCode) => {
  compileAndRender(newCode);
}, { immediate: true });

onMounted(() => {
  compileAndRender(props.code);
});
</script>

<template>
  <div ref="previewContainer" class="preview-container">
    <div id="preview-root"></div>
  </div>
</template>

<style scoped>
.preview-container {
  height: 100%;
  padding: 1rem;
  overflow: auto;
}

.error {
  color: #ff4d4f;
  background: #fff2f0;
  border: 1px solid #ffccc7;
  border-radius: 4px;
  padding: 1rem;
  margin: 1rem;
}

.error h3 {
  margin: 0 0 0.5rem;
}

.error pre {
  margin: 0;
  white-space: pre-wrap;
  font-family: monospace;
  font-size: 0.9em;
}
</style>