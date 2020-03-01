# ES6模块体系及DDN开发规范


## 概述

得益于 ES6 和 TS的模块体系，DDN区块链可以快速拆解和迭代。没有这些模块化的基础，我们后面所有的工作都会受阻，可插拔、可视化、可配置等功能就成了一句空话，面向不同企业给予“多链并行，跨链互通”也就非常困难。

按照敏捷开发的思路，DDN区块链整体上同样基于“默认优于配置”，在底层开发的代码层面就规定了一些默认的约束和规范。本章咱们就简单的介绍DDN区块链开发中用到的关于模块的组织和实践，方便大家在使用DDN进行深度开发的过程中可以更加便捷。

## 关于 JavaScript 的模块体系

JavaScript 最早用在浏览器上，而且很长一段时间里，没有自己的模块（module）体系，无法从小文件开始拆分并拼装成一个大程序，所谓的程序员大牛根本不认为它就是编程语言。与其形成鲜明对比的，就连 CSS 都有`@import`，更别说C/C++、Java、ruby、Python等语言了。

当然，今天这种情形得到改观，特别是在 ES6 出现之后，在语言层面上，实现了模块功能，而且相当简单，必将取代 CommonJS 和 AMD 规范，成为统一浏览器和服务器通用的模块解决方案。但是，在现阶段，还会有些小小的麻烦，核心就在于 ES6 的模块规范与 CommonJS 和 AMD 模块体系差别上。所以，只要搞明白这一点，使用起来就如虎添翼，简单明了了。

## ES6 模块特点

作为脚本语言，CommonJS 和 AMD 模块体系，只能在运行时确定模块的依赖关系，这个应该是理所当然的事情。而 ES6 模块的设计却是走了静态化的路线，与那些编译类的语言有些类似，在编译时就能确定模块关系，以及输入和输出的变量，这就为 Javascript 带来了很多可能。比如进一步拓宽 JavaScript 的语法，引入宏（macro）和类型检验（type system）这些只能靠静态分析实现的功能。

另一个方面是，ES6 模块自动采用严格模式，不管你有没有在模块头部加上`"use strict";`，默认限制如下：

- 变量必须声明后再使用
- 函数的参数不能有同名属性，否则报错
- 不能使用`with`语句
- 不能对只读属性赋值，否则报错
- 不能使用前缀 0 表示八进制数，否则报错
- 不能删除不可删除的属性，否则报错
- 不能删除变量`delete prop`，会报错，只能删除属性`delete global[prop]`
- `eval`不会在它的外层作用域引入变量
- `eval`和`arguments`不能被重新赋值
- `arguments`不会自动反映函数参数的变化
- 不能使用`arguments.callee`
- 不能使用`arguments.caller`
- 禁止`this`指向全局对象
- 不能使用`fn.caller`和`fn.arguments`获取函数调用的堆栈
- 增加了保留字（比如`protected`、`static`和`interface`）

其中，尤其需要注意`this`的限制。ES6 模块之中，顶层的`this`指向`undefined`，即不应该在顶层代码使用`this`。

## Node.js 使用 ES6 语法

DDN区块链基于Node.js平台，而 Node.js 遵从 CommonJS，对 ES6 模块的处理比较麻烦，因为它与 ES6 模块格式是不兼容的。目前，Node.js 解决方案是，将两者分开，ES6 模块和 CommonJS 采用各自的加载方案。

默认情况下，Node.js 要求 ES6 模块采用.mjs后缀文件名，或者在 `package.json` 里配置字段 `type` 为 `module`。也就是说，只要脚本文件里面使用import或者export命令，那么就必须采用.mjs后缀名。require命令不能加载.mjs文件，会报错，只有import命令才可以加载.mjs文件。反过来，.mjs文件里面也不能使用require命令，必须使用import。

目前，这项功能还在试验阶段。安装 Node v8.5.0 或以上版本，要用--experimental-modules参数才能打开该功能。

```
$ node --experimental-modules my-app.mjs
```

在实际的开发过程中，我们采取的手段有两个。一是直接使用上面 Node.js 对 ES6 的规范，二是通过 Babel 等第三方工具，将 ES6 规范的代码编译（配置工具即可）成 对应格式 的代码来使用。DDN区块链采取的是后者，因为毕竟各种配套的工具比较齐备，不必操心其中的不兼容性等潜在问题。

## DDN区块链模块研发规范

DDN区块链全部使用 ES6 的模块规则，并在此基础上，规定如下用法：

- 模块名字与文件名字保持一致，比如：import Address from './address';
- 模块内部导出接口命令，只能使用一个`export { }`的命令形式，其他模块导入使用`import { } from `的形式导入，也就是说模块的变量名称可能是NPM包内部唯一的；
- 使用NPM包的变量与包名字保持一致，比如：import Dapp from `@ddn/dapp`;
- NPM包导出命令，只能使用`export default`命令。

下面，详细说明其中要点。

ES6 模块功能主要由两个命令构成：`export`和`import`。`export`命令用于规定模块的对外接口，`import`命令用于输入其他模块提供的功能。

