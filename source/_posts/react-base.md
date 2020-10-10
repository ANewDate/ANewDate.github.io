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
  ### 3. 实现: 简单路由拦截
  - 版本: react-router-dom 5.2.0
  - 与vue的区别: 没有路由拦截钩子, 听说github上说就是作者希望react-router是灵活的...
  - 此处为超简单的实现: 没有token则跳Login页
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
  ![结果图](/img/react/menu.png)
  - 数据来源: 本地Mock
  ```js
  export default Mock.mock('/menu', 'post', {
    isSuccess: true,
    message: 'success',
    records: [
      {
        title: '用户中心',
        key: "sub1",
        children: [
          { title: '组织管理', key: '/user/org', component: 'views/Org', icon: "a" },
          { title: '岗位管理', key: '/user/station', component: 'views/Station', icon: "b" },
          { title: '用户管理', key: '/user/user', component: 'views/User', icon: "c" },
        ],
      },
      {
        title: '权限管理',
        key: "sub2",
        children: [
          { title: '菜单配置', key: '/auth/menu', component: 'views/Menu', icon: "a" },
          { title: '角色管理', key: '/auth/role', component: 'views/Role', icon: "b" },
        ],
      },
    ],
  })
  ```
  - 渲染菜单 及 路由
  [代码图](/img/react/react-menu.png)
  ### 5. 初识: redux + thunk 
  redux的[文档](https://www.redux.org.cn/), WTF, 这什么鬼
  #### 创建store
  ```js
  import {createStore, applyMiddleware } from "redux"
  import thunk from "redux-thunk"
  import reducer from "./reducer"
  // 此处reducer与store关联起来
  const store = createStore(reducer, applyMiddleware(thunk) )
  export default store
  ```
    **thunk中间件能使store.dispatch的参数不止是对象, 可以是一个函数(dispatch)=>{}**
  #### store与UI的连接
    API: `connect`
    **connect会往组件props对象注入方法及state, withRouter会注入history对象**
    - 同步action
      ```jsx
      const mapStateToProps = () => ({})
      const mapDispatchToProps = (dispatch) => ({
        handleSetTenant: (value) => dispatch(setTenant(value)),
        handleSetToken: (value) => dispatch(setToken(value)),
      })
      export default withRouter(connect(mapStateToProps, mapDispatchToProps)(Login))
      ```
    - 异步action
      *注意*: 至少触发两次action, 数据拉取中; 数据拉取成功
      ```js
      // dispatch(func) dispatch的参数换成函数
      dispatch(getTodoList()).then(() => {
        console.log('store.getState(): ', store.getState());
      })
      //action
      function getTodoList () {
        return dispatch => {
          dispatch({type: actionType.FETCH_TODO_LIST})
          return axios.get("/todo").then(res => {
            console.log('res: ', res)
            dispatch({type: actionType.TODO_LIST_RECEIVE, todoList: res.data.records})
          })
        }
      }
      ```
## 生命周期
  ![生命周期](/img/react/circle.png)
## END
  对于react有初步的了解, 对于react的开发, 正式环境区分, 生命周期, 高阶组件及mobX状态等了解甚少
  