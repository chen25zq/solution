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

#### 初始化时
1. onMounted 页面加载钩子获取数据给各个字段赋值
  获取最外层容器的可视区域高度，将其设置成 screenHeight 变量的值
  可显示列表项数 visibleCount：screenHeight / itemHeight，即屏幕可视区域的高度除以每一项的高度(向下取整)
  startIndex 开始索引初始化为 0， startOffset 开始偏移量初始化为 0
  endIndex 结束索引：初始化为 startIndex + visibleCount

2. 那此时要渲染的数据就可以根据上面的这些初始化数据计算得到
  visibleData：从原始数据中截取 startIndex 到 endIndex 的数据

#### 滚动事件：
  当页面滚动时，要根据滚动的距离来计算 startIndex 和 endIndex，并重新渲染 visibleData
  1. 获取到 scrollTop 距离顶部的距离
  2. startIndex 计算：scrollTop / props.itemHeight
  3. endIndex 计算：startIndex + visibleCount
  4. startOffset 偏移量计算：scrollTop - scrollTop % props.itemHeight

#### 偏移量值是用来计算得到滚动的值
  之所以要减去 scrollTop % itemSize，是为了将 scrollTop 调整到一个与 itemHeight 对齐的位置，避免显示不完整的列表项
  如果 itemHeight 为 100px 的话，那 偏移量的到的值就是一个整数，这样能过更好的对齐

  visibleCount：为计算属性，当屏幕可视区域的高度发生变化时，会重新计算 visibleCount 值 
  visibleData：为计算属性，当 startIndex 和 endIndex 发生变化时，会重新计算 visibleData 值

## 动态高度问题

1. 在这里的实现是给定了一个固定的高度，每一个 item 的高度是固定的。但是在实际开发中并不是所有 item 的高度都是固定的，比如有些 item 可能是图片，有些 item 可能是文字。所以我们需要动态计算 item 的高度。

不固定高度会导致的问题：

- 列表总高度：listHeight = listData.length * itemSize
- 偏移量：startOffset = scrollTop - (scrollTop % itemSize)
- 数据起始索引： startIndex = Math.floor(scrollTop / itemSize)

这些属性的计算方式就不能安装上面的进行计算了

真实要解决的问题是：
1. 如何获取真实高度？
    - 如果能获得真实高度，就好解决。但实际渲染之前是很难拿到每项的真实高度。因此采用一个预估高度渲染出真实dom，再根据dom实际情况更新真实高度，然后将这个真实高度缓存。得到一个缓存列表
2. 相关属性如何计算？
    - 之前的计算方式无法继续使用，因为都是针对固定之来计算
    - 这需要根据缓存列表重写计算属性、滚动回调函数
3. 列表渲染项目有什么变化？
    - 因为用于渲染页面元素的数据是根据 **开始/结束索引** 在 **数据列表** 中筛选出来的，所以只要保证索引的正确计算，那么**渲染方式是无需变化**的。
   - 对于开始索引，我们将原先的计算公式改为：在 **缓存列表** 中搜索第一个底部定位大于 **列表垂直偏移量** 的项并返回它的索引
   - 对于结束索引，它是根据开始索引生成的，无需修改。

## 实现步骤

1. onMounted 时初始化列表项的位置信息, 利用预估=假设的高度来建立缓存列表，缓存列表的每一项包含下标、高度、顶部位置、底部位置。

```js
positions = props.listData.map((_, index) => ({
    index, // 列表项的下标
    height: props.estimatedItemSize, // 列表项的高度，这里采用预估的高度
    top: index * props.estimatedItemSize, // 列表项的顶部位置，根据下标和预估高度计算
    bottom: (index + 1) * props.estimatedItemSize // 列表项的底部位置，也是根据下标和预估高度计算
  }))
```

2. 开启监听：

```js
watch(() => props.listData, initPostions)
```

3. 更新偏移量，在 滚动对应的处理函数 中获取到缓存列表的第一个元素的 底部 bottom 属性作为偏移量的值进行设置

4. 在页面更新数据之后，更新缓存列表的高度信息，并重新计算缓存列表的位置信息。此时需要在 onUpdated 回调方法中调用 nextTick 方法，确保缓存列表的更新已经完成。

```js
onUpdated(() => {
  // 这里之所以使用 nextTick，是为了确保DOM更新完毕后再去获取列表项的位置信息
  nextTick(() => {
    if (!items.value || !items.value.length) return
    // 1. 更新列表项的高度
    updateItemsSize()
    // 2. 更新虚拟列表的高度
    listHeight.value.style.height = positions[positions.length - 1].bottom + 'px'
    // 3. 更新列表的偏移量
    setStartOffset()
  })
})
```

5. 此时可以拿到渲染后的node节点，根据 getBoundingClientRect 方法获取到真实的高度，并与预估的高度进行比较，如果存在差值，则需要更新缓存列表的高度信息，并重新计算缓存列表的位置信息。


```js
// 用于创建列表项元素的引用
const items = ref([])

items.value.forEach((node, index) => {
  // 获取列表项实际的高度
  let height = node.getBoundingClientRect().height
  // 计算预估高度和真实高度的差值
  let oldHeight = positions[index].height // 拿到该项的预估高度
  let dValue = oldHeight - height
  if (dValue) {
    // 如果存在差值，那么就需要更新位置信息
    positions[index].bottom -= dValue
    positions[index].height = height
    // 接下来需要更新后续所有列表项的位置
    for (let i = index + 1; i < positions.length; i++) {
      positions[i].top = positions[i - 1].bottom
      positions[i].bottom -= dValue
    }
  }
})
```