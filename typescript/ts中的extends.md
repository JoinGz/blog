## 理解 ts 中的 extends

### 回顾 js 中的 extends

```javascript
// 用于继承
class Person {
  constructor(name) {
    this.name = name
  }
  sayName() {
    console.log(`my name is ${this.name}.`)
  }
}

class Gz extends Person {
  constructor(name) {
    super(name)
  }
}
```

### ts 中的 extends

查看官网中的一段话

> When the type on the left of the extends is assignable to the one on the right, then you’ll get the type in the first branch (the “true” branch); otherwise you’ll get the type in the latter branch (the “false” branch).

当 extends 左边的类型可赋值给右边的类型时，你会在第一个分支中获得该类型(“true”分支);否则你将在后面的分支中获得类型(“false”分支)。

1.除和 js 一样的继承外，还可以继承类型

```typescript
type Person = {
  name: string
}

interface Gz extends Person {
  age: number
}

let gz: Gz = {
  name: 'gz',
  age: 18,
}
```

1.基础类型
```typescript

type bool = 'k' extends 'k' ? boolean : string  // boolean
type bool_1 = 'k' extends 'k1' ? boolean : string  // string

type bool_2 = 100 extends 100 ? boolean : string  // boolean
type bool_3 = 500 extends 100 ? boolean : string  // string
```
我用js的全等`===`来`理解此种情况

2.对象类型

```typescript
type bool_4 = {} extends {name:string} ? boolean : string  // string
type bool_5 = {name:string} extends {} ? boolean : string  // boolean
```

根据ts中定义，当 extends 左边的类型可赋值给右边的类型时是true。 

因为第一行代码中{}(左边)没有满足name:string(右边)的约束所以为false,此时获得结果 string

因为第二行代码中{name:string}(左边)满足{}(右边)的约束(此时没有任何键值的约束)所以为true,此时获得结果 boolean

3.条件类型

```typescript

type bool_6<T> = T extends string ? "true" : "false"
type boolStringTrue = bool_6<''> // "true"
type boolStringFalse = bool_6<1> // "false"
type boolStringTrueAndFalse = bool_6<1 | ''> // "true" | "false"

```

前面两个很好理解，第三个为什么出现这种现象，我们需要了解一下联合类型的处理过程

```typescript
type boolStringTrueAndFalse = bool_6<1 | ''> // "true" | "false"

// 第一步 把1放入bool_6执行

type result_number = bool_6<1> // "false"

// 第二步 把""放入bool_6执行

type result_string = bool_6<""> // "true"

// 第三步  合并

type result = "false" | "true"

```

>对于使用extends关键字的条件类型（即上面的条件表达式类型），如果extends前面的参数是一个泛型类型，当传入该参数的是联合类型，则使用分配律计算最终的结果。分配律是指，将联合类型的联合项拆成单项，分别代入条件类型，然后将每个单项代入得到的结果再联合起来，得到最终的判断结果。