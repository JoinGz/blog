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

