---
title: Vue运行时模块-runtime-createApp（根组件挂载并初始化流程梳理）
subtitle: 根组件挂载并初始化流程梳理
link: runtime-createApp
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

# createApp 作用

1. 返回一个带有`mount`方法的对象`App`
2. 在`mount`方法里会调用`h`函数，来根据通过`createApp`方法传递的**组件对象**生成虚拟 DOM`vnode`，然后调用 render 函数来将`vnode`转换为真实 DOM，然后挂载在指定的 DOM 节点

# 完整代码

```typescript
import { isString } from '../utils/index';
import { render } from './render';
import { h } from './vnode';

export function createApp(rootComponent: Record<string, any>) {
  const app = {
    mount(rootContainer: HTMLElement | string) {
      if (isString(rootContainer)) {
        if (document.querySelector(rootContainer)) {
          rootContainer = document.querySelector(rootContainer) as HTMLElement;
        } else {
          return;
        }
      }
      render(h(rootComponent, null, null), rootContainer as HTMLElement);
    },
  };
}
```

# 执行 render 函数：将虚拟 DOM 节点转换为真实 DOM 节点并挂载到指定的容器上面

- 获取容器里之前的 vnode`preVnode`，然后调用`patch`函数来将虚拟 dom 转化为真实 DOM，并挂载到对应的 DOM 节点上面。
- 如果`preVnode`存在，现在的 vnode 为`null`, 则卸载`preVnode`对应的真实 dom。
- 最后将 vnode 赋值给容器的`_vnode`属性，作为下次执行时的之前的 vnode

# 完整代码

```typescript
export function render(vnode: TypeVnode | null, container: VElement) {
  const prevVnode = container._vnode;
  if (vnode) {
    patch(prevVnode, vnode, container);
  } else {
    if (prevVnode) unmount(prevVnode);
  }
  container._vnode = vnode;
}
```

# 执行 patch 函数，来通过 shapeFlag 根据虚拟 DOM 的不同类型调用不同的函数处理

- 对于 Element 节点，调用`processElement`方法进行处理
- 对于文本节点，调用`processText`方法进行处理
- 对于`Fragment`节点，调用`processFragment`方法进行处理
- 对于`Component`节点，调用`processComponent`方法进行处理

# 完整代码

```typescript
export function patch(
  oldVnode: TypeVnode | null | undefined,
  newVnode: TypeVnode,
  container: VElement,
  anchor: VElement | VChildNode | null = null
) {
  if (oldVnode && !isSameVnodeType(oldVnode, newVnode) && oldVnode.el) {
    anchor = oldVnode.anchor || oldVnode.el.nextSibling;
    unmount(oldVnode);
    oldVnode = null;
  }

  const { ShapeFlag } = newVnode;
  if (ShapeFlag & ShapeFlags.ELEMENT) {
    processElement(
      oldVnode as TypeElementVnode | null | undefined,
      newVnode as TypeElementVnode,
      container,
      anchor
    );
  } else if (ShapeFlag & ShapeFlags.TEXT) {
    processText(
      oldVnode as TypeTextVnode | null | undefined,
      newVnode as TypeTextVnode,
      container,
      anchor
    );
  } else if (ShapeFlag & ShapeFlags.FRAGMENT) {
    processFragment(
      oldVnode as TypeFragmentVnode | null | undefined,
      newVnode as TypeFragmentVnode,
      container,
      anchor
    );
  } else if (ShapeFlag & ShapeFlags.COMPONENT) {
    processComponent(
      oldVnode as TypeComponentVnode | null | undefined,
      newVnode as TypeComponentVnode,
      container,
      anchor
    );
  }
}
```

# 执行 processComponent 方法进行对组件虚拟 DOM 的处理

- 判断 oldVnode 是否为空，为空则执行`mountComponent`函数进行挂载
- 不为空则进行更新操作

# 完整代码

```typescript
function processComponent(
  oldVnode: TypeComponentVnode | null | undefined,
  newVnode: TypeComponentVnode,
  container: VElement,
  anchor: VElement | VChildNode | null = null
) {
  if (!oldVnode) mountComponent(newVnode, container, anchor);
  else {
    newVnode.component = oldVnode.component as Instance;
    newVnode.component.next = newVnode;
    (newVnode.component.update as EffectFn)();
  }
}
```

