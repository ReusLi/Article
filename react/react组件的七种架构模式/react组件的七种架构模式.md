### 原文链接 :

[7 architectural attributes of a reliable React component ](https://dmitripavlutin.com/7-architectural-attributes-of-a-reliable-react-component/#1singleresponsibility)

react是一个基础类库, 你可以基于react组件高复用的优势, 把用户界面分割成一个个小的组件

基于组件的开发模式对生产率而言是有很大好处的, 一个复杂的系统通常由一个个简单并易于维护的模块组成. 但是, 只有把代码设计得非常好, 才能让一个个模块可以自由组合并且达到高复用度.

所以, 哪怕是业务场景复杂, 时间赶或者在毫无准备的情况下接到了紧急需求, 你也要保证代码设计的不要偏离自由组合以及高复用性这两个核心点

![Alt text]("https://github.com/ReusLi/Article/blob/master/react/react%E7%BB%84%E4%BB%B6%E7%9A%84%E4%B8%83%E7%A7%8D%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F/work-life-balance.jpg?raw=true")