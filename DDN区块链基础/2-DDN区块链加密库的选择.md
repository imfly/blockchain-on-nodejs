DDN区块链加密库的选择
--------------

DDN是基于Node.js平台，无论前端还是后台都要使用 JavaScript 语言。所以，选择一个基于 JavaScript 语言的性能卓越的密码库是我们要考虑的重要问题之一。当然，最好是能够兼顾前端（浏览器端）和性能，这样就能实现前端和后台的统一了。

## 浏览器端加密的限制

目前，要把加密函数暴露给浏览器的 JavaScript，只有三个选项：

### 1、使用插件

插件是指运行在浏览器中，可以由 JavaScript 调用的，编译过的代码。比如，Java 和 Flash 中存在的加密库。另外一个选项是使用 Chrome 浏览器的 NaCl 客户端（Native Client） 程序，它允许运行由 C 或者 C++ 编译出的机器代码。

这样的做法通常性能很高，但是需要用户安装浏览器插件程序，或者NaCl客户端程序只能用于 Chrome 浏览器，所以可移植性不是很好，用户体验较差。

### 2、使用 Web 加密 API

人们已经提案给 JavaScript 提供原生的基本加密接口，让 Web 应用可以更快地加密解密。但是，要想主流浏览器采用这项技术还需要很长一段时间，远水解不了近渴。现在，能在多数浏览器中使用的只有crypto.getRandomValues()函数。

### 3、直接用 JavaScript 加密

这种方案的优点就在于高度的可移植性。所有的浏览器都可以执行 JavaScript，也就意味着所有的浏览器都可以调用 JavaScript 写成的加密库。但是，在 JavaScript 中加密存在两个缺陷：安全性和性能，不过现在可以找到替代方案。

安全性方面，Math.random()函数不是随机数的良好来源，所以不可能得到足够的随机数用来加密，这是事实。所以，建议在现代浏览器，全部使用 crypto.getRandomValues() 函数以取得足够数量的随机数，从而弥补该不足。

性能方面，浏览器端 JavaScript 加密在速度等诸多方面都不服务端。但是反过来，我们又何尝需要浏览器端提供那么高的性能呢？很多时候，在浏览器端，不需要大规模的加密计算，特别是在端到端的信息加密方面，现代浏览器的性能已经能够满足需求，比如：用户使用浏览器生成地址或者签名交易。这些正好是DDN区块链这种分布式的应用程序的使用场景。

## 加密库的选择

性能要求不高，不代表我们不追求性能。近年来，JavaScript 加密的性能有极大提升。以下两个库`js-nacl`和`TweetNaCl.js`就是这其中的代表，他们都可以在Node.js后台和浏览器前端执行，性能都有大幅度提升。不过，他们实现的原理完全不同。

### 两个库的逻辑关系

js-nacl(js库) --> libsodium(C库，方便安装的NaCl封装版) --> NaCl(密码库) --> 加密算法（sha256等）

TweetNaCl.js --> TweetNaCl（C库，可审计的高安全密码库） --> NaCl(密码库) --> 加密算法（sha256等）

`NaCl` (读作 “salt”) 是一个 C 语言的库，提供对称式密钥加密解密和公钥签名认证的应用函数。当然，其他库也提供这方面的功能，不过　NaCl 在安全性、易用性和速度上都有所提升。它由密码学人士编写，在加密社区广为人知，受到信赖。问题之一是 NaCl 是 C 语言，而不是 JavaScript 编写的。

libsodium 是 NaCl 的一个分支。着重于易于移植，可交叉编译和可安装打包。并有和 NaCL 兼容的 API，进一步增加了易用的扩展API。libsodium 的目标是提供构建高层密码学工具所需的核心算法。libsodium 的设计强调高安全，基础算法的性能也全面超越 NIST 标准下的绝大多数其他实现。libsodium 支持一系列编译器和操作系统，包括 iOS、Android。

TweetNaCl 号称是世界上第一个可审计的高安全性密码库。TweetNaCl只支持100条Tweet，同时支持应用程序使用的所有25个C语言的 NaCl 函数。TweetNaCl 是一个自包含的公共域C库，因此可以很容易地集成到应用程序中。

总之，这两个库最终都是要实现使用 Javascript对 NaCl 的API接口调用。

### 两个库的区别

**js-NaCl：编译成 JavaScript 的 NaCl 加密库**

`js-NaCl`，是把 NaCl 编译成 LLVM 的字节码，然后用 `emscripten` 将这些字节码编译成 JavaScript。并且，LLVM 编译器能在编译时作许多优化，所以得到的 JavaScript 代码也会得到优化。

更好的是，`emscripten` 编译出的代码是 JavaScript 的子集，也叫做 `asm.js`。你可以将 asm.js 当作很像 JavaScript 的汇编语言。浏览器遇到了 asm.js 的代码块时，会将其编译成高效的机器码，运行速度接近原生代码。

目前主要只有 Firefox 浏览器支持 asm.js 的优化。这就使 js-nacl 在 Firefox 中的加密解密非常迅速，视具体操作的不同，比 Chrome 浏览器的速度快 2 至 8 倍。但是即使是 Chrome，js-nacl 也很快，超过了我们所测试的其他所有加密库。

**TweetNaCl.js: TweetNaCl的纯 JavaScript 迁移实现**

官方的说法是：

“The primary goal of this project is to produce a translation of TweetNaCl to JavaScript which is as close as possible to the original C implementation, plus a thin layer of idiomatic high-level API on top of it.”

简单解释就是：TweetNaCl.js的主要目的是生成一个将TweetNaCl转换为JavaScript的版本，该版本尽可能接近原始的C实现。

## 选择 TweetNaCl.js 的理由

我会在适当的时候对上述两个库做简单的性能测试，不过目前，我选择 `TweetNaCl.js` 的理由很简单，主要有这样几点：

1. 原生实现，让使用者更有主动权。我喜欢原生代码，纯 Javascript 实现，这样在必要的时候，我可以直接参与其中，提供必要的改善。当然，尽管不希望有这样的机会，但是作为程序员，谁又能保证未来某一天没有这个必要呢。
2. 良好的使用体验。该库提供的API比较清晰简单，使用起来更加直观。
3. 良好的社区支持。对比`js-NaCl` 590 的使用者，该库 4016274 的使用量，简直不是一个数量级的。

## 参考

https://byronhe.gitbooks.io/libsodium/content/

http://cace-project.eu/nacl/

http://wiki.ucis.nl/NaCl#JavaScript_implementation

https://tweetnacl.js.org/#/

http://tweetnacl.cr.yp.to/

http://nacl.cr.yp.to/
