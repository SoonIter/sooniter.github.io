---
title: 闭包和作用域[[Scopes]]
description: 一篇我在知乎里的回答，JS基础里函数、[[Scopes]]和闭包
date: 2022-10-18
lang: zh
duration: 5min
image: https://pic2.zhimg.com/80/v2-e5e2c04a518f9b14375913c7fabc961c_720w.jpg
---
[[TOC]]

本篇来源于我在知乎的一篇回答，以下为原链接

[这段代码算不算闭包？- 知乎](https://www.zhihu.com/question/551227690/answer/2719847401)

## 这段代码算不算闭包？
```javascript
function fn () {
  var a = 12
  console.log(a)
}

function fn1() {
  return fn()
}

fn1()
```

闭包，就是指函数创建时会捕获(capture)到其所用的变量，这个行为会导致上级作用域变量无法释放。

js里的所有函数都是闭包。
----------------------
（在mdn角度，是，但不经典）

如果js里所有函数都是闭包的话，那确实该和普通行为作一下区分了。
由于在js中我主要拿“闭包”用来解释变量没有被释放的行为，所以我说，不是。

## “创建函数，静态作用域就被『闭合』了”
我们尝试创建这个html在chrome里进行调试，并用`console.dir`打印函数

```html
<script>
  function fn () {
    var a = 12;
    console.log(a);
  }

  function fn1 () {
    return fn();
  }
  console.dir(fn1)
</script>
```

![](https://pic2.zhimg.com/80/v2-e5e2c04a518f9b14375913c7fabc961c_720w.jpg)


可以观察到在Chrome DevTools，函数有一个 `[[Scopes]]` 属性，用于调试。

而且我可以说，这个属性不论你在哪打印这个函数，结果都是一样的，不论是你`return`或者`export`出去，还是把这个函数赋值给 `a.fn` 还是`a.xxx.fn`。

这个也算是佐证了 该函数引了哪些作用域已经确定了。

在如图这个场景下，fn1引的是上级作用域Global里的fn，是一个函数，函数也是变量，因此fn不会被释放（起码在fn1还在的时候）。但是，大家本来在Global里就不会被释放，不经典。

## 改一改让它成为经典
我们尝试修改一下，让他俩都被包在fn2里

```html
<script>
  function fn2(){
    function fn () {
      var a = 12;
      console.log(a);
    }

    function fn1 () {
      return fn();
    }
    console.dir(fn1)
  }

  fn2();
</script>

</html>
```
![](https://pic3.zhimg.com/80/v2-4fc30dae322ce26b0b3ba520a27207a6_720w.webp)

此时我们熟悉的那个词来了，Closure闭包，fn被引了

但可惜这个闭包只是昙花一现...

fn2结束，fn1朝生夕死，fn也随之消亡。

于是我们想个办法，让fn1不被释放，最简单的方法就是return出去。

```javascript
  function fn2(){
    function fn () {
      var a = 12;
      console.log(a);
    }

    function fn1 () {
      return fn();
    }
    return fn1;   // 在这里return出去，fn1
  }

  var fnnnn = fn2();
```

怎么说，把fn视为一个变量a，和下面的经典是不是一样了。

```javascript
function fn2(){
    var a = 2;

    function fn1 () {
      return a;
    }
    return fn1;
}

var fnnnn = fn2();
```

## 到底是不是闭包？

字典是死的，但是字典里的字是活的。

说shui服会变成说shuo服，在js里，“闭包”也成为了阐释某种行为的代表，只有当该特定行为出现时，我们用”闭包“来解释他。

一定要是函数嵌套吗，我觉得无所谓了，不必过分纠结这个词。

你能使用”闭包“这个词，和同事达成共识，讲出来为什么有的变量没有被释放即可。
