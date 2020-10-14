---
title: react入坑指南
date: 2020-10-10 19:19:44
tags: react
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
  redux的使用理解:
  1. 创建store, 关联reducer
  2. store.state 与 React Component连接(通过Connect API向Component.prop注入)
  3. React Component 通过 store.dispatch(action) 调用 reducer 改变 store.state
  ```js
  // 创建store
  import { createStore } from 'redux'
  import root from './reducer'
  const store = createStore(root)
  export default store

  // store 与 React Component 关联，根组件注入
  ```
## 生命周期
  ![生命周期](/img/react/circle.png)
## END
  对于react有初步的了解, 对于react的开发, 正式环境区分, 生命周期, 高阶组件及mobX状态等了解甚少
  