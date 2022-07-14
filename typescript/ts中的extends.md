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

#### 1.类中的extends, 除和 js 一样的继承外,还可以继承类型

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
#### 2.条件类型中的extends

查看官网中的一段话

> When the type on the left of the extends is assignable to the one on the right, then you’ll get the type in the first branch (the “true” branch); otherwise you’ll get the type in the latter branch (the “false” branch).

当 extends 左边的类型可赋值给右边的类型时(当左边的类型约束完全符合右边的类型约束时)，你会在第一个分支中获得该类型(“true”分支);否则你将在后面的分支中获得类型(“false”分支)。

1.情况一
```typescript
interface Animal {
  live(): void;
}
interface Dog extends Animal {
  woof(): void;
}
 
type Example1 = Dog extends Animal ? number : string; // number

type Example2 = RegExp extends Animal ? number : string; // string
```

2.情况二

```typescript
type Example3 = 'k' extends 'k' ? "true" : "false"  // "true"
type Example4 = 'k' extends 'k1' ? "true" : "false"  // "false"

type Example5 = 100 extends 100 ? "true" : "false"  // "true"
type Example6 = 500 extends 100 ? "true" : "false"  // "false"
```

我用js的全等`===`来理解此种情况

2.情况三

```typescript
type Example7 = {} extends {name:string} ? "true" : "false"  // "false"
type Example8 = {name:string} extends {} ? "true" : "false"  // "true"
```

根据ts中定义，当 extends 左边的类型可赋值给右边的类型时是true(当左边的类型约束完全符合右边的类型约束时)。 

因为第一行代码中{}(左边)没有满足name:string(右边)的约束所以为false,此时获得结果 "false"

因为第二行代码中{name:string}(左边)满足{}(右边)的约束(此时没有任何键值的约束)所以为true,此时获得结果 "true"

3.情况四

```typescript

type Example9<T> = T extends string ? "true" : "false"
type boolStringTrue = Example9<''> // "true"
type boolStringFalse = Example9<1> // "false"
type Example10 = Example9<1 | ''> // "true" | "false"

```

前面两个很好理解，第三个为什么出现这种现象，我们需要了解一下联合类型的处理过程

```typescript
type boolStringTrueAndFalse = Example9<1 | ''> // "true" | "false"

// 第一步 把1放入Example9执行

type result_number = Example9<1> // "false"

// 第二步 把""放入Example9执行

type result_string = Example9<""> // "true"

// 第三步  合并

type result = "false" | "true"

```

>对于使用extends关键字的条件类型（即上面的条件表达式类型），如果extends前面的参数是一个泛型类型，当传入该参数的是联合类型，则使用分配律计算最终的结果。分配律是指，将联合类型的联合项拆成单项，分别代入条件类型，然后将每个单项代入得到的结果再联合起来，得到最终的判断结果。

通常，分配性是所需的行为。为避免这种行为，您可以extends用方括号将关键字的每一侧括起来。

```typescript
type Example10<T> = [T] extends [string] ? "true" : "false"
type Example11 = Example10<1 | ''> // "false"

type Example12<T> = [T] extends [1 | ''] ? "true" : "false"
type Example13 = Example12<1 | ''> // "true"
```

此时，传入的参数T会被当成一个整体，从而不在分配。在Example10中 `1 | ''` 不能够分配给类型 `string`所以false。
Example13中值相等所以为true

#### []的多种情况

1.只扩左边

```typescript
type Example14<T> = [T] extends string ? "true" : "false"
type boolString3 = Example14<1 | ''> // "false"
type boolString4 = Example14<string> // "false"
```
这种情况下，应该永远都是false,此时就是普通的写法，没有更改分配性。相当于 `[] extends string` 所以永远为false。（自己理解，如有不对还请不吝赐教）

2.只扩右边
```typescript
type Example15<T> = T extends [string] ? "true" : "false"
type boolString5 = Example15<1 | ''> // "false"
type boolString6 = Example15<['', 1]> // "false"
type boolString7 = Example15<['']> // "true"
```
此时也没有更改分配性，T需要符合\[string\]的约束。就是符合这个元祖的约束即可。

经过上面的情况梳理，取消掉分配性需要extends两边同时加上括号。

### 特殊的never

```typescript
type Example16<T> = T extends string ? "true" : "false"
type boolString8 = Example16<never> // never
```

当传入never时，此表达式没有执行，得到never。因为never是所有类型的子类型，是一个空的联合类型。因为没有联合项分配所以就没有执行。如果需要执行，我们可以阻止分配性，使用我们刚学的[]。

```typescript
type Example17<T> = [T] extends [string] ? "true" : "false"
type boolString9 = Example17<never> // "true"
```

### 总结
ts中的extends关键字，主要用法就是
+ 继承、拓展约束
+ 条件类型
  - 当左边的类型约束完全符合右边的类型约束时
  - 分配条件类型