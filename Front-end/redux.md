## Redux的原理和应用
Redux简化了组件之间的通信方式。
Redux作为应用程序中唯一数据变化的来源。
Redux的可预测性：state的变化一定是由某个action触发产生的，变化的state是新的state。
Redux通过纯函数更新store，本质上是产生了新的store，纯函数就是reducer。

### Redux reducer
store通过`dispatch`函数把state变化发送给dispatcher，调用`reducer`函数获得新的state。
所有的action可以被所有的reducer接收到，然后判断`acton.type`决定是否处理。

### Redux in React
在React中使用redux需要引入react-redux包，然后通过connect方法生成一个高阶组件，包含了访问store的功能。

在初始化时需要通过定义`mapStateToProps`方法，将store中的感兴趣的state字段返回给组件的props；通过定义`mapDispatchToProps`方法，使用store的dispatch方法生成相应的dispatcher函数传递给组件的props，这样组件可以在特定的时刻调用dispatch去分发事件。

### 一个计数器的例子
```javascript
import React from 'react';
import { bindActionCreators, combineReducers } from 'redux';
import { configureStore } from '@reduxjs/toolkit';
import { Provider, connect } from 'react-redux';

const initialState = { count: 0 };

// Reducer
const counterReducer = (state = initialState, action) => {
    switch(action.type) {
        case "ADD_ONE":
            return { count: state.count + 1 };
        case "MINUS_ONE":
            return { count: state.count - 1 };
        case "CUSTOM_COUNT":
            return { count: state.count + action.payload.count };
        default:
            break;
    }
    return state;
};

// Empty reducer
const printReducer = (state = {}) => state;

// Create store
const store = configureStore(
    {
        reducer: combineReducers({counterReducer, printReducer})
    }
);

// Actions
function addOne() {
    return { type: "ADD_ONE" };
}

function minusOne() {
    return { type: "MINUS_ONE" };
}

function customCount(count) {
    return { type: "CUSTOM_COUNT", payload: { count }};
}

// Dispatch actions
//const addOneDispatcher = bindActionCreators(addOne, store.dispatch);

store.subscribe(() => {
    console.log(store.getState().counterReducer.count);
});

//addOneDispatcher();
store.dispatch(minusOne());
store.dispatch(customCount(10));

const ReduxCounter = (props) => {
    const { count, plusOne, minusOne } = props;
    return (
        <div className="counter">
            <button onClick={minusOne}>-</button>
                <span style={{ display: "inline-block", margin: "0 10px" }}>
                    {count}
                </span>
            <button onClick={plusOne}>+</button>
        </div>
    );
}

function mapStateToProps(state) {
    return {
        count: state.count
    };
}

function mapDispatchToProps(dispatch) {
    return bindActionCreators({
        plusOne: addOne,
        minusOne: minusOne
    }, dispatch);
}

const ConnectedReduxCounter = connect(mapStateToProps, mapDispatchToProps)(ReduxCounter);
const ReactReduxCounter = () => {
    return (
        <Provider store={store}>
            <ConnectedReduxCounter />
        </Provider>
    );
};
export default ReactReduxCounter;
```

### Redux异步请求
对于异步请求的action，Redux在dipatcher内部会有一个middleware层，用于截获action，并发起异步调用并获得结果，根据结果再dispatch新的action到reducer。

### Redux数据不可变
Redux中的store的数据是不可变的，每次action更新state，都是重新生成新的state对象。这样有利于更新前后的比较（只需比较引用即可），可以有利于调试。

### Redux Toolkit的使用
#### Redux Slice
一个slice是指多个reducer逻辑的切片，代表某个应用的某个方面，可以配置在store的reducer属性下，如：
```javascript
import { configureStore } from '@reduxjs/toolkit'
import usersReducer from '../features/users/usersSlice'
import postsReducer from '../features/posts/postsSlice'
import commentsReducer from '../features/comments/commentsSlice'

export default configureStore({
  reducer: {
    users: usersReducer,
    posts: postsReducer,
    comments: commentsReducer
  }
})
```
这里`configureStore`方法内部会调用`combineReducers`方法，将多个slice合并为一个reducer。

Redux Toolkit 有一个名为 `createSlice` 的函数，它负责生成 `action` 类型字符串、`action creator` 函数和 action 对象的工作。如：
```javascript
export const counterSlice = createSlice({
  name: 'counter',
  initialState,
  // The `reducers` field lets us define reducers and generate associated actions
  reducers: {
    increment: (state) => {
      // Redux Toolkit allows us to write "mutating" logic in reducers. It
      // doesn't actually mutate the state because it uses the Immer library,
      // which detects changes to a "draft state" and produces a brand new
      // immutable state based off those changes
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
    // Use the PayloadAction type to declare the contents of `action.payload`
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    },
  },
});
```
这里`name`属性和reducer的名字（increment等）会自动拼接并生成action的名字（`counter/increment`)。如注释所说，Redux Toolkit允许我们写“可变”的逻辑，实际上内部使用了Immer库，监测state的变化并生成新的state对象返回，保持了不可变的特性。

同时，createSlice还生成一个reducer函数，响应上述的action，如：
```javascript
const newState = counterSlice.reducer(
  { value: 10 },
  counterSlice.actions.increment() //此处用于获取action对象，也是自动生成。
)
console.log(newState)
```

