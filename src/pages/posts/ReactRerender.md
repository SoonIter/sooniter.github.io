---
title: React的useCallback和useEvent
description: 这是我在知乎上的一篇回答，切题地讲一讲，除了这些API，React真正减少rerender的思路
date: 2022-08-30T00:00:00.000+00:00
lang: zh
duration: 20min
image: https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b89d9b11f664eff833f9722b3f8d715~tplv-k3u1fbpfcp-zoom-1.image
---

[[TOC]]

## 先说结论

useCallback、useMemo、memo、shouldUpdate，性能优化的API，不出性能问题你是不该使用的，盲目的使用会造成负优化，甚至还会造成延迟闭包的bug，这个结论是共识。

可是这个只是一方面， 我主要想切题地讲一讲，除了这些API，React真正减少rerender的思路

## React的粗粒度，可能伤及无辜，罪魁祸首 —— jsx的灵活让props难以区分变与不变

App是一个组件，Box是一个子组件

```typescript
 const App = () => {
   return <Box h="200px" w="200px" {/*.....*/}>hello</Box>
 }
 // jsx 编译后
 const App = () => {
   return /*#__PURE__*/ React.createElement(Box, { h: "200px", w: "200px" }, "hello");
 };
```

React采用的策略是全等比较，props比了第一回的`{ h: "200px", w: "200px" }`和第二回的`{ h: "200px", w: "200px" }`，虽然看似相等，但是function每次都创建了新的object类型，地址不同，照样渲染，即使是空对象也没辙，也如泄洪一般，从脑袋顶一直渲染到脚底板。Box是会被渲染的

```typescript
 const App = () => {
   return <Box>hello</Box>  //Box无memo，照样render
 }
 // jsx 编译后
 const App = () => {
   //此处props是空对象
   return /*#__PURE__*/ React.createElement(Box, {}, "hello");
 };
```

这是React的App级的更新策略 为什么采用这样的策略呢？因为props太多啦，假设是一个复杂的props

`{ h: "200px", w: "200px" 此处省略200多个key-value}`

我如果浅比较diff半天，结果发现只变了一个prop，还是要让自组件rerender，所以干脆，直接rerender得了。

这样虽然保证万无一失，但是这样的粗粒度对

`props不常改变或传参较少，rerender性能昂贵的组件 `不太友好。

毕竟是一个大框架，这样的边角问题虽然存在，那我还是要处理的，于是就把shouldUpdate一系列api暴露出来，把问题抛给开发者呗。

反观Vue这边的template，从写法上就不难看出，可以直接找到变和不变。

```typescript
 <Box w="50px" h="50px" :color="color" />
```

## 如何解决？

在该例子中，得益于JSX语法的灵活，React采用App级更新，并未对props进行区分。一旦App根部触发更新，泄洪便开始了。

```typescript
 const App = () => {
   const [count, setCount] = useState(0);
   return (
     <>
       <button onClick={() => setCount((a) => a + 1)}>Update</button>
       <A />  {/* 或是 <A width="50px"/>，都是不变的props */}
       <B count={count}/>
     </>
   );
 };
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b89d9b11f664eff833f9722b3f8d715~tplv-k3u1fbpfcp-zoom-1.image)


### 解决方法1 状态下沉

既然你知道了状态提升，那么我想聊一聊状态下沉。

很简单，和状态提升反着来，我们只有A用到了count，那么我们就该把const [count, setCount] = useState(0);移到B去，能动小树，就不要动大树。

```typescript
 const App = () => {
   return (
     <>
       <A />
       <B />
     </>
   );
 };
```

这样做的理由就是，

组件围绕状态进行拆分，达到State 和 View的对应关系

（PS：这在React基本上是不可能完美完成，即使对应不是很精准，大差不差也行）

Vue和Solid这样利用proxy以state为中心，依赖追踪，State和View的对应关系粒度更细，性能也更好。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b77951790c384711900e1d938f876e2a~tplv-k3u1fbpfcp-zoom-1.image)




### 解决方法2 shouldUpdate剪枝

使用组件优化的api，shouldComponentUpdate、memo、PureComponent等， 原理呢，就是一个高阶组件，相当于给A造了一个父节点，隔离层。

该例里，A组件未传props，不可能由父级改变，套memo，正收益，收益的大小取决于A组件的复杂程度。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a0e3c0e878b243e69907a34e4d1aa93f~tplv-k3u1fbpfcp-zoom-1.image)



### 解决方法3 Provider + 状态管理

状态下沉让state和view ，一一对应很考验布局组件的功底

而布局组件，Provider可以是很好的辅助道具。官方也说了，优秀的组件一定有Provider

### 该如何做状态提升？

Redux等React状态管理库就是这一个原理，往往要在App根部加装context.Provider就是证明（也是一个聪明的缓存机制，与App同一个生命周期）。理论上来说，把状态写入管理库，让多数组件回归无状态组件，是可以提高性能的。

缺点是： state和 改变state的method组合成一个数据模型，我叫做一个model。如果只用一个库统管所有state，各种method十分混乱，可读性差，难以维护。

所以能创建多个model，并使用hook来切分model的Pinia，是非常好的状态管理方案。

```typescript
 const App = () => {
    return {....}
 }
 export default ()=>(
   <某某XProvider>
     <App/>
   </某某XProvider>
 )
```



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de19b03c2911404d868bc10507d4074b~tplv-k3u1fbpfcp-zoom-1.image)



```typescript
const Leaf = memo(() => {
   const title = useContext(myContext);
   return <span>{title}</span>
})
```

一个足够小的叶节点，挖空了树干，减少了中间的rerender。

