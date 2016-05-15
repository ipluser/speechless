# Flux 应用架构
**Flux**是Facebook用来构建客户端Web应用的一种应用架构体系。它是一种类似MVC的架构，但是它更加简单、清晰，是一种单向数据流的架构设计。

**Note**

请事先对[React](http://facebook.github.io/react/)和[ES6](http://es6.ruanyifeng.com/)进行了解。

由于Flux有很多种实现框架，本文中是采用Facebook官方的[Flux](https://github.com/facebook/flux)。

## 安装实例
本文中以一个简单的**Todo List**实例来讲解**Flux**：

```
$ git clone https://github.com/ipluser/react-flux-demo.git
$ cd react-flux-demo
$ npm start
```

浏览器将会显示**Todo**界面（若未显示，请访问`http://127.0.0.1:8080`）：

![Demo](../../public/img/architecture/flux__demo.png)

## 核心思想
**Flux**是一种单向数据流的架构体系，其核心思想：

| Name | Description |
|:-----|:------------|
| view | 视图层 |
| Action | 行为动作层，视图层发出的动作如`click event` |
| Dispatcher | 分发中心，接受动作和调用数据流向中的回调函数 |
| Store | 数据层，管理应用状态，若状态改变，马上通知View层重新渲染 |

**Flux**数据流向图：

![Flux Data Flow](../../public/img/architecture/flux__data-flow.png)

上图中可以看出**Flux**总是一种单向的流动，不存在数据双向流动。数据的流向总是清晰可见，可以很清楚的知道应用当前的状态和数据情况。

## View
打开**react-flux-demo**中的<mark>main</mark>入口文件:

```js
// public/scripts/main.jsx
import React from 'react';
import ReactDOM from 'react-dom';
import TodoController from './components/todoController.jsx';

ReactDOM.render(<TodoController />, document.getElementById('app'));

```

此处采用[Controller View Pattern](http://blog.andrewray.me/the-reactjs-controller-view-pattern/)，引用了<mark>TodoController</mark>，其管理应用状态和行为：

```js
// public/scripts/components/todoController.jsx
import React from 'react';

import TodoAction from '../actions/todoAction.js';
import Todo from './todo.jsx';

export default React.createClass({
  newItem() {
  	TodoAction.addItem('new item');
  },

  render() {
  	return <Todo newItem={this.newItem} />;
  }
})

```

<mark>TodoController</mark>对<mark>Todo</mark>组件指定了实际动作。而<mark>Todo</mark>代码非常简单：

```js
// public/scripts/components/todo.jsx
import React from 'react';

import '../../styles/components/todo.scss';

export default function Todo(props) {
  let list = props.items.map((item, index) => {
    return <li className="color--red" key={index}>{item}</li>;
  });

  return <div className="todo">
    <ul>{list}</ul>
    <button className="todo__click-btn" onClick={props.newItem}>Todo</button>
  </div>;
}
```

<mark>Todo</mark>组件不包含状态，仅为逻辑组件，使组件更轻薄，更易测试和通用。

<mark>TodoController</mark>给<mark>Todo</mark>指定了一个动作，当点击Todo按钮，将会触发<mark>TodoAction.addItem</mark>动作。

## Action
**View**的实际动作交由<mark>TodoAction</mark>管理，其通知**Dispatcher**发生的动作类型和数据：

```js
// public/scripts/actions/todoAction.js
import AppDispatcher from '../dispatcher.js';
import TodoConstant from '../constants/todoConstant.js';  // 动作类型-常量对象

class TodoAction {
  addItem(text) {
    AppDispatcher.dispatch({
      actionType: TodoConstant.ADD_ITEM,
      text
    });
  }
}

export default new TodoAction();
```

<mark>TodoConstant</mark>常量对象：

```js
// public/scripts/constants/todoConstant.js
export default {
  ADD_ITEM: 'TODO_ADD_ITEM'
}
```

## Dispatcher
<mark>Dispatcher</mark>注册所有的行为动作**Actions**，若发生行为动作，将会分发给**Stores**。<mark>Dispatcher</mark>有点类似路由或网关，当发生动作时，分发给指定的**Stores**做相应的状态处理：

```js
// public/scripts/dispatcher.js
import {Dispatcher} from 'flux';

import TodoStore from './stores/todoStore';
import TodoConstant from './constants/todoConstant';

let AppDispatcher = new Dispatcher();

TodoStore.dispatchToken = AppDispatcher.register(function (payload) {
  switch(payload.actionType) {
  	case TodoConstant.ADD_ITEM:
  	  TodoStore.addItem(payload.text);
  	  break;
  }
});

export default AppDispatcher;
```

当发生**Todo - ADD_ITEM**动作时，触发**TodoStore.addItem**。

## Store
<mark>TodoStore</mark>管理应用状态，当状态发生改变时，会调用那些监听了该状态更新的**View**回调函数：

```js
// public/scripts/stores/todoStore.js
import EventEmitter from 'events';

class TodoStore extends EventEmitter {
  constructor() {
  	super();
  	this.items = [];
  }

  getAll() {
  	return this.items;
  }

  addItem(text) {
  	this.items.push(text);
  	this.change();
  }

  change() {
  	this.emit('change');
  }

  addListener(name, callback) {
  	this.on(name, callback);
  }

  removeListener(name, callback) {
  	this.removeListener(name, callback);
  }
}

export default new TodoStore();
```

## View
<mark>TodoController</mark>初始化状态，同时监听<mark>TodoStore</mark>的状态更新。一旦状态发生改变，将会触发**onListChange**事件，设置应用状态，并以属性形式传递给<mark>Todo</mark>组件，使其重新渲染：

```js
// public/scripts/components/todoController.jsx
import React from 'react';

import TodoAction from '../actions/todoAction.js';
import TodoStore from '../stores/todoStore.js';
import Todo from './todo.jsx';

export default React.createClass({
  getInitialState() {
  	return {
  	  items: TodoStore.getAll()
  	}
  },

  componentDidMount() {
  	TodoStore.addListener('change', this.onListChange);
  },

  componentWillUnmount() {
  	TodoStore.removeListener('change', this.onListChange);
  },

  newItem() {
  	TodoAction.addItem('new item');
  },

  onListChange() {
    this.setState({
      items: TodoStore.getAll()
    });
  },

  render() {
  	return <Todo items={this.state.items} newItem={this.newItem} />;
  }
})
```

## 致谢
- [Facebokk Flux](https://facebook.github.io/flux/docs/overview.html)
- [Andrew - Controller-View](http://blog.andrewray.me/the-reactjs-controller-view-pattern/)
- [ruanyifeng - Flux 架构入门教程](http://www.ruanyifeng.com/blog/2016/01/flux.html)

## 源代码
[react-flux-demo](https://github.com/ipluser/react-flux-demo)
