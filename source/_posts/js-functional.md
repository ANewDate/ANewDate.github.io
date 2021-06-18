---
title: 读书笔记-JS函数式编程
date: 2020-12-11 09:45:35
updated:
tags: book
categories: 
- web前端
---
近状, 工作几月, 经验几天。 重复处理我司旧项目CRM业务需求, 也曾考虑重构, 奈何条件不允许。遂读书, 寻求突破。感觉通篇在流水, 但希望通过这种方式能让自己对某些思想理解更加深刻.

```md
函数式编程的哲学就是假定副作用是造成不正当行为的主要原因。
只要是跟函数外部环境发生的交互就都是副作用。
我们不能消除副作用, 但是我们可以控制副作用, 通过函数式编程
```
## 1. 函数真的没什么特殊
```md
函数真没什么特殊的，你可以像对待任何其他数据类型一样对待它们——把它们存在数组里，当作参数传递，赋值给变量...等等
```
这个没什么好说的, 高阶函数, 贴合平时使用的回调函数, 给函数传递函数参数, 函数返回函数等场景
但是, **传递函数的同时, 应当注重函数的 "打开方式"**
```js
// 看起来酷炫, 实则无用
var getServerStuff = cb => ajaxCall(json => cb(json));

var getServerStuff = ajaxCall
```
## 2. 纯函数:减少函数副作用
```md
纯函数是这样一种函数，即相同的输入，永远会得到相同的输出，而且*没有任何可观察的副作用*。
```
**简单理解, 不被外界影响, 输入与输出之间唯一映射**
下面两段代码的区别在于: 比较的minimum值是否可被系统改变。
```js
// 不纯的函数
var minimum = 21;
var checkAge = function(age) {return age >= minimum;};
```
```js
// 纯函数
var checkAge = function(age) {var minimum = 21;return age >= minimum;};
```
**编写纯函数应该明确依赖, 即从形参便可看出此函数的依赖**, 即传入的外部状态应当在形参中定义
```js
// 不纯的
var signUp = function(attrs) {
  var user = saveUser(attrs);  welcomeUser(user);
};
var saveUser = function(attrs) {
  var user = Db.save(attrs);    
};
var welcomeUser = function(user) {    
  Email(user, ...);    
};
// 纯的
var signUp = function(Db, Email, attrs) {
  return function() {
    var user = saveUser(Db, attrs);    
    welcomeUser(Email, user);  
  };
};
var saveUser = function(Db, attrs) { ...};
var welcomeUser = function(Email, user) {    ...};
```

## 3. curry and compose
### 3.1 curry
```md
只传递给函数一部分参数来调用它，让它返回一个函数去处理剩下的参数。
```
这里推荐一个大神的博客, 便于理解[函数柯里化](https://github.com/mqyqingfeng/Blog/issues/42), 函数柯里化, 用习惯了就很依赖, 但是我司项目没有身影
*这里把要操作的数据放到最后一个参数*, 利于`compose`
```js
var curry = require('lodash').curry
var match = curry(function(what, str) {
  return str.match(what)
})
var replace = curry(function(what, replacement, str) {
  return str.replace(what, replacement)
})
var findSpace = match(/\s+/g) // 通过传入不同的正则得到不同的匹配函数
var map = curry(function(f, ary) {
  return ary.map(f)
})
```

### 3.2 compose
```js
// 代码从右往左进行
var compose = function(f, g) {
  return function(x) {
    return f(g(x))
  }
}
```
**pointfree模式: 不必告诉我你是谁(数据)**
```md
pointfree: 函数无须提及将要操作的数据是什么样的
```
```js
  var toUpperCase = function(x) { return x.toUpperCase(); };
  // 非 pointfree，因为提到了数据：word
  var snakeCase = function (word) {
    return word.toLowerCase().replace(/\s+/ig, '_');
  };
  // pointfree
  var snakeCase = compose(replace(/\s+/ig, '_'), toLowerCase);
```
习惯了命令式函数, 再看这种声明式函数的写法这个反差还是比较大的，但是总的来着，这种函数写法更加优雅，易读性更高，当然这是建立在函数命名简单易懂的前提下(当然，有函数签名更好)

**compose函数的debug方法: trace函数**
<font size=2>通过`trace`函数可以知道数据在传输过程中的值</font>
```js
var trace = curry(function(tag, x){console.log(tag, x);return x;});
// after split [ 'The', 'world', 'is', 'a', 'vampire' ]
var dasherize = compose(join('-'),map(toLower),trace("after split"),split(' '),replace(/\s{2,}/ig, ' '));
```

### 3.3 线上例子
[获取🐱的图片显示在网页上](https://codesandbox.io/s/js-hanshushibiancheng-3qb2z?file=/src/index.js)
**compose+ curry**的组合感觉是是函数式编程的精粹吧, 可以习惯使用这种写法


## 4. Hindley-Milner类型签名
```md
简单易读的函数注释, 暴露函数的行为与目的
```
```js
// sayHi :: String -> String 这个String可以随便命名表示某种类型的值
var sayHi = (str) => {alert(str)};
// id :: a -> a
var id = function(x) {return x};
```
```js
// map :: (a -> b) -> [a] -> b
var map = curry(function(f, xs) {
  return xs.map(f)
})

// reduce :: (b -> a -> b) -> b -> [a] -> b
var reduce = curry(function(f, x, xs) {
  return xs.reduce(f, x)
})
```
```md
**map函数类型解读:**
  - map函数的第一个参数f是一个函数, 接收类型a的参数(此数据来源于map函数的第二个参数a数组)，返回类型b
  - map函数的第二个参数是一个a类型数组, 此参数会传递给函数f作为参数
  - map函数返回的是 b 类型的数据

**reduce函数类型解读:**
  - 第一个参数是一个函数, 此函数接收两个参数, b,a, 这两个参数来源于reduce函数的第二及第三个参数
  - 返回b
```

## 5. END
此书后面还提到了functor之类的概念, 个人觉得现阶段难以施展.
一些收获: 个人觉得 Hindley-Milner类型签名真的是此书给我最大的收获, 当然函数柯里化也是JS中的黑魔法 
**读过的书也许会忘掉, 但是收获的却是思想**, 努力吧, 打工人


