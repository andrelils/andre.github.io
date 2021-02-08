# React源码学习笔记
## React API 简介
> 自从Facebook把React和ReactDOM分包发布之后，React就不仅仅是一开始的前端框架了。目前大部分的框架逻辑都放在了ReactDom里面。
本文的React版本是`V17.0.1`。借用React官网的一段话来介绍`什么是React`。

> React是一个用于构建用户界面的JavaScript库。

> 声明式： React使创建交互式UI变得轻松自如。为应用程序中的每个状态设计简单的视图，当数据更改时，React将有效地更新和呈现正确的组件。声明式视图使您的代码更具可预测性，更易于理解和易于调试。

> 基于组件：构建封装的组件，以管理其自身的状态，然后将它们组合成复杂的UI。由于组件逻辑是用JavaScript而不是模板编写的，因此您可以轻松地通过应用程序传递丰富的数据并将状态保持在DOM之外。

> 一次学习，随处编写：我们不会对您的其余技术栈做任何假设，因此您可以在React中开发新功能而无需重写现有代码。React还可以使用Node在服务器上进行渲染，并使用React Native来支持移动应用程序。

```javascript
export {
  Children = {
    map,  
    forEach,
    count,
    toArray,
    only,
  },
  createRef,
  Component,
  PureComponent,
  createContext,
  forwardRef,
  lazy,
  memo,
  useCallback,
  useContext,
  useEffect,
  useImperativeHandle,
  useDebugValue,
  useLayoutEffect,
  useMemo,
  useReducer,
  useRef,
  useState,
  useMutableSource, 
  useMutableSource as unstable_useMutableSource,
  createMutableSource,
  createMutableSource as unstable_createMutableSource,
  Fragment,
  Profiler,
  unstable_DebugTracingMode,
  StrictMode,
  Suspense,
  createElement,
  cloneElement,
  isValidElement,
  version,
  __SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED,
  createFactory,
  useTransition,
  useTransition as unstable_useTransition,
  startTransition,
  startTransition as unstable_startTransition,
  useDeferredValue,
  useDeferredValue as unstable_useDeferredValue,
  SuspenseList,
  SuspenseList as unstable_SuspenseList,
  unstable_LegacyHidden,
  unstable_createFundamental,
  unstable_Scope,
  unstable_useOpaqueIdentifier,
  unstable_getCacheForType,
  unstable_Cache,
  unstable_useCacheRefresh,
}  from 'React' 
````
## 必备知识点
- `$$typeof`是什么东西？是用来干什么的。
> 其实这个东西是为了来防止XSS攻击（不知道的自行百度）。React 0.14 修复手段是在虚拟DOM中添加`$$typeof`，使用 `Symbol` 标记每个 React 元素（element）：`Symbol`类型是非常重要的，因为JSON不支持 `Symbol` 类型。 所以即使服务器存在用JSON作为文本返回安全漏洞，JSON 里也不包含 `Symbol.for('react.element')`。React 会检测 `element.$$typeof`，如果元素丢失或者无效，会拒绝处理该元素。<br> 详情见: https://blog.csdn.net/z591102/article/details/108531302

## Children
这个对象提供了一系列处理props.children的方法，用来将`ReactNodeList`的处理成一个数组。这样就方便于我们自己去对children进行一系列操作。
 
 >  `React.children.map`的用法`React.Children.map(props.children,(child)=>{ })`。第一个参数是父组件的`props.children`，第二个参数是源码中定义的 `MapFunc` 返回一个回调函数，提供一个参数`child:?React$Node`。
## CreateRef
新的`ref`用法，React即将抛弃`<div ref="myDiv" />`这种`string ref`的用法，将来你只能使用两种方式来使用ref。`createRef`的源码如下：
```javascript
type RefObject = {|
  current: any,
|};

// an immutable object with a single mutable value
export function createRef(): RefObject {
  const refObject = {
    current: null,
  };
  if (__DEV__) {
    Object.seal(refObject);
  }
  return refObject;
}
```
例子如下：
```javascript
// Ref例子 class模式
class App extends React.Component{

