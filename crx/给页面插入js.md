### 动态插入

使用 `chrome.scripting.executeScript` 语法

```typescript
chrome.scripting.executeScript({
  target: { tabId: tab?.id as number },
  func: () => {
    console.log(`injected`)
  },
})
```

### 声明式注入

使用 `content_scripts` 声明式注入

```json
// manifest.json
"content_scripts": [
    {
      "matches": ["*://*.baidu.com/*"],
      "js": ["content-script.js"],
      "run_at": "document_idle"
    }
  ],
```

`run_at`分别有 3 个值：

- `document_start` 脚本是在任何 css 文件之后，但在构建任何其他 DOM 或运行任何其他脚本之前注入的。
- `document_end` DOMContentLoaded 之后 onload 之前
- `document_idle` 默认值，浏览器选择一个时间进行注入，在 document_end 之后或 onload 之后。取决于文档的复杂程度和加载时间

### 声明文件加注入式

使用 `web_accessible_resources` 声明文件

```json
// manifest.json
"web_accessible_resources": [
    {
      "resources": ["baidu.js"],
      "matches": ["https://*.baidu.com/*"]
    }
  ]
```

```ts
chrome.scripting.executeScript({
  target: { tabId: tab?.id as number },
  func: () => {
    const s = document.createElement('script')
    const url = chrome.runtime.getURL('baidu.js')
    s.setAttribute('src', url)
    document.documentElement.appendChild(s)
  },
})
```

```js
// baidu.js
console.log(window.$) // 可以拿到插入页面的js对象的值
```

#### 三种方式对比

|  动态插入   | 声明式  | 声明文件加注入式  |
| ---- | ---- | ---- |
| ---- | ---- | ---- |
| ---- | ---- | ---- |
| ---- | ---- | ---- |

| JS种类 | 可访问的API| DOM访问情况| JS访问情况| 直接跨域 |
| -- | -- | -- | -- | -- |
| injected script(声明文件加注入式) | 和普通JS无任何差别，不能访问任何扩展API | 可以访问 | 可以访问 | 不可以 
| injected script(动态插入) | 只能访问 extension、runtime等部分API | 可以访问 | 不可用 | 不可以 
content script (声明式)|只能访问 extension、runtime等部分API|可以访问|不可以|不可以
popup.js|可访问绝大部分API，除了devtools系列|不可直接访问|不可以|可以
background.js|可访问绝大部分API，除了devtools系列|不可直接访问|不可以|可以
devtools js|只能访问 devtools、extension、runtime等部分API|可以访问devtools|可以访问devtools|不可以
