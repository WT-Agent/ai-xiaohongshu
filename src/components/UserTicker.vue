<template>
  <div class="glass-card ticker-card">
    <div class="ticker-wrapper">
      <div v-for="(item, index) in tickerItems" :key="index" class="ticker-item">
        <span>{{ item.text }}</span>
        <span style="color: var(--success-color); font-weight: bold;">[生成完成]</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const tickerItems = ref([
  {
    "text": "用户 138****6611 创作的小红书减脂便当爆款获赞 1W+"
  },
  {
    "text": "用户 189****3322 获得了小个子穿搭爆款封面排版模板"
  },
  {
    "text": "用户 131****8800 完成了小众护肤品闺蜜语气种草文案"
  },
  {
    "text": "用户 156****7799 生成了 AI 效率工具合集引流小红书图文"
  },
  {
    "text": "用户 173****1100 取得了评论区求链接与私域导流控评 SOP"
  }
]);
</script>
