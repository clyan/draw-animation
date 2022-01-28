## PixiJS 是什么
> 简而言之： 就是方便绘制webgl或者canvas的工具， 同时默认采用webGL，当浏览器不支持时默认使用canvas作为备选方案
> 注： canvas只是用于绘图，绘图与动画是两回事

WebGL 是用于访问用户 GPU 以实现快速渲染和高级效果的 JavaScript API

## PixiJS 不是什么
1. PixiJS 不是框架，没有像Unity 之类的库提供完善的游戏开发全链路解决方案，仅仅是用于渲染图形，Pixi是2D渲染引擎，如果需要渲染2D模型那么three更合适。
2. pixi不提供数据存储，配合Cookies、Web Storage等配合使用
3. pixi不提供构建Ui的库
4. pxi不提供音频,使用[PixiJS Sound 插件](https://github.com/pixijs/sound), 或者[Howler.js](https://howlerjs.com/) （web应用音频解决方案）

## Pixi架构
### 自定义选择pixi模块
> 减少文件大小， 不需要引入全部的pixi[PixiJS 自定义工具](https://pixijs.io/customize/)
**主要核心库**
@pixi/core：	PixiJS 系统的核心是渲染器，它显示场景图并将其绘制到屏幕上。PixiJS 的默认渲染器基于 WebGL。

@pixi/display：	创建场景图的主要显示对象：要显示的可渲染对象树，例如精灵、图形和文本。有关更多详细信息，请参见场景图。

@pixi/loader：	加载器系统提供了用于异步加载图像和音频文件等资源的工具。

@pixi/ticker：	Tickers 提供基于时钟的周期性回调。您的游戏更新逻辑通常会在每帧响应一次滴答时运行。您可以同时使用多个代码。

@pixi/app:	Application 是一个简单的帮助器，它将 Loader、Ticker 和 Renderer 包装到一个方便易用的对象中。非常适合快速入门、原型设计和构建简单项目。

@pixi/interaction:	PixiJS 支持触摸和基于鼠标的交互——使对象可点击、触发悬停事件等。

@pixi/accessibility:	通过我们的显示系统编织的是一组丰富的工具，用于启用键盘和屏幕阅读器的可访问性。


**主要模块**

1. 容器Container (类似div) 
2. 显示对象DisplayObject，所有的精灵，文本，复杂图形，容器的父类
> 通常不会直接使用 DisplayObject - 在子类中使用它的函数和属性。
   + position位置：	X 和 Y 位置以像素为单位，并更改对象相对于其父对象的位置，也可直接用作object.x/object.y
   + rotation回转：	旋转以弧度指定，顺时针旋转对象 (0.0 - 2 * Math.PI)
   + angle角度：	角度是旋转的别名，以度而不是弧度 (0.0 - 360.0) 指定
   + pivot枢：	点对象旋转，以像素为单位 - 还设置子对象的原点
   + alpha：	不透明度从 0.0（完全透明）到 1.0（完全不透明），由子代继承
   + scale缩放：	Scale 指定为百分比，1.0 为 100% 或实际大小，并且可以为 x 和 y 轴独立设置
   + skew偏斜：	Skew 在 x 和 y 中变换对象，类似于 CSS skew() 函数，并以弧度指定
   + visible可见的：	对象是否可见，作为布尔值 - 防止更新和渲染对象和子对象
   + renderable可渲染：	对象是否应该被渲染 - 当false，对象仍然会被更新，但不会被渲染，不影响孩子
3. 纹理Texture
   1. 什么是纹理： 我的理解为，屏幕上绘制的所有东西都可以称作纹理，图片、视频、svg等等
   2. 使用流程： Source Image > Loader > BaseTexture > Texture： 原图像 -> 加载器加载图像 -> BaseTexture 管理加载完的img像素源
   3. 销毁BaseTexture 释放内存destroy()， 或者销毁并清除缓存PIXI.utils.destroyTextureCache()
   4. BaseTexture 与 Texture的关系： https://ithelp.ithome.com.tw/articles/10242060
      1. 如一个精灵图里又10个图片，都是同一个图片，那么这一个图片就是BaseTexture， 10个组合成的精灵图就是一个Texture
4. 图形Graphics
   1. 绘制形状的工具,同时可以使用图形制作mask蒙版
   2. 常见的形状函数
      1. Line线
      2. Rect矩形
      3. RoundRect圆形矩形
      4. Circle圆圈
      5. Ellipse椭圆
      6. Arc弧
      7. Bezier and Quadratic Curve贝塞尔曲线和二次曲线
   3. 其他的形状，使用@pixi/graphics-extras
      1. Torus环面
      2. Chamfer Rect倒角矩形
      3. Fillet Rect圆角矩形
      4. Regular Polygon正多边形
      5. Star星星
      6. Rounded Polygon圆角多边形
   4. 案例解释
    ```js
    let obj = new PIXI.Graphics();
    obj.beginFill(0xff0000);
    // 绘制矩形， 只是将需要绘制的矩形数据添加到几何列表Geometry List中，并没有真正的绘制
    obj.drawRect(0, 0, 200, 100);
      // 添加至屏幕上，真正的绘制
    app.stage.addChild(obj);
    ```
   5. 注意事项
      1. 内存溢出，使用destroy()释放内存
      2. 改变图形形状时，不需要删除和重新创建，只需要清除Geometry List几何列表，在添加新的
      3. 如果需要使用透明度，重叠图片，采用AlphaFilter 或 RenderTexture。
5. 相互作用(也就是事件)
   1. DisplayObject 派生对象（Sprite、Container 等）都可以通过将其interactive属性设置为 来实现交互true。
   2. PixiJS 支持三种类型的交互事件——鼠标、触摸和指针。鼠标事件由鼠标移动、点击等触发。触摸事件由具有触摸功能的设备触发。并为两者触发指针事件
   3. 命中，也就是点击对象判断是否命中，内部处理时也是遍历所有的可点击对象
      1. 使得对象的一部分可点击，使用DisplayObject 的 hitArea属性，设置可点击的位置
      2. 优化：优化点击查找对象的时间，如一个背景层使用background.interactiveChildren = false减少点击的对象，加快查找速度
   4. 注意点
      1. pixi中点击事件没有冒泡和捕获
      2. 如果要支持冒泡，则需要在子对象的事件处理代码中显式地重新触发父对象的事件
6. 渲染循环（简单理解也就是动画）
   1. Ticker, 用于执行动画，不断循环
      1. 因为通常屏幕为60fps,但有些新的显示器可以支持144fps及更高，可以设置minFPS和maxFPS属设置范围，但运行环境复杂，使用delta回调中传递的只更合适
7. 渲染树
   1. 跟dom数差不多，不在过多描述
8. 精灵
   1. Sprite 是 PixiJS 中最简单、最常见的可渲染对象。它们代表要在屏幕上显示的单个图像。每个Sprite都包含一个要绘制的纹理，以及在场景图中运行所需的所有转换和显示状态。
   2. Alpha、色调和混合模式
       1. Alpha 是标准的显示对象属性。您可以使用它通过在一段时间内将每个精灵的 alpha 从 0.0 设置为 1.0 来将精灵淡入场景中。
       2. 着色允许您将每个像素的颜色值乘以一种颜色。例如，如果你有一个地下城游戏，你可以通过设置来显示一个角色的中毒状态obj.tint = 0x00FF00，这会给角色一个绿色的色调。
       3. 混合模式改变渲染时像素颜色添加到屏幕的方式。三种主要模式是add，它将每个像素的 RGB 通道添加到您的 sprite 下的任何内容（对发光和照明很有用），multiply其工作方式类似tint，但在每个像素的基础上，以及screen，它覆盖像素，使任何内容变亮在他们下面。
   3. Pivot轴心 vs Anchor锚点 、
      1. Pivot： Sprite 左上角的偏移量，以像素表示。它默认为 (0, 0)， 100 像素 x 50 像素的 Sprite，并且希望将轴心点设置为图像的中心，您可以将轴心点设置为 (50, 25) - 宽度的一半，
      2. anchor仅适用于 Sprite， 锚点在每个维度中以百分比指定，从 0.0 到 1.0。要使用锚点围绕纹理的中心点旋转，您需要将 Sprite 的锚点设置为 (0.5, 0.5) - 宽度和高度的 50%。
      3. 设置锚点或枢轴不仅会影响旋转原点，还会影响精灵相对于其父对象的位置。
9. 文本
   1.  PIXI.Text
       1.  静态文本
       2.  少量文本对象
       3.  高保真文本渲染（例如字距调整）
       4.  文本布局（行和字母间距）
   2.  PIXI.BitmapText
       1.  动态文本
       2.  大量文本对象
       3.  较低的内存


## 第三方库
### 缓动动画
[tweenjs](https://github.com/tweenjs/tween.js/) 注意不要跟createjs的tweenJS的搞混了