---
title: css挖坑爬坑之div高宽相等
date: 2017-09-26 22:06
tags: [前端, css]
---
目标效果
对于这么一个头像，外面是一个圆角的div里面是一个img，对于外面的div我要使他高度等于宽度。

<!--more-->


发现问题
开始的时候设置宽度为100%然后高度为100%，这样子对于宽度来说的话可以撑满页面宽度，但是高度的话对于设置了父节点高度就会撑满父节点高度，没有设置父节点高度就不会有高度。

解决问题
网上找到了解决方法就是设置div的宽度然后不设置高度，然后设置他的after的margin-top为100%；因为类似于margin这种属性他的百分比相对于父元素宽度

然后上代码
```
    .head{
      position: relative;
      width:100%;
      border-radius: 50%;
      overflow: hidden;
      &:after {
        content: '';
        display: block;
        margin-top: 100%; //margin 百分比相对父元素宽度计算
      }
      img{
        height: 100%;
        width: 100%;
        position: absolute;
        top: 0;
        left: 0;
      }
    }
```

 