---
layout: post
title: "Higher Order Reducers conception"
date: 2017-10-26 9:55:00 +0300
categories: redux, reducer
comments: true
---
Sometimes it’s useful to extend reducer logic or have multiple instances of the same reducers with some minor differences between each other.

These kinds of issues could be easily resolved using Higher Order Reducers.
<!--more-->
<img src="https://images.unsplash.com/photo-1498642145410-c61aa8483bcb?dpr=1&auto=format&fit=crop&w=2980&q=60&cs=tinysrgb">
To understand Higher Order Reducers conception properly, it’s important to understand the main conception  -  [Higher Order Functions](//en.wikipedia.org/wiki/Higher-order_function).

Higher Order Reducer is like Higher Order Function except these differences  -  it **optionally** takes a reducer function as an argument and **always** returns a new reducer function as a result.

---

Let’s imagine the situation, where you need to handle API actions in your application. In that case, you need to create a reducer which handles `REQUEST` , `SUCCESS` and `FAILURE` actions for every single API call. That leads to a huge code duplication for the big apps.

We can create a Higher Order Reducer to solve that problem:
```javascript
const createAsyncReducer = (constants) => (state, action) => {
  switch (action.type) {
    case constants[0]:
      return {
        ...state,
        isFetching: true
      }
    case constants[1]:
      return {
        ...state,
        isFetching: false,
        data: action.payload,
        error: void 0,
      }
    case constants[2]:
      return {
        ...state,
        isFetching: false,
        error: action.payload
      }
    default: return state;
  }
}
```
Basicaly, we are creating a factory function, which accepts `constants` array and returns new reducer, which changes the state, when given constants are dispatched.

With it’s help, we can create multiple instances of the reducers, which are dealing with different constants:
```javascript
const userReducer = createAsyncReducer([
  "USER_REQUEST",
  "USER_SUCCESS",
  "USER_FAILURE"
]);

const todosReducer = createAsyncReducer([
  "TODOS_REQUEST",
  "TODOS_SUCCESS",
  "TODOS_FAILURE"
]);
```
Everything is fine up to the moment, when we want to handle some custom actions in the reducers. For example we want to update `todosReducer` state after `ADD_TODO` action was fired.

To achieve more flexibility, we can create another Higher Order Reducer, for handling all initial reducer actions and custom actions as well:
```javascript
const withAddTodo = reducer => (state, action) => {
  const nextState = reducer(state, action);
  switch (action.type) {
    case "ADD_TODO":
      return {
        ...nextState,
        data: [...nextState.data, action.payload]
      };
    default:
      return nextState;
  }
};

const customReducer = withAddTodo(todosReducer);
```

When we have lot's of that kind Higher Order Reducers, we can use well-known functional programming technique, called *function composition*, in order to compose them all together and apply to the needed reducer:

```javascript
const customReducer = compose(
  withAddTodo,
  withRemoveTodo,
  withSomethingElse
)(todosReducer);
```

---

Using these patterns, you can avoid code duplication in the reducers by creating reducer factories and write more clean and reusable code.