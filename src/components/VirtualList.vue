<template>
  <div class="virtual-list-container" ref="virtualListRef" @scroll="throttleScroll">
    <div class="virtual-item-container" :style="{height: itemHeight * Math.ceil(listData.length / rowNum) - 16 + 'px'}">
      <template
        v-for="(item, index) of visibleItems"
        :key="index"
      >
          <slot name="item" :item="item" :index="index" :style="getItemStyle()"></slot>
      </template>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from 'vue';
import { throttle } from 'lodash';

const props = defineProps({
  listData: {
    type: Array,
    required: true,
    default: () => []
  },
  itemHeight: {
    type: Number,
    required: true,
    default: 1
  },
  rowNum: {
    type: Number,
    default: 1
  }
});

const { listData, itemHeight, rowNum } = props;

const virtualListRef = ref();

// const itemHeight = ref(176);
const visibleCount = ref(0);
const startIndex = ref(0);
const top = ref(0);

// 计算课件项目数量
const initVisibleCount = () => {
  const container = virtualListRef.value;
  if (container) {
    const containerHeight = container.clientHeight - 32 * 2;
    visibleCount.value = (Math.ceil(containerHeight / itemHeight) + 4) * rowNum;
  }
};

// 计算可见项目索引范围内的 数据
const visibleItems = computed(() => {
  const endIndex = startIndex.value + visibleCount.value;
  return listData.slice(startIndex.value, endIndex);
});

// 处理滚动函数 节流处理
const onScroll = () => {
  const container = virtualListRef.value;
  if (container) {
    const scrollTop = container.scrollTop;
    const n = Math.floor(scrollTop / itemHeight);
    const start = n + (rowNum - 1) * n;
    if (start !== startIndex.value) {
      // 距离顶部的偏移量 (可以理解为 0 索引的块 距离当前 页面 的偏移量)
      const offsetY = scrollTop - (scrollTop % itemHeight);

      // 这里的top用于设置translateY  transform:translateY(${top}px)
      top.value = offsetY;
      startIndex.value = start;
    }
  }
};
const throttleScroll = throttle(onScroll, 50);

// 组件挂在时计算可见项目数
onMounted(() => {
  initVisibleCount();
  window.addEventListener('resize', initVisibleCount);
});

// 清理资源
onUnmounted(() => {
  window.removeEventListener('resize', initVisibleCount);
});

// 计算每个 item 的 transform 样式
const getItemStyle = () => {
  return `transform: translateY(${top.value}px)`;
};
</script>

<style scoped>
.virtual-list-container {
  width: 100%;
  height: 100%;
  padding: 32px;
  overflow-y: auto;

  &::-webkit-scrollbar {
    width: 0;
    height: 8px;
    background-color: transparent;
  }
}

.virtual-item-container {
  display: flex;
  flex-wrap: wrap;
  align-content: flex-start; /* 子元素在容器内顶部对齐 */
}
</style>