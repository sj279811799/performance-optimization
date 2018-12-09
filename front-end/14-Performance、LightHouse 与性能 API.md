# Performance、LightHouse 与性能 API

## Performance 面板

Chrome提供的性能调试工具，主要有FPS，CPU，火焰图，Summary饼图。可以录制一段渲染过程，Performance会给我们展示整个过程中浏览器都做了哪些事。

## LightHouse

网站分析工具，会帮我们生成网站的评估报告。

## W3C 性能 API

通过调用API，可以拿到实际渲染的时间，从而对性能进行分析。

## Mobx Trace

Mobx内置调试工具，开启后，每次mobx导致页面重新渲染，都会在浏览断点，并显示详细的触发信息及参数值。

```js
// 引入
import { trace } from "mobx"

// 开启
trace(true)
```

## Chrome浏览器插件：Mobx、React

Chrome上有很多React和Mobx的调试工具，可以显示虚拟DOM节点和Mobx变量，帮助调试。
