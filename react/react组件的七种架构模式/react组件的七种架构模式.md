### 原文链接 :

[7 architectural attributes of a reliable React component ](https://dmitripavlutin.com/7-architectural-attributes-of-a-reliable-react-component)

react是一个基础类库, 你可以基于react组件高复用的优势, 把用户界面分割成一个个小的组件

基于组件的开发模式对生产率而言是有很大好处的, 一个复杂的系统通常由一个个简单并易于维护的模块组成. 但是, 只有把代码设计得非常好, 才能让一个个模块可以自由组合并且达到高复用度.

所以, 哪怕是业务场景复杂, 时间赶或者在毫无准备的情况下接到了紧急需求, 你也要保证代码设计的不要偏离自由组合以及高复用性这两个核心点

不幸的是, 经常会有一些错误的观点出现在我们的视线中, 比如:

* 写功能大而全的组件
* 组件之间紧耦合
* 不需要单元测试

这些错误的观点, 导致我们增加了许多[技术债务]("https://www.nczonline.net/blog/2012/02/22/understanding-technical-debt/"), 慢慢地, 我们会发现代码的维护和新功能的增加变得异常困难

当我在写react应用时, 我经常会问自己3个问题:
* **如何正确地构造组件?**
* **组件达到什么规模时需要分割?**
* **如何设计组件之间的通信才不会让组件之间变得紧耦合?**

幸运的是, 健壮的组件之间是有共同特征的, 让我们一起来学习这7种让组件变得更健壮的架构模式


## 目录
* [1 "单一职责"](#1)
* [1.1多重责任的陷阱](#1.1)
* [1.2案例研究：使组件承担一项责任](#1.2)
* [1.3案例研究：HOC赞成单一职责原则](#1.3)

* [2 "封装"](#2)
* [2.1信息隐藏](#2.1)
* [2.2沟通](#2.2)
* [2.3案例研究：封装修复](#2.3)

* [3 "可组合的"](#3)
* [3.1组成好处](#3.1)

* [4 "可重复使用的"](#4)
* [4.1跨应用程序重用](#4.1)
* [4.2重用第三方库](#4.2)

* [5 "纯"或"几乎纯"](#5)
* [5.1案例研究：从全局变量中纯化](#5.1)
* [5.2案例研究：从网络请求中净化](#5.2)
* [5.3将几乎纯粹转化为纯粹](#5.3)

* [6 "可测试"和"测试"](#6)
* [6.1案例研究：可测试的手段设计良好](#6.1)
* [7 "有意义"](#7)
* [7.1组件命名](#7.1)
* [7.2案例研究：编写不言自明的代码](#7.2)
* [7.3表现力的楼梯](#7.3)

* [8 做不断的改进](#8)

* [9 可靠性很重要](#9)

* [10 结论](#10)

<h1 id="1">1.单一职责</h1>

单一职责是写组件时经常需要考虑的原则之一

单一职责要求组件只负责一件事情, 只受一个因素的影响

这个"职责"可以是渲染一个列表,展示一个时间空间,发起一个http请求,绘画一个图表,或者懒加载一张图片等. 

为什么"只有一个理由可以改变组件"这个原则是非常重要的? 因为这样做能够让组件低耦合并且容易控制. 通过单一职责原则去控制组件的规模, 让组件专心实现本身的逻辑, 这对代码的实现, 后续的维护, 重用和测试都是极其方便的.

让我们看以下的例子:

用例1: 一个组件,它的职责是获取远程调用的数据, 那么相应地, 当远程调用的逻辑改变时, 就相当于有一个可以影响组件变动的因素. 这个因素可以是:

* URL地址改变了

* response的格式有变动

* 你希望换一个HTTP的类库


用例2: 一个表格组件, 它的数据包是一个数组的结构, 那么, 可以影响组件变动的因为可以是: 

* 要求分页

* 要求空数据时显示提示语

以上的描述, 你觉得这个表格组件是不是一个单一职责的组件呢? 答案是: 是的. 所以, 请把分页和提示分拆成2个职责吧.

关于单一职责原则, 还有一种说法是: 你必须清楚地知道这种设计模式总是围绕一个"核心点"的. 拿上面的例子来说, "核心点" 就是 远程请求逻辑 和 渲染数据逻辑.

如果你对单一职责模式还不明白的地方, 可以看[这篇文章](https://8thlight.com/blog/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html)

一个项目, 在从0到1的阶段, 往往需要在代码层面经过高频的修改, 这时候是最适合使用单一职责模式的阶段.


<h2 id="1.1">1.1 多重职责的陷阱</h2>

当一个多职责的组件产生了一个小bug时, 第一次看似乎是小问题, 对整个组件而言是无足轻重的, 而且很容易修复, 因此: 

* 你开始编程修复这个bug, 不需要关注这个组件内各个职责的耦合度, 在修复过程中也不需要重新规则组件的结构设计

* 修复过程中也不需要创造新的组件来分担职责

这就是大多数项目, 产品最开始的过程, 快速, 不注重职责分离地编程. 这样的后果是当代码规模变大后, 后续的迭代和维护变得异常复杂和困难.

当一个组件拥有多个职责时, 就意味着有多个可以影响它变动的因素存在. 这时候, 最常见的问题出现了: 因为修改其中一个职责, 而影响了其他职责的功能, 这是非常致命的.

这种多职责的设计是脆弱的, 而且难以预测和控制.

比如:  `<ChartAndForm>` 组件实现了2个功能 
1. 绘制一个chart表 
2. 处理为这个chart表提供数据的表单
这时候, `<ChartAndForm>`自然就有了2个可以让它变动的因素

当你需要对表单进行一个变动时(如把`input`换成`select`), 可能会影响到chart表的布局. 

解决多职责组件的方式是把它们分拆成多个单一职责的小组件.

最糟糕的多职责组件叫 `<God Component>`  , 比如: `<Application>`, `<Manager>`, `<BigContainer>`, `<Page>` 这些组件动辄超过500行代码.

<h2 id="1.2">1.2 案例研究, 让你的组件只拥有一种职责</h2>

假如, 有一个组件, 目前它作用是通过一个HTTP请求获取当前的天气数据, 拿到数据后会通过更新状态来把数据显示在页面上. 代码实现大概是这样的:

```js
import axios from 'axios';  

// Problem: A component with multiple responsibilities 
class Weather extends Component {  
   constructor(props) {
     super(props);
     this.state = { temperature: 'N/A', windSpeed: 'N/A' };
   }

   render() {
     const { temperature, windSpeed } = this.state;
     return (
       <div className="weather">
         <div>Temperature: {temperature}°C</div>
         <div>Wind: {windSpeed}km/h</div>
       </div>
     );
   }

   componentDidMount() {
     axios.get('http://weather.com/api').then(function(response) {
       const { current } = response.data; 
       this.setState({
         temperature: current.temperature,
         windSpeed: current.windSpeed
       })
     });
   }
}
```

在处理类似的问题时, 要经常问自己: 我需要把这个组件分拆成更小的组件吗?  最标准的答案是: 根据组件的职责来决定.

`<Weather>` 组件有2个可以影响它变动的因素:

* 取数操作, 在react的componentDidMount阶段, 可能因为url或数据格式变动而需要改动
* 数据的可视化操作, 在react的render阶段, 当要求数据的展现形式发生改变时, 逻辑会需要变动

因此, 我们需要把这2个职责分离开来, 变成 `<WeatherFetch>` 和 `<WeatherInfo>`,
`<WeatherFetch>` 负责取数, `<WeatherInfo>` 负责展示, 代码如下:

```js
import axios from 'axios';  

// Solution: Make the component responsible only for fetching
class WeatherFetch extends Component {  
   constructor(props) {
     super(props);
     this.state = { temperature: 'N/A', windSpeed: 'N/A' };
   }

   render() {
     const { temperature, windSpeed } = this.state;
     return (
       <WeatherInfo temperature={temperature} windSpeed={windSpeed} />
     );
   }

   componentDidMount() {
     axios.get('http://weather.com/api').then(function(response) {
       const { current } = response.data; 
       this.setState({
         temperature: current.temperature,
         windSpeed: current.windSpeed
       });
     });
   }
}
```

这样做的好处是什么?

例如: 你可能需要用 `async/await` 来代替  `promises`

```js
// Reason to change: use async/await syntax
class WeatherFetch extends Component {  
   // ..... //
   async componentDidMount() {
     const response = await axios.get('http://weather.com/api');
     const { current } = response.data; 
     this.setState({
       temperature: current.temperature,
       windSpeed: current.windSpeed
     });
   }
}
```

又例如: 你可能需要在风速为0的时候不是显示 `0km/h`, 而是显示 `calm`

```js
// Reason to change: handle calm wind  
function WeatherInfo({ temperature, windSpeed }) {  
   const windInfo = windSpeed === 0 ? 'calm' : `${windSpeed} km/h`;
   return (
     <div className="weather">
       <div>Temperature: {temperature}°C</div>
       <div>Wind: {windInfo}</div>
     </div>
   );
}
```

无论是哪一种, 都因为2个职责被分离了, 而降低了影响相互间功能的可能性

`<WeatherFetch>`和`<WeatherInfo>`有自己的责任。
一个组件的更改对另一个组件的影响很小。
这就是单一责任原则的优势：分离地进行修改，这样做对系统的影响是轻微并且可预测的。

<h2 id="1.3">1.3 用例学习: 高阶组件最爱单一职责模式</h2>

现在,我们来从高阶组件(HOC: height order component)的例子中学习一下单一职责的好处.

>高阶组件(HOC)是一个接收一个组件并返回一个新组件的函数。

HOC的常见用法是: 为组件提供额外的props属性或修改已经存在的props值, 这种技术被称为props代理.

```js
function withNewFunctionality(WrappedComponent) {  
  return class NewFunctionality extends Component {
    render() {
      const newProp = 'Value';
      const propsProxy = {
         ...this.props,
         // Alter existing prop:
         ownProp: this.props.ownProp + ' was modified',
         // Add new prop:
         newProp
      };
      return <WrappedComponent {...propsProxy} />;
    }
  }
}
const MyNewComponent = withNewFunctionality(MyComponent);  
```

您也可以通过修改包装组件呈现的元素来挂钩到呈现机制。
这种HOC技术被称为渲染劫持：

```js
function withModifiedChildren(WrappedComponent) {  
  return class ModifiedChildren extends WrappedComponent {
    render() {
      const rootElement = super.render();
      const newChildren = [
        ...rootElement.props.children, 
        // Insert a new child:
        `<div>New child</div>`
      ];
      return cloneElement(
        rootElement, 
        rootElement.props, 
        newChildren
      );
    }
  }
}
const MyNewComponent = withModifiedChildren(MyComponent);  
```

如果你想深入了解HOC, 可以阅读这篇文章: [react高阶组件的深入了解](https://medium.com/@franleplant/react-higher-order-components-in-depth-cf9032ee6c3e)

让我们一起来看一下, HOC是怎么体现单一职责原则的.

组件`<PersistentForm>`由一个输入框`input`和保存按钮`button`组成, `input`的值在输入后被保存在`localstorage`, 当改变输入值时, 点击保存按钮`button`, 会更新`localstorage`的值

代码如下:

```js
class PersistentForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = { inputValue: localStorage.getItem('inputValue') };
    this.handleChange = this.handleChange.bind(this);
    this.handleClick = this.handleClick.bind(this);
  }

  render() {
    const { inputValue } = this.state;
    return (
      <div>
        <input type="text" value={inputValue} 
          onChange={this.handleChange}/> 
        <button onClick={this.handleClick}>Save to storage</button>
      </div>
    )
  }

  handleChange(event) {
    this.setState({
      inputValue: event.target.value
    });
  }

  handleClick() {
    localStorage.setItem('inputValue', this.state.inputValue);
  }
}

ReactDOM.render(<PersistentForm />, document.getElementById('root')); 
```

当`input`输入框值变化时, 会在`handleChange`函数内更新数据, 当点击保存`button`时会在handleClick里保存数据

现在, `<PersistentForm>`组件有2个因素可影响它的变动:
* 表单`input`的管理
* 保存操作的管理

让我们重构`<PersistentForm>`, 让它变成单一职责: 

* **渲染表单字段并附加事件处理程序, 这个过程它不应该知道如何直接使用存储**

```js
class PersistentForm extends Component {  
  constructor(props) {
    super(props);
    this.state = { inputValue: props.initialValue };
    this.handleChange = this.handleChange.bind(this);
    this.handleClick = this.handleClick.bind(this);
  }

  render() {
    const { inputValue } = this.state;
    return (
      <div className="persistent-form">
        <input type="text" value={inputValue} 
          onChange={this.handleChange}/> 
        <button onClick={this.handleClick}>Save to storage</button>
      </div>
    );
  }

  handleChange(event) {
    this.setState({
      inputValue: event.target.value
    });
  }

  handleClick() {
    this.props.saveValue(this.state.inputValue);
  }
}
```

组件从prop initialValue接收存储的输入值，并使用prop的函数saveValue（newValue）保存输入值。

`props`由使用道具代理技术的Persistence（）HOC提供。

现在`<PersistentForm>`组件已经符合SRP原则, 只有一个因素可以影响它的变动, 因为保存管理的因素已经被转移到了父组件上

```js
function withPersistence(storageKey, storage) {  
  return function(WrappedComponent) {
    return class PersistentComponent extends Component {
      constructor(props) {
        super(props);
        this.state = { initialValue: storage.getItem(storageKey) };
      }

      render() {
         return (
           <WrappedComponent
             initialValue={this.state.initialValue}
             saveValue={this.saveValue}
             {...this.props}
           />
         );
      }

      saveValue(value) {
        storage.setItem(storageKey, value);
      }
    }
  }
}
```

withPersistence() 是一个高阶组件, 它不需要知道表单是如何实现的, 它只关心自己的工作, 提供初始化的值和保存的方法.

把 `<PersistentForm>` 和 `withPersistence()` 写成一个组件 `<LocalStoragePersistentForm>`. 

```js
const LocalStoragePersistentForm  
  = withPersistence('key', localStorage)(PersistentForm);

const instance = <LocalStoragePersistentForm />;  
```

这时`withPersistence()`的任何修改都不会干扰到`<PersistentForm>`的`初始化`和`保存`逻辑.

反之, `<PersistentForm>`的任何修改也不会干扰到表单的处理逻辑.

我们再次重申SRP原则: 允许单独修改模块, 并且尽可能少地影响其他模块.

而且会使模块的重用性增加了, 你可以在任何地方引入这个模块:

```js
const LocalStorageMyOtherForm  
  = withPersistence('key', localStorage)(MyOtherForm);

const instance = <LocalStorageMyOtherForm />;  
```

你可以很容易地修改逻辑, 比如把储存方式改成sessionStorage: 

```js
const SessionStoragePersistentForm  
  = withPersistence('key', sessionStorage)(PersistentForm);

const instance = <SessionStoragePersistentForm />;  
```

In situations when composition is ineffective `(还没想好怎么翻译)`

`props proxy(属性代理)` 和 `render highjacking(渲染劫持)` 的高阶组件技术让组件职责分离的做法变得非常轻松.

<h1 id="2">2. 封装</h2>

> 封装一个组件, 指的是为组件提供props来控制它的行为, 同时不暴漏它的内部实现和结构

根据组件依赖程度，可以区分2种耦合类型：

* 松耦合: 当应用程序组件对其他组件知之甚少或根本不了解时
* 紧耦合: 当应用程序组件知道彼此的许多细节时

松耦合的目的是减少组件之间的关联.

封装, 是设计组装时的基本原则, 也是松耦合的关键

<h2 id="2.1"> 2.1 信息隐藏 </h2>

一个封装得好的组件, 应该是会把内部实现隐藏, 只对外提供props来控制组件的行为

隐藏内部实现是必要的, 因为组件的内部实现不应该被其他的组件知道, 或者说内部实现不应该跟其他的组件产生关联关系

一个React的组件, 有可能是功能组件, 或者基础组件(底层建筑). 或者是方法的实例. 这里实现都应该被封装在组件内部.

封装, 隐藏实现, 降低对其他组件的依赖, 都是松耦合的关键.