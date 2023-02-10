## JavaScript模块系统
### 概念
依赖图：以**有向图**来表示应用程序中个模块的依赖关系。

依赖分析：对静态分析友好（非动态依赖）的模块系统可以让打包工具更容易将代码处理为较少的文件。

循环依赖：要构建没有循环依赖的程序几乎不可能，包括CommonJS、AMD和ES6的所有模块系统都支持循环依赖。在包含循环依赖的程序中，模块的加载顺序可能出人意料，需要恰当封装。加载器会执行**深度优先**的依赖加载。

### ES6之前的模块

使用立即调用函数表达式：
```javascript
var globalFlag = true;
var Foo = (function(flag) {
    var internalModule = (function(){
        return {
            getName: function() { return "internal";}
        };
    })();
    return {
        bar: 'baz',
        get: function() {
            if (flag)
                console.log(this.bar);
        }
        internal: internalModule
        
    };
})(globalFlag);
Foo.get();
Foo.internal.getName();
```

将函数返回值赋值给变量，就是**为模块创建了命名空间**。模块内部可以定义模块，形成命名空间的嵌套。如果模块要是外部的变量，可以传递给立即执行函数。可以通过传入已有模块的方式进行扩展：

```javascript
// Foo 为已有模块
var Foo = (function() {
    FooModule.newFunction = function() {
        console.log("Newly added function of Foo");
    }
    return FooModule;
})(Foo || {});
```

### CommonJS模块
CommonJS规范概述了**同步声明依赖**的模块定义，主要用于服务端实现模块化组织代码，假设加载没有网路延迟，不能直接在浏览器环境中运行。

CommonJS通过`require`方法指定依赖。无论一个模块在requre中被引用多少次，**模块永远是单例**。模块加载遵循依赖图，且是同步的。CommonJS也支持动态依赖。如：
```javascript
if (condition) {
    var mod = require('./moduleA');
}
```

模块可以没有任何公共接口，一旦被引入，也会在加载后执行模块体。模块的一个主要用途是托管类定义，将类定义作为module.exports的值导出。

要在浏览器中运行CommonJS代码，需要提前把模块文件以正确的顺序打包好，把全局属性转换为**原生的JavaScript结构**，将模块代码**封装在函数闭包**中，最终只提供一个文件。

### 异步模块定义（AMD）
AMD以浏览器为目标执行环境，需要考虑网络延迟的问题。

AMD模块实现的核心是用函数包装模块定义，包装模块的函数是全局的define函数的参数，由AMD加载器库实现。与CommonJS不同，AMD支持**指定模块的字符串标识符**。

```javascript
define('ModuleA', ['ModuleB'], function(moduleB) {
    return {
        do: moduleB.do()
    };
});
```
AMD中也可以使用require和exports作为依赖来定义模块（动态加载）。

### ES6模块系统
CommonJS与AMD的冲突正式我们现在享用的ES6模块规范诞生的温床。

ES6模块需要将`script`标签的`type`属性指定为`module`，可以同时使用defer和async属性。type为module的脚本都可以作为**入口模块**，入口模块的数量没有限制，但相同模块**只会被加载一次**。

浏览器**首先解析入口模块**，确定它的依赖，然后发送对依赖模块的请求。这些依赖请求返回后，浏览器就会解析其内容，确定它们是否还有依赖，如果二次依赖还没有被加载，则会发送更多请求，这个**异步递归的过程持续到整个应用程序的依赖图都加载完成**。加载大型应用的深度依赖图可能要花费很长时间。

除了模块是单例、只能加载一次、支持循环依赖等特点，ES6的还有一些新特性：
- 默认在严格模式下执行
- 模块不共享全局命名空间
- 模块顶级this值是undefined
- 模块中的var声明不会添加到window对象
- 模块是异步加载和执行的

使用`export`关键字进行导出，支持**命名导出**和**默认导出**两种方式。export在模块中出现的位置没有限制。

```javascript
// export 命名导出的若干方式
export const foo = 'foo';
export { foo };
export { foo as myFoo };
```
默认导出只能有一个，就好像模块与被导出的值是一回事。

```javascript
export default foo;
export { foo as default };
```

import语句也可以在模块中的任何位置，但推荐在顶部。导入模块的路径**不能是拼接字符串**。

没有导出的模块也可以被导入，以利用其副作用（比如css文件）。
```javascript
import './foo.js';
```
也可以从其他模块export某些定义，如在bar.js中再导出foo.js中的所有导出对象：

```javascript
export * from './foo.js';
```

`type`属性和`nomodule`属性可以给不支持ES6模块的系统提供兼容性，如：
```html
<!-- 支持module的浏览器执行这里，不支持的会忽略这一行-->
<script type="module" src="app.js"></script>
<!-- 支持module的浏览器会忽略这里，不支持的会执行这一行-->
<script nomodule src="app-legacy.js"></script>
```
