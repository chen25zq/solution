<template>
  <div id="app">
    <!-- <VirtualList :listData="data" :itemSize="100" :estimatedItemSize="150" /> -->
    <V2 :listData="data" :itemSize="100" />
  </div>
</template>

<script setup>
import { ref } from 'vue'
// import VirtualList from './components/VirtualList.vue'
import V2 from '@/components/V6.vue'
import { faker } from '@faker-js/faker'

// 创建一个长列表的数据
let d = []
for (let i = 0; i < 10000; i++) {
  d.push({
    id: i + 1,
    // value: faker.lorem.sentences() // 生成一个随机的句子
    value: `第${i+1}数据`
  })
}
const data = ref(d)
</script>

<style>
html {
  height: 100%;
}
body {
  height: 100%;
  margin: 0;
}
#app {
  height: 100%;
}
</style>
