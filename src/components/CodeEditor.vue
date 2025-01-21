<script setup lang="ts">
import { ref, onMounted, watch } from 'vue';
import * as monaco from 'monaco-editor';
import { useElementSize } from '@vueuse/core';

const props = defineProps<{
  modelValue: string;
}>();

const emit = defineEmits<{
  (e: 'update:modelValue', value: string): void;
}>();

const editorContainer = ref<HTMLElement | null>(null);
const { width, height } = useElementSize(editorContainer);
let editor: monaco.editor.IStandaloneCodeEditor | null = null;

onMounted(() => {
  if (!editorContainer.value) return;

  editor = monaco.editor.create(editorContainer.value, {
    value: props.modelValue, // 初始值
    language: 'vue', // 语言
    theme: 'vs-dark', // 主题
    minimap: { enabled: false }, // 禁用缩略图
    fontSize: 14, // 字体大小
    lineNumbers: 'on', // 显示行号
    automaticLayout: true, // 自动布局
    scrollBeyondLastLine: false, // 禁用自动滚动到最后一行
    wordWrap: 'on', // 自动换行
    formatOnPaste: true, // 粘贴时自动格式化
    formatOnType: true, // 输入时自动格式化
    autoClosingBrackets: 'always', // 自动闭合括号
    autoIndent: 'full', // 自动缩进
    suggestOnTriggerCharacters: true, // 触发字符时显示建议
  });

  editor.onDidChangeModelContent(() => {
    const value = editor?.getValue() || '';
    emit('update:modelValue', value);
  });
});

watch(() => props.modelValue, (newValue) => {
  if (editor && newValue !== editor.getValue()) {
    editor.setValue(newValue);
  }
});

watch([width, height], () => {
  editor?.layout();
});
</script>

<template>
  <div ref="editorContainer" class="editor-container"></div>
</template>

<style scoped>
.editor-container {
  height: 100%;
  width: 100%;
  overflow: hidden;
}
</style>