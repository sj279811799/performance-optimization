# DOM 优化原理与基本实践

## DOM 为什么这么慢

JS引擎和渲染引擎间调用的开销不可忽略，所以每一次操作DOM，都要有所消耗。

当我们对DOM进行操作后，会触发回流和重绘。

回流：当对DOM修改，导致几何尺寸发生变化，需要重新计算元素的几何属性，然后将结果计算出来，又叫重排。

重绘：当对DOM修改，没有导致尺寸变化，不需重新计算，只要重新绘制即可。

## 给你的 DOM “提提速”

根据上面的几个问题，优化的方法就是`减少 DOM 操作：少交“过路费”、避免过度渲染`

加入我们需要向DOM中的一个元素中写入1000次一样的字符串，这个有两点优化可以考虑：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>DOM操作测试</title>
</head>
<body>
  <div id="container"></div>
</body>
</html>
```

```js
for(var count=0;count<10000;count++){ 
  document.getElementById('container').innerHTML+='<span>我是一个小测试</span>'
} 
```

- 减少JS操作DOM次数，获取DOM元素，将变量缓存下来

```js
// 只获取一次container
let container = document.getElementById('container')
for(let count=0;count<10000;count++){ 
  container.innerHTML += '<span>我是一个小测试</span>'
}
```

- DOM更改次数太多，会导致多次的重绘或回流

```js
let container = document.getElementById('container')
let content = ''
for(let count=0;count<10000;count++){ 
  // 先对内容进行操作
  content += '<span>我是一个小测试</span>'
} 
// 内容处理好了,最后再触发DOM的更改
container.innerHTML = content
```

上面的例子每次操作都相同，所以可以合并，对于不同的操作，我们可以用`DOM Fragment`来处理

```js
let container = document.getElementById('container')
// 创建一个DOM Fragment对象作为容器
let content = document.createDocumentFragment()
for(let count=0;count<10000;count++){
  // span此时可以通过DOM API去创建
  let oSpan = document.createElement("span")
  oSpan.innerHTML = '我是一个小测试'
  // 像操作真实DOM一样操作DOM Fragment对象
  content.appendChild(oSpan)
}
// 内容处理好了,最后再触发真实DOM的更改
container.appendChild(content)
```

DocumentFragments是一个DOM节点，但不是主树中的一部分，存在内存中，每次修改不会导致回流。最后将文档片段放到DOM中，片段会被其子元素代替。