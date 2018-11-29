# Event Loop 与异步更新策略

## Micro-Task 与 Macro-Task

macro（宏任务）：setTimeout、setInterval、setImmediate、script、I/O操作、UI渲染等。

micro（微任务）：process.nextTick、Promise、MutationObserver等。

## Event Loop 过程解析

- 初始状态：调用栈空。micro队列空，macro队列里有且只有一个script脚本。

- 全局上下文（script标签）被推入调用栈，同步代码执行。执行过程中产生的任务会被推入对应的队列里。同步执行完，被移除macro队列，整个过程本质是队列的macro-task的执行和出队过程。

- macro-task出队时，任务是一个一个执行。micro-task出队时，任务是一队一队执行的。处理micro队列，会逐个执行队列中任务并出队，直到队列清空。

- 执行渲染操作，更新界面。

- 检查是否存在Web worker任务，如果有，对齐进行处理。

上面过程循环往复，直到两个队列清空。

## 渲染时机

假如要在异步队列更新DOM，放在micro还是macro中？

假如放在macro中，比如使用setTimeout：

```js
// task是一个用于修改DOM的回调
setTimeout(task, 0)
```

根据上一步中的Event Loop过程，由于本次执行的script也是macro任务，本次不会被执行，所以要等到第二次才会更新。

如果使用micro，比如Promise：

```
Promise.resolve().then(task)
```

执行完本次script，接下来就是micro，于是DOM被修改，接下来就是render，不需要等到第二次循环。

所以，要在异步任务中的更新DOM，最好包装在micro任务中。

# 异步更新策略
