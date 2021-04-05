## Vue && React

### 1、Vue 的响应式原理

vue 通过双向数据绑定，来实现了 View 和 Model 的同步更新。而双向数据绑定主要是通过 *数据劫持* 和 *发布订阅者模式* 来实现的  
首先我们通过 *Object.defineProperty* 方法来对 Model 数据各个属性添加访问器属性，以此来实现数据的劫持，因此当 Model 中的数据发生变化的时候，我们可以通过配置的 setter 和 getter 方法来实现对 View 层数据更新的通知  
对于文本节点的更新，我们使用了发布订阅者模式，属性作为一个主题，我们为这个节点设置一个订阅者对象，将这个订阅者对象加入这个属性主题的订阅者列表中。当 Model 层数据发生改变的时候，Model 作为发布者向主题发出通知，主题收到通知再向它的所有订阅者推送，订阅者收到通知后更改自己的数据

vue2 Object.defineProperty

```javascript
// 基础
/** descriptor
  * configurable:true|false, 是否可以删除目标属性或是否可以再次修改属性的特性
  * enumerable:true|false, 此属性是否可以被枚举
  * value:任意类型的值,
  * writable:true|false 属性的值是否可以被重写
  */
Object.defineProperty(obj, prop, descriptor)
// 存取器 getter setter
let obj = {};
let testKey = 7
Object.defineProperty(obj, 'testKey', {
  get: function () {
    return testKey;
  },
  set: function (newValue) {
    testKey = newValue
  }
})
```

