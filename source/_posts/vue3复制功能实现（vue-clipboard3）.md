---
title: vue3复制功能实现（vue-clipboard3）
date: 2023-05-26 13:56:45
tags: vue
categories: 技术
---

![](https://w.wallhaven.cc/full/m3/wallhaven-m3zjx1.jpg)

* 安装 `vue-clipboard3`

```sh
$ npm install --save vue-clipboard3
```

* 在 `setup () {}` 中使用：

```vue
<template>
  <button @click="touchCopy">复制链接</button>
</template>

<script>
import { defineComponent } from 'vue'
// 导入插件
import useClipboard from 'vue-clipboard3'

export default defineComponent({
  setup () {
    // 点击复制
    function touchCopy () {
      // 调用
      copy('拷贝内容')
    }
    // 使用插件
    const { toClipboard } = useClipboard()
    const copy = async (msg) => {
      try {
        // 复制
        await toClipboard(msg)
        // 复制成功
      } catch (e) {
        // 复制失败
      }
    }
    // 导出
    return {
      // 点击复制
      touchCopy
    }
  }
})
</script>
```

* 在 `<script setup>` 中使用：

```vue
<template>
  <button @click="touchCopy">复制链接</button>
</template>

<script setup>
// 导入插件
import useClipboard from 'vue-clipboard3'

// 点击复制
function touchCopy () {
  // 调用
  copy('拷贝内容')
}
// 使用插件
const { toClipboard } = useClipboard()
const copy = async (msg) => {
  try {
    // 复制
    await toClipboard(msg)
    // 复制成功
  } catch (e) {
    // 复制失败
  }
}
</script>
```

