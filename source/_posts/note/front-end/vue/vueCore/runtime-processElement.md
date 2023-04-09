---
title: Vue运行时模块-runtime-processElement
subtitle: 对Element类型的vnode的处理
link: runtime-processElement
catalog: true
date: 2023-02-14 10:21:11
tags:
  - 前端
  - Vue
  - Vue原理刨析
  - Vue运行时模块刨析
  - processElement
categories:
  - [笔记, 前端, Vue, Vue原理实现]
---

# 执行 processElement 函数来对 Element 类型的 vnode 做处理

- 判断 oldVnode 是否为 null，是则执行 mountElement 函数对 vnode 进行挂载
- 不是则执行 patch 函数来对 vnode 做更新

# 完整代码

```typescript
function processElement(
  oldVnode: TypeElementVnode | null | undefined,
  newVnode: TypeElementVnode,
  container: VElement,
  anchor: VElement | VChildNode | null = null
) {
  if (!oldVnode) {
    mountElement(newVnode, container, anchor);
  } else {
    patchElement(oldVnode, newVnode);
  }
}
```

# 执行 mountElement 函数来对 Element 类型的 vnode 做挂载

- 根据 type 属性渲染真实 dom, 并赋值给 el
- 调用 patchProps 函数来给真实 dom 添加 props
- 通过 shapeFlag 来判断子节点是数组还是文本节点，如果是文本节点，则通过`textContent`属性直接挂载。如果是数组，则调用 mountChildren 函数进行挂载。（mountChildren 则是对数组做遍历以对每个元素调用 patch 函数进行挂载）

# 完整代码

```typescript
function mountElement(
  vnode: TypeElementVnode,
  container: VElement,
  anchor: VElement | VChildNode | null = null
) {
  const { type, props, ShapeFlag, children } = vnode;
  const el = (vnode.el = document.createElement(type));

  if (ShapeFlag & ShapeFlags.TEXT_CHILDREN) {
    el.textContent = children as string;
  } else if (ShapeFlag & ShapeFlags.ARRAY_CHILDREN) {
    mountChildren(children as TypeVnode[], container, anchor);
  }

  if (props) {
    patchProps(el, null, props);
  }
  container.insertBefore(el, anchor);
}
```

# 执行 patchElement 函数来对 Element 类型的虚拟 DOM 进行更新

- 将旧虚拟 DOM 的 el 属性赋值给新虚拟 DOM 的 el 属性
- 调用 patchProps 来更新属性
- 调用 patchChildren 来更新子节点

# 完整代码

```typescript
function patchElement(oldVnode: TypeElementVnode, newVnode: TypeElementVnode) {
  newVnode.el = oldVnode.el as HTMLElement;
  // patchProps
  patchProps(newVnode.el, oldVnode.props, newVnode.props);
  patchChildren(oldVnode, newVnode, newVnode.el);
}
```

# 执行 patchProps 来更新属性

- 遍历 newProps 的 key，通过 key 来获取对应的新旧属性值`next`, `prev`如果不相同则调用`patchDomProp`函数来进行属性的更新
- 遍历 oldProps 的 key，看 newProps 中是否有该 key，没有的话，则调用`patchDomProp`函数来进行属性的卸载。

# 完整代码

```typescript
export function patchProps(
  el: HTMLElement,
  oldProps: Record<string, any> | null,
  newProps: Record<string, any> | null
) {
  if (oldProps === newProps) return;
  oldProps = oldProps || {};
  newProps = newProps || {};
  for (const prop in newProps) {
    if (prop === 'key') continue;
    const prev = oldProps[prop];
    const next = newProps[prop];
    if (prev !== next) {
      patchDomProps(el, prop, prev, next);
    }
    for (const prop in oldProps) {
      if (prop != 'key' && !(prop in newProps))
        patchDomProps(el, prop, prev, null);
    }
  }
}
```

# 执行 patchDomProp 函数来进行对属性的更新

判断 key 是否为`class`, `style`, `eventName`, `传递空字符串则设置为true的布尔值属性名`，`自定义属性`，并进行相应的处理

# 完整代码

