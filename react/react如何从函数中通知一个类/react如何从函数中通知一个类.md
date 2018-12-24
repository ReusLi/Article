### 原文链接 :

[How Does React Tell a Class from a Function? ](https://overreacted.io/how-does-react-tell-a-class-from-a-function/)

下面是一个定义为函数的Greeting组件
```js
function Greeting() {
  return <p>Hello</p>;
}
```

其实, 也可以被定义成class的形式， 效果是一样的

```js
class Greeting extends React.Component {
  render() {
    return <p>Hello</p>;
  }
}
```

当你要渲染一个`<Greeting />`组件时， 你其实不需要关心它是如何被定义的

```js
// Class 或 function — 都可以这样使用
<Greeting />
```

但是, `React`内容其实是需要区分它被定义成`funtion`还是`class`的

如果`<Greeting />`被定义成函数, `React`会这样调用它

```js
// Your code
function Greeting() {
  return <p>Hello</p>;
}

// Inside React
const result = Greeting(props); // <p>Hello</p>
```

但是如果`Greeting`是一个类，`React`需要使用`new`运算符实例化它，然后在刚刚创建的实例上调用`render`方法：

```js
// Your code
class Greeting extends React.Component {
  render() {
    return <p>Hello</p>;
  }
}

// Inside React
const instance = new Greeting(props); // Greeting {}
const result = instance.render(); // <p>Hello</p>
```

在上面2个例子中, `React`的目的都是为了渲染节点(在例子中是`<p>Hello</p>`)
但是, 会因为组件的定义方式不同, 而有不同的步骤

那么, `React`是怎么知道一个组件是函数还是class呢?

事实上, 这个问题更多是跟`JavaScript`有关, 而不是`React`

这篇文章没有太多关于React本身的信息，但我们将讨论`new`，`this`，`class`，`arrow functions,(箭头函数)`，`prototype`，`__ proto__`，`instanceof`以及这些东西如何在`JavaScript`中协同工作的

（如果你真的只想知道答案，请滚动到最后。）

首先，我们需要理解为什么以不同方式处理函数和类很重要。
注意我们在调用类时如何使用new运算符：

```js
// If Greeting is a function
const result = Greeting(props); // <p>Hello</p>

// If Greeting is a class
const instance = new Greeting(props); // Greeting {}
const result = instance.render(); // <p>Hello</p>
```

让我们粗略地了解new运算符在JavaScript中的作用

在过去，`JavaScript`没有`class`。
但是，你可以用函数来模拟一个`class`。

```js
// Just a function
function Person(name) {
  this.name = name;
}

var fred = new Person('Fred'); // ✅ Person {name: 'Fred'}
var george = Person('George'); // 🔴 Won’t work
```

就像上面的代码, 如果没有new运算符, `george`就会指向`undefined`

```js
var fred = new Person('Fred'); // Same object as `this` inside `Person`
```

new运算符还可以让`fred`使用`Person.prototype`里的所有值

```js
function Person(name) {
  this.name = name;
}
Person.prototype.sayHi = function() {
  alert('Hi, I am ' + this.name);
}

var fred = new Person('Fred');
fred.sayHi();
```

这就是人们以前已经使用`funtion`来模拟`class`的方式

