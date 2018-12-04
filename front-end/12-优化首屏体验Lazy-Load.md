# 优化首屏体验——Lazy-Load 初探

## Lazy-Load

懒加载，指的是将图片延迟加载，只有当图片滚动到可视区域才加载图片，这对图片较多的页面性能提升很多。

## 实现Lazy-Load

首先获取可视区域高度:

`const viewHeight = window.innerHeight || document.documentElement.clientHeight `

然后获取元素距离顶部距离:

`getBoundingClientRect() `

最后对滚动增加监听，计算图片是否进入可视区域，然后显示图片