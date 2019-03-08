## 面试题

### 目录

- [JavaScript 实现继承的几种方式](#实现继承的几种方式)
- [JavaScript 创建对象的九种方法](#创建对象的九种方法)
- [算法题](#算法题)
  - [完美数](#完美数)
- [变量提升](#变量提升)
- [深拷贝和浅拷贝](#深拷贝和浅拷贝)
- [数组去重](#数组去重)
- [布局](#布局)
  - [两列等高布局](#两列等高布局)
  - [左侧不定宽，右侧自适应](#左侧不定宽，右侧自适应)
- [水平垂直居中](#水平垂直居中)

  - [多行文本垂直居中](#多行文本垂直居中)
  - [在某个盒子中居中](#在某个盒子中居中)

#### [实现继承的几种方式](https://juejin.im/entry/58dfbe0361ff4b006b166388 "JavaScript实现继承的几种方式")

1. 类式继承

```javascript
// 声明父类
function Animal() {
  this.name = "animal"

  this.type = ["pig", "cat"]
}
// 为父类添加共有方法

Animal.prototype.greet = function(sound) {
  console.log(sound)
}

// 声明子类
function Dog() {
  this.name = "dog"
}
// 继承父类
console.log(new Animal()) //new Animl运行一次，导致所有的Animal里的属性只能是唯一值
Dog.prototype = new Animal()
var dog = new Dog()
dog.greet("汪汪") //  "汪汪"
console.log(dog.type) // ["pig", "cat"]
//原理说明：在实例化一个类时，新创建的对象复制了父类的构造函数内的属性与方法并且将原型__proto__指向了父类的原型对象，这样就拥有了父类的原型对象上的属性与方法。
```

2. 构造函数继承

```javascript
// 声明父类

function Animal(color) {
  this.name = "animal"

  this.type = ["pig", "cat"]

  this.color = color
}

// 添加共有方法

Animal.prototype.greet = function(sound) {
  console.log(sound)
}

// 声明子类

function Dog(color) {
  Animal.apply(this, arguments)
}

var dog = new Dog("白色")

var dog2 = new Dog("黑色")

dog.type.push("dog")

console.log(dog.color) // "白色"

console.log(dog.type) // ["pig", "cat", "dog"]

console.log(dog2.type) // ["pig", "cat"]

console.log(dog2.color) // "黑色"
//原理是在子类种调用了父类方法,apply更改了父类中的this指向,父类中的this指向了子类,调用父类方法给子类的this赋值,不是通过prototype
// 但是，构造函数继承也是有缺陷的，那就是我们无法获取到父类的共有方法，也就是通过原型prototype绑定的方法：
dog.greet() // Uncaught TypeError: dog.greet is not a function
```

3. 组合继承
   > 其实就是将类式继承和构造函数继承组合在一起

```javascript
// 声明父类

function Animal(color) {
  this.name = "animal"

  this.type = ["pig", "cat"]

  this.color = color
}

// 添加共有方法

Animal.prototype.greet = function(sound) {
  console.log(sound)
}

// 声明子类

function Dog(color) {
  // 构造函数继承

  Animal.apply(this, arguments)
}

// 类式继承

Dog.prototype = new Animal()

var dog = new Dog("白色")

var dog2 = new Dog("黑色")

dog.type.push("dog")

console.log(dog.color) // "白色"

console.log(dog.type) // ["pig", "cat", "dog"]

console.log(dog2.type) // ["pig", "cat"]

console.log(dog2.color) // "黑色"

dog.greet("汪汪") // "汪汪"
```

4. 寄生组合式继承

> 寄生组合式继承强化的部分就是在组合继承的基础上减少一次多余的调用父类的构造函数

```javascript
function Animal(color) {
  this.color = color

  this.name = "animal"

  this.type = ["pig", "cat"]
}

Animal.prototype.greet = function(sound) {
  console.log(sound)
}

function Dog(color) {
  Animal.apply(this, arguments)
  this.name = "dog"
}

/* 使用Object.create()进行一次浅拷贝，将父类原型上的方法拷贝后赋给Dog.prototype */
Dog.prototype = Object.create(Animal.prototype)

Dog.prototype.constructor = Dog

Dog.prototype.getName = function() {
  console.log(this.name)
}
var dog = new Dog("白色")
console.log(dog)
var dog2 = new Dog("黑色")

dog.type.push("dog")

console.log(dog.color) // "白色"

console.log(dog.type) // ["pig", "cat", "dog"]

console.log(dog2.type) // ["pig", "cat"]

console.log(dog2.color) // "黑色"

dog.greet("汪汪") //  "汪汪"
// 在上面的例子中，我们并不像构造函数继承一样直接将父类Animal的一个实例赋值给Dog.prototype，而是使用Object.create()进行一次浅拷贝，将父类原型上的方法拷贝后赋给Dog.prototype，这样子类上就能拥有了父类的共有方法，而且少了一次调用父类的构造函数。
// Object.create()的浅拷贝的作用类式下面的函数：
function create(obj) {
  function F() {}

  F.prototype = obj

  return new F()
}

// 这里还需注意一点，由于对Animal的原型进行了拷贝后赋给Dog.prototype，因此Dog.prototype上的constructor属性也被重写了，所以我们要修复这一个问题：
// Dog.prototype.constructor = Dog;
```

5. extends 继承

```javascript
class Animal {
  constructor(color) {
    this.color = color
  }

  greet(sound) {
    console.log(sound)
  }
}

class Dog extends Animal {
  constructor(color) {
    super(color)
    this.color = color
  }
}

let dog = new Dog("黑色")

dog.greet("汪汪") // "汪汪"

console.log(dog.color) // "黑色"
// 不知道你有没有注意到一点，我在子类的构造方法中调用了super方法，它表示父类的构造函数，用来新建父类的this对象。
// 注意：子类必须在constructor方法中调用super方法，否则新建实例时会报错。这是因为子类没有自己的this对象，而是继承父类的this对象，然后对其进行加工。如果不调用super方法，子类就得不到this对象。
```

#### [创建对象的九种方法](https://www.cnblogs.com/ayyl/p/5968568.html "`JavaScript` 创建对象的 7 种方法")

#### 算法题

##### 完美数

```javascript
let wanmei = []
for (var i = 1; ; i++) {
  // 获取因数
  let yinshu = []
  for (var j = 1; j < i; j++) {
    if (i % j === 0) {
      yinshu.push(j)
    }
  }
  if (eval(yinshu.join("+")) === i) {
    wanmei.push(i)
  }
}
```

#### 变量提升

- 第一题

```javascript
function test() {
  var num = 123
  console.log(num)
}
test() //123(test函数中的局部变量num,这个很好理解我就不解释了)
console.log(num) //报错(全局变量中没有声明num变量,无法访问下一级函数作用域中的变量,这个是考察作用域链的知识)
```

- 第二题

```javascript
var str = "我是MT"
test()
function test() {
  console.log(str) //undefined
  var str = "哈哈哈"
  console.log(str) //"哈哈哈"
}
console.log(str) //我是MT
```

- 第三题

```javascript
function test() {
  if ("a" in window) {
    var a = 10
  }
  console.log(a)
}

test() //undefined
//test函数内部有变量申明var a;会提升到函数作用域的最前面,全局没有变量a,所以结果是undefined而不会报错.
```

- 第四题

  ```javascript
  if ("a" in window) {
    var a = 10
  }
  console.log(a) //10
  //这个题目是针对demo3做对比的, if语句没有作用域,var a提升到最前面,所以window中有'a',打印结果为10.
  ```

- 第五题

```javascript
if (!"a" in window) {
  var a = 10
}
console.log(a) //undefined
// 此题是demo4的一个变种,同理变量提升,只是没有进入判断语句,所以为undefined;
```

- 第六题

```javascript
var foo = 1
function bar() {
  if (!foo) {
    var foo = 10
  }
  console.log(foo)
}
bar() //   10
```

- 第七题

```javascript
function Foo() {
  getName = function() {
    console.log("1")
  }
  return this
}
Foo.getName = function() {
  console.log("2")
}

Foo.prototype.getName = function() {
  console.log("3")
}

var getName = function() {
  console.log("4")
}
function getName() {
  console.log("5")
}
Foo.getName() // 2
getName() // 4 原因：getName先提升了声明，后赋值
Foo().getName() //1 ? 4 ? 2 ?报错
getName() // ?    1
new Foo.getName() //  2
new Foo().getName() // 3
new new Foo().getName() // 3
```

#### [深拷贝和浅拷贝](https://juejin.im/post/5beb93de6fb9a049c30ac9ee "深拷贝和浅拷贝")

- 深拷贝和浅拷贝的区别
  - 浅拷贝
    > 将原对象或原数组的引用直接赋给新对象，新数组，新对象／数组只是原对象的一个引用
  - 深拷贝
    > 创建一个新的对象和数组，将原对象的各项属性的“值”（数组的所有元素）拷贝过来，是“值”而不是“引用”

##### 深拷贝

> 我们希望在改变新的数组（对象）的时候，不改变原数组（对象）

1. 通过 `JSON.parse(JSON.stringify())`

```javascript
var obj1 = { body: { a: 10 } }
var obj2 = JSON.parse(JSON.stringify(obj1))
obj2.body.a = 20
console.log(obj1)
// { body: { a: 10 } } <-- 沒被改到
console.log(obj2)
// { body: { a: 20 } }
console.log(obj1 === obj2)
// false
console.log(obj1.body === obj2.body)
// false
/* 这样做是真正的Deep Copy，这种方法简单易用。
但是这种方法也有不少坏处，譬如它会抛弃对象的constructor。也就是深拷贝之后，不管这个对象原来的构造函数是什么，在深拷贝之后都会变成Object。
这种方法能正确处理的对象只有 Number, String, Boolean, Array, 扁平对象，即那些能够被 json 直接表示的数据结构。RegExp对象是无法通过这种方式深拷贝。 */

// 也就是说，只有可以转成JSON格式的对象才可以这样用，像function没办法转成JSON。

var obj1 = {
  fun: function() {
    console.log(123)
  }
}
var obj2 = JSON.parse(JSON.stringify(obj1))
console.log(typeof obj1.fun)
// 'function'
console.log(typeof obj2.fun)
// 'undefined' <-- 没复制
// 要复制的function会直接消失，所以这个方法只能用在单纯只有数据的对象。
```

2. 递归拷贝

```javascript
function deepClone(initalObj, finalObj) {
  var obj = finalObj || {}
  for (var i in initalObj) {
    var prop = initalObj[i] // 避免相互引用对象导致死循环，如initalObj.a = initalObj的情况
    if (prop === obj) {
      continue
    }
    if (typeof prop === "object") {
      obj[i] = prop.constructor === Array ? [] : {}
      arguments.callee(prop, obj[i])
    } else {
      obj[i] = prop
    }
  }
  return obj
}
var str = {}
var obj = { a: { a: "hello", b: 21 } }
deepClone(obj, str)
console.log(str.a)
```

3. Lodash 库

```javascript
var _ = require("lodash")
var obj1 = {
  a: 1,
  b: { f: { g: 1 } },
  c: [1, 2, 3]
}
var obj2 = _.cloneDeep(obj1)
console.log(obj1.b.f === obj2.b.f)
// false
```

1. jquery
   > jquery 有提供一个\$.extend 可以用来做 Deep Copy。

```javascript
var $ = require("jquery")
var obj1 = {
  a: 1,
  b: { f: { g: 1 } },
  c: [1, 2, 3]
}
var obj2 = $.extend(true, {}, obj1)
console.log(obj1.b.f === obj2.b.f)
// false
```

#### [数组去重](https://juejin.im/post/5b0284ac51882542ad774c45 "数组去重")

1. 利用 `indexOf()`

```javascript
functon unique(arr) {
   let res = []
   for (let i = 0; i < arr.length; i++) {
     if (res.indexOf(arr[i]) === -1) {
       res.push(arr[i])
     }
   }
   return res
 }
 console.log(unique(arr))
//[ 1,   'true',   'a',   true,   false,   undefined,   null,   NaN,   NaN,   'NaN',   0,   {},   {}, [], []]


```

2. 利用 `includes()`

```javascript
functon unique(arr) {
 let res = []
 for (let i = 0; i < arr.length; i++) {
     if (!res.includes(arr[i])) {
   	    res.push(arr[i])
     }
 }
 return res
}
```

3. 使用 `Set`

```javascript
Array.prototype.unique = function() {
  const set = new Set(this)
  return Array.from(set)
}

Array.prototype.unique = function() {
  return [...new Set(this)]
}
```

4. 使用 `Map`

```javascript
Array.prototype.unique = function() {
  const newArray = []
  const tmp = new Map()
  for (let i = 0; i < this.length; i++) {
    if (!tmp.get(this[i])) {
      tmp.set(this[i], 1)
      newArray.push(this[i])
    }
  }
  return newArray
}

Array.prototype.unique = function() {
  const tmp = new Map()
  return this.filter(item => {
    return !tmp.has(item) && tmp.set(item, 1)
  })
}
```

#### 布局

##### 两列等高布局

1. 使用`table-cell`实现等高布局,使用后元素会在一排显示，两边的等高的高度由较高的一侧的高度决定

```html
<div class="container">
  <div class="container-left">
    <div class="content-left" style="height: 500px"></div>
  </div>
  <div class="container-right">
    <div class="content-right" style="height: 600px;"></div>
  </div>
</div>
```

```css
.container-left,
.container-right {
  display: table-cell;
  width: 500px;
}
```

2. 使用`margin + padding`

```html
<div class="container">
  <div class="left">
    <div class="box" style="height: 200px;"></div>
  </div>
  <div class="right">
    <div class="box" style="height: 800px;"></div>
  </div>
</div>
```

```css
.container {
  overflow: hidden;
}

.left,
.right {
  float: left;
  width: 300px;
  margin-bottom: -600px;
  padding-bottom: 600px;
}

.left {
  background-color: yellow;
}

.right {
  background-color: green;
}
```

##### 左侧不定宽，右侧自适应

```html
<!-- table-cell实现 -->
<div class="layout-table-cell">
  <div class="left">
    <div class="left-content"></div>
  </div>
  <div class="right"></div>
</div>
<!-- overflow实现 -->
<div class="layout-overflow">
  <div class="left">
    <div class="left-content"></div>
  </div>
  <div class="right"></div>
</div>
<!-- flex实现 -->
<div class="layout-flex">
  <div class="left">
    <div class="left-content"></div>
  </div>
  <div class="right"></div>
</div>
```

```css
/* css */
.layout-table-cell
  .left
    display table-cell
    width 0.01%
    background-color pink

    .left-content
      width 500px
      height 400px

  .right
    display table-cell
    background-color green
    line-height 1

.layout-overflow
  overflow hidden

  .left
    float left
    background-color red

    .left-content
      width 500px
      height 400px

  .right
    overflow hidden
    height 400px
    background-color yellow

.layout-flex
  display flex

  .left
    background-color red

    .left-content
      width 500px
      height 400px

  .right
    flex 1
    overflow hidden
    height 400px
    background-color yellow
```

#### 水平垂直居中

##### 多行文本垂直居中

1. 多行文本垂直居中

```html
<div class="box">
  <div class="content">基于行高实现的...</div>
</div>
```

```css
.box {
  line-height: 120px;
  background-color: #f0f3f9;
}
.content {
  display: inline-block;
  line-height: 20px;
  margin: 0 20px;
  vertical-align: middle;
}
```

##### 在某个盒子中居中

1. 盒子有宽高

   1. 使用 `position+margin：auto`

   ```html
   <div class="father">
     <div class="son"></div>
   </div>
   ```

   ```css
   .father {
     width: 800px;
     height: 500px;
     position: relative;
     background-color: yellow;
   }

   .son {
     position: absolute;
     top: 0;
     right: 0;
     bottom: 0;
     left: 0;
     width: 300px;
     height: 300px;
     margin: auto;
     background-color: green;
   }
   ```

   1. 使用 `position+margin` 盒子宽高的负值

   ```html
   <div class="father">
     <div class="son"></div>
   </div>
   ```

   ```css
   .father {
     width: 800px;
     height: 500px;
     position: relative;
     background-color: yellow;
   }

   .son {
     position: absolute;
     width: 300px;
     height: 300px;
     left: 50%;
     top: 50%;
     margin-left: -150px;
     margin-top: -150px;
     background-color: green;
   }
   ```

2. 盒子没有宽高

   1. 使用 `position+tranform`

   ```html
   <div class="father">
     <div class="son">
       <div class="content"></div>
     </div>
   </div>
   ```

   ```css
   .father {
     width: 800px;
     height: 500px;
     position: relative;
     background-color: yellow;
   }

   .son {
     position: absolute;
     left: 50%;
     top: 50%;
     transform: translate(-50%, -50%);
     background-color: green;
   }
   .content {
     width: 200px;
     height: 100px;
     background-color: pink;
   }
   ```

   1. 使用`writing-mode`居中盒子中的任意元素

   ```html
   <div class="center-box">
     <div class="center-wrap">
       <div class="center-content">
         任意内容
       </div>
     </div>
   </div>
   ```

   ```css
   .center-box {
     width: 300px;
     height: 300px;
     background-color: teal;
     writing-mode: tb-lr;
     writing-mode: vertical-lr;
     text-align: center;
   }

   .center-wrap {
     writing-mode: lr-tb;
     writing-mode: horizontal-tb;
     text-align: center;
     width: 100%;
     display: inline-block;
   }

   .center-content {
     display: inline-block;
     text-align: left;
     text-align: initial;
   }
   ```