# 执行 mountComponent 方法来进行对组件虚拟 DOM 的挂载

- 创建组件实例，给组件实例添加一些属性

```typescript
const instance: Instance = (vnode.component = {
  props: {}, // 组件声明的props
  attrs: {}, // 传递给组件的，但是组件没有声明的属性
  setupState: null, // setup函数返回的数据对象
  ctx: null, // 传递给组件的render函数作为参数的数据对象
  subTree: null, // 虚拟dom树
  update: null, // 组件的更新函数
  isMounted: false, // 判断是否需要挂载的标志变量
  next: null, // 存储新的组件虚拟dom
});
```

- 调用 updateProps 方法来给组件实例添加 props，attrs

```typescript
function updateProps(instance: Instance, vnode: TypeComponentVnode) {
  const { type: Component, props: vnodeProps } = vnode;
  for (const key in vnodeProps) {
    if (Component.props?.includes(key)) {
      instance.props[key] = vnodeProps[key];
    } else {
      instance.attrs[key] = vnodeProps[key];
    }
  }
  // 对props进行响应式处理
  instance.props = reactive(instance.props);
}
```

- 调用组件对象的 setup 函数来获取返回的数据对象，并传递 props 和上下文对象
- 初始化 ctx 数据对象
- 利用`effect`实现组件的自更新
  1. 将组件的挂载与更新操作注册为副作用函数依赖，并设置调度器 scheduler，在调度器里面调用`queuejob`从而使多次相同的更新操作，只执行最后的一次。
  2. 手动调用`instance.update`函数来进行挂载，挂载过程中响应式数据也完成了对依赖的收集。
  3. 当响应式数据发生改变时就会触发收集的依赖，就会执行 update 来进行组件的更新，并且由于调度器的存在，多次相同的更新，只会执行一次

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
      // 判断是否是被动更新
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

instance.update();
```

# 完整代码

```typescript
export function mountComponent(
  vnode: TypeComponentVnode,
  container: HTMLElement,
  anchor: VElement | VChildNode | null = null
) {
  const { type: Component } = vnode;

  const instance: Instance = (vnode.component = {
    props: {}, // 组件声明的props
    attrs: {}, // 传递给组件的，但是组件没有声明的属性
    setupState: null, // setup函数返回的数据对象
    ctx: null, // 传递给组件的render函数作为参数的数据对象
    subTree: null, // 虚拟dom树
    update: null, // 组件的更新函数
    isMounted: false, // 判断是否需要挂载的标志变量
    next: null, // 存储新的组件虚拟dom
  });

  updateProps(instance, vnode);

  // 源码：instance.setupState = proxyRefs(setupResult)
  // TODO
  instance.setupState = Component.setup?.(instance.props, {
    attrs: instance.attrs,
    emit,
  });

  // 源码：对ctx做了一个代理，先再props里面找，再到setupSate里面找
  // TODO
  instance.ctx = {
    ...instance.props,
    ...instance.setupState,
  };

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

  instance.update();

  function emit(event: string, ...payload: any[]) {
    const eventName = `on${event[0].toUpperCase() + event.slice(1)}`;
    const handler = instance.props[eventName];
    if (handler) {
      handler(...payload);
    } else {
      console.error('事件处理函数不存在');
    }
  }
}

function updateProps(instance: Instance, vnode: TypeComponentVnode) {
  const { type: Component, props: vnodeProps } = vnode;
  for (const key in vnodeProps) {
    if (Component.props?.includes(key)) {
      instance.props[key] = vnodeProps[key];
    } else {
      instance.attrs[key] = vnodeProps[key];
    }
  }
  // 对props进行响应式处理
  instance.props = reactive(instance.props);
}

function fallThrough(instance: Instance, subTree: TypeVnode) {
  if (Object.keys(instance.attrs).length) {
    subTree.props = {
      ...subTree.props,
      ...instance.attrs,
    };
  }
}
```

组件的挂载和更新都会调用 patch 函数来进行对 subtree 虚拟 DOM 树的挂载与更新，调用的 patch 函数里针对不同类型的 DOM 函数设计的处理方法又会调用 patch 函数，以此递归，知道所有的节点全部挂载 or 更新完毕
