# 多图片自动左右滚动，移动触摸版，vue2.0版

* [demo](https://github.com/lin09/demo/tree/master/image-scroll-vue)
* [预览](https://lin09.github.io/demo/image-scroll-vue/dist/index.html)
* ![扫一扫预览](https://lin09.github.io/demo/image-scroll-vue/public/qr.jpg) 扫一扫预览

## 安装
```
yarn add -D @lin09/image-scroll-vue
// 或
npm i -D @lin09/image-scroll-vue
```

## 使用
```
// App.vue

<template>
  <div id="app">
    <ImageScroll :images="images" height="292px" />
  </div>
</template>

<script>
import ImageScroll from '@lin09/image-scroll-vue'

export default {
  name: 'app',
  components: {
    ImageScroll
  },
  data() {
    return {
      images: [{
        imgUrl: 'https://gw.alicdn.com/imgextra/i4/1/O1CN01fFFjmR1BsUtzbDibn_!!1-0-lubanu.jpg_790x10000Q75.jpg_.webp',
        href: 'https://www.xx.xx'
      },{
        imgUrl: 'https://gw.alicdn.com/imgextra/i1/124/O1CN01QTazoU1CmpcKM3axV_!!124-0-luban.jpg_790x10000Q75.jpg_.webp'
      },{
        imgUrl: 'https://gw.alicdn.com/imgextra/i1/50/O1CN01x3WLUL1CEwJJq7Esw_!!50-0-lubanu.jpg_790x10000Q75.jpg_.webp'
      },{
        imgUrl: 'https://gw.alicdn.com/imgextra/i3/103/O1CN01IALwmx1CdDImsIMWb_!!103-0-lubanu.jpg_790x10000Q75.jpg_.webp'
      }]
    }
  }
}
</script>

<style>
body {
  margin: 0;
}
</style>
```

## 参数
* 数据
  * :images="[{ imgUrl: 'url', href: 'url' }, ...]"
* 高度
  * height="292px"
* 底点
  * pagination="false" // 隐藏