```typescript
function patchDomProps(el: HTMLElement, key: string, prev: any, next: any) {
  switch (key) {
    case 'class':
      el.className = next || '';
      break;
    case 'style':
      if (!next) {
        el.removeAttribute('style');
      } else {
        for (const styleName in next) {
          el.style[styleName as any] = next[styleName];
        }
        if (prev) {
          for (const styleName in prev) {
            if (!(styleName in next)) {
              el.style[styleName as any] = '';
            }
          }
        }
      }
      break;
    default:
      if (/^on[^a-z]/.test(key)) {
        if (prev !== next) {
          const eventName = key.slice(2).toLowerCase();
          if (prev) {
            el.removeEventListener(eventName, prev);
          }
          if (next) {
            el.addEventListener(eventName, next);
          }
        }
      } else if (domPropsRE.test(key)) {
        if (next === '' && typeof (el as any)[key] === 'boolean') {
          next = true;
        }
        (el as any)[key] = next;
      } else {
        if (next == null || next == false) {
          el.removeAttribute(key);
        } else {
          el.setAttribute(key, next);
        }
      }
      break;
  }
}

const domPropsRE = /[A-Z]|^(value|checked|selected|muted|disabled)$/;
```

# 执行 patchChildren 来更新子节点

- 判断新子节点是否为文本节点，如果是文本结点的话，则判断旧子节点是否为数组，为数组则遍历数组将其一一卸载，如果不是数组，则通过`textCotent`属性，完成新子节点(文本节点)的更新
- 判断新的子节点是否为 null, 如果为 null， 则判断旧子节点是否为数组，为数组则遍历数组将其一一卸载，如果不是数组，则通过`textCotent`属性，来将其设置为 null。
- 判断新的子节点是否为数组，如果是数组的话，判断旧子节点是否为 null 或者 文本节点，如果为 null or 文本节点的话，通过`textCotent`属性将其设置为 null。如果旧子节点为数组的话则判断其元素是否有 key 属性，有的话调用`patchKeyChildren`函数来进行更新，如果没有 key 则调用`patchUnKeyedChidren`函数来进行更新

# 完整代码

```typescript
function patchChildren(
  oldVnode: TypeElementVnode | TypeComponentVnode | TypeFragmentVnode,
  newVnode: TypeElementVnode | TypeComponentVnode | TypeFragmentVnode,
  container: HTMLElement,
  anchor: VElement | VChildNode | null = null
) {
  const { ShapeFlag: prevShapeFlag, children: oldChildren } = oldVnode;
  const { ShapeFlag, children: newChildren } = newVnode;

  if (ShapeFlag & ShapeFlags.TEXT_CHILDREN) {
    if (prevShapeFlag & ShapeFlags.ARRAY_CHILDREN) {
      unmountChildren(oldChildren as TypeVnode[]);
    }
    if (oldChildren !== newChildren) {
      container.textContent = newChildren as string;
    }
  } else if (ShapeFlag & ShapeFlags.ARRAY_CHILDREN) {
    if (prevShapeFlag & ShapeFlags.ARRAY_CHILDREN) {
      if (
        (oldChildren as TypeVnode[])[0] &&
        (oldChildren as TypeVnode[])[0].key &&
        (newChildren as TypeVnode[])[0] &&
        (newChildren as TypeVnode[])[0].key
      ) {
        patchkeyChildren(
          oldChildren as TypeVnode[],
          newChildren as TypeVnode[],
          container,
          anchor
        );
      } else {
        patchUnkeyChildren(
          oldChildren as TypeVnode[],
          newChildren as TypeVnode[],
          container,
          anchor
        );
      }
    } else {
      container.textContent = '';
      mountChildren(newChildren as TypeVnode[], container, anchor);
    }
  } else {
    if (prevShapeFlag & ShapeFlags.ARRAY_CHILDREN) {
      mountChildren(newChildren as TypeVnode[], container, anchor);
    }
    container.textContent = '';
  }
}
```

# 执行 patchUnKeyedChildren 函数来进行对没有 key 的子节点的更新

- 获取新的子节点数组的长度和旧的子节点数组的长度
- 利用 for 循环来进行新旧子节点的更新
- 如果新子节点数组的长度大于旧子节点数组的长度，则对多出来的新节点进行挂载
- 如果旧子节点数组的长度大于新子节点数组的长度，则对多出来的旧节点进行卸载

# 完整代码

```typescript
function patchUnkeyChildren(
  oldChildren: TypeVnode[],
  newChildren: TypeVnode[],
  container: VElement,
  anchor: VElement | VChildNode | null = null
) {
  const oldLength = oldChildren.length;
  const newLength = newChildren.length;
  const commonLength = Math.min(oldLength, newLength);
  for (let i = 0; i < commonLength; i++) {
    patch(oldChildren[i], newChildren[i], container, anchor);
  }
  if (newLength > oldLength)
    mountChildren(newChildren.slice(commonLength), container, anchor);
  if (oldLength > newLength) unmountChildren(oldChildren.slice(commonLength));
}
```

# 执行 patchKeyChildren 函数来进行对有 key 的子节点的更新

**这部分的讲解请看我的另外一篇文章**
