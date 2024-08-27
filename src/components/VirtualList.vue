<template>
    <!-- 外层容器 -->
    <div ref="listRef" class="infinite-list-container" @scroll="handleScroll">
        <!-- 元素高度为总列表高度，目的是为了形成滚动 -->
        <div ref="listHeight" class="infinite-list-plantom"></div>
        <!-- 可视区域 -->
        <div ref="content" class="infinite-list">
            <div
             ref="items"
             class="infinite-list-item"
             v-for="item in visibleData"
             :key="item.id"
            >
                {{ item.value }}
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, onMounted, watch, onUpdated, nextTick } from 'vue'
const props = defineProps({
    listData: {
        type: Array,
        default: () => []
    },
    itemSize: {
        type: Number,
        default: 150
    },
    // 预估高度
    estimatedItemSize: {
        type: Number,
        required: true
    }
})

// container 元素引用
const listRef = ref(null)
// 可视区域高度
const screenHeight = ref(0)
// 开始索引
const startIndex = ref(0)
// 结束索引
const endIndex = ref(0)
// 初始偏移量
// const startOffset = ref(0)

// 用于创建列表项元素的引用
const items = ref([])
// 用于引用phantom元素
const listHeight = ref(null)
// 用于引用list元素
const content = ref(null)

// 缓存列表，用于存储列表项的位置信息
let positions = []
// 用于初始化每个列表项的位置信息
const initPostions = () => {
    positions = props.listData.map((_, index) => ({
        index, // 列表项的下标
        height: props.estimatedItemSize, // 列表项的高度，这里采用预估的高度
        top: index * props.estimatedItemSize, // 列表项的顶部位置，根据下标和预估高度计算
        bottom: (index + 1) * props.estimatedItemSize // 列表项的底部位置，也是根据下标和预估高度计算
    }))
}

// 列表总高度
// const listHeight = computed(() => props.listData.length * props.itemSize);
// 可显示的列表项数
const visibleCount = computed(() => Math.ceil(screenHeight.value / props.itemSize));
// 列表显示数据
const visibleData = computed(() => 
    props.listData.slice(startIndex.value, Math.min(endIndex.value, props.listData.length))
);

// 向下位移的距离, 计算得到的偏移量能过更平滑的滚动效果
// const getTransform = computed(() => `translate3d(0, ${startOffset.value}px, 0)`)

// 关于查找 startIndex 的方法，可以使用二分查找法来进行优化
const binarySearch = (list, value) => {
  let start = 0
  let end = list.length - 1
  let tempIndex = null
  while (start <= end) {
    let midIndex = parseInt((start + end) / 2)
    let midValue = list[midIndex].bottom
    if (midValue === value) {
      return midIndex + 1
    } else if (midValue < value) {
      start = midIndex + 1
    } else if (midValue > value) {
      if (tempIndex === null || tempIndex > midIndex) {
        tempIndex = midIndex
      }
      end = end - 1
    }
  }
  return tempIndex
}
const getStartIndex = (scrollTop) => {
  return binarySearch(positions, scrollTop)
}

const handleScroll = () => {
    // 滚动操作时更新各项数据 当页面滚动时，要根据滚动的距离来计算 startIndex 和 endIndex，并重新渲染 visibleData
    let scrollTop = listRef.value.scrollTop;
    startIndex.value = getStartIndex(scrollTop);
    endIndex.value = startIndex.value + visibleCount.value;
    // 之所以要减去 scrollTop % itemSize，是为了将 scrollTop 调整到一个与 itemHeight 对齐的位置(也就是 itemSize 的整数倍)，避免显示不完整的列表项
    // startOffset.value = scrollTop - (scrollTop % props.itemSize);
    setStartOffset()
}

const setStartOffset = () => {
    let startOffset = startIndex.value >= 1 ? positions[startIndex.value - 1].bottom : 0;
    content.value.style.transform = `translate3d(0, ${startOffset}px, 0)`;
}

const updateItemSize = () => {
    items.value.forEach((node, index) => {
        // 获取列表项真实高度
        let height = node.getBoundingClientRect().height;
        // 计算预估高度和真实高度之间的差值
        let oldHeight = positions[index].height;
        let dValue = oldHeight - height;
        if (dValue) {
            // 如果存在差值，那么就需要更新位置信息
            positions[index].bottom -= dValue
            positions[index].height = height
            for(let i = index + 1; i < positions.length; i++) {
                positions[i].top = positions[i - 1].bottom
                positions[i].bottom -= dValue
            }
        }
    })
}

onMounted(() => {
    // 获取可视区域高度
    screenHeight.value = listRef.value.clientHeight;
    startIndex.value = 0;
    endIndex.value = startIndex.value + visibleCount.value;
    initPostions();
})

onUpdated(() => {
    // 这里之所以使用 nextTick，是为了确保DOM更新完毕后再去获取列表项的位置信息
    nextTick(() => {
        if (!items.value || !items.value.length) return;
        // 1.更新列表项高度信息
        updateItemSize();
        // 2. 更新虚拟列表高度
        listHeight.value.style.height = positions[positions.length - 1].bottom + 'px';
        // 3. 更新列表偏移量
        setStartOffset();
    })
})

watch(() => props.listData, initPostions);
</script>

<style scoped>
.infinite-list-container {
    height: 100%;
    overflow: auto;
    position: relative;
    -webkit-overflow-scrolling: touch;
}

.infinite-list-plantom {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    z-index: -1;
}

.infinite-list {
    left: 0;
    right: 0;
    top: 0;
    position: absolute;
    text-align: center;
}

.infinite-list-item {
    padding: 10px;
    color: #555;
    box-sizing: border-box;
    border-bottom: 1px solid #999;
}
</style>
