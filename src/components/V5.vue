<template>
  <div class="list-container" ref="list" @scroll="handleScroll">
    <div class="list-platform" :style="{ height: `${listHeight}px` }"></div>
    <div class="list-main" :style="{ transform: getTransform }">
        <div 
            class="list-item"
            v-for="item in visibleData"
            :key="item.id"
            :style="{ height: `${props.itemSize}px`, lineHeight: `${props.itemSize}px` }"
        >{{ item.value }}</div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
const props = defineProps({
    listData: {
        type: Array,
        default: () => [],
    },
    itemSize: {
        type: Number,
        default: 100,
    }
});

const list = ref(null);
const startIndex = ref(0);
const endIndex = ref(0);
const screenHeight = ref(0);
const startOffset = ref(0);

const listHeight = computed(() => props.listData.length * props.itemSize);
const visibleCount = computed(() => Math.ceil(screenHeight.value / props.itemSize));
const visibleData = computed(() => props.listData.slice(startIndex.value, Math.min(endIndex.value, props.listData.length)));

const getTransform = computed(() => `translate3d(0, ${startOffset.value}px, 0)`)

onMounted(() => {
    screenHeight.value = list.value.clientHeight;
    startIndex.value = 0;
    endIndex.value = startIndex.value + visibleCount.value;
});

const handleScroll = () => {
    const scrollTop = list.value.scrollTop;
    startIndex.value = Math.floor(scrollTop / props.itemSize);
    endIndex.value = startIndex.value + visibleCount.value;
    startOffset.value = scrollTop - (scrollTop % props.itemSize);
}

</script>

<style scoped>
.list-container {
    position: relative;
    height: 100%;
    overflow: auto;
    -webkit-overflow-scrolling: touch;
}

.list-platform {
    position: absolute;
    right: 0;
    top: 0;
    left: 0;
    z-index: -1;
}

.list-main {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    text-align: center;
}

.list-item {
    color: #555;
    box-sizing: border-box;
    border-bottom: 1px solid #999;
}
</style>