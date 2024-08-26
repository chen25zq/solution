# Solution 解决方案实现


### 1. 虚拟列表

##### vite + vue3

- 创建项目
 
```bash
yarn create vite solution --template vue
```

- vite 配置别名

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { fileURLToPath, URL } from 'node:url'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      "@": fileURLToPath(new URL("./src", import.meta.url))
    }
  }
})
```

### 实现步骤

- 初始化时
1. onMounted 页面加载钩子获取数据给各个字段赋值
  获取最外层容器的可视区域高度，将其设置成 screenHeight 变量的值
  可显示列表项数 visibleCount：screenHeight / itemHeight，即屏幕可视区域的高度除以每一项的高度(向下取整)
  startIndex 开始索引初始化为 0， startOffset 开始偏移量初始化为 0
  endIndex 结束索引：初始化为 startIndex + visibleCount

2. 那此时要渲染的数据就可以根据上面的这些初始化数据计算得到
  visibleData：从原始数据中截取 startIndex 到 endIndex 的数据

3. 滚动事件：
  当页面滚动时，要根据滚动的距离来计算 startIndex 和 endIndex，并重新渲染 visibleData
  1. 获取到 scrollTop 距离顶部的距离
  2. startIndex 计算：scrollTop / props.itemHeight
  3. endIndex 计算：startIndex + visibleCount
  4. startOffset 偏移量计算：scrollTop - scrollTop % props.itemHeight

4. 偏移量值是用来计算得到滚动的值
  之所以要减去 scrollTop % itemSize，是为了将 scrollTop 调整到一个与 itemHeight 对齐的位置，避免显示不完整的列表项
  如果 itemHeight 为 100px 的话，那 偏移量的到的值就是一个整数，这样能过更好的对齐

  visibleCount：为计算属性，当屏幕可视区域的高度发生变化时，会重新计算 visibleCount 值 
  visibleData：为计算属性，当 startIndex 和 endIndex 发生变化时，会重新计算 visibleData 值