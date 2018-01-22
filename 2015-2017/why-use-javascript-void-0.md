---
title: 为什么要使用 href="JavaScript:void(0);"？
date: 2016-11-18 18:00:00
tags: javascript
---

# 为什么要使用 href="JavaScript:void(0);"？
也许你写了 5 年 JavaScript，但你从未思考过为什么要使用 `JavaScript:void(0);` ；也许你是在作为一个前端新人时，看见前辈都在 `href=` 中加入 `JavaScript:void(0);` ，来阻止浏览器的默认行为，然后你也跟着这样写，久而久之，成为一种习惯，但你从未思考过，为什么要这样。

其实我在写这篇文章之前，我也不知道。也许我写完这篇文章之后，我还是不知道。


下面直接 **插** 入主题...

### 关于 javascript:void(0);

#### void 的定义：
> [void](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/void) is a prefix keyword that takes one argument and always returns `undefined`.

void 是 JavaScript 的一个关键字。它只接受一个参数，并总是返回 `undefined`

<!-- more -->

#### void 的用法：
> void expression

void 后面跟的是一个表达式。

看下面几个例子：

```
void 0;
void(0);
void(1);
void('hello');
void(new Date());
// 上面 5 条表达式，返回的都是 undefined
```

那么，返回 `undefined` 意味着什么呢？

它意味着： **浏览器不会做任何响应。**

那么，问题来了，为什么我们要在空链接那里加上 `href="javascript:void(0);"` ？

因为空链接 `<a>点击</a>` ，它是没有 `curson: pointer` 的手势的，而加上 `href="#"` 或者 `href="javascript:void(0);"` 后，空链接会出现手势，这样的界面更加友好，告诉用户，这是一个可以点击的元素。

那么，为什么不使用 `href="#"` 呢？单单敲一个 `#` 比 `javascript:void(0);` 少按 **18** 次键盘。

**因为 `href="#"` 是一个锚点，当 `#` 后面没有跟上页面元素的 id 时，点击后会直接滚动回页面顶部。而往往这种操作，不是我们希望的，所以我们一般会把空链接的 `href` 属性写成 `javascript:void(0);` ， 来阻止浏览器的默认行为。所以，使用了 `javascript:void(0);` 之后，既可以阻止浏览器有任何响应，又可以在此期间执行异步操作，又保持了 `cursor: point` 的样式。**

### 为什么使用的是 void(0) 而不是 void(100) 或其他呢？

`void(0)` 或 `void(100)` 的结果是一样的，那么 0 的好处在哪里呢？这里引用一句话：
> The only benefit of passing in 0 instead of some other argument is that 0 is short and idiomatic.

因为它更加简短（short）和符合语言习惯（idiomatic）。

### 使用 `href="javascript:void(0);"` 有什么坏处？

1. `javascript:` 是一个伪协议，是非标准化的协议（我们写代码最好还是按照标准的协议？也许吧）；
2. 不能平稳退化。当用户禁用了浏览器的 JS 后，这个链接什么意义也没有（不过现代的网页基本都包含了 JS 了吧，一般禁用后，很多都不能正常使用了，而且一般用户也不会去故意禁用 JS）；
3. 它没有包含位置信息，不利于爬虫搜索（也许这个有点意义）。

那么，最后，为什么要使用 `javascript:void(0);` 呢？

其实，我也不知道，也许是 **习惯**
😂


