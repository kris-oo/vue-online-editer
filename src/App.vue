<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { Layout, Button, message } from 'ant-design-vue';
import CodeEditor from './components/CodeEditor.vue';
import PreviewPane from './components/PreviewPane.vue';
import localforage from 'localforage';

const code = ref(`<template>
  <div class="demo">
    <h1>{{ message }}</h1>
    <button @click="count++">Count: {{ count }}</button>
  </div>
</template>

<script setup>
import {ref} from 'vue'
const message = ref('Hello Vue!')
const count = ref(0)
<\/script>

<style>
.demo {
  text-align: center;
  padding: 20px;
}
h1 { 
  color: #42b883;
  margin-bottom: 20px;
}
button { 
  padding: 8px 16px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
button:hover {
  background: #3aa876;
}
</style>`);

const previewKey = 'vue-editor-code';

onMounted(async () => {
  try {
    const savedCode = await localforage.getItem(previewKey);
    if (savedCode) {
      code.value = savedCode as string;
    }
  } catch (error) {
    message.error('Failed to load saved code');
  }
});

const saveCode = async () => {
  try {
    await localforage.setItem(previewKey, code.value);
    message.success('Code saved successfully');
  } catch (error) {
    message.error('Failed to save code');
  }
};
</script>

<template>
  <Layout class="editor-layout">
    <Layout.Sider width="50%" class="editor-sider">
      <div class="editor-header">
        <h2>Vue Editor</h2>
        <Button type="primary" @click="saveCode">Save</Button>
      </div>
      <CodeEditor v-model="code" class="code-editor" />
    </Layout.Sider>
    <Layout.Content class="preview-content">
      <h2>Preview</h2>
      <div class="preview-wrapper">
        <PreviewPane :code="code" />
      </div>
    </Layout.Content>
  </Layout>
</template>

<style scoped>
.editor-layout {
  height: 100vh;
  background: #fff;
}

.editor-sider {
  background: #1e1e1e;
  padding: 1rem;
}

.editor-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  color: #fff;
}

.code-editor {
  height: calc(100vh - 80px);
}

.preview-content {
  padding: 1rem;
  background: #fff;
}

.preview-wrapper {
  height: calc(100vh - 80px);
  border: 1px solid #eee;
  border-radius: 4px;
  overflow: hidden;
}

h2 {
  margin: 0 0 1rem;
  font-size: 1.5rem;
}
</style>