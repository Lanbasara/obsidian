# 思路的变化

## 这个仓库源代码的思路

这个仓库的技术思路是通过 service worker 来充当插件的 background
再 backgrand 里面其实是通过脚本注入把 js 文件直接注入主页面

大大说

```js
chrome.scripting.getRegisteredContentScripts(
  { ids: ["testing-scripts-gen"] },
  async (scripts) => {
    if (scripts && scripts.length) {
      await chrome.scripting.unregisterContentScripts({
        ids: ["testing-scripts-gen"],
      });
    }
    chrome.scripting.registerContentScripts([
      {
        id: "testing-scripts-gen",
        js: ["./content.js"],
        matches: ["<all_urls>"],
        runAt: "document_start",
        allFrames: true,
      },
    ]);
  }
);
```

在主页面注入的脚本中去做了 iframe 注入, 并且把一些拦截逻辑嵌入，并且把一些拦截逻辑做了进去. 拦截 XHR 和 fetch, iframe 来做弹窗页面, 以此实现的拦截修改

## 新的尝试思路

新的尝试思路主要希望做两件事

1. 通过 service worker 而不是拦截 XHR 手段来拦截请求
2. 尝试介入 indexDB 来实现一定程度的 api 自动化

### Service worker 拦截

在这个仓库原本代码的基础上把思路修改为不是注入 JS 脚本代码，而是尝试注入 service worker

这个实验目前的进度为, 实现了下面这样的架构: 
1. Service worker来充当Chrome extension的background
2. 在Service worker中来注入js脚本到主页面
3. 注入的js脚本, 给页面引入一个Servive worker, 在这里来完成拦截功能

到目前为止, 这个流程被成功实现了, 然而需要注意以下问题
1. 有两个不同的Service worker, 一个是chrome extension的backrgound, 一个是实现拦截功能的
2. 第二个Service worker, 需要和运行页面同源, 这是一个需要解决的问题, 我们不可能给所有的host都准备一个Service worker文件, 即使使用拦截返回的手段, 也很困难

# dah 
