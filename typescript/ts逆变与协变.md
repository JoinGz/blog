## 逆变与协变


### 自己的理解

逆变， 子类反推到父类。
协变， 父类顺推到子类。

### 理解实例

如： WhiteCat -> Cat -> Animal

当 `(Cat) => Cat` 时，什么类型是其子类？

- (Animal) => WhiteCat
- (WhiteCat) => Animal
- (Animal) => Animal
- (WhiteCat) => WhiteCat

- 首先拆分下，满足`(Cat) => void`的是？
  - 满足的意思，就是哪些可以符合约束`(Cat) => void`,让其在这个函数中怎么都不会报错。那么就是`(Animal)=>void`，为什么呢，因为`Cat`包含了`Animal`。`(Animal)=>void`中调用任何方法都是`Cat`有的，就不会报错。反之子类`WhiteCat`的方法`Cat`不一定有，会有报错风险。所以是逆变。(cat要完全包含的就是父类，子类的东西不一定有)
- 接下来满足`() => Cat`的是？
    返回值能符合`Cat`约束的是？ 肯定是就子类`WhiteCat`。子类是继承父类的。所以是协变。

一个有趣的现象：在 TypeScript 中， 参数类型是双向协变的 ，也就是说既是协变又是逆变的，而这并不安全。但是现在你可以在 TypeScript 2.6 版本中通过 --strictFunctionTypes 或 --strict 标记来修复这个问题。

那么为什么ts会让函数参数类型保留双变转换呢？下面是一个十分常见的例子：

```ts
interface Event { timestamp: number; }
interface MouseEvent extends Event { x: number; y: number }

function listenEvent(eventType: EventType, handler: (n: Event) => void) {
    /* ... */
}

// 虽然不安全，且编译无法通过，但是十分常见的使用方式
listenEvent(EventType.Mouse, (e: MouseEvent) => console.log(e.x, e.y));

// 为了保证编译通过，只能通过以下方式
listenEvent(EventType.Mouse, (e: Event) => console.log((e as MouseEvent).x, (e as MouseEvent).y));
listenEvent(EventType.Mouse, ((e: MouseEvent) => console.log(e.x, e.y)) as (e: Event) => void);

```

### 分配条件类型与逆变和协变

```ts
type Foo<T> = T extends { a: infer U, b: infer U } ? U : never;
type T10 = Foo<{ a: string, b: string }>;  // string
type T11 = Foo<{ a: string, b: number }>;  // string | number

type Bar<T> = T extends { a: (x: infer U) => void, b: (x: infer U) => void } ? U : never;
type T20 = Bar<{ a: (x: string) => void, b: (x: string) => void }>;  // string
type T21 = Bar<{ a: (x: string) => void, b: (x: number) => void }>;  // string & number
```

为什么出现这种情况？

**多个相同泛型变量在协变的地方会被推断为联合类型，在逆变的地方会被推断为交叉类型**

逆变位置到底是个什么？

之前我们的分析得知：函数参数是逆变的，而对象属性是协变的。

变量处于逆变位置就是这个变量是一个函数的参数。

[参考1](https://juejin.cn/post/6844904169921314829)
[参考2](https://jkchao.github.io/typescript-book-chinese/tips/covarianceAndContravariance.html#%E4%B8%80%E4%B8%AA%E6%9C%89%E8%B6%A3%E7%9A%84%E9%97%AE%E9%A2%98)
[参考3](https://segmentfault.com/a/1190000041743807)