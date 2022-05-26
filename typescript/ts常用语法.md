## 前言

总结一些`typescript`常用的语法，加深记忆，也在忘记的时候方便查阅。

## 遍历

ts 中遍历使用的是 `in` 关键字

- 遍历对象

```ts
type obj = {
  name: string
  age: number
}

type copyObj<T> = { [P in keyof T]: T[P] }

type copedType = copyObj<obj>
```

- 遍历数组

```ts
type arrayList = [1, 2, 3]

type forEach<T extends any[]> = { [P in T[number]]: P }

type result = forEach<arrayList>
```

数组中`T[number]`取得是数组中的值

## 获取数组的长度

`T['length']`


### 获取数组的index

`keyof [4,5,6]`

```ts
type copyArray<T extends unknown[]> = { [P in keyof T]: T[P] }

type exp1 = copyArray<[3,4]> // [3,4]
```

### 数组的值变为联合类型

```ts
type TupleToUnion<T extends any[]> = T[number]

type resultUnion = TupleToUnion<[123, '456', true]> //  123 | '456' | true
```

### [] 与 readonly []

```ts
type bool = [] extends readonly [] ? true : false // true
type exp1 = readonly [] extends  [] ? true : false // false
```

### 限制数组的扩宽

```ts

function copyArr<T>(list : T[]): T[] {
  return [...list]
}

function copyArr2<T extends readonly unknown[] | readonly [unknown]>(list: T): {
  [P in keyof T]: T[P]
} {
  return [...list]
}
function copyArr3<T extends unknown[] | [unknown]>(list: T): {
  [P in keyof T]: T[P]
} {
  return [...list]
}

// 不增加 | [unknown] 就会报错，原因思考中
function copyArr4<T extends unknown[]>(list: T): {
  [P in keyof T]: T[P]
} {
  return [...list] // error 不能将类型“T[number][]”分配给类型“{ [P in keyof T]: T[P]; }”。ts(2322)
}

let arr1 = copyArr(['1', 1]) // (string | number)[]
let arr2 = copyArr2(['1', 1]) // [string, number] 没有拓宽
let arr3 = copyArr3(['1', 1, {a: 'a'}]) // [string, number, {a: string;}] 没有拓宽
```


#### 关于限制数组的扩宽中问题的思考

```ts
// 不增加 | [unknown] 就会报错的原因
/**
 * 之前我主要的疑问为我其实已经实现了实现的返回值和定义函数的返回值一致，按理说ts不应该报错了。但忽略了数组一个重要的属性 长度 。在ts中要所有属性满足约束才能通过约束。
 * 我们来想想一般约束数组哪些属性：  值 和 长度
 * 值 我们使用的 {[P in keyof T]: T[P]} 来约束成功
 * 但是长度约束中，ts却不知道。因为 unknown[] 是一个不定长数组，而{[P in keyof T]: T[P]}是一个定长数组。所以定长的来约束不定长的就会报错
 * 
*/
function copyArr4<T extends unknown[]>(list: T): {
  [P in keyof T]: T[P]
} {
  return [...list] // error 不能将类型“T[number][]”分配给类型“{ [P in keyof T]: T[P]; }”。ts(2322)
}

let arr1 = copyArr(['1', 1]) // (string | number)[]
let arr2 = copyArr2(['1', 1]) // [string, number] 没有拓宽
let arr3 = copyArr3(['1', 1, {a: 'a'}]) // [string, number, {a: string;}] 没有拓宽
```

解决方案一: 返回值也改变为不定长数组

```ts

function copyArr5<T extends unknown[]>(list: [...T]): [...{
  [P in keyof T]: T[P]
}] {
  return [...list]
}


let arr5 = copyArr5(['1', 1, {a: 'a'}]) // [string, number, {a: string;}] 没有拓宽

```

方案一中的约束`T`为数组中的每一项，为什么呢？

这就是为了定长，把每一项取出来，就成为了定长数组。

方案二：

```ts
function copyArr3<T extends unknown[] | [unknown]>(list: T): {
  [P in keyof T]: T[P]
} {
  return [...list]
}

let arr3 = copyArr3(['1', 1, {a: 'a'}]) // [string, number, {a: string;}] 没有拓宽

```

方案二在约束中增加的`| [unknown]`使约束变为`T extends unknown[] | [unknown]`为什么也可以完成需求呢？

- `unknown[]`不定长数组
- `[unkonwn]`定长数组

这两个组成的联合类型就能被TS正确推断出来（我的推断，验证需要看TS实现细节）。

参考我的[提问](https://segmentfault.com/q/1010000041878054/a-1020000041882785?_ea=238927828)


### 如何删除值为never的属性

如
```ts
{
  a: string,
  b: number,
  c: never
}
// 处理后
{
  a: string,
  b: number
}
```

- 在联合类型中的never会被忽略

```ts
 type exp1 = string | never | number // string | number
```

所以，我们返回一个联合类型，其中值不是never的就保持原来的key,是never的返回never就可以剔除值为never的key了。

```ts
type deletedNever<T> = {[P in keyof T]: T[P] extends never ? never : P}[keyof T]
```

前面的`{[P in keyof T]: T[P] extends never ? never : P}`主要是把key转换为value,因为后面我们要`Pick`出需要的key。(把值为never的key剔除)

重点为后面的`[keyof T]`

如：

```ts
{
  a: 'a',
  b: 'b',
  c: never
}['a' | 'b' | 'c']
// 得到

'a' | 'b' | never

// ts剔除never后

//得到 'a' | 'b'

```
拿到需要的key了，我们在`Pick`一下即可。

完整代码

```ts
// 获取到value为never的key
type deletedNever<T> = {[P in keyof T]: T[P] extends never ? never : P}[keyof T]

// 在筛选就就ok

type uix<T> = Pick<T, deletedNever<T>>

type uiox = uix<{ name: string, ok: never, l: number }>
// {
//   name: string;
//   l: number;
// }
```

### extends {}



```ts
type bool = '' extends {} ? true : false // true
type bool2 = 0 extends {} ? true : false // true
type bool3 = true extends {} ? true : false // true
type bool4 = null extends {} ? true : false // false ( with strictNullChecks )
type bool5 = undefined extends {} ? true : false // false ( with strictNullChecks )
```

`{}`类型除了`null`和`undefined`都满足分配。包括原始类型。

`unknown`则是所有都满足。
```ts
type bool6 = null extends unknown ? true : false // true
type bool7 = undefined extends unknown ? true : false // true
```

[参考](https://stackoverflow.com/questions/61648189/typescript-generic-type-parameters-t-vs-t-extends)