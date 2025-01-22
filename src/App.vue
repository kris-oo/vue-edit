<template>
  <a-layout class="layout">
    <a-layout-header :style="headerStyle">Header</a-layout-header>
    <a-layout>
      <a-layout-sider :style="siderStyle">Sider</a-layout-sider>
      <a-layout-content :style="contentStyle">
        <MainView />
      </a-layout-content>
    </a-layout>
  </a-layout>
</template>
<script lang="ts" setup>
import type { CSSProperties } from 'vue';
import MainView from './components/MainView.vue';

const headerStyle: CSSProperties = {
  textAlign: 'center',
  color: '#fff',
  height: '32px',
  paddingInline: 0,
  lineHeight: '32px',
  backgroundColor: '#191919',
  borderBottom: '1px solid #383838',
};
const contentStyle: CSSProperties = {
  minHeight: '120px',
  color: '#fff',
};

const siderStyle: CSSProperties = {
  width: '200px',
  textAlign: 'center',
  color: '#fff',
  backgroundColor: '#181818',
  borderRight: '1px solid #383838',
};
</script>
<style scoped>
.layout {
  min-height: 100vh;
  background: #f0f2f5;
}
</style>