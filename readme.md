## 面试题

#### js 实现继承的几种方式

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

#### 完美数

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