### 1. 集中输出

一个模块就是一个独立的文件，该文件内部的所有变量、方法、类等，我们这里统一称为变量，外部是无法获取的。如果你希望外部能够读取他们，就必须使用`export`关键字输出，即提供接口（不是常量，而是变量），确保与模块内部的变量建立一一对应关系。ES6 允许使用多个`export`逐个输出，且可以出现在模块的任何位置，但是那样的体验非常不直观，我们约定：

模块的导出：

- 一个模块只能有一个`export`；
- `export`只能放在文件最末尾，并确保处在作用域的最顶层（比如：不能放在`if`语句里，否则报错，这是因为处于条件代码块之中，就没法做静态优化了，违背了 ES6 模块的设计初衷）；



NPM包的导出：

- 用`export default`指定默认输出，而不是使用单独的`export`命令，直接输出所要加载的变量名或函数名，这样无需了解模块细节，借助IDE代码补全，快速开发；
- 代码中使用模块中的变量，尽量携带导入的模块名字，避免名字冲突，尽量不要使用 `{ Address } = Asset` 过渡；

格式如下：

```javascript
// @ddn/utils/src/index.js
import AssetTypes from './asset-types';
import RuntimeState from './runtime-states';
import Address from './address';
import Amount from './amount';
import LimitCache from './limit-cache';
import Utils from './utils';
import Bignum from './bignumber';

export default {
    AssetTypes,
    RuntimeState,
    Address,
    Amount,
    LimitCache,
    Utils,
    Bignum
};
```

上面代码在`export default`命令后面，使用大括号指定所要输出的一组变量。这样做就可以在脚本尾部，一眼看清楚输出了哪些变量。通常情况下，`export`输出的变量就是本来的名字，如果想修改成别的名字，使用`as`关键字重命名即可，所以`export default`命令等同于`* as default`。

`export default`命令用于指定模块的默认输出，本质上就是输出一个叫做`default`的变量或方法，然后`import`的时候，系统允许你为它取任意名字。显然，一个模块只能有一个默认输出，因此`export default`命令只能使用一次。所以，import命令后面不用直接加大括号。

另外，`export`语句输出的接口，与其对应的值是动态绑定关系，即通过该接口，可以取到模块内部实时的值。这与 CommonJS 规范完全不同。CommonJS 模块输出的是值的缓存，不存在动态更新。

```javascript
export var foo = 'bar';
setTimeout(() => foo = 'baz', 500); // 输出变量`foo`，值为`bar`，500 毫秒之后变成`baz`
```

下面比较一下默认输出和正常输出。

```javascript
// 第一组
export default function crc32() { // 输出
  // ...
}

import crc32 from 'crc32'; // 输入

// 第二组
export function crc32() { // 输出
  // ...
};

import {crc32} from 'crc32'; // 输入
```

上面代码的两组写法，第一组是使用`export default`时，对应的`import`语句不需要使用大括号；第二组是不使用`export default`时，对应的`import`语句需要使用大括号。

### 2. 集中输入

使用`export`命令定义了模块的对外接口以后，其他 JS 文件就可以通过`import`命令加载这个模块。

- 在NPM包内的一个模块导入，


- NPM包的导入，代码如下：

```
// 不正确
import { Address } from '@ddn/utils'
console.log(Address); // 输出 undefined

// 正确
import Utils from '@ddn/utils'
console.log(Utils.Address);

或者
// 正确
const { Address } = Utils；
console.log(Address);
```


```javascript
// main.js
import { firstName, lastName, year } from './profile.js';

function setName(element) {
  element.textContent = firstName + ' ' + lastName;
}
```

- `import`命令输入的变量都是只读的，因为它的本质是输入接口。也就是说，不允许在加载模块的脚本里面，改写接口。也就是说，凡是输入的变量，即便能改写，也都要当作完全只读，不要轻易改变它的属性。
- `import`后面的`from`指定模块文件的位置，可以是相对路径，也可以是绝对路径，`.js`后缀可以省略。如果只是模块名，不带有路径，那么必须有配置文件，告诉 JavaScript 引擎该模块的位置。

- `import`命令是编译阶段执行的，在代码运行之前。因此，`import`命令具有提升效果，会提升到整个模块的头部，首先执行。所以也不能使用表达式和变量，这些只有在运行时才能得到结果的语法结构。
- `import`语句是 Singleton 模式。如果多次重复执行同一句`import`语句，那么只会执行一次，而不会执行多次。

```javascript
import { foo } from 'my_module';
import { bar } from 'my_module';

// 等同于
import { foo, bar } from 'my_module';
```

- 目前阶段，通过 Babel 转码，CommonJS 模块的`require`命令和 ES6 模块的`import`命令，可以写在同一个模块里面，但是不要这样做。因为`import`在静态解析阶段执行，所以它是一个模块之中最早执行的。下面的代码可能不会得到预期结果。

```javascript
require('core-js/modules/es6.symbol');
require('core-js/modules/es6.promise');
import React from 'React';
```

- 模块整体加载所在的那个对象，应该是可以静态分析的，所以不允许运行时改变。下面的写法都是不允许的。

