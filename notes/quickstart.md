# 1. React.createElement

```
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
```

is same with

```
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```

this is because `Babel compiles`

- React.createElement 使用时会创造一个对象（React elements），如下

```
// Note: this structure is simplified
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world!'
  }
};
```

- React element 是不可改变的 ，如果要更新，需要重新create

# 2. Rendering Elements

示例中有个`id`为`root`的`div`，这是一个根结构，所有的`react dom`会在这个里面

## ReactDOM.render()
可以通过`ReactDOM.render()`将react dom渲染到根节点中

```
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

在做update工作的时候，react 比较react element 和它的children，然后更新dom，降低dom渲染成本

# 3. Components and Props

`function`和es6中的`class`都可以作为一个组件,通常我们使用第二种

```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

```
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

# 4. State and Lifecycle

我们希望组件有子自更新的能力，能记住一些状态值，相比function，使用class更加方便

```
    // 我们希望组件能有自己的state值，而不是全部通过props传入，并且能够完成自更新，所以使用class创建组件会更方便
    class Clock extends React.Component {
        constructor(props) {
            super(props)
            this.state = {date: new Date()};
        }
        // componentDidMount 和 componentWillUnmount为生命周期函数
        componentDidMount() {
            // render 完成之后触发
            this.timerID = setInterval(
                    () => this.tick(),
                    1000
            );
        }
        componentWillUnmount() {
            clearInterval(this.timerID)
        }
        tick() {
            // 修改state时必须使用setState, 只有在constructor中可以直接用字面量给state赋值
            this.setState({
                date: new Date()
            });
        }
        render() {
            return (
                <div>
                    <h1>hi</h1>
                    <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
                </div>
            );
        }
    }

    ReactDOM.render(
            <Clock />,
            document.getElementById('root')
    )

```

- setState()
setState用于更新state，会将传入的对象和原有对象进行merge工作

```
this.setState({comment: 'Hello'});
```

当setState的值中有涉及到自身的计算时，可以采用如下写法

```
// Correct
this.setState((prevState, props) => ({
  counter: prevState.counter + props.increment
}));
```

# 5. Handling Events

函数绑定可以通过如下方式进行
```
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    this.state = {isToggleOn: true};

    // This binding is necessary to make `this` work in the callback
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick(e) {
    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}

ReactDOM.render(
  <Toggle />,
  document.getElementById('root')
);
```

需要注意的是

- 在函数中不能使用return false 来阻止默认事件，而应使用e.preventDefault

- 直接使用函数在触发事件的时候会存在this指向错误的问题，官方提供的解决方法是

    1. 在constractor中使用bind绑定this指向
    2. 通过字段绑定箭头函数的形式执行
    ```
    handleClick = () => {
        console.log('this is:', this);
      }
    ```
    3. 在返回的render模板中使用箭头函数
    ```
      render() {
        // This syntax ensures `this` is bound within handleClick
        return (
          <button onClick={(e) => this.handleClick(e)}>
            Click me
          </button>
        );
      }
    ```
    
# 6. Conditional Rendering

关于条件渲染感觉没什么特别好说的

```
function WarningBanner(props) {
  if (!props.warn) {
    return null;
  }

  return (
    <div className="warning">
      Warning!
    </div>
  );
}

class Page extends React.Component {
  constructor(props) {
    super(props);
    this.state = {showWarning: true};
    this.handleToggleClick = this.handleToggleClick.bind(this);
  }

  handleToggleClick() {
    this.setState(prevState => ({
      showWarning: !prevState.showWarning
    }));
  }

  render() {
    return (
      <div>
        <WarningBanner warn={this.state.showWarning} />
        <button onClick={this.handleToggleClick}>
          {this.state.showWarning ? 'Hide' : 'Show'}
        </button>
      </div>
    );
  }
}

ReactDOM.render(
  <Page />,
  document.getElementById('root')
);
```

# 7. Lists and Keys

列表渲染如下：
在外部调用数组map
```
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <ListItem key={number.toString()}
              value={number} />

  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}
```

直接嵌入return的模板中
```
function NumberList(props) {
  const numbers = props.numbers;
  return (
    <ul>
      {numbers.map((number) =>
        <ListItem key={number.toString()}
                  value={number} />

      )}
    </ul>
  );
}
```

关于key的使用方法和意义和vue中v-for的key类似

# 8.form
form 表单基本上基于上面的内容
需要注意一下的是，它通过value属性绑定textarea的内容、select的选中值什么的

- 在select中，可以通过这样的形式实现多选

    `<select multiple={true} value={['B', 'C']}>`
    
# 9.Lifting State Up
想要实现组件间的数据交流，官方通过将state提升到最近的父级组件中，然后通过props传给子组件

需要注意的是，由于之前提到的组件自身是不允许改变porps值的，所以改值函数也应该由父级的props传入

如下：
```
render() {
    const scale = this.state.scale;
    const temperature = this.state.temperature;
    const celsius = scale === 'f' ? tryConvert(temperature, toCelsius) : temperature;
    const fahrenheit = scale === 'c' ? tryConvert(temperature, toFahrenheit) : temperature;

    return (
      <div>
        <TemperatureInput
          scale="c"
          temperature={celsius}
          onTemperatureChange={this.handleCelsiusChange} />

        <TemperatureInput
          scale="f"
          temperature={fahrenheit}
          onTemperatureChange={this.handleFahrenheitChange} />

        <BoilingVerdict
          celsius={parseFloat(celsius)} />

      </div>
    );
  }
```

# 10.Composition vs Inheritance

- Composition

在实际开发中，我们第一时间并不清楚组件内部的内容，比如`dialog`组件，所以可以用标签内部写内容的方式来作为`children` prop传递给子组件，也可以通过自定义props的方式传值

```
function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
    // 嵌套的内容可以在FancyBorder 的 props.children中获取到
      <h1 className="Dialog-title">
        Welcome
      </h1>
      <p className="Dialog-message">
        Thank you for visiting our spacecraft!
      </p>
    </FancyBorder>
  );
}
```

嵌套的内容可以在FancyBorder 的 props.children中获取到

```
function FancyBorder(props) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  );
}
```

通过props传值

```
function SplitPane(props) {
  return (
    <div className="SplitPane">
      <div className="SplitPane-left">
        {props.left}
      </div>
      <div className="SplitPane-right">
        {props.right}
      </div>
    </div>
  );
}

function App() {
  return (
    <SplitPane
      left={
        <Contacts />
      }
      right={
        <Chat />
      } />
  );
}
```

- Specialization

    这个是关于通用组件和通用组件的特列问题
    
    意思是可以先做一个通用的基础组件，然后再基于这个组件写一个新的组件
    
# 11.Thinking in React
讲述了一个react构建页面的思想

1. 将整个原型图拆分成一个有层次的组件结构树
2. 做一个不带交互的静态版本，数据有上层组件传入
3. 定义组件的state，难点在于如何确定某个字段需要定义为state并且将状态放在哪个组件中
4. 由于react是单项数据流，所以需要增加反向数据流
