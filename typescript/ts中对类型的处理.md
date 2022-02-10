### 前言
在书写ts中我们可能会遇到如下情况：
```typescript
type str = {str: string};
type num = {num: number}

type obj = {
  str: str,
  num: num.num
}
```