# 禁用下拉刷新

Lavas apk 和 Chrome「添加到桌面」之后的效果一样，默认支持了「**下拉刷新**」功能，这个功能在大部分情况下是非常实用的，但是有很多时候开发者有禁用掉的需求。

## 场景

比如 twitter 在改造他们的 PWA 站点的时候，就遇到这种情况，需求时当用户下拉页面的时候刷新是不是有最新的 twitter 信息，这个过程应该是异步的，因为没有必要 reload 页面，只需要是一个异步懒加载的效果，可是在 Chrome「添加到桌面」之后每次用户下拉就变成了刷新页面，这样的效果 twitter 自然不能忍了。

## 如何禁用 Lavas app 的下拉刷新

一共有三种方法可以禁用掉 Lavas app 的下拉刷新：

- touch-action: none
- overflow-y: hidden
- preventDefault

### touch-action: none

可以通过设置 Root Element（也就是 html 元素）的 `touch-action` 属性值为 `none` 的方式来禁用 Lavas app 的下拉刷新的功能，这种情况下，只要开发者写了下面这段 css 代码，基本就能搞定。

```css
html {
    touch-action: none;
}
```

这种禁用方式有个弊端，就是必须要设置在页面的 Root Element（也就是 html 元素）上，我们来总结一下这种方式：

- 如果在 html 元素上设置了 `touch-action: none`，整个页面的 touch 事件失效，也就是说不仅禁用掉了下拉刷新功能，连整个页面都不能滚动。
- 当页面中元素含有 `overflow` 属性的时候，`touch-action: none` 禁用刷新失效。

```html
<!DOCTYPE html>
<html lang="en">
<head>
<style>
    html {
        touch-action: none;
    }
    .wrapper {
        /* 加上这个之后 touch-action: none 就会失效 */
        overflow: auto;
    }
    .content {
        /* 让页面产生滚动条 */
        height: 2000px;
    }
</style>
</head>
<body>
    <div class="wrapper">
        <div class="content">
            some text
        </div>
    </div>
</body>
</html>
```

从 `touch-action: none;` 在 Lavas app 上表现的特性看来，这种方式只是比较适用于禁止单页全屏并且无交互的静态页面下拉刷新了，一般情况下都不使用这种方式禁用 Lavas app 的下拉刷新功能。

### overflow-y: hidden

还有个最简单的方式在 Lavas app 上禁用下拉刷新的功能，只需要对 body 元素设置 `overflow-y: hidden;` 就搞定了。

```css
body {
    overflow-y: hidden;
}
```

通常这种做法会导致整个 body 不会产生滚动条了，如果既要禁用 Lavas app 的下拉刷新，又要页面可以正常上下滚动，只能人为的强制造一个 Dom 代替 body 标签行使最外层滚动容器的权利，这里有很多种做法，可以查看 Demo 源码看一种简单的实现。

通常这种做法在禁用 Lavas app 的自带的下拉刷新的时候非常常用，因为改动的代码非常少，并且相对于后面的 touchmove 的 preventDefault 方法也不影响页面自身的 touch 事件。基本算是 Lavas app 自身自带的一个无污染的 hack 了。

### e.preventDefault()

还有一种很常见的方式，也是我们想禁用浏览器自带事件时候最容易想到的方法，就是对 touchmove 事件进行 `e.preventDefault()` 处理，这样，就能够把浏览器的默认事件给阻止掉，所以我们禁用浏览器默认的下拉刷新的思路是：

- 当 `scrollTop` 等于 0 的时候
- 并且判断是下拉手势，判断 derection 向下的

满足以上的逻辑的，都 preventDefault 掉，照着这个思路，我们应该很快就有代码产出出来了：

```js
var lastY = 0;

window.addEventListener('touchmove', function (e) {
    var scrolly = window.pageYOffset || window.scrollTop || 0;
    var direction = e.changedTouches[0].pageY > lastY ? 1 : -1;

    if (direction > 0 && scrolly === 0) {
        e.preventDefault();
    }
    lastY = e.changedTouches[0].pageY;
}, {passive: false});
```

这样就能够使 `e.preventDefault()` 方法生效了，然后就可以实现禁用 Lavas app 的下拉刷新的功能了。
