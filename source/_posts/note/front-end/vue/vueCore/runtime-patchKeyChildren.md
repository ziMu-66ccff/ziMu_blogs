---
title: Vue运行时模块-runtime-patchKeyChildren
subtitle: patchKeyChildren
link: runtime-patchKeyChildren
catalog: true
date: 2023-02-14 10:21:11
tags:
  - 前端
  - Vue
  - Vue原理刨析
  - Vue运行时模块刨析
  - patchKeyChildren
categories:
  - [笔记, 前端, Vue, Vue原理实现]
---

# 简单介绍

vue3 中对带有 key 的子节点是利用快速 diff 算法进行更新的

# 快速 diff 算法基本原理

1. 首先进行预处理，设置一个指向数组头部的指针`j`，再通过新旧数组的长度-1 来获取指向新旧数组尾部的指针`newEndIndex`, `oldEndIndex`。
2. 判断新旧头节点，新旧尾节点的 key 是否相同，相同则调用 patch 函数进行，并将指向头节点的指针`j`向下移动，指向尾节点的指针`newEndIndex`, `oldEndIndex`向上移动，以此循环，直到遇到`key`不相同的，停止循环
3. 如果旧数组已经遍历完毕，而新数组还没有，则把新数组剩下的节点全部挂载，如果新数组已经遍历完毕，旧数组还没有，就把旧数组剩下的节点全部卸载。
4. 如果都没有遍历但是已经停止循环了，那么就开始进行接下来的操作
5. 维护一个元素个数和新数组中剩下的没有处理的节点个数一样多的数组`source`，然后在这个数组中存储新节点的 key 对应的旧节点的索引，但是会先初始化这个数组，让其元素全为`-1`
6. 我们通常想到的是遍历新数组，然后在 for 循环里面嵌套 for 循环遍历旧数组，然后找到 key 一样的就把索引值存进数组`source`, 但是这样就出现了循环的嵌套，时间复杂度就有点高，所以 vue 采用了下面的方法来填充`source`数组
7. 首先维护一个对象`keyindex`,这个对象中 key 为新数组中没有处理的新节点的 key，value 为对应的索引值。遍历新数组，从而填充`keyindex`
8. 遍历旧数组，通过`keyindex`找到 key 对应的新数组中存储的新节点，从而填充`source`，并调用 patch 函数进行更新。然后将新节点的索引值记录下来赋值给`pos`变量。通过比较`pos` 和 `新节点的索引大小来判断是否需要移动`(旧数组是按顺序遍历的，那么对应的新节点的索引旧应该也是从大到小的，pos 代表的就是前一个新节点的索引值，如果新节点的索引值小于 pos，那么就说明需要移动， 就将标志变量`move`设置为`true`)
9. 如果没有找到对应的新节点，那就说明这个旧节点的 key 对应的新节点在新数组中并不存在，那么就卸载这个旧节点。
10. 如果`move = true`那么就说明有节点需要移动，则进行接下来的操作
11. 求`source`数组的最长递增子数组`list`，并且这个`list`的元素值为`source`数组的索引值。这意味着`list`中存储的`source`数组中对应的索引值对应的新节点是不需要移动的。
12. 设置两个尾指针`i`,`s`，一个指向`list`数组的最后一个元素，一个指向`source`数组的最后一个元素。
13. 开始判断 source 数组存储的元素是否为`-1`，为-1 则将对应的新节点挂载，不为-1 则进一步判断`source[i]`是否和`s`是否相等，相等则说明不需要移动，就把指针`i`向上移动一位。不相等则说明需要移动，则把新节点对应的已经更新过了的旧节点对应的真实节点`el`插入到它后面一位虚拟 dom 对应的真实节点的前面

# 完整代码

```typescript
function patchkeyChildren(
  oldChildren: TypeVnode[],
  newChildren: TypeVnode[],
  container: VElement,
  anchor: VElement | VChildNode | null = null
) {
  let j = 0;
  let newStartVnode = newChildren[j];
  let oldStartVnode = oldChildren[j];
  let newEndIndex = newChildren.length - 1;
  let oldEndIndex = oldChildren.length - 1;
  let newEndVnode = newChildren[newEndIndex];
  let oldEndVnode = oldChildren[oldEndIndex];

  while (newStartVnode.key === oldStartVnode.key) {
    patch(oldStartVnode, newStartVnode, container);
    j++;
    newStartVnode = newChildren[j];
    oldStartVnode = oldChildren[j];
  }
  while (newEndVnode.key === oldEndVnode.key) {
    patch(oldEndVnode, newEndVnode, container, anchor);
    newEndIndex--;
    oldEndIndex--;
    newEndVnode = newChildren[newEndIndex];
    oldEndVnode = oldChildren[oldEndIndex];
  }
  if (oldEndIndex < j && newEndIndex >= j) {
    const anchorIndex = newEndIndex + 1;
    const anchor =
      anchorIndex < newChildren.length ? newChildren[anchorIndex].el : null;
    while (j <= newEndIndex) {
      patch(null, newChildren[j++], container, anchor);
    }
  } else if (newEndIndex < j && oldEndIndex >= j) {
    while (j <= oldEndIndex) {
      unmount(oldChildren[j++]);
    }
  } else {
    const count = newEndIndex - j + 1;
    const source = new Array(count);
    const keyindex: Record<any, any> = {};
    let moved = false;
    let pos = 0;
    let patched = 0;
    source.fill(-1);

    for (let i = j; i <= newEndIndex; i++) {
      keyindex[newChildren[i].key] = i;
    }
    for (let i = j; i <= oldEndIndex; i++) {
      const oldVnode = oldChildren[i];
      if (patched <= count) {
        const k = keyindex[oldVnode.key];
        if (typeof k !== 'undefined') {
          const newVnode = newChildren[k];
          patch(oldVnode, newVnode, container, anchor);
          patched++;
          source[k - j] = i;
          if (k < pos) {
            moved = true;
          } else {
            pos = k;
          }
        } else {
          unmount(oldVnode);
        }
      } else {
        unmount(oldVnode);
      }
    }
    if (moved) {
      const seq = getSequence(source);
      let s = seq.length - 1;
      let i = count - 1;
      for (i; i >= 0; i--) {
        if (source[i] === -1) {
          const pos = i + j;
          const newVnode = newChildren[pos];
          const nextPos = pos + 1;
          anchor =
            nextPos < newChildren.length ? newChildren[nextPos].el : null;
          patch(null, newVnode, container, anchor);
        } else if (source[i] !== s) {
          const pos = i + j;
          const newVnode = newChildren[pos];
          const nextPos = pos + 1;
          anchor =
            nextPos < newChildren.length ? newChildren[nextPos].el : null;
          container.insertBefore(newVnode.el as HTMLElement | Text, anchor);
        } else {
          s--;
        }
      }
    }
  }
}
```
