### 原文链接 :

[Why Do React Elements Have a $$typeof Property? ](https://overreacted.io/why-do-react-elements-have-typeof-property/)

```js
<marquee bgcolor="#ffa7c4">hi</marquee>
```

看到上面的代码, 你可能会觉得自己在写`jsx`

但实际上, 你在调用一个函数

```js
React.createElement(
  /* type */ 'marquee',
  /* props */ { bgcolor: '#ffa7c4' },
  /* children */ 'hi'
)
```

并且这个函数可以为你返回一个`Object`
我们将这个`Object`称为React元素
它告诉React下一步要呈现什么
而组件(component)返回的则是一个个`Object组成的树`

```js
{
  type: 'marquee',
  props: {
    bgcolor: '#ffa7c4',
    children: 'hi',
  },
  key: null,
  ref: null,
  $$typeof: Symbol.for('react.element'), // 🧐 这是什么
}
```

如果你用过React，你可能熟悉`type，props，key`和`ref`这些字段
但什么是`$$typeof`?

为什么它有一个`Symbol()`作为值?

这是你使用React时不需要知道的另一件事，但这会让你感觉很好
在这篇文章中还有一些关于安全性的提示，你可能想知道
也许有一天你会编写自己的UI库，所有这些都会派上用场

以前, 我们经常会做这样一件事: 自己构造`HTML`字符串, 然后`apeend`到页面上

但是xss攻击总是有的

为了防止此类攻击，你可以使用安全的API，例如document.createTextNode（）或textContent，它只处理文本。
你还可以通过在任何用户提供的文本中替换<，>等其他潜在危险字符来抢先"转义"输入

尽管如此，错误的成本很高，每次将用户编写的字符串插入输出时，记住它都很麻烦。
这就是像React这样的现代库默认情况下为字符串转义文本内容的原因：

```html
<p>
  {message.text}
</p>
```

如果`message.text`是带有`<img>`或其他标记的恶意字符串，则它不会变成真正的`<img>`标记。
`React`将转义内容，然后将其插入`DOM`。
所以，不要看到`<img>`标签，而只是看到它的标记。

要在`React`元素中呈现任意`HTML`，你必须编写`dangerouslySetInnerHTML = {{__ html：message.text}}`。
这种写法, 是通过很笨拙的方式去编写一个功能实现。
但它意味着高度可见，以便你可以在代码审查和代码库审计中捕获它。

那么, 这是否意味着React完全免受注入攻击?

事实上并没有. `HTML`和`DOM`提供了[大量的攻击面](https://github.com/facebook/react/issues/3473#issuecomment-90594748)，对于React或其他UI库而言，这些攻击面太难或太慢。

大多数剩余的攻击向量涉及属性。
例如，如果你渲染`<a href={user.website}>`，请注意其网站为`"javascript：stealYourPassword()"`的用户。
通过`<div {... userData}>`的方式处理用户的输入信息是很少见并且很危险的

React[可以](https://github.com/facebook/react/issues/10506)随着时间的推移提供更多保护，但在许多情况下，这些都是服务器问题的结果，无论如何都[应该](https://github.com/facebook/react/issues/3473#issuecomment-91327040)修复它们

当然，过滤文本内容是一个合理的第一道防线，可以捕获大量潜在的攻击。
这样我们可以保证代码是安全的，这不是很好吗?

```html
// 自动过滤
<p>
  {message.text}
</p>
```

但是, 这也不是总是正确的, 所以, $$typeof的作用就来了

React的元素被设计成`Object`的形式

```js
{
  type: 'marquee',
  props: {
    bgcolor: '#ffa7c4',
    children: 'hi',
  },
  key: null,
  ref: null,
  $$typeof: Symbol.for('react.element'),
}
```

虽然通常使用`React.createElement()`创建它们，但它不是必需的。
`React`有一些有效的用例来支持像我刚刚上面所做的那样编写的普通元素对象。
当然，你可能不希望像这样编写它们 - 但这对于优化编译器，在工作程序之间传递UI元素或者将JSX与React包解耦是有用的。

但是，如果你的服务器有一个漏洞，允许用户存储任意JSON对象，而客户端代码需要一个字符串，这可能会成为一个问题：

```js
// 服务端有一个漏洞: 可以让用户保存json数据
let expectedTextButGotJSON = {
  type: 'div',
  props: {
    dangerouslySetInnerHTML: {
      __html: '/* put your exploit here */'
    },
  },
  // ...
};
let message = { text: expectedTextButGotJSON };

// 这样的情况在React 0.13时是很危险的
<p>
  {message.text}
</p>
```

在这种情况下，`React 0.13`很容易受到XSS攻击。
再次澄清一下，这种攻击取决于现有的服务器漏洞。
尽管如此，`React`可以更好地保护开发者受它侵害。
从`React 0.14`开始:

`React 0.14`开始是使用`Symbol`标记每个`React`元素：

```js
{
  type: 'marquee',
  props: {
    bgcolor: '#ffa7c4',
    children: 'hi',
  },
  key: null,
  ref: null,
  $$typeof: Symbol.for('react.element'),
}
```

这是有效的，因为你不能只把符号放在JSON中。
因此，即使服务器具有安全漏洞并返回JSON而不是文本，该JSON也不能包含`Symbol.for（'react.element'）`。
`React`将检查元素。`$$typeof`，如果元素丢失或无效，将拒绝处理该元素。
使用`Symbol.for()`的好处是，元素在`iframe`和`worker`等环境之间是全局的。
因此，即使在更奇特的条件下，此修复也不会阻止在应用程序的不同部分之间传递可信元素。
同样，即使页面上有多个React副本，它们仍然可以`"agree"`有效的`$$typeof`值。

那些不支持`Symbols`的浏览器呢?
那么, 他们将没有得到这种额外的保护。
React仍然在元素上包含`$$typeof`字段以保持一致性，但它设置为一个数字 - `0xeac7`。
为什么是这个数字?
因为`0xeac7`有点像`"React"`。
