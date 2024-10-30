<script setup lang="ts">
import { ref, watch, onMounted } from 'vue';
import { createApp } from 'vue';
import Antd from 'ant-design-vue';
import { parse } from '@vue/compiler-sfc';

const props = defineProps<{
  code: string;
}>();

const previewContainer = ref<HTMLDivElement | null>(null);
let previewApp: any = null;

const compileAndRender = async (code: string) => {
  if (!previewContainer.value) return;

  try {
    // 清空之前的内容
    previewContainer.value.innerHTML = '<div id="preview-root"></div>';

    // 解析 SFC
    const { descriptor } = parse(code);

    // 提取 template 和 script 内容
    const templateContent = descriptor.template?.content || '';
    const scriptContent =
      descriptor.script?.content || descriptor.scriptSetup?.content || '';

    // 匹配 script setup 中定义的变量名称
    const variableNames = Array.from(
      scriptContent.matchAll(/\b(const|let|var|ref|reactive)\s+(\w+)/g),
      (match) => match[2]
    );

    // 去除 import 语句和 export 关键词
    const processedScript = scriptContent
      .replace(/import\s+.*?from\s+['"].*?['"]/g, '')
      .replace(/export\s+default\s*{/, 'return {')
      .replace(/export\s*{[^}]*}/, '')
      .replace(/export\s+const/g, 'const');

    // 动态创建并执行 setup 函数
    const setupFn = new Function(
      'Vue',
      'Antd',
      `
      with (Vue) {
        ${processedScript}
        return { ${variableNames.join(', ')} };
      }
    `
    );

    const component = {
      template: templateContent,
      setup() {
        return setupFn({ ...Antd, ref });
      },
      errorCaptured(err: Error) {
        console.error('运行时错误:', err);
        return false;
      },
    };

    // 销毁之前的应用
    if (previewApp) {
      previewApp.unmount();
    }

    // 创建并挂载新应用
    previewApp = createApp(component);
    previewApp.use(Antd); // 注册 Ant Design Vue 组件
    previewApp.mount('#preview-root');

    // 添加样式
    const styles = document.querySelectorAll('style[data-preview]');
    styles.forEach((style) => style.remove());

    if (descriptor.styles.length > 0) {
      const styleElement = document.createElement('style');
      styleElement.setAttribute('data-preview', 'true');
      styleElement.textContent = descriptor.styles
        .map((s) => s.content)
        .join('\n');
      document.head.appendChild(styleElement);
    }
  } catch (error: any) {
    console.error('编译错误:', error);
    previewContainer.value.innerHTML = `
      <div class="error">
        <h3>编译错误</h3>
        <pre>${error.message}</pre>
      </div>
    `;
  }
};

// 监听代码变化
watch(
  () => props.code,
  (newCode) => {
    compileAndRender(newCode);
  },
  { immediate: true }
);

// 初始化加载
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
