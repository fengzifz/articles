---
title: Redux 的 dispatch 和 bindActionCreators 总结
date: 2017-12-18 21:10:23
tags: Redux
---

使用过 redux 的朋友，对 `dispatch` 和 `bindActionCreators` 都不会陌生。但两者究竟有什么区别，相信很多新手是不知道的。

先来回顾一下 redux，简单来说，redux 是用来管理 `state` 的一个组件，它包含三个重要的基本概念：Action，Reducer 和 Store。

<!-- more -->

### 基本概念
#### Action
用户只能通过 Action 把数据传到 Store，它是 Store 数据的唯一来源。要改变数据，只能通过 Action 来改变。

Action 是一个以 JavaScript Plain Object 的形式存在，如：

```
{
	type: USER_LOGIN
}
```

其中，`type` 是必要的属性。除了 `type` 之外，其他的可以根据自己的需求添加，例如：

```
{
	type: USER_LOGIN,
	name: 'damon'
}
```

官方推荐使用函数式编程来定义 Actions：

```
export function userLogin(name) {
	return {
		type: 'USER_LOGIN'
	}
}

export function userLogout() {
	return {
		type: 'USER_LOGOUT'
	}
}
```

#### Store

Store 就是保存数据的地方，一个应用只有一个 Store。通过 `createStore` 方法来创建 store：

```
import { createStore } from 'redux';

const store = createStore(reducer);
```

#### Reducer

Store 接收到一个 Action，通过某些逻辑，需要给出一个新的状态，这种弄过程就叫 Reducer。

一般会在 reducer 定义初始的 state，当某些原因发生一些未知的 Actions 时，需要返回初始 state。

```
import { Map } from 'immutable';

const initState = Map({
	login: false，
	userId: null,
	name: null,
	age: null,
	gender: null,
	height: null
});
```

然后根据 Action 来返回不同的 state：

```
export default (state = initState, action) {
	switch (action.type) {
		case 'USER_LOGIN':
			return state.set('login', true);
		case 'USER_LOGOUT':
			return state.set('login', false);
		default:
			return state;
	}
} 
```

### 关于 dispatch 和 bindActionCreators

我们在定义好 store、reducer 和 actions 等之后，那么在组件当中如何读取和改变 state 呢？

redux 提供一个极其重要的方法：`connect`。

`connect` 接收四个参数：

- mapStateToProps
- mapDispatchToProps
- mergeProps
- options

这里重点说第二个参数：`mapDispatchToProps`。

假设我们有一个组件是：`LoginComponent`：

```
import React, { Component } from 'react';
import { connect } from 'react-redux';
import * as UserActions from '../Actions/UserActions';

class LoginComponent extends Component {
	
	login() {
		// 一些逻辑处理...
		
		this.props.userLogin();
	}
}

export default connect(
	(state) => ({
		// 一些状态，这里省略...
	}),
	(dispatch) => ({
		userLogin: () => dispatch(UserActions.userLogin())
	})
)(LoginComponent);
```

参数 `mapDispatchToProps` 的作用就是把 actions 作为 props 分发到 LoginComponent 上面，让其可以调用。在上面例子中，我们 `userLogin` 这个 action 包装成一个可以调用的方法。

#### 在 Action 中传递一个参数时...

现在，我们把 `userLogin` 这个 action 修改一下：

```
export function userLogin(userId) {
	return {
		type: 'USER_LOGIN',
		userId: userId
	}
}
```

把对应的 reducer 也修改一下，在用户登录时，设置一个 `userId` 属性：

```
export default (state = initState, action) {
	switch (action.type) {
		case 'USER_LOGIN':
			return state.set('login', true)
							.set('userId', action.userId);
		case 'USER_LOGOUT':
			return state.set('scanning', false);
		default:
			return state;
	}
} 
```

但是，我们在 `LoginComponent` 里面读取用户的 `this.props.userId` 属性时，发现它是 `undefined`。

原因很简单，是因为在 dispatch 一个 action 时，没有把对应的 userId 属性传递进去，我们修改一下 `connect` 方法：

```
export default connect(
	(state) => ({
		// 一些状态，这里省略...
	}),
	(dispatch) => ({
		userLogin: (userId) => dispatch(UserActions.userLogin(userId))
	})
)(LoginComponent);
```

注意到，这次把 `userId` 作为参数，把 `userLogin` 绑定到 `LoginComponent` 上面。那么再次去获取 `this.props.userId` 的值时，就不在是 `undefined` 了。

#### 在 Action 中传递多个参数时...

但是，如果当我们需要设置很多用户的信息时，这样操作就会比较麻烦：

修改 connect:

```
export default connect(
	(state) => ({
		// 一些状态，这里省略...
	}),
	(dispatch) => ({
		userLogin: (userId, name, age, gender, height) => dispatch(UserActions.userLogin(userId, name, age, gender, height))
	})
)(LoginComponent);
```

修改 action：

```
export function userLogin(userId, name, age, gender, height) {
	return {
		type: 'USER_LOGIN',
		userId: userId,
		name: name,
		age: age,
		gender: gender,
		height: height
	}
}
```

修改 reducer：

```
export default (state = initState, action) {
	switch (action.type) {
		case 'USER_LOGIN':
			return state.set('login', true)
							.set('userId', action.userId)
							.set('name', action.name)
							.set('height', action.height)
							.set('age', action.age)
							.set('gender', action.gender);
		case 'USER_LOGOUT':
			return state.set('scanning', false);
		default:
			return state;
	}
}
```

可以看到，这样需要传递多个参数时，就会很麻烦，在 reducer、action 和 connect 里面，都要写很多属性。

所以，redux 提供了 `bindActionCreators` 这个方法，看一下 `bindActionCreators` 实现的一部分源码：

```
function bindActionCreator(actionCreator, dispatch) {
  return (...args) => dispatch(actionCreator(...args));
}

/*
 * @param actionCreators
 * @param dispatch
 * @return {actionKey: (...args) => dispatch(actionCreator(...args))}
 */
export default function bindActionCreators(actionCreators, dispatch) {
  return mapValues(actionCreators, actionCreator =>
    bindActionCreator(actionCreator, dispatch)
  );
}
```

它可以不管参数的数量，把 actions 和 dispatch 组合起来，生成 `mapDispatchToProps` 需要的内容。

那么，现在只需要修改一下 connect 方法：

```
import { bindActionCreators } from 'redux';

export default connect(
	(state) => ({
		// 一些状态，这里省略...
	}),
	(dispatch) => ({
		userLogin: bindActionCreators(UserActions.userLogin, dispatch)
	})
)(LoginComponent);
```

又或者用更加简单的写法：

```
import { bindActionCreators } from 'redux';

export default connect(
	(state) => ({
		// 一些状态，这里省略...
	}),
	{
		userLogin: UserActions.userLogin
	}
)(LoginComponent);
```

### 总结

总结一下，`dispatch` 和 `bindActionCreators` 归根到底，其作用是一样的。但如果 actions 需要传入多个参数时，我们可以借助 `bindActionCreators` 来精简我们的代码。

##### 参考资料

- [redux 官方文档](https://redux.js.org/docs/api/bindActionCreators.html)
- [What is difference between dispatch and bindActionCreators?
](https://stackoverflow.com/questions/41342540/what-is-difference-between-dispatch-and-bindactioncreators)
- [React 实践心得：react-redux 之 connect 方法详解](http://taobaofed.org/blog/2016/08/18/react-redux-connect/)
