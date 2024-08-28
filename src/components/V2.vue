<template>
  <div ref="list" class="infinite-list-container" @scroll="handleScroll">
    <div class="infinite-list-phanto" :style="{ height: `${listHeight}px` }"></div>
    <div class="infinite-list" :style="{ transform: getTransform }">
        <div 
          class="infinite-list-item"
          v-for="item in visibleData"
          :key="item.id"
          :style="{ 
            height: `${props.itemSize}px`,
            lineHeight: `${props.itemSize}px`,
          }"
        >{{ item.value }}</div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
const props = defineProps({
    listData: {
        type: Array,
        default: () => []
    },
    itemSize: {
        type: Number,
        default: 50
    }
})

const list = ref(null);

const startIndex = ref(0);
const endIndex = ref(0);
const screenHeight = ref(0);
const startOffset = ref(0)

// 列表总高度
const listHeight = computed(() => props.listData.length * props.itemSize);
// 列表可见项数量 向上取整
const visibleCount = computed(() => Math.ceil(screenHeight.value / props.itemSize));
// 列表可见项数据
const visibleData = computed(() => props.listData.slice(startIndex.value, Math.min(endIndex.value, props.listData.length)));

// 滚动高度
const getTransform = computed(() => `translate3d(0, ${startOffset.value}px, 0)`)

onMounted(() => {
    screenHeight.value = list.value.clientHeight;
    startIndex.value = 0;
    endIndex.value = startIndex.value + visibleCount.value;
})

const handleScroll = () => {
    const scrollTop = list.value.scrollTop;
    startIndex.value = Math.floor(scrollTop / props.itemSize);
    endIndex.value = startIndex.value + visibleCount.value;
    startOffset.value = scrollTop - (scrollTop % props.itemSize);
}

</script>

<style scoped>
.infinite-list-container {
    height: 100%;
    overflow: auto;
    position: relative;
    -webkit-overflow-scrolling: touch;
}
.infinite-list-phanto {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    z-index: -1;
}
.infinite-list {
    position: absolute;
    top: 0;
    right: 0;
    left: 0;
    text-align: center;
}
.infinite-list-item {
    color: #555;
    box-sizing: border-box;
    border-bottom: 1px solid #999;
}
</style>