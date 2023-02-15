## 浏览器的重排与重绘（《高性能JavaScript》）

### 何时发生重排？
当DOM的变化影响了元素的几何属性（宽和高），浏览器需要重新计算元素的几何属性，浏览器会使渲染树中受影响的部分失效，并重新构造渲染树，这个过程成为重排（reflow）。重排后，浏览器会将受影响的部分绘制到屏幕中，这个过程是重绘（repaint）。

下述过程会发生重排：
1. 添加或删除可见的DOM元素。
2. 元素位置改变。
3. 元素尺寸改变。
4. 内容改变（文本改变或图片被不同尺寸的图片替代）。
5. 页面渲染初始化。
6. 浏览器窗口尺寸变化。

### 如何优化？
浏览器会通过队列化修改并批量执行来优化重排过程，然而有些操作会刷新队列并立即执行重排，比如获取布局信息（读取offsetTop，scrollLeft，clientBottom，getComputedStyle等方法属性）。注意，通过这些属性修改样式会刷新队列，即使你在获取最近未发生改变的或者与最新改变无关的布局信息。例如，如果遇到计算computed样式的语句，将其放到样式修改代码的末尾，这样重拍次数最少。

改变样式时尽量使用css或者style.cssText属性一次性修改。

另一种方法可以减少修改DOM操作的重排：使元素脱离文档流。具体做法：1. 隐藏元素，应用修改，重新显示。2. 使用document fragment在当前DOM外构建子树，再拷贝回文档。3. 将原始元素拷贝到一个脱离文档的节点中，修改副本，再替换原始元素。

尽量减少布局信息的获取次数，获取后把它赋值给局部变量，然后再操作局部变量。

动画里对重排的优化：使用绝对定位页面上的动画元素，使其脱离文档流。动画结束再恢复定位。

在元素很多时避免使用:hover伪选择器。

使用JavaScript执行动画时，传统方法是通过定时器来定期执行动画函数，这种做法的问题是不好掌控定时的周期，太短了浏览器还没有刷新渲染，太长了动画不够平滑顺畅。因此，最佳周期是遵循浏览器的刷新率大约17ms，但问题是定时器函数无法精确实现这个级别的周期。

### RequestAnimationFrame方法
requestAnimationFrame方法就是解决上述问题，它通知浏览器某些JavaScript代码要执行动画了，浏览器可以在运行某些代码后进行优化。它的参数是一个在重绘前调用的回调函数。可以将多个requestAnimationFrame方法串联起来，实现循环，因为每次调用该函数，回调函数只会执行一次。

一个例子：节流滚动操作的事件处理器。

```javascript
let enabled = true;

function scrollHandler() {
    console.log("This handler is expensive and should control running times");
}

window.addEventListener('scroll', () => {
    if (enabled) {
        enabled = false;
        requestAnimationFrame(scrollHandler);
        // Only handle scroll every 100ms
        setTimeout(() => {
            enabled = true;
        }, 100);
    }
});
```