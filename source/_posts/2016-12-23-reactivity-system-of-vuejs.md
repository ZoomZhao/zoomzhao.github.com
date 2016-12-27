---
layout: post
title: Vuejs 中的 Reactivity 的实现
---

Reactivity (Vue 中文文档中翻译为响应式) 是 Vue 的非常关键的一部分。通过 Model 与视图的双向绑定，让使用者可以简单的更新页面状态，不再需要直接操作 DOM 进行处理。

除了 Reactivity，Vue 中还有很多其他的部分，包括：模版解析、VDOM、组件等。
为了更好的学习 Vue 中 Reactivity 的实现，将其中 Reactivity 相关部分拿出来成为了单独的部分。这样更容易单独的看这一部分的实现。
相关的代码在 [vue-observer](https://github.com/ZoomZhao/vue-observer)

<!-- more -->

运行：

```
npm install;
npm run dev;
```

通过访问 `example/base` 下的 `index.html`，并修改源码，了解其中的实现。


Reactivity 监控变化的处理流程参考 Vue 文档中给出的图片：

![vue-reactivity](https://cn.vuejs.org/images/data.png)


下面从文件介绍以及处理流程两个方面，介绍 Vue 中 Reactivity 的具体实现。

## 文件功能介绍

### instance 

instance 中相关的文件，主要是两个：

1.  `state.js`

    Vue 中初始化传入的 `props`，`data`，`compute`，`watch` 的逻辑

2.  `render.js`

     vue-observer 示例中将 Vue 的 `render` 方法作了精简，并将 `render` 方法的调用（其实是 `new Watcher()`）从 lifecycle 移到了 render.js 文件中。

### observer

vue-observer 的这部分代码，基本完全与 Vue 中的代码一致，是 Vue 中 Reactivity 实现的核心代码。主要包含以下几个文件

1. `inde.xjs`

    包含类 `Observer`，主要是为各个需要观察的目标埋点 （`new Dep()`）
    
2. `dep.js`

    包含类 `Dep`  （观察点 类）
    
3. `watcher.js`

    包含类 `Watcher`  （观察者 类）
    
4. `scheduler.js`

    观察者的调度逻辑，主要负责处理，多个 Watcher 需要被调用的情况。防止多次被调用以及在特定情况下按顺序调用。
    
5. `array.js`

    辅助方法，用于重写 Array 的一些方法，使 Array 的变化可以被观察。


其中的两个重要的类，属性方法如下：
#### Dep 类

**属性**

- `target`  静态属性，全局唯一，标识当前正在处理的 Watcher
- `id`  ID
- `subs`  Watcher 的列表


**方法**

- `addSub`  添加 Watcher
- `removeSub`  移除 Watcher
- `depend`  为自己添加当前 target Watcher，并更新 Watcher 的观察列表
- `notify`  通知所有 Watcher 更新


#### Watcher

根据传入的 expression，得到 value，传给 cb 进行处理。

**属性**

- `deep` 用于决定是否完整的轮训，来保证每一个属性都被重新计算依赖
- `user` 是否是用户创建的，是的话会尝试捕获 cb 调用时产生的异常，并提醒
- `lazy` 是否 new 之后不立即执行
- `sync` 是否需要在每次需要 update 的时候立即执行 （不设置的话会交由 scheduler 处理）
- `expression` 传入的方法或表达式
- `cb` 执行之后的回调函数
- `id` 
- `active` 是否当前激活状态
- `dirty` 是否是脏数据
- `deps` 与下面的三个一起，用来做 观察目标 （Dep）的动态更新
- `newDeps`
- `depIds`
- `newDepIds`
- `getter` 从 expression 来的 getter 方法
- `value` 当前的取值

**方法**

- `get` 默认初始化之后调用，调用传入的 expression，收集依赖，并调用 `cleanupDeps` 更新依赖关系
- `addDep` 把自身添加到制定 Dep 的观察列表中，另外如果依赖有变化则记录
- `cleanupDeps` 更新依赖关系
- `update` 更新
- `run` 具体的执行方法，负责调用 `get` 然后根据情况决定是否调用 `cb`
- `evaluate` 只是调用 `get`，用于清除 `dirty` 状态
- `depend` 把收集的 deps 统一进行处理，跟 `evaluate` 一样，都是为 `lazy` 的 Watcher 服务的
- `teardown` 销毁


### util + share 一些工具代码

一些工具方法

## Reactivity 相关处理流程

大概了解了各个文件的功能之后，我们从处理流程的角度，了解下 Vue 中 Reactivity 的具体处理过程。

Reactivity 的处理包含三个部分：

- 初始化观察目标，负责埋点
- 初始化 Watcher 并进行依赖收集，等待变更出现
- 出现变更，进行更新


### 初始化观察目标

初始化的时候（在 Vue 中 `new Vue` 或者创建 `Component`），根据传入的 `$options` 进行初始化处理。

- 初始化 `$options.props`
- 初始化 `$options.data`

  通过调用 `observer/index.js` 文件中的 `observer` 方法，进行埋点。

  - 每个对象（Object 和 Array）都会被分配个观察者 `Observer`
  
    主要是用于对象变更的观察，例如 Array 的 push 操作；另外一个使用的地方是 Vue.set ，可以在初始化之后依然提供观察的能力。
    这里有一个很厉害的细节，`Observer` 对象被放到了 Object 的 `__ob__` 属性中，既可以在拿到对象的时候都可以回去到对应的 `Observer`，还起到缓存的作用，防止多次观察同一对象，一举两得。

  - 每个属性也会被分配个观察者

  这里的观察者采用了闭包的方式，通过 Object.defineProperty 来处理 getter 和 setter。在 get 的时候，处理观察关系，将当前 Watcher 加入到属性的观察者列表中。set 的时候，通过 dep.notify() 触发观察者进行更新。

  这里也有个 childOb 的概念，如果当前的属性存在 childOb，那么 childOb 也要同时被当前 Watcher 观察。

### 初始化 Watcher 并进行依赖收集，等待变更出现

初始化 Watcher 分为三个部分：

- 初始化 $options.computed 的 Watcher

  Vue 中可以通过给 `$options` 传入 computed 对象来处理一些比较复杂的显示逻辑。这个时候就要根据所依赖的属性，进行观察和重新计算。computed 对应的 Watcher 通过 makeComputedGetter 方法创建，他的特点是，创建的为 Lazy Watcher，new 之后不会立即调用，在 computed 的属性被 get 的时候才需要重新计算。

  对于这类 Watcher，处理的流程为

  1. 初始化的时候，只设置 dirty 为 true，不直接执行传入的 expression。
  2. 每次 Dep 通过 update 的时候只设置 Watcher 的 dirty 为 true，标识需要重新计算。
  3. 在需要获取最新值得时候（这里就是 computed 属性被 get 的时候）调用 watcher.evaluate() 方法，计算 watcher 的最新值，并且手动调用 watcher.depend() 方法来重新计算依赖。

- 初始化 $options.watch 的 Watcher

  `$options.watch` 提供观察 `data` 的功能，在 `data` 被修改之后，触发提供的回调函数。

  具体的实现过程中，使用了 Vue 暴露出的全局 API：`Vue.$watch`，创建一个普通的 watcher，在指定的 expression 的发生改变的时候触发 cb。

- 初始化 render Watcher

  在 Vue 的处理流程中，会把 temple 编译为 render 函数（具体的实现会更加复杂），在 Vue Component 初始化完成的时候，会创建一个 Watcher(render, patchVDom)，根据传入的 render 函数创建依赖关系，如果发现变化，则 PatchVDom， 实现 View 的更新。

### 出现变更，进行更新

经过 Watcher 的初始化，Vue 中的 Watcher，分为三类（不包括用户调用 `Vue.$watch` 传入 sync 参数的情况）
    
- Computed Watcher（lazy Watcher），update 的时候只标记 dirty 状态，不会进入 queue（后面具体讲队列），属性被 get 的时候直接调用 `watcher.evaluate()` 进行取值。
- 普通 Watcher，包含用户通过 `$options.watch` 或 `Vue.$watch` 生成的 watch，在 update 的时候会进入 queue
- Component Watcher，render 方法作为 expOrFn 的 watcher，实现 Model -> DOM 的关键 Watcher，在 update 的时候会进入 queue

在 Model 被改变的时候，对应的 dep 被触发 notify 方法。dep 会调用所有 watcher 的 update 方法。

- Computed Watcher 的处理逻辑在 *初始化 $options.computed 的 Watcher* 部分已经做了描述
- 其他的 watcher （不包括设置 sync 为 true，下同）都会交由 queueWatcher 进行处理

想过逻辑存在于文件 `observer/scheduler.js`

queueWatcher 的作用主要有以下几点：

1. 同一个 watcher 只会被添加到列表中一次，防止被多次调用
2. 如果正在执行队列，则会判断当前插入的 watcher id，如果已经过了，则立即执行，否则插入到合适位置
3. 当前处理的所有 watcher 被插入到队列之后进行执行

执行队列的时候，会先对需要执行的 watcher 进行排序处理：

1. 大的方面，从父组件到子组件的方向执行，因为父组件总是先被创建的。
2. 用户定义的 watcher 先于 render watcher 被处理（用户定义的 watcher 可能会影响 Model 的取值）
3. 如果子组件被父组件干掉了，那么它会被跳过。

这个顺序也是 watcher 被创建的顺序，这样的话，按照 watcher 的 id 从小到大进行排列就可以了。

然后执行 排重 + 排序 之后的 watcher 队列，这样就保证了，每次数据变更，只会按顺序处理一次需要更新的 watcher。


----

这就是 Vue 中的 Reactivity 的具体实现。

可以看到，这部分的整体的实现真的是非常的好。

其中 Dep 和 Watcher 这两个类的设计非常优秀。在 Watcher 中存储了 Dep 的列表，也在 Dep 中存储了 Watcher 的列表。通过这样的设计，既可以在 Watcher 实例中，初始化及更新依赖关系（通过 Deps 和 newDeps），也可以在 dep 中直接通知所有的 watcher 进行更新。

scheduler 的引入很好的解决了什么时候调用 watcher 的问题，最大化的减少了 DOM 操作，再加上 VDOM，就完成了 Model 到 View 的高性能的映射。
 






​	

