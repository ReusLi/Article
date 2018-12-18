### 原文链接 :

[Why Do We Write super(props)? ](https://overreacted.io/why-do-we-write-super-props/)


我写过无数次 super(props)

```js
class Checkbox extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isOn: true };
  }
  // ...
}
```

当然,  [class fields proposal](https://github.com/tc39/proposal-class-fields) 让我们跳过了super(props)的步骤

```js
class Checkbox extends React.Component {
  state = { isOn: true };
  // ...
}
```

当React 0.13在2015年增加对普通类的支持时，[计划](https://reactjs.org/blog/2015/01/27/react-v0.13.0-beta-1.html#es7-property-initializers)使用这样的语法。定义构造函数和调用super（props）总是打算成为临时解决方案，直到class字段提供符了替代方案。


让我们重新看一下这个ES6的例子
```js
class Checkbox extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isOn: true };
  }
  // ...
}
```

在JavaScript中，super指的是父类构造函数。（在我们的示例中，它指向React.Component实现。）

重要的是，在调用父构造函数之前，不能在构造函数中使用它。
比如：

```js
class Checkbox extends React.Component {
  constructor(props) {
    // 🔴 这里还不能使用 `this`
    super(props);
    // ✅ 这里可以使用 `this` 了
    this.state = { isOn: true };
  }
  // ...
}
```

有一个很好的理由说明为什么JavaScript在你使用`this`之前要强制执行父构造函数。
看下面的例子：

```js
class Person {
  constructor(name) {
    this.name = name;
  }
}

class PolitePerson extends Person {
  constructor(name) {
    this.greetColleagues(); // 🔴 这里不能使用 `this` 请阅读以下原因
    super(name);
  }
  greetColleagues() {
    alert('Good morning folks!');
  }
}
```

想想一下, 在之后的维护中, 我们可能会修改`greetColleagues`函数变成:
把用户名称`this.name`加进去

```js
greetColleagues() {
  alert('Good morning folks!');
  alert('My name is ' + this.name + ', nice to meet you!');
}
```

但是我们忘记了, `greetColleagues()`可能会在`this.name`被定义之前被调用的, 所以, 这样的代码会很难预料的错误

为了避免这样的情况, JavaScript强制要求: 如果你想在构造函数中使用它，你必须首先调用super, 让父类先执行自己的逻辑!

同样的情况也使用与React组件

```js
constructor(props) {
  super(props);
  // ✅ Okay to use `this` now
  this.state = { isOn: true };
}
```

上面的代码会有一个问题: 为什么`super`的时候要传递 `props` 参数? 

你可能认为将props传递给super是必要的，方便的React.Component构造函数可以初始化this.props

```js
// 在react内部
class Component {
  constructor(props) {
    this.props = props;
    // ...
  }
}
```

这就是传递props的原因, [它做了什么 ?](https://overreacted.io/why-do-we-write-super-props/)

但不知道为什么，即使你没有使用`props`参数调用`super()`，你仍然可以在渲染和其他方法中访问`this.props`

这是怎么实现的？ 实际上，在调用构造函数后，React也会在实例上分配`props`

```js
// 在react内部
const instance = new YourComponent(props);
instance.props = props;
```

因此，即使你忘记将`props`传递给`super()`，React仍会在之后设置它们。
这是有原因的。

当`React`添加对`Class`的支持时，它不仅仅增加了对`ES6`类的支持。
`React`目标是尽可能支持更多的类抽象。
目前还不清楚`ClojureScript，CoffeeScript，ES6，Fable，Scala.js，TypeScript`或其他解决方案在定义组件方面的进展。
所以`React`故意不关心是否需要调用`super()` - 即使是`ES6`类。

那么这是否意味着你可以只写`super()`而不是`super(props)`?
建议是不要这么做
当然，`React`稍后会在你的构造函数运行后分配`this.props`

但是`this.props`在`super()`和构造函数结束之间的这个时间段, 还是会等于`undefined`的：

```js
// 在React内部
class Component {
  constructor(props) {
    this.props = props;
    // ...
  }
}

// 你的代码
class Button extends React.Component {
  constructor(props) {
    super(); // 😬 We forgot to pass props
    console.log(props);      // ✅ {}
    console.log(this.props); // 😬 undefined 
  }
  // ...
}
```

如果在从构造函数调用的某个方法中发生这种情况，则调试可能变成更困难。
这就是为什么我建议你写`super(props)`的原因，即使它不是绝对必要的：

```js
class Button extends React.Component {
  constructor(props) {
    super(props); // ✅ We passed props
    console.log(props);      // ✅ {}
    console.log(this.props); // ✅ {}
  }
  // ...
}
```

这样可以确保在构造函数退出之前设置`this.props`

最后一点React用户可能会觉得好奇

你可能已经注意到，当你在`Class`中使用`Context API`时（使用遗留的`contextTypes`或`React 16.6`中添加的现代`contextType API`），`context`将作为第二个参数传递给构造函数

那么为什么我们不写`super(props, context)`呢?
其实是可以这样写的，但`context`的使用频率较低，所以这个陷阱并没有那么多

随着`Class fields`的提议，这些陷阱大都会消失
如果没有显式构造函数，则会自动传递所有参数
这允许像`state = {}`这样的表达式包含对`this.props`或`this.context`的引用

有了Hooks，我们甚至没有`super`或者`this`
但那是另一个话题了