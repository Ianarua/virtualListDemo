<template>
  <!-- 外层容器，用于滚动 -->
  <div class="virtual-list-container" ref="virtualListRef" @scroll="throttleScroll">
    <!-- 虚拟列表项的容器，其高度根据项目数量和行数动态计算 -->
    <div class="virtual-item-container" :style="{height: itemHeight * Math.ceil(listData.length / rowNum) - 16 + 'px'}">
      <!-- 遍历可见的项目 -->
      <template
        v-for="(item, index) of visibleItems"
        :key="index"
      >
        <!-- 插槽，用于渲染每个列表项，传递项目数据、索引和样式 -->
        <slot name="item" :item="item" :index="index" :style="getItemStyle()"></slot>
      </template>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from 'vue';
import { throttle } from 'lodash';

// 定义组件的属性
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

// 解构属性，方便后续使用
const { listData, itemHeight, rowNum } = props;

// 引用虚拟列表的容器
const virtualListRef = ref();

// 用于存储可见项目数量的响应式变量
const visibleCount = ref(0);
// 用于存储当前滚动开始的索引
const startIndex = ref(0);
// 用于存储每个项目需要向上偏移的像素值
const top = ref(0);

// 初始化可见项目数量
const initVisibleCount = () => {
  const container = virtualListRef.value;
  if (container) {
    const containerHeight = container.clientHeight - 32 * 2; // 容器高度减去上下 padding
    visibleCount.value = (Math.ceil(containerHeight / itemHeight) + 4) * rowNum;
  }
};

// 计算可见项目索引范围内的数据
const visibleItems = computed(() => {
  const endIndex = startIndex.value + visibleCount.value;
  return listData.slice(startIndex.value, endIndex);
});

// 处理滚动事件，节流处理以提高性能
const onScroll = () => {
  const container = virtualListRef.value;
  if (container) {
    const scrollTop = container.scrollTop;
    const n = Math.floor(scrollTop / itemHeight);
    const start = n + (rowNum - 1) * n;
    if (start !== startIndex.value) {
      // 计算偏移量
      const offsetY = scrollTop - (scrollTop % itemHeight);
      top.value = offsetY;
      startIndex.value = start;
    }
  }
};
const throttleScroll = throttle(onScroll, 50);

// 组件挂载时初始化可见项目数量，并监听窗口大小变化
onMounted(() => {
  initVisibleCount();
  window.addEventListener('resize', initVisibleCount);
});

// 组件卸载时移除事件监听器，避免内存泄漏
onUnmounted(() => {
  window.removeEventListener('resize', initVisibleCount);
});

// 计算每个项目的 transform 样式
const getItemStyle = () => {
  // 根据当前索引和起始索引计算偏移量
  return `transform: translateY(${top.value}px)`;
};
</script>

<style scoped>
.virtual-list-container {
  width: 100%;
  height: 100%;
  padding: 32px;
  overflow-y: auto;
  /* 隐藏滚动条 */
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