![响应式](https://cn.vuejs.org/images/data.png)

这个方式它有什么缺陷？ 

- 它无法低耗费的监听到数组下标的变化，所以通过数组下标添加元素，不能实时响应（数组操作的问题）
- 它只能劫持对象的属性，从而需要对每个对象，每个属性进行遍历。如果属性值是对象，还需要深度遍历
 
vue3 Proxy  
Proxy可以劫持整个对象， 并返回一个新的对象
Proxy不仅可以代理对象，还可以代理数组。还可以代理动态增加的属性

```javascript
// 选择代理属性
const target = {
  notProxied: "original value",
  proxied: "original value"
};
const handler = {
  get: function(target, prop, receiver) {
    if (prop === "proxied") {
      return "replaced value";
    }
    return Reflect.get(...arguments);
  }
};
const proxy = new Proxy(target, handler);
console.log(proxy.notProxied); // "original value"
console.log(proxy.proxied);    // "replaced value"
```


### 2、写 Vue 项目时为什么要在列表组件中写 key，其作用是什么


vue都是采用 diff 算法来对比新旧虚拟节点，从而更新节点。
在diff 函数交叉对比中，当新节点跟旧节点 头尾交叉对比 没有结果时，会根据新节点的key去对比旧节点数组中的key，从而找到相应旧节点（这里对应的是一个 key => index 的 map 映射）。
如果没有找到就认为是一个新增节点。
而如果没有 key，那么就会采用遍历查找的方式去找到对应的旧节点。
一种一个 map 映射，另一种是遍历查找。相比而言，map 映射的速度更快。

### 3、为什么 Vuex 的 mutation 中不能做异步操作

纯函数，给定同样的输入返回同样的输出，可预测性

### 4、Vue 的父组件和子组件生命周期钩子执行顺序是什么

加载渲染过程：父 beforeCreate -> 父 created -> 父 beforeMount -> 子 beforeCreate -> 子 created -> 子 beforeMount -> 子 mounted -> 父 mounted；  
子组件更新过程：父 beforeUpdate -> 子 beforeUpdate -> 子 updated -> 父 updated；
父组件更新过程：父 beforeUpdate -> 父 updated；  
销毁过程：父 beforeDestroy -> 子 beforeDestroy -> 子 destroyed -> 父 destroyed；
 
### 5、vue的v-for中需不需要用事件代理

首先vue的里面是没有主动去做事件代理的，但是要不要自己去做需要分情况。首先事件代理的作用是两个：

- 减少内存的占用率
- 动态节点的事件绑定问题
vue已经解决了动态节点的事件绑定问题，内存占用的消耗和数据量相关
参考文章：https://www.jianshu.com/p/52b0562846af

### 6、双向绑定和 vuex 是否冲突

有，怎么解决https://muyiy.cn/question/frame/47.html

### 7、Vue 中 computed 和 watch 的差异

1. computed是计算出一个新的属性并挂载到vue的实例上，而watch是监听一个原有的属性
2. computed是带有缓存性质的，只有当依赖的属性变化的时候第一次访问该属性才会被调用；而watch是数据变化就会调用
3. 使用中computed适用一个数据被多个数据影响；watch使用一个数据影响多个数据

### 8、组件之间通讯

https://juejin.cn/post/6844903887162310669
https://segmentfault.com/a/1190000022083517


### 9、vue 如何优化首页的加载速度？vue 首页白屏是什么问题引起的？如何解决呢？ 

引起问题的原因：  
现在的前端框架，例如vue react 都是 JS 驱动，在 JS 代码解析完成之前，页面不会展示任何内容，也就是所谓的白屏。
用户是极其不喜欢看到白屏的，什么都没有展示，用户很有可能怀疑网络或者应用出了什么问题。 拿 Vue 来说，在应用启动时，Vue 会对组件中的 data 和 computed 中状态值通过 Object.defineProperty 方法转化成 set、get 访问属性，以便对数据变化进行监听。 而这一过程都是在启动应用时完成的，这也势必导致页面启动阶段比非 JS 驱动（比如 jQuery 应用）的页面要慢
解决方式：  
ssr（千人千面的缓存问题需要考虑）
预渲染
骨架屏
 
参考文章： https://muyiy.cn/question/frame/119.html


### 10、v-if、v-show、v-html 的原理是什么，它是如何封装的？

v-if会调用addIfCondition方法，生成vnode的时候会忽略对应节点，render的时候就不会渲染； v-show会生成vnode，render的时候也会渲染成真实节点，只是在render过程中会在节点的属性中修改show属性值，也就是常说的display； v-html会先移除节点下的所有节点，调用html方法，通过addProp添加innerHTML属性，归根结底还是设置innerHTML为v-html的值

### 11、vue的nextTick原理

2.4之前是微任务的实现（MicroTask），目前是默认微任务，特殊场景下例如v-on的实现中使用宏任务。实现宏任务的方法依次检测**setImmediate** 、 **MessageChannel** 、 **setTimeout**

### 12、vue的生命周期

```javascript
// 顺序
Vue.prototype._init = function(options) {
  initLifecycle(vm)
  initEvents(vm)
  initRender(vm)
  callHook(vm, 'beforeCreate') // 拿不到 props data
  initInjections(vm)
  initState(vm)
  initProvide(vm)
  callHook(vm, 'created')
}
// beforeMount
// VDOM -> DOM
// mounted
```

### 13、vuex

## React

### 1、什么是虚拟 DOM

虚拟 DOM (VDOM)是真实 DOM 在内存中的表示。UI 的表示形式保存在内存中，并与实 际的 DOM 同步。这是一个发生在渲染函数被调用和元素在屏幕上显示之间的步骤，整 个过程被称为调和

### 2、类组件和函数组件之间的区别是什么

类组件可以使用其他特性，如状态 state 和生命周期钩子。
当组件只是接收 props 渲染到页面时，就是无状态组件，就属于函数组件，也被称为 哑组件或展示组件
类组件的性能要低于函数组件，因为类组件使用的时候是实例化的，而函数的组件是 直接执行函数返回结果即可
函数组件没有 this，没有生命周期，没有状态，但是类组件都有

### 3、react 的 refs

Refs 是一个 获取 DOM 节点或 React 元素实例的工具 什么时候用?(场景)

- 管理焦点，文本选择或媒体播放 • 触发强制动画
- 集成第三方 DOM 库

使用情况
在 dom 元素中使用 在类组件上面添加 ref
在函数组件里面使用 ref 是不生效的，这个时候可以使用 forwardRef

```javascript
function CustomTextInput(props) {
// 这里必须声明 textInput，这样 ref 才可以引用它 const textInput = useRef(null);
function handleClick() {
    textInput.current.focus();
}
  return (
    <div>
      <input
        type="text"
        ref={textInput} />
      <input
        type="button"
        value="Focus the text input"
        onClick={handleClick}
/> </div>
);
```

### 4、什么是高阶组件

高阶组件(HOC)是 React 中用于复用组件逻辑的一种高级技巧。HOC 自身不是 React API 的一部分，它是一种基于 React 的组合特性而形成的设计模式。 具体而言，高阶组件是参数为组件，返回值为新组件的函数，即将组件转化成另外一 个组件

### 5、什么是受控||不受控组件

#### 受控组件(被 react 控制)

在 HTML 中，表单元素(如 input、textarea、select)之类的表单元素通常可以自己 维护 state，并根据用户的输入进行更新。而在 React 中，可变状态(mutable state)通常保存在组件的 state 属性中，并且只能通过 setState()来更新。 在 此，我们将 React 的 state 作为唯一的数据源，通过渲染表单的 React 组件来控制用 户输入过程中表单发送的操作。 这个“被 React 通过此种方式控制取值的表单输入元 素”被成为受控组件。

#### 不受控制组件(自身维护的状态值变化)

从字面意思来理解:不被 React 组件控制的组件。在受控制组件中，表单数据由 React 组件处理。其替代方案是不受控制组件，其中表单数据由 DOM 本身处理。文件 输入标签就是一个典型的不受控制组件，它的值只能由用户设置，通过 DOM 自身提供 的一些特性来获取。

### 6、constructor 中的 super 和 props 参数一起使用的目的是什么

在调用方法之前，子类构造函数无法使用 this 引用 super()
在 ES6 中，在子类的 constructor 中必须先调用 super 才能引用 this 在 constructor 中可以使用 this.props

### 7、setState 到底是同步还是异步

setState 是没有异步处理的，但是从表现上看是异步的，这个是因为本身的实现机制 的原因，为了优化做了多次更改的合并
React 会将多个 setState 的调用合并为一个来执行，也就是说，当执行 setState 的时候，state 中的数据并不会马上更新 因为他并不是异步处理

```javascript
// 在异步里面
state = {
    number:1
};
componentDidMount(){
    setTimeout(()=>{
      this.setState({number:3})
      console.log(this.state.number)
},0) }
// 在原生事件里面
state = {
    number:1
};
componentDidMount() {
    document.body.addEventListener('click', this.changeVal, false);
}
changeVal = () => {
    this.setState({
number: 3 })
    console.log(this.state.number)
}
```

所以根据上面的例子我们可以得到结论:setState 本身并不是异步，只是因为 react 的性能优化机制体现为异步。在 react 的生命周期函数或者作用域下为异步，在原生 的环境下为同步

### 8、react 的 hook

#### Hook 是什么?

Hook 是一个特殊的函数，它可以让你“钩入” React 的特性，在不编写 class 的情 况下使用 state 以及其他的 React 特性

#### 什么时候我会用 Hook?

如果你在编写函数组件并意识到需要向其添加一些 state，以前的做法是必须将其它 转化为 class。现在你可以在现有的函数组件中使用 Hook
使用 hook 的原因?

- 【代码复用】Hook 使你在无需修改组件结构的情况下复用状态逻辑
- 【代码管理】复杂组件(被状态逻辑和副作用充斥)Hook 将组件中相互关联
   的部分拆分成更小的函数(比如设置订阅或请求数据)，而并非强制按照生命周期划分
- Hook 使你在非 class 的情况下可以使用更多的 React 特性

#### useState

```javascript
// useState
import React, { useState } from 'react';
function Example() {
// 声明一个叫 "count" 的 state 变量 const [count, setCount] = useState(0); return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
</div> );
}
```

#### useEffect

可以把useEffect Hook看成是componentDidMount、componentDidUpdate以及 componentWillUnMount 的函数组合  
如果要同步的话用到 useLayoutEffect

```javascript
// class 的方式 componentDidMount() {
  document.title = `You clicked ${this.state.count} times`;
}
componentDidUpdate() {
  document.title = `You clicked ${this.state.count} times`;
}
// hook 的方式
import React, { useState, useEffect } from 'react'; useEffect(() => {
  document.title = `You clicked ${count} times`;
});
```

#### Hook 规则

- 不要在循环，条件或嵌套函数中调用 Hook， 确保总是在你的 React 函数的 最顶层调用他们
- 只在 React 函数中调用 Hook，不要在普通的 JavaScript 函数中调用 Hook

### 9、15和16的diff

#### tree diff

跨层级的节点移动不会复用，直接删除新建  
对于不同的节点类型 react 会直接删除旧的节点，新建一个新节点

#### component diff

如果是同一个类型的组件，根据新节点的 props 去更新原来根节点的组件实例，触发 一个更新过程，重新对比虚拟 DOM 如果不是同类型的组件，则替换整个组件下面的所有子节点

#### element diff

当节点处于同一个层级的时候，三种方案:

- INSERT_MARKUP 插入
- MOVE_EXISTING 移动
- REMOVE_NODE 删除

在没有 key 的情况下，是按照顺序对比的;在有 key 的情况下有点考虑移动然后是添 加，在完成新集合的所有节点 diff 后，对老集合进行型循环遍历，如果新集合中没 有，老集合才有的，需要进行删除

#### 16中对于数组的 diff

第一轮遍历的核心逻辑是复用和当前节点索引一致的旧节点，一旦出现不能复用的情 况就跳出遍历。
当第一轮遍历结束后，会出现两种情况:

- newChild 已经遍历完，只需要把所有剩余的旧节点都删除即可。
- 旧节点已经遍历完了，开始第二轮遍历，只需要把剩余新的节点全部创建完毕即可。

找出可以复用的子节点，否则创建。原理是将旧节点放置在 map 结构中，循环新节点 看是否有匹配的项，匹配则在 map 中删除，并移动位置，循环结束后将 map 中还存 在的旧节点删除。  

重点:删除新建移动的操作都只是先做标记

### 10、Fiber 、Fiber tree

传统 diff 算法面对数量庞大层级复杂的节点业务，大量 dom 的更新占用过多的内存和 计算时间将导致渲染 UI 不流畅所以用户时间操作得不到及时反馈，所以利用了 fiber 的架构。

fiber 的目标:

- 把可以中断的工作拆分成小任务
- 对正在做的工作调整优先级次序、重做、复用上次（做一半）的成果
- 在父子任务之间从容切换，以支持react执行过程中的布局刷新
- 支持render返回多个元素
- 更好的支持error boundary

```javascript
DOM
真实 DOM 节点
-------
Instances
React 维护的 vDOM tree node  
-------
Elements
描述 UI 长什么样子(type, props)

// Instances 层增加的实例 
effect
每个 workInProgress tree 节点上都有一个 effect list
用来存放 diff 结果
当前节点更新完毕会向上 merge effect list(queue 收集 diff 结果)
---- 
workInProgress
workInProgress tree 是 reconcile 过程中从 fiber tree 建立的当前进度 快照，用于断点恢复
----
fiber
fiber tree 与 vDOM tree 类似，用来描述增量更新所需的上下文信息
```

fiber 的本质上是一个链表  

参考文章:

fiber   
http://www.ayqy.net/blog/dive-into-react-fiber/ https://segmentfault.com/a/1190000018250127 https://www.zhihu.com/question/49496872

链表   
https://blog.csdn.net/weixin_42312342/article/details/88957617

### 11、react 的优化

两个方向，一个是减少 render 的次数，一个是减少 diff 计算的量。函数式组件每次 render 都会从新执行调用函数;类组件里面主要是用 shouldComponentUpdate 生命周 期和 PureComponent 组件去减少 render 的次数

- React.memo:等同于 PureComponent，用它包裹子组件，当父组件需要重新 render 的时候，如果传给自己的 props 不变，就不会触发重新 render。 memo 可以添加第二个参数，是个函数，参数为前后 props，返回 true 不需 要重新 render。
- useCallback:应用场景是父组件向子组件传递方法，当父组件重新渲染时， 代码都会重新执行。所以就算子组件包裹了 React.memo，也会重新渲染。可 以通过 useCallback 进行记忆传递的方法，并将记忆的方法传递给子组件。
- useMemo:如果在组件有个变量的值需要大量的计算才可以得出，因为函数组 件重新渲染就会重新执行代码，所以该变量的值也会重新计算，就可以 useMemo 做计算结果缓存。

### 12、react-router 的原理

hash 路由:核心是监听了 load 和 onHashChange 事件，在页面刷新或者 URL hash 改变时渲染不同页面组件。  
history API 路由:核心是通过 replaceState 和 pushState 去改变页面 URL，通过 popState 事件监听 history 对象改变的时候改变页面

### 13、react-redux

#### 什么是 redux

redux 只是一个实现了 Flux 思想的数据流框架。所谓 Redux，就是将动作(action) 变换成 state 转换函数(reducer)，然后放到一个统一的地方(store)来 setState 而 已。它具有单向性，唯一性和可预测性

![WechatIMG33.png](https://upload-images.jianshu.io/upload_images/7900525-9c82de787332da41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 如何流转的

- View触发dispatch
- 进入reducer，修改store中的state
- 将新的state和props传入handleChange中，生成更符合页面的props 4. 传给原始根节点重新render

![WechatIMG32.png](https://upload-images.jianshu.io/upload_images/7900525-bf298b5ad5d5334b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 为什么

- Redux 把流程规范了
- 更加注重数据的单一流向性，所有的 Component
- 让程序只拥有一个 Listener，不需要在每一个组件中进行自己的 state 管理，
一切所需的数据都从上游作为 props 传进來了
- 与后端做同构也是相当好的选择

参考  
https://alisec- ued.github.io/2016/11/23/%E5%9B%BE%E8%A7%A3Redux%E6%95%B0%E6%8D%AE%E6%B5%81 (%E4%B8%80)/

### 14、connect 的原理

connect 方法是一个高阶组件，主要的两个参数都是函数，命名为 mapStateToProps 和 mapDispatchToProps，内部原理是获取 store 添加订阅后，将 state 和 dispatch 分别传入上面两个方法，返回需要的 state 和 改变 state 的方法添加到 UI 组件的 props 上

```javascript
const mapStateToProps = (state) => {
  return {
    admin: state.app.hasAdmin,
  };
};
// getOverallInvestmentList 为后端提供的一个方法 const mapDispatchToProps = (dispatch) => ({
  getOverallInvestmentList() {
    return dispatch(getOverallInvestmentList());
}, });
export default connect(
  mapStateToProps,
  mapDispatchToProps,
)(OperateList);
```

备注:前提是在应用顶层已经使用 provider 组件，并应用初始化时创建 store。如下

```javascript
import { Provider } from 'react-redux';
import { LocaleProvider } from 'antd';
import { Router } from 'react-router-dom';
import App from './app';
const Root = ()=>{
  return <Provider store={store}>
      <Router history={history}>
        <LocaleProvider locale={locale_cookie === locales.EN ? enUS :
zhCN}>
          <App />
        </LocaleProvider>
      </Router>
  </Provider>;
};
ReactDOM.render(<Root />, document.getElementById('root'));
```
