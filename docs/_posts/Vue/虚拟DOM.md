## 1. DOM的缺陷

我们如果深谙浏览器原理的话，可能会知道DOMAPI的诸多弊病，比如访问DOM元素的属性可能会触发`强制同步布局`，生成或者删除DOM元素更是会重新更新整条`渲染流水线`，因为浏览器中渲染进程只有一个主线程，所以这些操作都有可能会大大降低渲染效率

当然这些情况只针对复杂的页面，执行一次重排或者重绘都是非常耗时的

## 2. 什么是虚拟DOM

DOM要解决的问题

* 将页面改变的内容应用到虚拟DOM上，而不是直接应用到DOM上
* 变化被应用到虚拟DOM上，并不直接去渲染页面，而是调整虚拟DOM的状态，这样操作虚拟DOM的代价就很轻了
* 虚拟DOM收集到足够的改变，再一次性应用到真实的DOM上

> 虚拟DOM的定义，简单来说就是用JS去描述一个DOM节点

```
<div class="a" id="b">我是内容</div>

{
  tag:'div',        // 元素标签
  attrs:{           // 属性
    class:'a',
    id:'b'
  },
  text:'我是内容',  // 文本内容
  children:[]       // 子元素
}
```

以React为例

![](https://static001.geekbang.org/resource/image/cf/90/cf2089ad62af94881757c2f2de277890.png)

* 创建阶段。根据JSX创建虚拟DOM，然后由虚拟DOM创建真实DOM，再触发渲染流水线输出页面
* 更新阶段。如果数据发生了改变，则创建一个新的虚拟DOM，然后比较这两个树，找出变化的地方，并且将变化的地方一次性更新到真实的DOM上，最后渲染引擎更新渲染流水线，生成新的页面

## 3. 虚拟DOM的特点

* 充当JS和DOM之间的中间层，可以尽量避免JS频繁操作DOM引发的性能问题
* 本质上就是一个JS对象，通过diff算法比较新老DOM的差别，来达到最小化局部更新的目的
* 跨平台
* 比直接操作DOM拥有更好的性能，除此之外diff算法也节约了效率损耗
* 适合比较大型的项目，即比较复杂的页面

## 4. Vue中的虚拟DOM

### VNode类

首先，在Vue中，最基础的是VNode类，通过这些类，我们可以实例化出各种各样的虚拟DOM节点

完整的VNode类

```
class VNode {
    constructor (tag, data, children, text, elm) {
        constructor (
        tag?: string,
        data?: VNodeData,
        children?: ?Array<VNode>,
        text?: string,
        elm?: Node,
        context?: Component,
        componentOptions?: VNodeComponentOptions,
        asyncFactory?: Function
      ) {
        this.tag = tag
        this.data = data
        this.children = children
        this.text = text
        this.elm = elm
        this.ns = undefined
        this.context = context
        this.fnContext = undefined
        this.fnOptions = undefined
        this.fnScopeId = undefined
        this.key = data && data.key
        this.componentOptions = componentOptions
        this.componentInstance = undefined
        this.parent = undefined
        this.raw = false
        this.isStatic = false
        this.isRootInsert = true
        this.isComment = false
        this.isCloned = false
        this.isOnce = false
        this.asyncFactory = asyncFactory
        this.asyncMeta = undefined
        this.isAsyncPlaceholder = false
      }
    
      // DEPRECATED: alias for componentInstance for backwards compat.
      /* istanbul ignore next */
      get child (): Component | void {
        return this.componentInstance
      }
    }
}
```

其中比较重要的字段

* tag: 当前节点的标签名
* data: 当前节点的数据对象，具体包含哪些字段可以参考vue源码types/vnode.d.ts中对VNodeData的定义
* children: 数组类型，包含了当前节点的子节点
* text: 当前节点的文本，一般文本节点或注释节点会有该属性
* elm: 当前虚拟节点对应的真实的dom节点
* key: 节点的key属性，用于作为节点的标识，有利于patch的优化

### VNode节点

#### 空节点

```
function createEmptyVNode () {
    const node = new VNode();
    node.text = '';
    return node;
}
```

#### 注释节点

```
export const createEmptyVNode = (text: string = '') => {
  const node = new VNode()
  node.text = text
  node.isComment = true
  return node
}
```

#### 文本节点

```
// 创建文本节点
export function createTextVNode (val: string | number) {
  return new VNode(undefined, undefined, undefined, String(val))
}
```

#### 克隆节点

克隆节点主要是为了模板编译优化时使用的

```
function cloneVNode (node) {
    const cloneVnode = new VNode(
        node.tag,
        node.data,
        node.children,
        node.text,
        node.elm
    );
    return cloneVnode;
}
```

### 元素节点

这个节点是更通用的一类节点，拥有比如tag，data，children等属性的节点

## 5. DOM-Diff算法

VNode最大的用途就是在数据变化前后生成真实DOM对应的虚拟DOM节点，然后就可以对比新旧两份VNode，找出差异所在，然后更新有差异的DOM节点，最终达到以最少操作真实DOM更新视图的目的

那么如何对比新旧两个VNode，这背后肯定是有一套算法支撑的，这套算法就是大名鼎鼎的diff算法

在Vue中，这样的过程称之为`patch`。其实如果先不考虑看这个算法的具体实现，给一个JS描述的虚拟DOM树，让我们对比两个树，并且尽可能完成最小的更新，我们会怎么设计呢？很简单，旧的没有新的有，则新增，新的没有旧的有则删除，新旧都有，则再进一步递归更新

diff算法也是这样的思路，再复杂的算法背后其实也是`新增`，`删除`，`更新`三个核心的操作在支撑

宏观来看，diff算法大致分为如下的步骤（我们从两个根VNode开始遍历），注意下面的针对old的操作实际上都是真实的DOM操作，因为old本质就是DOM的数据化表达

* 如果new和old完全一致，则直接复用（完全一致通过key和selector判断 ）
* 如果new和old都属于static节点，则直接复用
* 如果new和old都属于文字节点，且两者的text不一样，则删除old的文字，并且用new的文字替代
* 如果new和old都是元素节点且存在子节点，则进入两者的子节点的patch
* 如果只有new有子节点，首先我们看old是否为text节点，如果是的话清空old的text内容，然后将new的子节点添加至old中
* 如果只有old有子节点，直接清空old中的子节点

### 如何patch子节点

diff算法最为核心的部分在于如果新旧虚拟DOM树都包含子节点，应该采取怎样的策略去更新

最简单的办法是双重循环，即每遍历一个新节点时都会遍历一遍老虚拟DOM树中寻找存不存在相同的节点，这样做很简单，但是也会带来巨大的性能开支

然后我们再说一下Vue中的diff算法，它相比传统的diff算法最大的变化就在于它只进行同级比较，因为一般的跨层级的DOM节点变化是很少的，所以Vue中忽略了这一点，节省了大量的计算需要。

首先Vue中为了避免双重循环，使用了四个指针来表示oldStart，oldEnd，newStart，newEnd。我们假设现在有两个VNode序列，分别是newVNodes和oldVNodes

首先我们需要对四个节点进行比对

* newStart和oldStart对比，如果符合sameVNode，则直接进行patchVNode操作，即对根节点的那个操作一样
* 然后是newEnd和oldEnd的对比
* newEnd和oldStart的对比，如果符合sameVNode，则将真实DOM中的oldStart位置的节点移动至对应newEnd的地方
* newStart和oldEnd的对比，如果符合sameVNode，则将真实DOM中oldEnd位置的节点移动到对应newStart的地方
* 如果上述四种匹配都不满足，则再次采用双重循环的过程

注意`sameVNode`的判定标准，它包含key一致，tag一致，isComment一致和data一致，还有input类型一致。

一定要注意这里的key一致，也就是说两个节点都没有key的话也是符合要求的，这也就是很多情况下在列表渲染中会出现`就地复用的情况`，因为对于两个列表节点没有key的情况，在上述的匹配过程中第一项就匹配上了，于是就出现了这样的情况

[1,2,3,4] => [1,2,4,3]，对于新数组的第三项4，diff算法会认为它是从3变成了4，对于第四项3，diff算法则是反过来，换句话说，没有用key绑定的话，3和4两个列表节点diff并不会复用而是选择原地复用！



