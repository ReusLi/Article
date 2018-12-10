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

