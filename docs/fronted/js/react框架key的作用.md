# 深入理解 key（react）
## 一 react 组件元素的 diff 算法
![屏幕快照 2019-10-08 下午11.55.39](https://shenggao.oss-cn-beijing.aliyuncs.com/blog/2020/02/29/ping-mu-kuai-zhao-20191008-xia-wu115539.png)

## 二 key 的理解
### 概述 
react 中的key 属性，它是一个特殊的属性，它的出现不是给开发者用的（例如你为一个组件设置key之后不能获取组件的这个key props），而是给react自己用的。
### key 的作用
key不是用来提升react的性能的，不过用好key对性能是有帮组的。
react 利用 key 来识别组件，它是一种身份标识标识。每个 key 对应一个独立的组件，在前后渲染过程中，相同的 key会被 react 认为是同一个组件。
* key 相同，会重新渲染 props 属性。
* key 不同，即在下次渲染中有新的 key 值出现，会销毁之前的组件，然后重新创建该组件。

### key 的使用场景
#### 1 用数组动态创建子组件的情况。
   **尽量不用 index 作为列表的唯一 key。**
```jsx
{this.state.data.map((v,idx)=><Item key={idx} v={v} />)}
// 开始时：['a','b','c']=>
<ul>
    <li key="0">a <input type="text"/></li>
    <li key="1">b <input type="text"/></li>
    <li key="2">c <input type="text"/></li>
</ul>

// 数组重排 -> ['c','b','a'] =>
<ul>
    <li key="0">c <input type="text"/></li>
    <li key="1">b <input type="text"/></li>
    <li key="2">a <input type="text"/></li>
</ul>
```
分析：
1. 重新排序后，组件重新生成虚拟 dom,然后新旧虚拟 dom 进行 diff 对比。
2. 拿 key 为 0 的组件举例。获取到新虚拟 dom 中 key 为 0 的组件，然后从旧的虚拟 dom 中找到 key 为 0 的组件。由同一个 key 可判断是同一个组件进行 diff。react 会保证组件的实例不变。只更新 props 值。然后递归遍历他的子节点，发现只有文本节点前后不一致，调用 componentWillReceiveProps -> componentWillUpdate -> render 依次更新组件的属性。
3. 因为 input 元素在对比中并没有变化，且其是非受控组件，又与父组件传入的任一props没有关联，所以其输入的 value 值会保存在 input 中，导致排序后，内容和组件的顺序错乱。

**key的值要稳定唯一**
1. 不能使用random来生成key的值。
    * random 函数的不确定性，会导致同一组件的 key 值渲染前后不一致。react 认为其不是同一个组件，会先将其从 dom 中移除，然后再重新生成 dom。不会重新渲染 props。性能开销相对较大。
2. 可以自定一个 count 解决。每次数组有一个子项，便向子项数据中加入 key 这一属性，遍历时将 key 赋值给属性。保证 key 值得唯一性。

#### 2 为一个复杂逻辑组件添加一个 key,然后再次渲染时修改它的 key。以达到先销毁该组件实例，再去重新渲染该组件的目的。实为复原组件的默认状态。
### key 的其他注意事项
* key值的唯一是有范围的，即在数组生成的同级同类型的组件上要保持唯一，而不是所有组件的key都要保持唯一。
* 不仅仅在数组生成组件上，其他地方也可以使用key，主要是react利用key来区分组件的，相同的key表示同一个组件，react不会重新销毁创建组件实例，只可能更新；key不同，react会销毁已有的组件实例，重新创建组件新的实例。