  constructor() {
    this.appRef = React.createRef()
  }

  render() {
    return <div ref={this.appRef} />
    // or
    return <div ref={(current) => this.appRef = current} />
  }
  
}
```
## Component && PureComponent
这两个类是React的组件类。里面会提供一些方法，比如`Component.prototype.forceUpdate`和`Component.prototype.setState`。这两个类的区别在于`isPureReactComponent`属性不一样。`Component`的`isPureReactComponent`为`{}`。`PureComponent`的`isPureReactComponent`属性为`true`。这个属性用来判断是否是继承的`PureComponent`，如果是的话就`shallowEqual`比较新旧`state`和`props`。React中对比一个ClassComponent是否需要更新，如果是`Component`组件则是看有没有`shouldComponentUpdate`方法，然后默认渲染。而如果是`PureComponent`还需要比较比较新旧`state`和`props`。总结来说`PureComponent`是一个更有性能的`Component`版本。
```javascript
//Avoid an extra prototype jump for these methods.
Object.assign(pureComponentPrototype, Component.prototype);
pureComponentPrototype.isPureReactComponent = true;
```
## CreatContent
这是官方出的`context`方案。主要是提供了`Provide`和`Consumer`。
```javascript
// 源码中用到的方法
Object.defineProperties(Consumer, {...} // 主要是给Consumer对象新增或修改一些属性。
//例子
const {Provide,Consumer} = React.creatContent(defauleValue)

const ProvideDemo = (props)=>(
  // contextValue 就是传下去的上下文
  <Provide value={'contextValue'}>
     {props.children}
  </Provide>	
)
 // contextValue 就是传下去的上下文,Consumer会将上下文的数据作为参数传入renderProps渲染的函数之内
const ConsumerDemo = ()=(
<Consumer>
    {
      (contextValue)=>{return <span>{contextValue}</span>}
    }
 </Consumer>
)
```
## ForwardRef

`forwardRef`主要是为了来传递ref，将外部的props传入给包裹的组件。

## Lazy
懒加载，主要是用来异步加载一些组件。原理是通过`import`方法，`import`在`webpack`中最终会调用`requireEnsure`方法，动态插入script来请求js文件，类似jsonp的形式。`React.lazy`不能单独使用，需要配合`React.suspense`，`suspence`是用来包裹异步组件，添加`loading`效果等。
```javascript
const demo = React.lazy(()=>import('./component/Lazy'))

//在fallback里面处理loading效果等
<React.suspense fallback={<div>loading</div>}>
   <demo></demo>
</React.suspense>
```
 
## memo
`memo`其实和`PureComponent`类似。它是用来创建函数的时候使用的，而`PureComponent`是在创建类的时候使用的。所以`memo`大多数用于hook模式，而`PureComponent`大多数用于class模式。
```javascript
// memo
const children = function(){
   return '1'
}
const demo = React.memo()

//PureComponent
class child extend React.PureComponent{
  render(){}
}

```

## Hooks
源码在`React Dom`的`server`的`ReactPartialRendererHooks`中。

* useState
`useState`其实只是一个语法糖，底层的东西实际上是`useReduce`。<br>
 1、`createWorkInProgressHook`创建一个`workInProgressHook`对象用来判断是否是第一次渲染，如果是则需要进行初始化操作。这里初始化`initialState`，并且记录在`workInProgressHook.memoizedState`和`workInProgressHook.baseState`上
```javascript
   // 第一次渲染主要就是初始化操作
    let initialState;
    if (reducer === basicStateReducer) {
      // Special case for `useState`.
      initialState =
        typeof initialArg === 'function'
          ? ((initialArg: any): () => S)()
          : ((initialArg: any): S);
    } else {
      initialState =
        init !== undefined ? init(initialArg) : ((initialArg: any): S);
    }
     workInProgressHook.memoizedState = initialState;
    const queue: UpdateQueue<A> = (workInProgressHook.queue = {
      last: null,
      dispatch: null,
    });
    const dispatch: Dispatch<A> = (queue.dispatch = (dispatchAction.bind(
      null,
      currentlyRenderingComponent,
      queue,
    ): any));
    return [workInProgressHook.memoizedState, dispatch];
```
如果不是，则是re-render。我们在上面的初始化代码中可以看到`queue`对象其实很简单，就是一个`last`指针和`dispatch`方法。`dispatch`是用来记录更新`state`的方法的。
```javascript
// This is a re-render. Apply the new render phase updates to the previous
    // current hook.
    const queue: UpdateQueue<A> = (workInProgressHook.queue: any);
    const dispatch: Dispatch<A> = (queue.dispatch: any);
    if (renderPhaseUpdates !== null) {
      // Render phase updates are stored in a map of queue -> linked list
      const firstRenderPhaseUpdate = renderPhaseUpdates.get(queue);
      if (firstRenderPhaseUpdate !== undefined) {
        renderPhaseUpdates.delete(queue);
        let newState = workInProgressHook.memoizedState;
        let update = firstRenderPhaseUpdate;
        do {
          // Process this render phase update. We don't have to check the
          // priority because it will always be the same as the current
          // render's.
          const action = update.action;
          if (__DEV__) {
            isInHookUserCodeInDev = true;
          }
          newState = reducer(newState, action);
          if (__DEV__) {
            isInHookUserCodeInDev = false;
          }
          update = update.next;
        } while (update !== null);

        workInProgressHook.memoizedState = newState;

        return [newState, dispatch];
      }
    }
    // 对应我们的 const [state, updateState] = useState('defaultVal')
    return [workInProgressHook.memoizedState, dispatch];
```
- useCallBack、useMemo
`useCallBack`主要用于处理函数产生不必要的渲染。根据参数`deps`的内容是否改变，决定是返回存储的老方法，还是返回新的方法并记录。它其实也是一个语法糖，底层的东西是`useMemo`。
```javascript
export function useCallback<T>(
  callback: T,
  deps: Array<mixed> | void | null,
): T {
  return useMemo(() => callback, deps);
}
```
- useContext
 通过Context实例去获取上下文的值。
```javascript
// 获取到的上下文的值
const context = useContext（fidContext）

function useContext<T>(
  context: ReactContext<T>,
  observedBits: void | number | boolean,
): T {
  if (__DEV__) {
    currentHookNameInDev = 'useContext';
  }
  resolveCurrentlyRenderingComponent();
  const threadID = currentPartialRenderer.threadID;
  validateContextBounds(context, threadID);
  return context[threadID];
}
```
- useEffect
`effect` 函数将在 `componentDidAmount` 时触发和 `componentDidUpdate` 时有条件触发。可?以返回一个函数(returnFunction)，`returnFunction` 将会在 `componentWillUnmount` 时触发和在 `componentDidUpdate` 时先于 `effect` 有条件触发。<br>
与 `componentDidAmount` 和 `componentDidUpdate` 不同之处是，`effect` 函数触发时间为在浏览器完成渲染之后。 如果需要在渲染之前触发，需要使用 `useLayoutEffect`。<br>
第二个参数 array 作为有条件触发情况时的条件限制。<br>
如果不传，则每次 `componentDidUpdate` 时都会先触发 `returnFunction`（如果存在），再触发 `effect`。<br>
如果为空数组[]，`componentDidUpdate` 时不会触发 `returnFunction` 和 `effect`。<br>
如果只需要在指定变量变更时触发 `returnFunction` 和 `effect`，将该变量放入数组。

- useImperativeHandle
`useImperativeHandle`的核心其实也是`useLayoutEffect`的调用，接受一个父组件的ref参数，挂载到Component上。实现父组件通过ref可以调用子组件的方法和变量。
```javascript
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeMethods(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));
  return <input ref={inputRef} ... />;
}
FancyInput =  React.forwardRef(FancyInput);
```

