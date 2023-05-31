---
title: VUE3实现组件刷新
date: 2023-05-31 13:54:08
tags: VUE3
categories: 技术
---

VUE3中实现组件刷新

<img src="https://img2.baidu.com/it/u=4154503761,1097204876&fm=253&fmt=auto&app=138&f=JPEG?w=888&h=500" style="zoom:50%;" />

provide ： 向子组件以及子孙组件传递数据。接收两个参数，第一个参数是 key，即数据的名称；第二个参数为 value，即数据的值
inject ： 接收父组件或祖先组件传递过来的数据。接收一个参数 key，即父组件或祖先组件传递的数据名称

通过依赖注入（provide和inject）实现自定义页面刷新事件

原理： 给app.vue中router-view绑定v-if事件，在函数中控制v-if的值在短时间内由true到false再到true,从而使页面达到刷新效果

**刷新实现**

APP.VUE

```vue
<script setup>
import { RouterView } from 'vue-router'
import { ref, provide, nextTick } from 'vue'

const isRouterActive = ref(true)
provide('reload', () => {
  isRouterActive.value = false
  nextTick(() => {
    isRouterActive.value = true
  })
})
</script>

<template>
	<router-view v-if="isRouterActive"></router-view>
</emplate>
```

**刷新页面**
需要用到刷新事件的子组件：

```vue
<script lang="ts" setup>
import { reactive, inject  } from 'vue'
//刷新页面
//注入刷新事件,这里括号中的参数要对应上前面provide中的第一个参数 
const reload: any = inject('reload')
const jump = (item: any) => {
	....
  	....
  	reload();
    ....
    ....
}
</script>
```

