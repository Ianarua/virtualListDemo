<template>
  <div class="bot-list" ref="virtualListRef" @scroll="throttleScroll">
    <div class="bot-item-container" :style="{height: itemHeight * Math.ceil(listData.length / 4) - 16 + 'px'}">
      <div
        v-for="item of visibleItems"
        :key="item"
        :class="['bot-item', item === 0 ? 'new' : '']"
        :style="`transform:translateY(${top}px)`"
      >
        <template v-if="item === 0">
          {{ item }}
        </template>
        <template v-else>
          {{ item }}
        </template>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from 'vue';
import { throttle } from 'lodash';

const virtualListRef = ref();

const listData = ref<number[]>([]);

const initData = () => {
  for (let i = 0; i < 10000; i++) {
    listData.value.push(i);
  }
  // listData.value.unshift(-1);
};

const itemHeight = ref(176);
const visibleCount = ref(0);
const startIndex = ref(0);
const top = ref(0);

// 计算课件项目数量
const initVisibleCount = () => {
  const container = virtualListRef.value;
  if (container) {
    const containerHeight = container.clientHeight - 32 * 2;
    visibleCount.value = (Math.ceil(containerHeight / itemHeight.value) + 4) * 4;
  }
};

// 计算可见项目索引范围内的 数据
const visibleItems = computed(() => {
  const endIndex = startIndex.value + visibleCount.value;
  return listData.value.slice(startIndex.value, endIndex);
});

// 处理滚动函数 节流处理
const onScroll = () => {
  const container = virtualListRef.value;
  if (container) {
    const scrollTop = container.scrollTop;
    const n = Math.floor(scrollTop / itemHeight.value);
    const start = n + 3 * n;
    if (start !== startIndex.value) {
      // 距离顶部的偏移量 (可以理解为 0 索引的块 距离当前 页面 的偏移量)
      const offsetY = scrollTop - (scrollTop % itemHeight.value);

      // 这里的top用于设置translateY  transform:translateY(${top}px)
      top.value = offsetY;
      startIndex.value = start;
    }
  }
};
const throttleScroll = throttle(onScroll, 50);

// 组件挂在时计算可见项目数
onMounted(() => {
  initData();
  initVisibleCount();
  window.addEventListener('resize', initVisibleCount);
});

// 清理资源
onUnmounted(() => {
  window.removeEventListener('resize', initVisibleCount);
});
</script>

<style scoped>
.bot-list {
  width: 100%;
  height: 100%;
  padding: 32px;
  font-family: PingFang SC;
  overflow-y: auto;

  &::-webkit-scrollbar {
    width: 0px;
    height: 8px;
    background-color: transparent;
  }
}

/**
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 10px;
  justify-content: space-between; 盒子之间平均分配间隔
*/
.bot-item-container {
  display: flex;
  flex-wrap: wrap;
  align-content: flex-start; /* 子元素在容器内顶部对齐 */
  margin-left: -8px; /* 负间距来抵消第一个和最后一个盒子的额外间距 */
  margin-right: -8px; /* 同上 */
}

.bot-item {
  height: 160px;
  flex-basis: calc(25% - 16px); /* 减去间距的宽度 */
  padding: 20px 28px;
  margin: 0 8px 16px;
  border-radius: 12px;
  background-color: aquamarine;
  cursor: pointer;
  transform: translateY(0);

  &.new {
    background-color: burlywood;
  }
}
</style>