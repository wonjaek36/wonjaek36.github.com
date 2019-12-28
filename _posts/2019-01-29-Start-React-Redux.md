---
layout: post
title: "Redux Example Code with React"
date: 2019-01-29 15:36 +0900
category: [framework, react]
tags: [react, redux, javascript]
---

### Environment
- Node v8.12.0
- npm v6.4.1
- yarn 1.9.4

### Create React Page
{% highlight shell %}
$ npx create-react-app start-react
$ cd start-react
$ yarn start # npm start
{% endhighlight %}

### Install Redux
{% highlight shell %}
$ yarn add --save redux react-redux
# npm install --save redux react-redux
{% endhighlight %}

### Redux 디렉토리 구조
{% highlight shell %}
src
├── actions
│       └── index.js
├── components
│       └── App.js
├── index.js
└── reducers
        └── index.js
{% endhighlight %}

### Action index.js example
{% highlight javascript %}
export const SET_ALERT = 'SET_ALERT';
export const DISABLE_ALERT = 'DISABLE_ALERT';

export function setAlert = (message) => ({type: SET_ALERT, message});
export function disableAlert = () => ({type: DISABLE_ALERT, message: ""});
{% endhighlight javascript %}

### Reducer index.js example
{% highlight javascript %}
import { SET_ALERT, DISABLE_ALERT } from './../actions';
import { combineReducers } from 'redux';

const initialState = {
	message: ""
}

const messageReducer = (state = initialState, action) => {
	
	switch(action.type) {
		case SET_ALERT:
			return {...state, message: action.message};
		case DISABLE_ALERT:
			return {...state, message: action.message};
		default:
			return state;
	}
}

const reducerApp = combineReducers({
	messageReducer
}) // can add multiple reducer

export default exampleReducer;
{% endhighlight javascript %}

### Component에서 State 및 Dispatch(Action) 사용
{% highlight javascript %}
import React from 'react';
import { setAlert, disableAlert } from './../actions';

class example extends React.Component {
	...
}

const mapStateToProps = (state) => {
	return {
		message: state.messageReducer.message
	}
}

const mapDispatchToProps = (dispatch) => {
	return {
		onAlert: (message) => dispatch(setAlert(message)),
		offAlert: () => dispatch(disableAlert())
	}
}
{% endhighlight javascript %}

[Official Document](https://redux.js.org/basics/basic-tutorial)