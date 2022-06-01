## 前置条件
  了解分配条件类型(Distributive Conditional Types)基本特点

## 前言

今天在做一道类型体操的题目时，我的实现如下：

```ts

// 我的答案

type ReplaceKeys<U, T, Y> =  U extends U ? {
  [P in keyof U]: P extends T ?  P extends keyof Y ?  Y[P] : never : U[P]
} : never


// 参考答案
type ReplaceKeys<U, T, Y> =  {
  [P in keyof U]: P extends T ?  P extends keyof Y ?  Y[P] : never : U[P]
}
```

对比两个答案，发现我多写了一个`extends`。我写这个的目的就是为了联合类型的分配律，返回联合类型。但其他人没写`extends`也实现了一样的功能。不禁让我对自己理解的分配律产生怀疑。


### 几个例子


```ts
// exp
type A<T> = string extends T ? "yes" : "no"

type exp = A<'1' | 2>
```
`exp`中的例子会满足分配条件类型吗？

不会，因为检查的类型是字符串，它不是泛型类型参数。

```ts
// exp1
type B<T> = { x: T } extends { x: number } ? "yes" : "no"

type exp1 = B<'1' | 2>
```
`exp`中的例子会满足分配条件类型吗？

不会，因为检查的类型是`{ x: T }` ，其中包含类型参数`T`，但不是裸类型参数(naked type parameter)。


```ts
// exp2
type C<T> = T extends string ? "yes" : "no"
```

`exp`中的例子会满足分配条件类型吗?

是的

然后我查阅官网对其的定义为

> Distributive Conditional Types
When conditional types act on a generic type, they become distributive when given a union type. 

> 机翻： 当条件类型作用于泛型类型时，它们在给定联合类型时成为分配类型。

这里出现了一个新的名词`条件类型`

#### 什么是条件类型？

[TypeScript 2.8新增条件类型](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html)

一些简单的例子
```ts
T extends U ? X : Y // extends 中
```

```ts
// 不在条件类型中，非分配类型
type D2<T> =  keyof T 
type exp3 = D2<{a: string} | {b: number}>
// 映射类型中，也是分配类型
type f1<T> =  {[P in keyof T]: boolean }
type exp4 = f1<{a: string} | {b: number}>
```

## 总结

使用分配条件类型的条件

1. 泛型
2. 泛型约束的值为裸类型(且在extends左边)
3. 条件类型中或映射类型中
