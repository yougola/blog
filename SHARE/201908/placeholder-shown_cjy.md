# CSS :placeholder-shown伪类实现Material Design占位符交互效果
### 一、Material Design规范中占位符交互效果
Material Design风格占位符交互效果官方示意见此[demo](https://material-components.github.io/material-components-web-catalog/#/component/text-field?type=filled)页面。
现在这种设计在移动端很常见，相信不少人设计项目中有实现过这种交互，而且，大部分是利用JS实现的。（ps:weex  不支持这个样式）

实际上，我们可以借助CSS :placeholder-shown伪类，纯CSS，无任何JS，实现这样的占位符交互效果。

:placeholder-shown表示，当输入框的placeholder内容显示的时候，输入框干嘛干嘛。

:placeholder-shown伪类目前兼容性如下：
[兼容性链接](https://caniuse.com/#search=%3Aplaceholder-shown)
![](../assets/images/pic1.png)

兼容性还是很不错的，在移动端我们可以放心使用。因为就算一些老手机不支持，也不过是传统的placeholder占位符效果，并没有什么损失
### 二、placeholder-shown  优点
纯CSS实现，要比JS实现好一千倍，代码少，性能高，样式调整方便，上手简单容易，可谓是前端必备技能了。

### 三、实现原理
[jsbin 编辑链接](https://jsbin.com/jisidoqazi/edit?html,css,output)
拿一个输入框举例，HTML结构如下：
```
<div class="input-fill-box">
    <input class="input-fill" placeholder="邮箱">
    <label class="input-label">邮箱</label>
</div>
```
首先，让浏览器默认的placeholder效果不可见，我们可以让颜色透明即可，如下CSS：

/* 默认placeholder颜色透明不可见 */
```
.input-fill:placeholder-shown::placeholder {
    color: transparent;
}
.input-fill{
  margin: 0;
  font-size: 16px;
  line-height: 1.5;
  outline: none;
  padding: 20px 16px 6px;
  border: 1px solid transparent;
   background: #f5f5fa;
  border-radius:10px;
  transition: border-color .25s;
}
然后，后面的.input-label这个label元素代替成为我们肉眼看到的占位符。我们可以采用绝对定位：
.input-fill-box {
    position: relative;
}
.input-label {
    position: absolute;
    left: 16px; top: 14px;
    pointer-events: none;
  color:#BEC1D9;
   padding: 0 2px;
    transform-origin: 0 0;
}
```
最后，对这个label元素在输入框focus时候，以及非placeholder显示的时候进行重定位（缩小并位移到上方）：
```
.input-fill:not(:placeholder-shown) ~ .input-label,
.input-fill:focus ~ .input-label {
    transform: scale(0.75) translate(0, -14px);
}
.input-fill:focus
{
    border-color: #283282;
}
```
### 四、清除按钮
1.html 部分
input上  required是必要属性，配合CSS伪类实现我们的效果。
```
<code>
 <a href="javascript:" class="clear">close</a>
 </code>
 ```
 2.CSS部分
使用的是:valid伪类。这是CSS3中新增伪类，IE10+以及其他现代浏览器支持，表示表单合法。由于HTML中的<input>有HTML5表单验证属性required. 于是，如果文本框没有内容，则不合法；有内容，则合法，就会触发这里的:valid伪类选择器。而这里:valid伪类控制后面的清除按钮显示，于是就实现了我们想要的效果。
啊，对了。IE11浏览器下不是所有的文本框都有黑色的叉叉吗，会跟这里的自定义清除按钮重叠，::-ms-clear { display: none; }这段代码可以去之~~
```
 .clear{
  position:absolute;
  top:10px;
  right:-20px;
   display: none;
    transition: all .25s;
}
.input-fill::-ms-clear { display: none; }
.input-fill:valid + .clear { display: inline; }
.input-fill:not(:focus) + .clear { display: none; }
```
3.实现的优点
此方法相比传统JS实现的好处在于，更简单了。JS的话还要侦听输入事件(input)等，比较折腾。CSS的话完全浏览器自身事件特性，显然，高效简单的多。

4.实现的不足
不足在于，兼容性。IE9-以下的浏览器只能点蜡烛了。

不过，写写原型啊，demo；或者渐进增强使用；或者移动端开发等，都可以试试这个新技能。
