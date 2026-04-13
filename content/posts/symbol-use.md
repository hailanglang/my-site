+++
date = '2026-04-13T21:52:01+08:00'
draft = false
title = 'Symbol 产生的背景以及应用场景'
+++


## 一、Symbol 产生的核心需求/背景

在 ES6 引入 Symbol 之前，JavaScript 中对象的属性名只能使用字符串或数字，这就导致了一系列问题，Symbol 的出现正是为了解决这些痛点，主要满足三个核心需求：

1.  解决对象属性命名冲突问题，避免属性名重写覆盖；
1.  实现非可迭代对象的迭代化，比如普通对象默认无法使用 for...of 遍历，Symbol 可解决这一问题；
1.  实现对象的不可枚举、半隐藏属性，让某些属性不被常规遍历方法暴露。

## 二、Symbol 应用场景

Symbol 的所有作用都围绕上述需求展开，每个作用对应具体的实战场景，以下结合代码示例详细说明：

### 1. 解决属性命名冲突

核心逻辑：Symbol 是全局唯一的值，即使描述符相同，两个 Symbol 也不相等，用它作为对象属性名，可彻底避免命名冲突。

```
// 示例：两个描述符相同的 Symbol，作为属性名不会冲突
const age1 = Symbol('age');
const age2 = Symbol('age'); // 与 age1 描述相同，但互不相等

const obj = {};
obj[age1] = 123; // 给 obj 添加 age1 对应的属性
obj[age2] = 456; // 给 obj 添加 age2 对应的属性

console.log(obj[age1]); // 123（不会被 age2 覆盖）
console.log(obj[age2]); // 456
console.log(age1 === age2); // false（证明两个 Symbol 不相等）
```

### 2. 实现不可枚举、半隐藏属性

核心逻辑：Symbol 作为对象属性名时，默认不可枚举，不会被 for...in、Object.keys() 遍历到，也无法被 JSON 序列化、普通拷贝方法复制，实现属性的半隐藏
注意：可通过 Object.getOwnPropertySymbols() 获取。

```
const sym = Symbol('privateProp');
const obj = {
  name: '张三',
  [sym]: '这是半隐藏属性' // Symbol 作为属性名
};

// 1. 无法被 for...in 遍历
for (let key in obj) {
  console.log(key); // 只输出 name，不会输出 sym 对应的属性名
}

// 2. 无法被 Object.keys() 获取
console.log(Object.keys(obj)); // ['name']

// 3. 无法被 JSON 序列化
console.log(JSON.stringify(obj)); // {"name":"张三"}（看不到 Symbol 属性）

// 4. 无法被 Object.assign 拷贝（浅拷贝也不会复制 Symbol 属性）
const newObj = Object.assign({}, obj);
console.log(newObj[sym]); // undefined

// 5. 可通过 Object.getOwnPropertySymbols() 获取（并非完全私有）
console.log(Object.getOwnPropertySymbols(obj)); // [Symbol(privateProp)]
```

### 3. 实现非可迭代对象的迭代化（Symbol.iterator）

核心逻辑：Symbol.iterator 是 Symbol 类的静态属性，相当于一个“迭代标记”。给普通对象的 obj[Symbol.iterator] 赋值一个生成器函数，就能让该对象变成可迭代对象，支持 for...of 遍历，遍历逻辑由生成器函数定义。

先明确两个关键知识点（结合代码理解）：

-   生成器函数：用 *function 声明，返回一个生成器对象，内部可使用 yield 关键字，每次调用生成器对象的 next() 方法，就会执行到下一个 yield 处；
-   next() 方法返回值：格式为 { value: 此次 yield 的值, done: 布尔值 }，done 为 true 表示迭代结束，false 表示仍有后续值。

```
// 示例：让普通对象变成可迭代对象，支持 for...of 遍历
const person = {
  name: '张三',
  age: 20,
  hobbies: ['游戏', '跑步', '看书'],
  // 给对象添加 Symbol.iterator 属性，赋值生成器函数
  [Symbol.iterator]: function* () {
    // 遍历对象的所有值，依次 yield 出去
    yield this.name;
    yield this.age;
    yield* this.hobbies; // yield* 用于迭代数组（批量 yield）
  }
};

// 此时 person 是可迭代对象，可使用 for...of 遍历
for (let item of person) {
  console.log(item); // 依次输出：张三、20、游戏、跑步、看书
}

// 单独调用生成器对象的 next() 方法（单步迭代）
const generator = person[Symbol.iterator]();
console.log(generator.next()); // { value: '张三', done: false }
console.log(generator.next()); // { value: 20, done: false }
console.log(generator.next()); // { value: '游戏', done: false }
console.log(generator.next()); // { value: '跑步', done: false }
console.log(generator.next()); // { value: '看书', done: false }
console.log(generator.next()); // { value: undefined, done: true }（迭代结束）
```

### 4. 内置符号改变语言默认行为

除了 Symbol.iterator，JavaScript 还有多个内置 Symbol（Well-known Symbols），可用于重写语言的默认行为，比如 Symbol.hasInstance 可自定义 instanceof 的判断逻辑，以下是最常用的示例：

```
// 示例：用 Symbol.hasInstance 自定义 instanceof 判断
class MyClass {
  // 静态方法，重写 Symbol.hasInstance
  static [Symbol.hasInstance](instance) {
    // 自定义判断逻辑：只要实例有 name 属性，就认为是 MyClass 的实例
    return instance.hasOwnProperty('name');
  }
}

const obj1 = { name: '张三' };
const obj2 = { age: 20 };

console.log(obj1 instanceof MyClass); // true（符合自定义逻辑）
console.log(obj2 instanceof MyClass); // false（不符合自定义逻辑）
```

其他常用内置符号（简单说明）：

-   Symbol.asyncIterator：用于实现异步迭代，支持 for await...of 遍历；
-   Symbol.toPrimitive：自定义对象转原始值（如 +obj、String(obj)）的逻辑；
-   Symbol.toStringTag：自定义 Object.prototype.toString() 的返回标签（如 [object MyClass]）。

## 三、总结

Symbol 的核心价值的是“唯一标识”和“可定制化语言行为”，主要解决对象属性冲突、实现半隐藏属性、让普通对象可迭代，再结合内置符号，能灵活改写 JavaScript 的默认逻辑，是前端开发中处理对象和迭代场景的重要工具。