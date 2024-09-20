<template>
  <div class="container">
    <div class="header"></div>
    <div class="main">
      <div class="side"></div>
      <div class="content">
        <!--        <BotList/>-->
        <Virtual-list :list-data="listData" :item-height="176" :row-num="rowNum">
          <template
            #item="{item, style}"
          >
            <div
              :class="['virtual-item', item === 0 ? 'new' : '']"
              :style="[style, { width: `calc(100% / ${rowNum} - 16px)` }]"
            >
              <template v-if="item === 0">
                {{ item }}
              </template>
              <template v-else>
                {{ item }}
              </template>
            </div>
          </template>
        </Virtual-list>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">

// import BotList from '@/views/BotList.vue';
import VirtualList from '@/components/VirtualList.vue';
import { ref } from 'vue';

const listData = ref<number[]>([]);
for (let i = 0; i < 35; i++) {
  listData.value.push(i);
}
const rowNum = ref(6);
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
}

.container {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}

.header {
  width: 100%;
  height: 64px;
  background-color: #26293b;
}

.main {
  width: 100%;
  height: calc(100vh - 64px);
  display: flex;

  .side {
    width: 280px;
    height: 100%;
    background-color: #26293b;
  }

  .content {
    flex: 1;
    background-color: #f3f6fd;
    overflow-y: auto;
  }
}

.virtual-item {
  height: 160px;
  padding: 20px 28px;
  margin: 0 8px 16px;
  border-radius: 12px;
  background-color: aquamarine;
  cursor: pointer;

  &.new {
    background-color: burlywood;
  }
}
</style>