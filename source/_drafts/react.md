---
title: react
comments: true
date: 2020-06-15 15:16:09
tags:
---

react 学习笔记
[react](https://react.docschina.org/docs/getting-started.html)
<!--more-->

## 基础

### 元素
```javascript

// 可以加括号也可以不加
const element = <h1>Hello, world!</h1>;
const element = (
  <h1>
    Hello, world!
  </h1>
);

// JavaScript 代码需要大括号包裹
const user = {
  firstName: 'Harper',
  lastName: 'Perez'
};

// 可以通过 引号 或者 大括号，对元素进行赋值，但是不能同时使用
const element = <div tabIndex="0"></div>;
const element = <img src={user.avatarUrl}></img>;
```

### 组件

React 会将以小写字母开头的组件视为原生 DOM 标签。例如，<div / >是 html 的标签，而 <Welcome /> 是一个组件。

组件中，props 是不可被更改的。

组件中，states 是私有的，完全受控于组件。

```javascript
// 下面两种组件在 react 中等效
// 组件名称必须大写字母开头
// 函数组件
function Welcome(props) {
    return <h1>Hello,{props.name}</h1>;
}
// class 组件
class Welcome extends React.Component {
    render() {
        return <h1>Hello,{props.name}</h1>;
    }
}

// 组件渲染
ReactDOM.render(
    <Welcome name={foo} />,
    document.getElementById('root')
);

// state
class Clock extends React.Component {
    constructor(props) {
        super(props);
        this.state = {date: new Date()};
    }
    // 声明周期方法
    componentDidMount() {
        this.timerID = setInterval(
            () => this.tick(),
            1000
        );
    }
    componentWillUnmount() {
        clearInterval(this.timerID);
    }
    tick() {
        // 使用 setState() 修改
        this.setState({
            date: new Date()
        });
    }
    render() {
        return (
            <div>
                <h1>Hello,world!</h1>
                <h2>It is {this.state.date.toLocaleString()}.</h2>
            </div>
        );
    }
}

// this.state 和 this.props 的更新可能是异步的，不要依赖他们的值来更新状态
// 正确的更新方式
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
this.setState(function(state, props) {
  return {
    counter: state.counter + props.increment
  };
});
```

### 事件处理

React 事件的命名采用小驼峰式（camelCase），而不是纯小写。

```javascript
// 在 React 中另一个不同点是你不能通过返回 false 的方式阻止默认行为。
// 你必须显式的使用 preventDefault
function ActionLink() {
    function handleClick(e) {
        e.preventDefault();
        console.log('The link was clicked.');
    }
    return (
        <a href="#" onClick={handleClick}>
            Click me
        </a>
    );
}

// 绑定事件
class Toggle extends React.Component {
    constructor(props) {
        super(props);
        this.state = {isToggleOn: true};

        // 为了在回调中使用 ‘this’，这个绑定必不可少
        this.handleClick = this.handleClick.bind(this);
    }

    handleClick() {
        this.setState(state => ({
            isToggleOn: !state.isToggleOn
        }));
    }

    render() {
        return (
            <button onClick={this.handleClick}>
                {this.state.isToggleOn? 'ON' : 'OFF'}
            </button>
        );
    }
}

// 传递参数
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

### 条件渲染

在 JavaScript 中，true && expression 总是会返回 expression, 而 false && expression 总是会返回 false。

```javascript
function Mailbox(props) {
  const unreadMessages = props.unreadMessages;
  return (
    <div>
      <h1>Hello!</h1>
      {unreadMessages.length > 0 &&
        <h2>
          You have {unreadMessages.length} unread messages.
        </h2>
      }
    </div>
  );
}
```

### 列表 key

```javascript
function ListItem(props) {
  // 正确！这里不需要指定 key：
  return <li>{props.value}</li>;
}

function NumberList(props) {
  const numbers = props.numbers;
  return (
    <ul>
        {numbers.map((number) =>
            // 在这里指定 key
            <listItem key={number.toString()} value={number} />
        )}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

### 表单

```javascript
// input 输入
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {input: ''};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('提交的名字: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          名字:
          <input type="text" value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="提交" />
      </form>
    );
  }
}

// select
<select value={this.state.value} onChange={this.handleChange}>
    <option value="grapefruit">葡萄柚</option>
    <option value="lime">酸橙</option>
    <option value="coconut">椰子</option>
    <option value="mango">芒果</option>
</select>

// textarea
<textarea value={this.state.value} onChange={this.handleChange} />

// 文件 input 标签
<input type="file" />

// 处理多个输入
// 我们可以给每个元素添加 name 属性，并让处理函数根据 event.target.name 的值选择要执行的操作。
class Reservation extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      isGoing: true,
      numberOfGuests: 2
    };

    this.handleInputChange = this.handleInputChange.bind(this);
  }

  handleInputChange(event) {
    const target = event.target;
    const value = target.name === 'isGoing' ? target.checked : target.value;
    const name = target.name;

    this.setState({
      [name]: value
    });
  }

  render() {
    return (
      <form>
        <label>
          参与:
          <input
            name="isGoing"
            type="checkbox"
            checked={this.state.isGoing}
            onChange={this.handleInputChange} />
        </label>
        <br />
        <label>
          来宾人数:
          <input
            name="numberOfGuests"
            type="number"
            value={this.state.numberOfGuests}
            onChange={this.handleInputChange} />
        </label>
      </form>
    );
  }
}
```

### 状态提升

### 组合 vs 继承

### react