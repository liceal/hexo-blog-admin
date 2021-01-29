---
title: VUE自动引入组件
date: 2020-04-22 15:04:11
tags: vue
---

> 这个教程讲如何只用创建一个目录就可以自动引入组件并且展示。
>
> 平时个人做一些练习，对一个知识点划分区域，会分成好几个组件，然后引入在一个主入口展示。为了方便，我们会用到for循环来列出所有组件，但是需要在JS里配置好引入的包，然后使用component动态引入组件，重复操作很多，所以出个此教程方便日常练习模块引入。

需要安装 webpack

<!-- more -->

#### DOM

```vue
<div v-for="(name,key) in requireComponents" :key="key">
    <component :is="name"/>
</div>
```

使用component 动态渲染组件

#### JS

```js
export default {
  data() {
    return {
      requireComponent: []
    };
  },
  mounted(){
      /**
       * 参数
       * src 下的 @/指定文件夹
       * 是否 包含子目录 中的文件
       * 匹配文件 正则表达式
       * @return Array
       */
      const files = require.context(
        "@/views",
        true,
        /\.\/.*\/.*\.vue$/
      ); //返回数组 ['./dirName/index.vue']
      
      let i = 0;
      files.keys().forEach(element => {
        const fileName = element.match(/\.\/(.*)\//)[1]; // dirName
        this.requireComponent.push(fileName);
        this.$options.components[fileName] = () => import("./" + fileName);// 注册组件
      });
  }
}
```

只要创建一个文件夹 文件夹内创建 index.vue 主入口 即可，页面会找到所有文件夹名字并且引入渲染:grin:





