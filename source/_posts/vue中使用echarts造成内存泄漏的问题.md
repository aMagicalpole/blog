---
title: vue中使用echarts造成内存泄漏的问题
copyright: true
date: 2020-11-30 21:49:27
tags: [ vue, echarts, 内存泄漏]
categories: 前端
author: Jerry Liu
top:
description: 之前在vue结合echarts的项目中遇到一个问题，我在切换路由时进行chrome快照，发现一个echarts内存泄漏的问题。
---

&emsp;之前在vue结合echarts的项目中遇到一个问题，我在切换路由时进行chrome快照，发现每次切换路由内存稳步增加十几兆，后面结合快照分析发现是echarts内存泄漏的问题。

```
 this.chart=echarts.init(document.getElementById(dom));
var option={
    //.....................
}
this.chart.setOption(option);
```

 &emsp;在页面中总共使用了十几个图表，由于在每次加载路由时，对每个图表进行了初始化，创建了echarts实例，但是在销毁组件的时候并没有销毁echarts实例。查询资料，官方提供了dispose()方法对实例进行销毁，并释放了实例。用上过后，内存果然不再增长。

```
beforeDestory(){
  echarts.dispose(this.chart);
  this.chart = null;
}
```

注：因为dispose是完全对实例进行了销毁，所以假如要重新构建echarts需要重新使用init方法进行初始化.
