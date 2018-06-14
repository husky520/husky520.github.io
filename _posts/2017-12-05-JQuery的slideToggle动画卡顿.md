---
layout: master
title:  "JQuery的slideToggle动画卡顿"
categories: Javascript
---
**事件起因**
笔者在用 JQuery 制作一个竖向手风琴菜单时（如图1），发现用`slideToggle`类的动画过程中出现卡顿。

![图1](http://upload-images.jianshu.io/upload_images/5908325-d16406dbdc394bba.png)

JS代码如下（仅仅为了强调使用了slide动画...）：
~~~ js
$('.menu-item').click(function () {
      var $this = $(this);

      if ($this.hasClass('open')) {
            $this.removeClass('open');
            $this.siblings('.menu-item-second').slideUp();
      } else {
            $this.addClass('open');
            $this.parent().siblings().children('.menu-item').removeClass('open');
            $this.parent().siblings().children('.menu-item-second').slideUp();
            $this.siblings('.menu-item-second').slideDown('100');
      }
 });
~~~
***

**原因**
后来发现原因是在前面的 css 代码中，对执行动画的元素使用了`transition`属性，并且过渡的效果写的是`all`，代码如下：
~~~ css
.menu-item-second li {
    background-color: #ddd;
    transition: all 300ms;
}
.menu-item-second li:hover {
    background-color: #fff;
}
~~~
***

**修改**
修改后的代码将`all`属性改为`background-color`后恢复流畅效果！
~~~ css
 transition: background-color 300ms;
~~~

***

**分析**
查阅 JQuery 文档后发现：
> 通过高度变化来切换所有匹配元素的可见性，并在切换完成后可选地触发一个回调函数。
> 这个动画效果只调整元素的高度，可以使匹配的元素以“滑动”的方式隐藏或显示。

`slideToggle` , `slideDown` , `slideUp`原理是调整高度值，`transition: all 300ms;` 过渡的效果也包含高度变化，两者产生不良反应。
***

**拓展**
另一方面，笔者之前写过一个案例，其中对同一个元素同时使用过 `slideDown` 和
 `slideUp`，两种效果的动画时长不同，但是却不会影响其流畅度。
***

能力所限，未深入底层。