```javascript
import * as circle from './circle';

// 下面两行都是不允许的
circle.foo = 'hello';
circle.area = function () {};
```

### 3. 接口转发

在一个模块之中，先输入后输出**同一个模块**，`import`语句可以与`export`语句写在一起，这就是所谓的接口转发。转发出去的变量在当前模块是无法使用的（因为直接导出了）。

```javascript
export { foo, bar } from 'my_module';

// 可以简单理解为
import { foo, bar } from 'my_module';
export { foo, bar }; // 当前模块不能直接使用`foo`和`bar`
```

接口转发的好处：

- 主要是模块的接口改名和整体输出。对于第三方模块，具名接口改默认接口，更加适用。

```javascript
// 接口改名
export { foo as myFoo } from 'my_module';

// 整体输出
export * from 'my_module'; // 与 默认接口有区别，这里忽略`my_module`模块的`default`方法，将其他各种变量都原封不动的输出，类似继承
```

默认接口的写法如下。

```javascript
export { default } from 'foo';
```

具名接口改为默认接口的写法如下。

```javascript
export { es6 as default } from './someModule';

// 等同于
import { es6 } from './someModule';
export default es6;
```

同样地，默认接口也可以改名为具名接口。

```javascript
export { default as es6 } from './someModule';
```

- 实现模块的继承

假设有一个`circleplus`模块，继承了`circle`模块。

```javascript
// circleplus.js

export * from 'circle';
export var e = 2.71828182846;
export default function(x) {
  return Math.exp(x);
}
```

上面代码中的`export *`，表示再输出`circle`模块的所有属性和方法。然后，又输出了自定义的`e`变量和默认方法。也可以将`circle`的属性或方法，改名后再输出。

```javascript
// circleplus.js

export { area as circleArea } from 'circle';
```

上面代码表示，只输出`circle`模块的`area`方法，且将其改名为`circleArea`。

### 4. 常量处理

`const`声明的常量只在当前代码块有效。如果想设置跨模块的常量（即跨多个文件），或者说一个值要被多个模块甚至多个NPM包共享，DDN区块链采用下面的写法。

```javascript
// constants.js 模块
export const A = 1;
export const B = 3;
export const C = 4;

// test1.js 模块
import * as constants from './constants';
console.log(constants.A); // 1
console.log(constants.B); // 3

// test2.js 模块
import {A, B} from './constants';
console.log(A); // 1
console.log(B); // 3
```

如果要使用的常量非常多，可以建一个专门的`constants`目录，将各种常量写在不同的文件里面，保存在该目录下。

```javascript
// constants/db.js
export const db = {
  url: 'http://my.couchdbserver.local:5984',
  admin_username: 'admin',
  admin_password: 'admin password'
};

// constants/user.js
export const users = ['root', 'admin', 'staff', 'ceo', 'chief', 'moderator'];
```

然后，将这些文件输出的常量，合并在`index.js`里面。

```javascript
// constants/index.js
export {db} from './db';
export {users} from './users';
```

使用的时候，直接加载`index.js`就可以了。

```javascript
// script.js
import {db, users} from './constants/index';◊
```

### 5. 动态加载

为了根据运行时需要加载不同模块，或者根据不同条件或路径加载不同模块，就需要用到动态加载。鉴于DDN区块链的灵活架构，必然大量用到这一功能，ES5阶段使用的是`require`函数，但是上面说了，不建议与`import`混用，我们将慢慢规范统一处理成这里的方法逻辑。

`import()`方法负责动态加载，可以取代`require`的动态加载功能。`import`命令能够接受什么参数，`import()`函数就能接受什么参数，两者区别主要是后者为动态加载。`import()`类似于 Node 的`require`方法，区别主要是前者是异步加载，后者是同步加载。

`import()`返回一个 Promise 对象。比如：

```javascript
const main = document.querySelector('main');

import(`./section-modules/${someVariable}.js`)
  .then(module => {
    module.loadPageInto(main);
  })
  .catch(err => {
    main.textContent = err.message;
  });
```

另外，`import()`函数与所加载的模块没有静态连接关系，这点也是与`import`语句不相同。`import()`加载模块成功以后，这个模块会作为一个对象，当作`then`方法的参数。

如果想同时加载多个模块，可以采用下面的写法。

```javascript
Promise.all([
  import('./module1.js'),
  import('./module2.js'),
  import('./module3.js'),
])
.then(([module1, module2, module3]) => {
   ···
});
```

或者用在 async 函数之中

```javascript
async function main() {
  const myModule = await import('./myModule.js');
  const {export1, export2} = await import('./myModule.js');
  const [module1, module2, module3] =
    await Promise.all([
      import('./module1.js'),
      import('./module2.js'),
      import('./module3.js'),
    ]);
}
main();
```

## 参考

- https://nodejs.org/dist/latest-v12.x/docs/api/modules.html
- https://www.cnblogs.com/myfirstboke/p/10563597.html
- http://es6.ruanyifeng.com/#docs/module-loader
- https://www.jianshu.com/p/ad427d8879cb