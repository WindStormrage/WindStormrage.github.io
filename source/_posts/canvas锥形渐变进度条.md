---
title: canvas锥形渐变进度条
date: 2020-05-28 16:41
tags: [canvas]
---
# 从一个渐变圆角进度条浅出画一个圆

## 开始
这一切需要从一个（简单）的需求开始，在最开始对设计第一眼看到这张图的时候，感觉挺简单的嘛，直接用echarts饼图模拟出来一个就好了

![image](http://p9.qhimg.com/dm/200_200_/t01b9c45ade0385f4e1.png)

<!--more-->

## echarts

然后上echarts试了一下发现实现不出来了

![image](http://p3.qhimg.com/dm/200_200_/t0161cd9dfa2f5a7997.png)

设计图这边采用的是锥形渐变，而echarts只有线性渐变和径向渐变。

## css

然后准备换种方案，css就有锥形渐变，然后通过conic-gradient加上mask画出了一个[渐变的环形](https://jsbin.com/holuxusiyi/1/edit?html,css,output)然后可以再通过剪裁实现出进度的展示。

但是存在两个问题，一个是conic-gradient属性兼容性不好ie和火狐都不支持，二个是后来发现了还存在一个需求进度条的两端需要有圆角，然后这种实现方式就不行了。
> 其实在写这篇文章的时候才想到一个方法就是在两端加上两个半圆形，不过得计算半圆形的位置。

## Canvas & SVG
在我的理解中在页面上作图总共有四种方式。
* dom+css
* Canvas
* SVG
* WebGL

WebGL是一头雾水还是试试Canvas和SVG吧，因为更熟悉Canvas一些，我这边就采用Canvas来试试。

Canvas可以轻松的实现[圆角和环形](https://jsbin.com/ceyehazaxo/1/edit?html,js,output)，但是他的api里面居然没有锥形渐变

然后就想着尝试手动来实现一个锥形渐变，然后查阅资料看到了一篇文章[手把手教你画圆锥渐变](https://juejin.im/post/5cc506f7f265da03474e0a73)，就是相当于画圆嘛，我们可以通过一条线一条线的画从而画出一个圆，然后把两端渐变的颜色通过计算找到中间画圆的每一条线的颜色组合起来就是一个渐变的效果了。

然后问题就是给你两个色值怎么计算中间的线段的颜色，其实对于rgba的颜色我们可以看到他是由四个数字组成的，那我把这四个数字分别求出四组长度相同且组内间隔相同的中间值那就可以得到颜色的中间值了，然后在搭配上张老师硬核的色值转换[JS HEX十六进制与RGB,HSL颜色的相互转换](https://www.zhangxinxu.com/wordpress/2010/03/javascript-hex-rgb-hsl-color-convert/)那就可以实现出我们想要的效果了。

通过一个开始颜色和一个结束颜色，默认是rgba的颜色，num是分段数，就可以求出中间每一段的颜色了
```javascript
  // 把颜色分段
  const beginColor = begin.slice(5, -1).split(',').map(item => Number(item))
  const endColor = end.slice(5, -1).split(',').map(item => Number(item))
  // 分段后的颜色储存在这个数组
  const middleColor = [[], [], [], []]
  // 循环rgba四种
  beginColor.forEach((item, index) => {
    // 当前的值每段颜色之间的间隔
    const differ = (endColor[index] - item) / (num - 1)
    // 循环分段数的次数
    for(let i = 0; i< num; i++) {
      // 每次加上这个间隔
      middleColor[index].push((item + differ * i).toFixed(2))
    }
  })
  console.log(middleColor)
  console.log(num)
```

然后绘制的话就是一段一段的画了，麻烦的地方就是计算每次从多少的角度画到多少的角度

```javascript
  for(let i = 0; i< num; i++) {
    ctx.beginPath()
    // 这里是每次绘制的过程
    // 每次绘制一段小圆弧
    // 最后一段只需要画一段就好    
    if(i === num - 1) 
      ctx.arc( 150 * dpr,150 * dpr, 100 * dpr, (Math.PI * 2 * value) / num * i, (Math.PI * 2 * value) / num * (i + 1));
    else
      ctx.arc( 150 * dpr,150 * dpr, 100 * dpr, (Math.PI * 2 * value) / num * i, (Math.PI * 2 * value) / num * (i + 2));
    ctx.lineWidth = 60;
    // ctx.lineCap = "round";
    ctx.strokeStyle= `rgba(${middleColor[0][i]}, ${middleColor[1][i]}, ${middleColor[2][i]}, ${middleColor[3][i]})`;
    ctx.stroke();   
    ctx.closePath() 
  }
```

结果非常的顺利，就是自己想要的结果，具体怎么一段一段的画，怎么求出颜色的中间值[看这里](https://jsbin.com/coconipuhi/2/edit?js,output)

## 优化
其实把绘制的过程放慢看

![image](http://p8.qhimg.com/t01c635d899a51a97a3.gif)

就是这个过程，每次画一段，每段不同的颜色组合起来就是一个渐变色，然后分段数再加多一点就会靠上去很流畅。

在完成后发现了几个问题，首先是在分段数很少的时候就会出现一块一块的间隔

![image](http://p6.qhimg.com/t01f57c7542eaebbe58.png)

就像这样，我大概分析了一下，猜测这个间隔出现的原因应该是我计算每一段的角度的时候肯定有除不尽的，我就四舍五入了，应该就会产生一些小间隔。

然后我就觉得把分段数提高应该就会好一些，然后就发现分段数高间隔会生成类似于摩尔纹的东西

![image](http://p5.qhimg.com/t0162ef7e517cdc4087.png)

然后就开始思考怎么去消除，最后想到了一种方案就是在每次绘制的时候绘制两段的长度，然后移动只移动一段的长度，就会下一段覆盖在上一段，就不会有间隔了，然后颜色渐变也还是一段一段的不会有影响。

![image](http://p4.qhimg.com/t01894f2312db2bc371.gif)

然后还有一个问题就是有锯齿，不清楚，解决方案也很简单，就是把你的画布放大指定倍数，然后半径也放大同样的倍数，最后dom的高宽不变，就会让绘制的图形更加的清晰。

## 总结
到此这个问题就算是解决了，然后我还顺便写了一个库，大家有兴趣的可以去使用一下，我还加上了数字，动画也可以支持多段渐变[gradient-ring-progress](https://github.com/WindStormrage/gradient-ring-progress)

通过这次的需求我收获到了，做东西需要完全的去了解了需求再去确定实现方案然后再动手，实现方案其实有非常多种，我们需要找到的是最合适的解决方案。最后抄袭张老师的一句话
> 然而一个人的积累总是有限，而创意总是无限的，因此一定还有其他更好更妙更简单的实现，欢迎分享欢迎指教！

## 参考资料

[CSS conic-gradient()锥形渐变简介](https://www.zhangxinxu.com/wordpress/2020/04/css-conic-gradient/?shrink=1)

[手把手教你画圆锥渐变](https://juejin.im/post/5cc506f7f265da03474e0a73)

[JS HEX十六进制与RGB,HSL颜色的相互转换](https://www.zhangxinxu.com/wordpress/2010/03/javascript-hex-rgb-hsl-color-convert/)

