---
title: Vue运行时模块-runtime-updateComponent（组件的被动更新）
subtitle: 组件的被动更新
link: runtime-updateComponent
catalog: true
date: 2023-02-14 10:21:11
tags:
  - 前端
  - Vue
  - Vue原理刨析
  - Vue运行时模块刨析
  - createApp
categories:
  - [笔记, 前端, Vue, Vue原理实现]
---

# 组件的被动更新发生的场景

父组件里嵌套了子组件，父组件在进行自更新的时候，会调用 patch 函数来更新`subtree`，subtree 更新的过程中就会调用 patch 函数来进行对子组件的更新，patch 函数发现是对组件的更新，就会调用 processComponent 函数，Component 函数就会调用 updateComponent 函数来进行对组件的更新

# 执行 updateComponent 来完成组件的被动更新

- 将旧虚拟 DOM`oldVnode`的组件实例赋值给新虚拟 DOM 的组件实例
- 将新虚拟 DOM 赋值给组件实例的`next`属性
- 调用组件实列的 update 函数进行被动更新

# 完整代码

```typescript
function updateComponent(
  oldVnode: TypeComponentVnode,
  newVnode: TypeComponentVnode
) {
  newVnode.component = oldVnode.component as Instance;
  newVnode.component.next = newVnode;
  (newVnode.component.update as EffectFn)();
}
```

# 执行组件实例上的 update 函数来进行组件的被动更新

- 发现组件实例上的`next`属性不为空，则说明是被动更新，进行接下来的操作
- 因为是被动更新，是父组件中的响应式数据发送改变引起的，而父组件的响应式数据可能传递给了子组件的 props，所以调用 updateProps 来进行对子组件的 props，attrs 的更新，并更新 ctx 数据对象
- 接下来走正常更新流程，调用子组件的 render 函数来获取 vnode，并调用 normalizeVnode 来使 vnode 标准化（因为子组件的 render 函数返回的可能是一组 vnode，需要将这一组 vnode 转换成 fragment 类型的 vnode）。调用 fallThrough 函数给 subtree 添加 props，调用 patch 函数渲染 subtree

# 完整代码

```typescript
instance.update = effect(
  () => {
    if (!instance.isMounted) {
      instance.subTree = normalizeVnode(Component.render(instance.ctx));
      fallThrough(instance, instance.subTree);
      patch(null, instance.subTree, container, anchor);
      instance.isMounted = true;
      vnode.el = instance.subTree.el;
    } else {
      if (instance.next) {
        vnode = instance.next;
        instance.next = null;
        updateProps(instance, vnode);
        instance.ctx = {
          ...instance.props,
          ...instance.setupState,
        };
      }

      const prev = instance.subTree;
      instance.subTree = normalizeVnode(Component.render(instance.ctx));

      fallThrough(instance, instance.subTree);

      patch(prev, instance.subTree, container, anchor);
      vnode.el = instance.subTree.el;
    }
  },
  {
    lazy: true,
    scheduler(fn) {
      queueJob(fn);
    },
  }
) as EffectFn;
```
