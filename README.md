# learn-es6
重温es6

##模块(Module)
在ES6之前，要想实现模块加载功能，最主要的有CommonJS(用于服务端)和AMD(用于浏览器)两种。

####AMD

随着RequireJS成为最流行的实现方式，异步模块规范（AMD）在前端界已经被广泛认同。

下面是只依赖jquery的模块foo的代码：

```js
// 文件名: foo.js
define(['jquery'], function ($) {
    // 方法
    function myFunc(){};
 	// 暴露公共方法
    return myFunc;
});
```

还有稍微复杂点的例子，下面的代码依赖了多个组件并且暴露多个方法:

```js
// 文件名: foo.js
define(['jquery', 'underscore'], function ($, _) {
// 方法
function a(){}; // 私有方法，因为没有被返回(见下面)
function b(){}; // 公共方法，因为被返回了
function c(){}; // 公共方法，因为被返回了
     // 暴露公共方法
    return {
        b: b,
        c: c
    }
});
```

定义的第一个部分是一个依赖数组，第二个部分是回调函数，只有当依赖的组件可用时（像RequireJS这样的脚本加载器会负责这一部分，包括找到文件路径）回调函数才被执行。

注意，依赖组件和变量的顺序是一一对应的（例如，jquery->$, underscore->_）。

同时注意，我们可以用任意的变量名来表示依赖组件。假如我们把$改成$$，在函数体里面的所有对jQuery的引用都由$变成了$$。

还要注意，最重要的是你不能在回调函数外面引用变量$和_，因为它相对其它代码是独立的。这正是模块化的目的所在！

####CommonJS

如果你用Node写过东西的话，你可能会熟悉CommonJS的风格（node使用的格式与之相差无几）。因为有Browserify，它也一直被前端界广泛认同。

就像前面的格式一样，下面是用CommonJS规范实现的foo模块的写法：

```js
// 文件名: foo.js
// 文件名: foo.js
// 依赖
var $ = require('jquery');
// 方法
function myFunc(){};
 
// 暴露公共方法（一个）
module.exports = myFunc;
```

还有更复杂的例子，下面的代码依赖了多个组件并且暴露多个方法：

```js
// 文件名: foo.js
var $ = require('jquery');
var _ = require('underscore');
 
// methods
function a(){};    //    私有方法，因为它没在module.exports中 (见下面)
function b(){};    //    公共方法，因为它在module.exports中定义了
function c(){};    //    公共方法，因为它在module.exports中定义了
 
// 暴露公共方法
module.exports = {
    b: b,
    c: c
};
```

####UMD: 通用模块规范

既然CommonJs和AMD风格一样流行，似乎缺少一个统一的规范。所以人们产生了这样的需求，希望有支持两种风格的“通用”模式，于是通用模块规范（UMD）诞生了。

不得不承认，这个模式略难看，但是它兼容了AMD和CommonJS，同时还支持老式的“全局”变量规范：

```js
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD
        define(['jquery'], factory);
    } else if (typeof exports === 'object') {
        // Node, CommonJS之类的
        module.exports = factory(require('jquery'));
    } else {
        // 浏览器全局变量(root 即 window)
        root.returnExports = factory(root.jQuery);
    }
}(this, function ($) {
    // 方法
    function myFunc(){};
    // 暴露公共方法
    return myFunc;
}));
```

保持跟上面例子一样的模式，下面是更复杂的例子，它依赖了多个组件并且暴露多个方法:

```js
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD
        define(['jquery', 'underscore'], factory);
    } else if (typeof exports === 'object') {
        // Node, CommonJS之类的
        module.exports = factory(require('jquery'), require('underscore'));
    } else {
        // 浏览器全局变量(root 即 window)
        root.returnExports = factory(root.jQuery, root._);
    }
}(this, function ($, _) {
    // 方法
    function a(){};    // 私有方法，因为它没被返回 (见下面)
    function b(){};    // 公共方法，因为被返回了
    function c(){};    // 公共方法，因为被返回了
    // 暴露公共方法
    return {
        b: b,
        c: c
    }
}));
```

ES6模块的设计思想，是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS和AMD模块，都只能在运行时确定这些东西(运行时加载)。

ES6模块还有以下好处：
* 一、静态加载。
* 二、不再需要UMD模块格式了，将来服务器和浏览器都会支持ES6模块格式。目前，通过各种工具库，其实已经做到了这一点。
* 三、将来浏览器的新API就能用模块格式提供，不再必要做成全局变量或者navigator对象的属性。
* 四、不再需要对象作为命名空间（比如Math对象），未来这些功能可以通过模块提供。

###知识点一：ES6的模块自动采用严格模式，不管你有没有在模块头部加上"use strict";

###知识点二：export命令

输出了三个变量

```js
// profile.js
export var firstName = 'Michael';
export var lastName = 'Jackson';
export var year = 1958;
```

输出的一组变量(优先考虑)

```js
// profile.js
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;

export {firstName, lastName, year};
```

输出函数或类（class）

```js
export function multiply(x, y) {
  return x * y;
};
```

需要特别注意的是，export命令规定的是对外的接口，必须与模块内部的变量建立一一对应关系。

```js
// 报错
export 1;

// 报错
var m = 1;
export m;
```

```js
// 写法一
export var m = 1;

// 写法二
var m = 1;
export {m};

// 写法三
var n = 1;
export {n as m};
```

同样的，function和class的输出，也必须遵守这样的写法。

```js
// 报错
function f() {}
export f;

// 正确
export function f() {};

// 正确
function f() {}
export {f};
```

另外，export语句输出的接口，与其对应的值是动态绑定关系，即通过该接口，可以取到模块内部实时的值。

```js
export var foo = 'bar';
setTimeout(() => foo = 'baz', 500);
上面代码输出变量foo，值为bar，500毫秒之后变成baz。
```

这一点与CommonJS规范完全不同。CommonJS模块输出的是值的缓存，不存在动态更新，详见下文《ES6模块加载的实质》一节。

最后，export命令可以出现在模块的任何位置，只要处于模块顶层就可以。如果处于块级作用域内，就会报错，下一节的import命令也是如此。这是因为处于条件代码块之中，就没法做静态优化了，违背了ES6模块的设计初衷。



