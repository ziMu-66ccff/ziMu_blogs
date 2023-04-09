---
title: Vue运行时模块-runtime-processFragment
subtitle: 对Fragment类型的vnode的处理
link: runtime-processFragment
catalog: true
date: 2023-02-14 10:21:11
tags:
  - 前端
  - Vue
  - Vue原理刨析
  - Vue运行时模块刨析
  - processFragment
categories:
  - [笔记, 前端, Vue, Vue原理实现]
---

# Fragment 节点简单介绍

是一种模板节点，实际上并不会渲染，只会渲染它的子节点

# 执行 processFragment 来完成对 Fragment 类型的虚拟 DOM 的处理

- 通过`oldVnode.el`来获取 Fragment 的头锚点，如果没有就创建一个空文本节点作为头锚点，通过`oldVnode.anchor`来获取 Fragment 的尾锚点，如果没有就创建一个空文本节点来作为尾锚点。
- 如果`oldVnode.el`为 null，则执行挂载操作，先挂载头锚点，再挂载尾锚点，然后调用 patchChildren，并利用`anchor`参数来将子节点全部挂载到尾锚点的前面
- 如果不为 null，则执行更新操作，一样调用 patchChildren 函数，并利用`anchor`参数来对子节点进行更新。

# 完整代码

```typescript
function processFragment(
  oldVnode: TypeFragmentVnode | null | undefined,
  newVnode: TypeFragmentVnode,
  container: VElement,
  anchor: VElement | VChildNode | null = null
) {
  const fragmentStartAnchor = (newVnode.el = oldVnode
    ? (oldVnode.el as Text)
    : document.createTextNode(''));
  const fragmentEndAnchor = (newVnode.anchor = oldVnode
    ? (oldVnode.anchor as Text)
    : document.createTextNode(''));
  if (oldVnode == null) {
    container.insertBefore(fragmentStartAnchor, anchor);
    container.insertBefore(fragmentEndAnchor, anchor);
  } else {
    patchChildren(oldVnode, newVnode, container, fragmentEndAnchor);
  }
}
```
