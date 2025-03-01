<script lang="ts" setup>
import { onMounted, onUnmounted, ref } from 'vue';

import { message } from 'ant-design-vue';

import { sseSse } from '#/api/system/systemSse';

const eventSource = ref<EventSource | null>(null);
const messages = ref<any[]>([]);

async function createSSEConnection() {
  try {
    // 假设使用一个固定的 uid，实际使用时应该是动态的
    const uid = 'test-user-123';

    // 先调用 API 进行连接初始化
    await sseSse({ uid });

    // 创建 SSE 连接
    const sse = new EventSource(`/api/sse/${uid}`);

    // 使用 addEventListener 监听事件
    sse.addEventListener('message', (event) => {
      const data = JSON.parse(event.data);
      messages.value.push(data);
      message.info('收到新消息');
    });

    sse.addEventListener('open', () => {
      message.success('SSE 连接已建立');
    });

    sse.addEventListener('error', (error) => {
      message.error('SSE 连接错误');
      console.error('SSE Error:', error);
      sse.close();
    });

    eventSource.value = sse;
  } catch (error) {
    message.error('建立 SSE 连接失败');
    console.error('Setup SSE failed:', error);
  }
}

// 组件挂载时建立连接
onMounted(() => {
  createSSEConnection();
});

// 组件卸载时关闭连接
onUnmounted(() => {
  if (eventSource.value) {
    eventSource.value.close();
    eventSource.value = null;
  }
});
</script>

<template>
  <div class="p-4">
    <h2 class="mb-4 text-lg font-medium">SSE 消息列表</h2>
    <div class="rounded-lg border p-4">
      <template v-if="messages.length > 0">
        <div
          v-for="(msg, index) in messages"
          :key="index"
          class="mb-2 rounded bg-gray-50 p-2"
        >
          <pre>{{ JSON.stringify(msg, null, 2) }}</pre>
        </div>
      </template>
      <div v-else class="text-gray-500">暂无消息</div>
    </div>
  </div>
</template>
