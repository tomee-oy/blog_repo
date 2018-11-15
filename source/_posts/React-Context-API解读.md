---
title: React Context API解读
date: 2018-04-15 10:45:26
tags: [react]
categories: 技术
---

### 一、为什么需要context api
--
在react开发过程中，我们可能会遇到这样的需求：某组件需要将值传给它的孙子，甚至嵌套更深的后代组件，如果不考虑context api，我们的实现方式，便是采用props逐层向下传递，直到需要获取到值的组件为止：

```javascript
// 顶层组件 grandParent.js
class GrandParent extends Component{
	render(){
	  return <Parent text='我是一个跨组件变量传递示例'/>
	}
}

// 中间层组件 parent.js
class Parent extends Component{
	constructor(props){
		super(props)
	}
	render(){
		return <Child text={this.props.text}/>
	}
}

// 需要获取顶层组件的text值的底层组件 child.js
class Child extends Component{
	constructor(props){
		super(props)
	}
	render(){
		return this.props.text
	}
}
```

首先，应该肯定的是这种方式是可以实现的。但是一旦嵌套层数太多，在每一层组件里都要用`this.props`去接收上层组件传来的变量，那么，你的代码将陷入链式地狱，并且变得难以维护。由此，context api应运而生。

### 二、旧版context api
--
关于context api是用来干什么的，在上一部分实际已经阐述出来了：解决跨组件数据传递的问题。在context api中，我们只需要在数据提供者（即顶层组件）中宣称：“本宝宝可以提供你们想要的xxx数据。”，然后在需要数据的组件中“回应”称：“本大大需要你的数据，请给我一份。”即可完成一次完整的跨组件数据传递：

```javascript
// 顶层组件 grandParent.js
import PropTypes from 'prop-types'
class GrandParent extends Component{
	getChildContext(){
		return {
			text: '我是旧版context api示例'
		}
	}
	render(){
		return <Parent />
	}
}
GrandParent.childContextTypes = {
	text: PropTypes.string
}

// 中间层组件 parent.js
class Parent extends Component{
	render(){
		return <Child />
	}
}

// 需要获取顶层组件的text值的底层组件 child.js
import PropTypes from 'prop-types'
class Child extends Component{
	render(){
		return this.context.text
	}
}
Child.contextTypes = {
	text: PropTypes.string
}

```
较于props的传递方式，context api不用在每个中间层组件中都显示地将props中的text设置到子组件的属性中，对于跨组件的数据传递确实提供了方便。但这种方式的缺陷在于：由于中间层组件并不依赖于context的存在，如果中间层某个组件采用`shouldComponentUpdate`做组件优化，限制了组件更新的条件，而组件更新只与props、state有关，与context并无关系。所以，顶层组件更新，可能导致底层组件并不会更新。

### 三、新版context api
--
新版context api从形式上来看，更符合react风格：组件化的处理方式，用Provider来包裹顶层组件，用Consumer来包裹底层获取数据的组件，同样是宣称“本宝宝可以提供数据”，也同样是“本大大需要你的数据”，新版api不再像旧版本那样对组件产生额外的除生命周期以外的钩子。

```javascript
// 创建供全局使用的context store.js
import React from 'react'
const context = React.createContext()
export const Provider = context.Provider
export const Consumer = context.Consumer

// 顶层组件 grandParent.js
import { Provider } from './store.js'
const store = {
  text: '新版context api示例'
}
class GrandParent extends Component{
	render(){
		return (
			<Provider value = {store}>
        <MsgContainer />
      </Provider>
    )
	}
}

// 中间层组件 parent.js
class Parent extends Component{
	render(){
		return <Child />
	}
}

// 需要获取顶层组件的text值的底层组件 child.js
import { Consumer } from './store.js'
class Child extends Component{
	render(){
		return (
			<Consumer>
        {
          context => {
            return context.text
          }
        }
      </Consumer>
    )
	}
}

```
实际上，新版的context api仍然存在缺陷。假如某个组件要获取的数据有多个数据来源，那么，其组件结构类似以下的形式：
```javascript
<A>
	<B>
		<C>
			...
		</C>
	</B>
</A>
```
是不是很熟悉的嵌套地狱！如果真的遇到这种需求，我们应该依赖于第三方框架类似redux、mobx等来实现。这些框架的底层实际上也是用context api来实现的。

### 四、redux将被取代？
--
社区有人讨论说，随着context api的优化，会不会导致redux被丢弃？答案是：不会！因为无论是新版的context api还是旧版的context api，解决的都是跨层级组件间的数据传递的问题。原生的context api并不能强大到做系统级的数据管理。所以，context api只是让我们理解原理，真正的应用，正是这些强大的框架赋予我们的力量！