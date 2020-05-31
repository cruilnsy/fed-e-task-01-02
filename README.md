# 崔锐 | Part 1 | 模块二

## 简答题

### 1. 描述引用计数的工作原理和优缺点。

解答：

内部通过一个引用计数器，来维护当前对象的引用数，从而判断当前引用数是否为0，来决定是不是垃圾对象。引用关系改变时，计数器增减相应数字，当值为零时，GC开始工作，将其所在的对象空间进行回收和释放再使用。

优点

- 发现垃圾时立即回收
- 减少程序卡顿时间（空间沾满，就会回收）

缺点

- 无法回收循环引用的对象
- 资源消耗较大，需要频繁的修改引用数

### 2. 描述标记整理算法的工作流程。

解答：

标记整理可以看做是标记清除的增强。首先，标记阶段，遍历所有对象找标记活动对象（递归标记）。其次，清除阶段，遍历所有对象，找到没有标记的对象，先执行整理，活动对象移动，地址上连续，然后把回收未活动的空间，同时把之前的活动对象的标记清除。

### 3. 描述V8中新生代存储区垃圾回收的流程。

解答：

新生代内存区分为二个等大小空间（From 和 To 是等大）。From 为使用空间，To为空闲空间。活动对象先存储于 From 空间，From 空间引用到一定程度后，触发GC操作，活动对象标记整理，然后拷贝到 To 中。拷贝的过程中会出现晋升（新生代到老生代）。两个条件会晋升，其一，当一轮GC下来，还存活的新生代；其二，To 空间的使用率超过25%。最后，From 与 To 实现交换，空间完成释放。

### 4. 描述增量标记算法在何时使用，及工作原理。

解答：

当垃圾回收的时候，会阻塞JavaScript的程序执行，所以中间会有空档期（程序执行后，会停下，去执行垃圾回收操作）。标记增量，就是将当前一整段的垃圾回收操作拆分成多个小步，组合着完成当前回收，从而替代之前一口气完成的垃圾回收操作。好处就是 垃圾回收和程序执行交替的完成。

## 代码题1

基于以下代码完成下面的四个练习

```jsx
const fp = require('lodash/fp')

// 数据
// horsepower 马力，dollar_value 价格，in_stock 库存
const cars = [
    {name: "Ferrari FF", horsepower:660, dollar_value:700000, in_stock: true},
    {name: "Spyker C12 Zagato", horsepower: 650, dollar_value: 648000, in_stock: false},
    {name: "Jaguar XKR-S", horsepower: 550, dollar_value: 132000, in_stock: false},
    {name: "Audi R8", horsepower: 525, dollar_value: 114200, in_stock: false},
    {name: "Aston Martin  One-77", horsepower: 750, dollar_value: 1850000, in_stock: true},
    {name: "Pagani Huayra", horsepower: 700, dollar_value: 1300000, in_stock: false},
]
```

### 练习1：

使用函数组合 `fp.flowRight()` 重新实现下面这个函数

```jsx
let isLastInStock = function(cars) {
    // 获取最后一条数据
    let last_car = fp.last(cars)
    // 获取最后一条数据的 in_stock 属性值
    return fp.prop('in_stock', last_car)
}
```

解答：

```jsx
let isLastInStock = fp.flowRight(fp.prop('in_stock'), fp.last)
```

### 练习2：

使用 `fp.flowRight()`、`fp.prop()` 和 `fp.first()` 获取第一个 car 的 `name`

解答：

```jsx
let getFirstName = fp.flowRight(fp.prop('name'), fp.first)
```

### 练习3：

使用帮助函数 `_average` 重构 `averageDollarValue`，使用函数组合的方式实现

```jsx
let _average = function(xs) {
    return fp.reduce(fp.add,  0, xs) / xs.length
}
// <- 无须改动

let  averageDollarValue =  function (cars)  {
    let dollar_values = fp.map(function(car) {
        return car.dollar_value
    }, cars)
    return _average(dollar_values)
}
```

解答：

```jsx
let averageDollarValue = fp.flowRight(_average, fp.map(fp.prop('dollar_value')))
```

### 练习4：

使用 `flowRight` 写一个 `sanitizeNames()` 函数，返回一个下划线连接的小写字符串，把数组中的 `name` 转换为这种形式：例如：`sanitizeNames(["Hello World"]) ⇒ ["hello_world"]`

```jsx
let _underscore = fp.replace(/\W+/g, '_')
// <-- 无须改动，并在 sanitizeNames 中使用它
```

解答：

```jsx
let sanitizeNames = fp.map(fp.flowRight(_underscore, fp.toLower))
```

## 代码题2

基于下面提供的代码，完成后续的四个练习

```jsx
// support.js
class Container {
    static of (value) {
        return new Container(value)
    }
    constructor (value) {
        this._value = value
    }
    map (fn) {
        return Container.of(fn(this._value))
    }
}

class Maybe {
    static of (x) {
        return new Maybe(x)
    }
    isNothing () {
        return this._value === null || this._value === undefined
    }
    constructor (x) {
        this._value = x
    }
    map (fn) {
        return this.isNothing ? this : Maybe.of(fn(this._value))
    }
}

module.exports = {
    Maybe,
    Container
}
```

### 练习1：

使用 `fp.add(x, y)` 和 `fp.map(f, x)` 创建一个能让 functor 里的值增加的函数 `ex1`

```jsx
const fp = require('lodash/fp')
const { Maybe, Container } = require('./support')

let maybe = Maybe.of([5, 6, 1])
let ex1 = // ... 你需要实现的位置
```

解答：

```jsx
let addOn = 4
let ex1 = maybe.map(fp.map(fp.add(addOn)))
```

### 练习2：

实现一个函数 `ex2`，能够使用 `fp.first` 获取列表的第一个元素

```jsx
const fp = require('lodash/fp')
const { Maybe, Container } = require('./support')

let xs = Container.of(['do', 'ray', 'me', 'fa', 'so', 'la', 'ti', 'do'])
let ex2 = // ... 你需要实现的位置
```

解答：

```jsx
let ex2 = xs.map(fp.first)
```

### 练习3：

实现一个函数 `ex3`，使用 `safeProp` 和 `fp.first` 找到 `user` 的名字的首字母

```jsx
const fp = require('lodash/fp')
const { Maybe, Container } = require('./support')

let safeProp = fp.curry(function (x, o) {
    return Maybe.of(o[x])
})
let user = { id: 2, name: "Albert" }
let ex3 = // ... 你需要实现的位置
```

解答：

```jsx
let ex3 = safeProp('name')(user).map(fp.first)
```

### 练习4：

使用 Maybe 重写 ex4，不要有 if 语句

```jsx
const fp = require('lodash/fp')
const { Maybe, Container } = require('./support')

let ex4 = function (n) {
    if (n) {
        return parseInt(n)
    }
}
```

解答：

```jsx
let ex4 = function (n) {
    return Maybe.of(n).map(parseInt)._value
}
```
