---
title: 合理编写JavaScript
date: 2017-07-28 17:45:48
tags:
- javascript
- coding
categories:
- javascript
- 基础
---

### 类型
#### 原始值
存取直接作用于它自身。

- **string**
- **number**
- **boolean**
- **null**
- **undefined**

```javascript
var foo = 1;
var bar = foo;
bar = 9;

console.log(foo, bar); // => 1, 9
```
<!-- more -->

#### 复杂类型
存取时作用于它自身值的引用。

- **object**
- **array**
- **function**

```javascript
var foo = [1, 2];
var bar = foo;
bar[0] = 9;

console.log(foo[0], bar[0]); // => 9, 9
```


### 单引号

```javascript
// bad
var name = "Bob Parr";

// good
var name = 'Bob Parr';
```



### 函数

#### 函数表达式

```javascript
// 匿名函数表达式
var anonymous = function() {
  return true;
};

// 命名函数表达式
var named = function named() {
  return true;
};

// 立即调用的函数表达式（IIFE）
(function () {
  console.log('Welcome to the Internet. Please follow me.');
}());
```

> 永远不要在一个非函数代码块（if、while 等）中声明一个函数，把那个函数赋给一个变量。浏览器允许你这么做，但它们的解析表现不一致。

```javascript
// bad
if (currentUser) {
  function test() {
    console.log('Nope.');
  }
}

// good
var test;
if (currentUser) {
  test = function test() {
    console.log('Yup.');
  };
}
```



### 数组
#### 拷贝
拷贝数组时，使用 Array#slice

```javascript
var len = items.length;
var itemsCopy = [];
var i;

// bad
for (i = 0; i < len; i++) {
  itemsCopy[i] = items[i];
}

// good
itemsCopy = items.slice();
```



### 对象
#### 访问属性
通过变量访问属性时使用中括号 []
```javascript
var luke = {
  jedi: true,
  age: 28
};

function getProp(prop) {
  return luke[prop];
}

var isJedi = getProp('jedi');
```



### 判断

#### 比较运算符 & 等号

> 优先使用 === 和 !== 而不是 == 和 !=.
>
> 条件表达式例如 if 语句通过抽象方法 ToBoolean 强制计算它们的表达式并且总是遵守下面的规则：
>

- 对象 被计算为 true
- Undefined 被计算为 false
- Null 被计算为 false
- 布尔值 被计算为 布尔的值​
- 数字 如果是 +0、-0 或 NaN 被计算为 false，否则为 true 
- 字符串 如果是空字符串 '' 被计算为 false，否则为 true 






### 空格

#### 缩进
使用 2 个空格作为缩进。

```javascript
// bad
function () {
∙∙∙∙var name;
}

// bad
function () {
∙var name;
}

// good
function () {
∙∙var name;
}
```



### 提升
#### 变量声明
变量声明会提升至作用域顶部，但赋值不会。

```javascript
// 我们知道这样不能正常工作（假设这里没有名为 notDefined 的全局变量）
function example() {
  console.log(notDefined); // => throws a ReferenceError
}
```

```javascript
// 但由于变量声明提升的原因，在一个变量引用后再创建它的变量声明将可以正常工作。
// 注：变量赋值为 true 不会提升。
function example() {
  console.log(declaredButNotAssigned); // => undefined
  var declaredButNotAssigned = true;
}

// 解释器会把变量声明提升到作用域顶部，意味着我们的例子将被重写成：
function example() {
  var declaredButNotAssigned;
  console.log(declaredButNotAssigned); // => undefined
  declaredButNotAssigned = true;
}
```

#### 匿名函数表达式
匿名函数表达式会提升它们的变量名，但不会提升函数的赋值。

```javascript
function example() {
  console.log(anonymous); // => undefined
  anonymous(); // => TypeError anonymous is not a function
  var anonymous = function () {
    console.log('anonymous function expression');
  };
}
```

#### 命名函数表达式
命名函数表达式会提升变量名，但不会提升函数名或函数体。

```javascript
function example() {
  console.log(named); // => undefined
  named(); // => TypeError named is not a function
  superPower(); // => ReferenceError superPower is not defined
  var named = function superPower() {
    console.log('Flying');
  };
}
```



```javascript
// 当函数名跟变量名一样时，表现也是如此。
function example() {
  console.log(named); // => undefined
  named(); // => TypeError named is not a function
  var named = function named() {
    console.log('named');
  }
}
```

#### 函数声明
函数声明提升它们的名字和函数体。

```javascript
function example() {
  superPower(); // => Flying
  function superPower() {
    console.log('Flying');
  }
}
```



### 逗号

> 额外的行末逗号：不需要。这样做会在 IE6/7 和 IE9 怪异模式下引起问题。同样，多余的逗号在某些 ES3 的实现里会增加数组的长度。在 ES5 中已经澄清了

```javascript
// bad
var hero = {
  firstName: 'Kevin',
  lastName: 'Flynn',
};

var heroes = [
  'Batman',
  'Superman',
];

// good
var hero = {
  firstName: 'Kevin',
  lastName: 'Flynn'
};

var heroes = [
  'Batman',
  'Superman'
];
```



### 分号

使用分号。

```javascript
;(function () {
  var name = 'Skywalker';
  return name;
})();
```



### 构造函数
#### 为原型追加方法
给对象原型分配方法，而不是使用一个新对象覆盖原型。覆盖原型将导致继承出现问题：重设原型将覆盖原有原型！

```javascirpt
function Jedi() {
  console.log('new jedi');
}

// bad
Jedi.prototype = {
  fight: function fight() {
    console.log('fighting');
  },

  block: function block() {
    console.log('blocking');
  }
};

// good
Jedi.prototype.fight = function fight() {
  console.log('fighting');
};

Jedi.prototype.block = function block() {
  console.log('blocking');
};
```



#### 方法链 
方法可以返回 this 来实现方法链式使用。

```javascript
// bad
Jedi.prototype.jump = function jump() {
  this.jumping = true;
  return true;
};

Jedi.prototype.setHeight = function setHeight(height) {
  this.height = height;
};

var luke = new Jedi();
luke.jump(); // => true
luke.setHeight(20); // => undefined

// good
Jedi.prototype.jump = function jump() {
  this.jumping = true;
  return this;
};

Jedi.prototype.setHeight = function setHeight(height) {
  this.height = height;
  return this;
};

var luke = new Jedi();

luke.jump()
  .setHeight(20);
```
#### 变量名
使用 $ 作为存储 jQuery 对象的变量名前缀。
```javascript
// bad
var sidebar = $('.sidebar');

// good
var $sidebar = $('.sidebar');
```