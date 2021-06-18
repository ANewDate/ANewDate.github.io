---
title: react入坑指南
date: 2020-10-10 19:19:44
tags: react
categories: 
- web前端
---
## 为什么入坑
  无他, 给自己充充电 (*之前有人问会不会`react + electron`, 汗颜: 不会*)

## react诞生背景
  网上说facebook开发团队开发facebook首页状态栏, 要实时显示当前好友请求, 消息列表、状态列表; 但是当功能上线的时候会出现: 如果有一条新的消息出现不能够及时的刷新显示的数字, 但点开一条新的消息数字又不能够立即的修改
  于是乎, react为了解决**状态分散难以追踪和维护**的问题(没有接触过传统MVC的项目, 个人是体验不到这种痛苦的)
## react v16+ 的学习
  ### 1. 脚手架创建项目
  ```shell
    npx create-react-app react-base
  ```
  *和vue-cli的一点区别: 没有过多的配置项, 只是简单的生成一个项目*
  ### 2. 样式: 自带样式作用域
  `*.module.css`的命令会自动打包为一个css对象,使用时引入像对象一样使用
  ```jsx
  import loginStyle from "./login.module.css"
  function login() {
    return (
      <div className={`${loginStyle.demo}`}>login</div>
    )
  }
  ```
  动态类名可以结合插件`classnames`使用
  ### 3. 实现: 简单路由拦截跳转
  版本: react-router-dom 5.2.0
  与vue的区别: 没有路由拦截钩子, 听说github上说就是作者希望react-router是灵活的...(这就需要自己封装高阶组件了)
  此处为超简单的实现: 没有token则跳Login页
    ```jsx
      const routes = [
        {path: '/login', component: Login},
        {path: '/', component: Home},
      ]

      function MyRouter (route) {
        return <Route path={route.path} render={() => <route.component {...route} routes={route.routes} /> } />
      }
      function hasToken () {
        return !!db.getValue('Token')
      }
      export default function () {
        console.log(' hasToken (): ',  hasToken ());
        return (
          <Router>
            <Switch>
              {
                routes.map((route, i) => {
                  return <MyRouter {...route} key={i} />
                })
              }
            </Switch>
              {
                !hasToken() ?  <Route path="*" render={() => (<Redirect to={{pathname: "/login"}} />) } /> : ''
              }
          </Router>
        )
      }
    ```
  ### 4. 实现: 动态渲染菜单
  [代码图](/img/react/menu.png)
  参照文档: [https://reactjs.org/docs/jsx-in-depth.html#choosing-the-type-at-runtime](https://reactjs.org/docs/jsx-in-depth.html#choosing-the-type-at-runtime)
  数据来源: 本地Mock
  ```jsx
  export default Mock.mock('/menu', 'post', {
    isSuccess: true,
    message: 'success',
    records: [
      {
        title: '用户中心',
        key: "sub1",
        children: [
          { title: '组织管理', key: '/user/org', component: 'Org', icon: "a" },
          { title: '岗位管理', key: '/user/station', component: 'Station', icon: "b" },
          { title: '用户管理', key: '/user/user', component: 'User', icon: "c" },
        ],
      },
      {
        title: '权限管理',
        key: "sub2",
        children: [
          { title: '菜单配置', key: '/auth/menu', component: 'Menu', icon: "a" },
          { title: '角色管理', key: '/auth/role', component: 'Role', icon: "b" },
        ],
      },
    ],
  })
  ```
  #### 问题:
  **Can't not find Module;**
  `<Route component={() => import('../views' + route.component))}`
  `import`引包的问题, 没找到一个好的解决办法
  **component should be a class/function;**
  `<Route component={route.component}`
  `component={Demo}` 会被编译成React.createElement(Demo,{})这种形式, 故直接返回一个路径字符串会报错，直接返回组件名也会报错
  **Suspense...**
  `<Route component={React.lazy(() => import(route.component))}`
  lazy不能单独使用, 应该配合Suspense使用
  ### 5. 初识: redux
  先上一张百度百科的图
  ![redux](/img/react/redux.jpg)
  与vuex的对比:
  1. 同为状态管理器, `vuex`为vue量身定做; `redux`就是一个纯JS库, 可以用在其他框架(我自己是没去试过怎么用的)
  2. vuex的同步可以使用`commit(mutation)`, 也可以用`dispatch(action)`; react同步异步都是用dispatch触发, 同步异步传参不同(这针对现在学的`thunk`)
  3. 刷新都会丢失,需要做数据持久化(似乎两个的思想都是来自与`flux`)
  4. 解决业务场景问题: 组件之间状态共享 
    
  
  redux的使用步骤:

  1. 创建`store`, 关联`reducer`
  2. `store.state` 与 `React Component`连接(通过`Connect` API向`Component.prop`注入)
  3. `React Component` 通过 `store.dispatch(action)` 调用 `reducer` 改变 `store.state`
  ```js
  // 创建store
  import { createStore } from 'redux'
  import root from './reducer'
  const store = createStore(root)
  export default store

  // reducer
  function setToken(state = '', action) {
    switch (action.type) {
      case actionType.SET_TOKEN:
        return action.payload
      default: {
        return state
      }
    }
  }
  export default combineReducers({tenant: setToken})

  // store 与 React Component 关联，根组件注入(略)
  // action及创建函数
  export const SET_TOKEN = "SET_TOKEN"
  export function setToken(value = '') {
    return {
      type: actionType.SET_TOKEN,
      payload: value,
    }
  }
  // store与UI关联
  function Login({handleSetToken}){
    // 调用handleSetToken即可更新state
  }
  const mapStateToProps = () => ({})
  const mapDispatchToProps = (dispatch) => ({
    handleSetToken: (value) => dispatch(setToken(value)),
  })
  export default withRouter(connect(mapStateToProps, mapDispatchToProps)(Login))
  ```
  **现阶段存在的一些问题:** 
    -  刷新页面, store的数据会丢失; 
    -  现在store.dispatch({type})是同步的, 如果异步, 该如何处理. 

  由此, 引入react的一些中间件
  ### redux-thunk: 异步action处理
  **`redux-thunk`帮我们做了什么**
  在没有引入`redux-thunk`之前, `store.dispatch()`的参数只能是一个Object, 引入这个库之后, `store.dispatch()`的参数可以是**Object || Function**
  为什么在此阶段处理异步操作:
  * 根据`redux`的设计原则, reducer应该为纯函数, 不涉及动态数据
  * UI 中触发更改state操作是通过 `store.dispatch()` 更改  
   

  ```js
  createStore(reducer, applyMiddleware(thunk))    // 好奇applyMiddleware是如何实现中间件功能的, 现在暂且先会用吧

  // action创建函数
  export function getTodoList() {
    return (dispatch) => {
      dispatch({ type: actionType.FETCH_TODO_LIST })
      return axios.get('/todo').then((res) => {
        dispatch({
          type: actionType.TODO_LIST_RECEIVE,
          todoList: res.data.records,
        })
      }).catch(err => {
        dispatch({
          type: actionType.TODO_LIST_RECEIVE
        })
      })
    }
  }
  // 其他使用和同步无异
  ```
  ### redux-persist: redux数据持久化
  [redux-persist文档](https://github.com/rt2zz/redux-persist)
  `redux-persist`帮我们做的事: 把state存储到浏览器storage
  ```js
  // persist.js store的创建有所改变
  import { createStore, applyMiddleware } from 'redux'
  import { persistStore, persistReducer } from 'redux-persist'
  import storage from 'redux-persist/lib/storage' // defaults to localStorage for web

  import rootReducer from './reducer'
  import thunk from 'redux-thunk'
  const persistConfig = {
    key: 'root',
    storage,
  }
  const persistedReducer = persistReducer(persistConfig, rootReducer)
  let store = createStore(persistedReducer, applyMiddleware(thunk))
  let persistor = persistStore(store)
  export {
    store,
    persistor
  }

  // 根组件配置
  import { Provider } from 'react-redux'
  import {persistor, store }  from "./store/persist.js"
  import { PersistGate } from 'redux-persist/es/integration/react';

  ReactDOM.render(
    <Provider store={store}>
        <PersistGate persistor={persistor}>
          <App />
        </PersistGate>
    </Provider>,
    document.getElementById('root')
  )
  ```
## 写在最后
  对于react有初步的了解, 但对于react的开发, 正式环境区分, 生命周期, 高阶组件及mobX状态等了解甚少
  后续一个方向, 了解react中间件的设计, `applyMiddleware`的实现, 实现一个不对数据做任何处理的中间件 
  有这种念头是因为, `react-redux`保证纯洁性的后果是, 所有脏苦累的活都交给中间件去做了
  
  最后 希望后面自己能对工作中的技术有一个阶段性的总结, 关于工作用到的技术, 关于个人的提升, 关于线下读的书...
  