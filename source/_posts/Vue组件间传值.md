---
title: Vue组件间传值
tags: study
---
> 最近把Pyblog重构成前后端分离的项目时遇到的问题: 当组件化页面元素后如何使页面组件间进行数据交换

## 使用插件 vue3-eventbus

使用eventbus可以实现
- 1. 安装依赖
- 2. 现在main.js中把event导入
```JavaScript
import eventBus from 'vue3-eventbus'
```
- 3. 具体使用到时 - 传出值的组件
```JavaScript
import bus from 'vue3-eventbus'

bus.emit('事件名',参数)
```

- 4. 具体使用到时 - 接收值的组件
```JavaScript
import bus from 'vue3-eventbus'

bus.on('事件名',回调函数处理接收参数)
```