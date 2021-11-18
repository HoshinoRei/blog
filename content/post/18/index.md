---
title: 关于 router.push() 会报错这件事
author: 星野玲
date: 2023-09-24T08:00:00+08:00
description: 好久没有更新文章了，今天小玲水一篇文章。说说自己在使用 Vue 的时候踩到的一个坑。
draft: false
tags:
  - Vue
  - Vue3
  - Vue Router

---

好久没有更新文章了，今天小玲水一篇文章。说说自己在使用 Vue 的时候踩到的一个坑。

这是一个 vue 文件，看似没有什么问题。

```html
<template>
  <!-- 创建一个退出登录按钮 -->
  <el-button type="danger" @click="logout">退出登录</el-button>
</template>

<script setup lang="ts">
import { ElButton, ElMessage } from 'element-plus'
import { useRouter } from 'vue-router'

// 定义一个名为logout的函数，用于处理退出登录的逻辑
function logout() {
  // 显示一个退出登录成功的消息提示框
  ElMessage({
    message: '退出登录成功',
    // 在消息提示框关闭时执行回调函数
    onClose: async () => {
      // 获取Vue Router实例
      const router = useRouter()
      // 执行路由跳转，跳转到名为'login'的路由
      await router.push({ name: 'login' })
    },
    type: 'success'
  })
}
</script>
```

不过，咱们把它运行起来，在点击退出登录按钮后，并没有成功跳转到名为 `login` 的路由，控制台报错：

```text
Uncaught (in promise) TypeError: Cannot read properties of undefined (reading 'push')
```

看样子，`router` 这个常量是 `undefined` 的。按理来说不应该呀，根据 [文档](https://router.vuejs.org/zh/guide/advanced/composition-api.html#%E5%9C%A8-setup-%E4%B8%AD%E8%AE%BF%E9%97%AE%E8%B7%AF%E7%94%B1%E5%92%8C%E5%BD%93%E5%89%8D%E8%B7%AF%E7%94%B1)，通过调用 `useRouter()` 函数应该会返回一个 Vue Router 实例。

小玲想了想，会不会是 `const router = useRouter()` 这个语句的位置不对，小玲把它放在这里的理由是 `router` 常量第一次使用的时候就是在这，想着挨近一点比较好。于是小玲移动 `const router = useRouter()` 到 `onClose` 外面。

```html
<template>
  <el-button type="danger" @click="logout">退出登录</el-button>
</template>

<script setup lang="ts">
import { ElButton, ElMessage } from 'element-plus'
import { useRouter } from 'vue-router'

function logout() {
  const router = useRouter()
  ElMessage({
    message: '退出登录成功',
    onClose: async () => {
      await router.push({ name: 'login' })
    },
    type: 'success'
  })
}
</script>
```

还是不行，那再移动到 `lougot()` 函数外面。

```html
<template>
  <el-button type="danger" @click="logout">退出登录</el-button>
</template>

<script setup lang="ts">
import { ElButton, ElMessage } from 'element-plus'
import { useRouter } from 'vue-router'

const router = useRouter()

function logout() {
  ElMessage({
    message: '退出登录成功',
    onClose: async () => {
      await router.push({ name: 'login' })
    },
    type: 'success'
  })
}
</script>
```

这次就正常了，小玲后来搜了搜，原来已经 [有人遇到过这个问题了](https://segmentfault.com/a/1190000043613525)。总而言之，`useRouter()` 一定要在 `script` 标签里的顶层调用。

至于为什么，[文档](https://router.vuejs.org/zh/guide/advanced/composition-api.html#%E5%9C%A8-setup-%E4%B8%AD%E8%AE%BF%E9%97%AE%E8%B7%AF%E7%94%B1%E5%92%8C%E5%BD%93%E5%89%8D%E8%B7%AF%E7%94%B1) 里有提到 `useRouter()` 是用来替代以前的写法中的 `this.$router`。

> 因为我们在 setup 里面没有访问 this，所以我们不能再直接访问 `this.$router` 或 `this.$route`。作为替代，我们使用 `useRouter` 和 `useRoute` 函数。

假设咱们还在使用以前的写法，如果不放在顶层，那么这个 `this` 指向谁呢？相信聪明的你一定能理解小玲的意思，小玲就不再赘述了。
