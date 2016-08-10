---
layout: post
title: "Redux saga as alternative way managing async flow in Redux applications"
date: 2016-08-02 21:35:00 +0300
categories: react, redux, redux-saga, saga
comments: true
---
When your application grows up, it becomes difficult to manage side-effects. In my opinion, using `redux-thunk` package is good for small applications, where your side effects are mostly CRUD operations with remote API. One of possible solutions to simplify managing side effects is `redux-saga` package.
<!--more-->

Redux-saga built using es6-generators. If you are not familiar with them, first I suggest you to read about it at this [link](http://gajus.com/blog/2/the-definitive-guide-to-the-javascript-generators).

This package provides convinient way to manage your side effects in redux applications. Along with actions, you should create `sagas`. They are generator functions, where you are will be handling your async flow.

### Installation

First, we should assign this library with our application:

```javascript
// store.js
import { createStore, applyMiddleware } from 'redux';
import rootReducer from './reducers';
import createSagaMiddleware from 'redux-saga';
// Importing not yet existing saga. This file will be created later.
import rootSaga from './sagas';

// Creating saga middleware instance.
const sagaMiddleware = createSagaMiddleware();

export default createStore(
  rootReducer,
  null,
  applyMiddleware(
    sagaMiddleware
  )
);
```

### Usage example

Below, I'll show you simple async example, which can be replacement for `redux-thunk` package.

Let's create our first saga:

```javascript
// sagas/index.js
import { call, put } from 'redux-saga/effects';

function* fetchTodos() {
  try {
    const result = yield call(api.fetchTodos);
    yield put({
      type: 'FETCH_TODOS_SUCCESS',
      todos: result
    });
  } catch(err) {
    yield put({
      type: 'FETCH_TODOS_FAILURE',
      message: err.message
    });
  }
}
```

There are some aspects above, that need to be explained.

To instruct middleware, what it should do, we need `yield` to it some effects. In this example, we are using two effects - `call` and `put`:

 - As first argument `call` effect accepts function, which should return `Promise`. By yielding this effect, we are telling to middleware, that it should call this promise and assign it result to `result` variable.
 - After promise will be resolved, we instructing middleware to dispatch an action to redux store, for this kind of stuff is responsible `put` efffect.

In our example, `api.fetchTodos` function should resolve promise, when response was succeeded and reject it when something goes wrong. This way we will fall into catch block, when response will be failed and dispatch corresponding action.

---

Next, we need to assign our created saga with corresponding action:

```javascript
// sagas/index.js
import { takeEvery } from 'redux-saga';

function* watchFetchTodos() {
  yield* takeEvery('FETCH_TODOS_REQUEST', fetchTodos);
}
```

This means, that every time we will be dispathching `FETCH_TODOS_REQUEST` action, would be executed `fetchTodos` saga.

And lastly we need to export array of watchers:

```javascript
// sagas/index.js
export default function* () {
  yield [
    watchFetchTodos()
  ];
};
```
