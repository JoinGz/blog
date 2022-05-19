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