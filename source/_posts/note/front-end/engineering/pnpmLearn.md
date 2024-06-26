---
title: 为什么选择pnpm来替代npm，yarn
link: pnpmLearn
catalog: true
date: 2023-10-31 16:51:37
tags:
  - 前端
  - 前端工程化
  - pnpm
  - 包管理器
categories:
  - [笔记, 前端, 前端工程化]
---

# npm, yarn, 遇到了什么问腿？

为了得到这个问题的答案，我们需要对 npm，yarn 执行`npm install` or `yarn install`后，在`node_modules`文件夹里面是怎么管理依赖的。

## npm3 版本之前对依赖的管理

npm3 版本之前在生成的`node_modules`文件夹对依赖的管理是++嵌套结构++的

假设我们有一个项目，它依赖于 `b` 包，`c `包，`b` 包又依赖于 `d` 包和 `f` 包， `c` 包又依赖于 `d` 包和 `f` 包
依赖关系如下：
![pnpmLearn01.png](https://i0.imgs.ovh/2023/10/31/AIEns.png)

当我们运行`npm install`的时候，生成的`node_modules`文件夹中对依赖的管理会是嵌套结构的，如下：

```
node_modules
├── b
|   └── node_modules
|       └── d
|       └── f
├── c
|   └── node_modules
|       └── d
|       └── e
```

我们可以发现，是和依赖关系对应的嵌套结构

## npm3 版本之前对依赖的管理方式:嵌套式的 node_modules 文件结构的缺陷

1. 嵌套的可能会非常深，就像 `d` 又依赖于 `d1`，`d1` 又依赖于 `d2`，`d2` 又依赖于 `d3`,如此下去，嵌套的就会非常深，有的操作系统可能就难以支持了
2. 同一个项目里会出现依赖重复安装，我们可以看到 d 包是被安装了两次的，在 b 包的`node_modules`里被安装了一次，在`c`包的`node_nodules`又被重复安装了一次
3. 不同的项目里都依赖同一个依赖的时候，这个依赖在磁盘里会被重复安装。比如`x`项目和`y`项目都依赖于`z`包，那么`z`包就会在磁盘里被安装两次

## npm3 版本之后和 yarn 对依赖的管理

npm3 以后的版本和 yarn 生成的`node_modules`文件夹是++扁平结构的++
根据我们上面的项目例子，它的`node_modules`文件夹结构应该是如下的：

```
node_modules
├── b
├── c
├── d
├── f
└── e

```

我们可以发现所有的依赖都被拍平了，是扁平化的，都被提升到了 node_modules 文件夹下面，而不是嵌套的。

## npm3 版本之后和 yarn 对依赖的管理方式：扁平式的 node_modules 文件结构 解决了之前的哪些问题

1. 解决了嵌套的非常深的问题
   采用了扁平式的结构，完全不存在嵌套
2. 解决了同一个项目里依赖重复安装的问题
   我们可以看到`d包`在`node_modules`文件夹下面是只安装了一次的。
   那么`b包` or `c包`包要怎么找到他们依赖的`d`包呢，因为他们自己目录下没有`node_modules`，就会到上层目录里找`node_modules`，就可以找到项目根目录下面的`node_modules`，里面就有他们需要的`d`包
   ++ps：依赖的查找方式是先在自己包目录下的 node_modules 目录下面找，如果不存在 or 没找到，就到上层目录的 node_modules 目录找，以此方式，不断的往上找++

## npm3 版本之后 和 yarn 对依赖的管理方式：扁平式的 node_modules 文件结构 带来了哪些新的问题, 还存在哪些问题

- 带来的新的问题
  1. 带了了`幽灵依赖的问题`
     - 什么是幽灵依赖
       在我们上面的例子里，我们项目`a`只依赖于`b`和`c`，也就是说`package.json`里的`dependencies`只声明了`b`和`c`，但是因为扁平化的结构，我们可以在项目里使用`package.json`的`dependencies`里没有声明的`d`, `e`, `f`。`d`, `e`, `f`这三个依赖没有在项目目录的`package.json`里声明，但是却可以使用，这种依赖就被称为`幽灵依赖`
     - `幽灵依赖`会造成什么后果
       `d`是被`b`依赖的，然后因为`扁平化`的结构，我们才能使用，那么如果有一天`b`不在依赖于`d`了，那么我们一旦`npm install`，`node_modules`里就不会再有`d`了，而我们的项目代码还在使用`d`，那么马上就会报错
- 之前存在但是没有得到解决的问题
  1. 不同的项目里都依赖同一个依赖的时候，这个依赖在磁盘里会被重复安装。

# 为什么选择 pnpm（pnpm 的优势）

## pnpm 对依赖的管理

pnpm 对依赖的管理是一种扁平式和嵌套式相结合的，利用了软连接和硬链接的一种结构

++ps: 这里简单的讲解一下什么是软连接，什么是硬链接++

- 硬链接
  在操作系统的文件系统里，磁盘的文件都会有一个编号叫索引节点号(Inode Index)， 而硬链接就是文件名直接指向这个索引号，从而找到磁盘里的文件内容。并且可以存在多个文件硬链接同一个索引节点号。只删除一个连接并不影响索引节点本身和其它的连接，只有当最后一个连接被删除后，文件的数据块及目录的连接才会被释放。也就是说，文件真正删除的条件是与之相关的所有硬连接文件均被删除。
- 软连接
  软连接也叫符号链接，它是一个保存有其他文件位置信息的文件，指向的是其他文件的位置信息，而不是磁盘里的文件的索引节点号。所以一旦它指向的那个文件被删除，它就找不到了
- 详细的讲解请看这篇文章
  [ Linux 软连接和硬链接](https://www.cnblogs.com/itech/archive/2009/04/10/1433052.html)

依旧是我们上面的例子，它的`node_modules`文件夹结构应该是如下的：

```
node_modules
├── .pnpm
    └── b // 硬连接 指向的是磁盘里的b包
        └── node_modules
            └── d // 软连接 指向的是 .pnpm/d
            └── f // 软连接 指向的是 .pnpm/f
    └── c
        └── node_modules
            └── d // 软连接 指向的是 .pnpm/d
            └── e // 软连接 指向的是 .pnpm/e
    └── d // 硬连接 指向的是磁盘里的d包
    └── e // 硬连接 指向的是磁盘里的e包
    └── f // 硬连接 指向的是磁盘里的f包
    └── node_modules
        └── b // 软连接 指向的是.pnpm/b
        └── c // 软连接 指向的是.pnpm/c
        └── d // 软连接 指向的是.pnpm/d
        └── e // 软连接 指向的是.pnpm/e
        └── f // 软连接 指向的是.pnpm/f
├── b // 软连接 指向的是 .pnpm/b
├── c // 软连接 指向的是 .pnpm/c
```

看到这个文件结构的时候第一感觉肯定是，好复杂啊，一下子多了好多东西，没关系，我接下来会逐一介绍为什么会是这样的，然后你就会发现这个结构真的是十分巧妙

- node_modules

  1.  我们会发现`node_modules`目录下的包文件和我们项目里的`package.json`里`dependencies`声明的包是一摸一样的。这样就可以有效的避免`幽灵依赖的问题`, 项目里就不能直接使用`d`, `e` ,`f`这些幽灵依赖了，因为`node_modules`里面没有。++完美解决幽灵依赖的问题++

  2.  好，聪明的你马上就会问，`b`, `c`都有各自的依赖呀，可是`node_modules/b`,`node_modules/c`里面没有`node_modules`呀，那怎么找到他们的依赖呢。那么现在就告诉你真相，`node_modules/`下面的包文件全部都是`软连接`，他们都指向`node_modules/.pnpm/`目录下 对应的包，也就是`node_modules/.pnpm/b`， `node_modules/.pnpm/c`,他们下面就有对应的`node_modules`来存储他们各自对应的依赖啦。

- node_modules/.pnpm
  1.  我们会发现`node_modules/.pnpm`目录下拥有着我们项目所有的依赖，`b`, `c`, `d` ,`e`, `f`，并且是扁平化的，而且他们都是`硬链接`。都指向磁盘里的`b`，`c`，`d`, `e` ,`f`。这样当我们在其他项目里面也有这些依赖的时候，就不需要在磁盘里面重复安装，直接一个`硬连接`指向磁盘里对应的文件就可以。++完美解决不同项目里同样的依赖在磁盘里重复安装的问题++
- node_modules/.pnpm/包文件/node_modules

  1. 我们会发现`node_modules/.pnpm/包文件/node_modules/`目录下拥有者包所需要的依赖， 而这些依赖其实都是软连接，指向`node_modules/.pnpm`目录下对应的包文件。在这里就是`node_modules/.pnpm/b/`下面有着`d`, `f`, 但其实这个`d`, `f`都是软连接，指向`node_modules/.pnpm/d`, `node_modules/.pnpm/f`。这样虽然`b`, `c`都依赖于`d`, 但是却不会重复安装，而是都指向`node_modules/.pnpm/d`。 ++完美解决同一项目里依赖重复安装的问题++

- node_modules/.pnpm/node_modules
  1. 我们会发现`node_modules/.pnpm/node_modules`这个目录下存有项目的所有依赖。当然，都是软连接，指向`node_modules/.pnpm`下面的包，那么这个文件夹到底是什么作用呢。
  2. 我们都知道`幽灵依赖`是不安全的，是很容易导致问题的，但是现实是，依旧很多第三方包使用了幽灵依赖，而我们的 pnpm 也对幽灵依赖做了兼容，在特定的情况下是允许幽灵依赖的，那么幽灵依赖在哪找呢，就是这个目录下面，这个目录是扁平化的，拥有项目所有的包。 ++实现了对幽灵依赖的支持 但是最好不要这样 因为幽灵依赖并不安全++

## pnpm 的优势，解决了哪些问题

1. 通过软连接避免了`幽灵依赖`的问题
2. 通过硬连接解决了`不同项目的相同依赖`在磁盘重复安装的问题，提升了速度
3. 通过软连接， `node_modules/.pnpm` 的`扁平化`结构解决了`同一项目里相同依赖`重复安装的问题
4. 通过软连接和`node_modules/.pnpm/node_modules`目录，兼容了`幽灵依赖`
