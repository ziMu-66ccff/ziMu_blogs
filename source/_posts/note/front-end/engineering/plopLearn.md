---
title: plop-一个基于模板的代码生成器
link: plopLearn
catalog: true
date: 2023-10-29 17:53:34
tags:
  - 前端
  - 前端工程化
  - plop
  - 脚手架
categories:
  - [笔记, 前端, 前端工程化]
---

# plop 是什么，为什么需要 plop

Plop is a little tool that saves you time and helps your team build new files with consistency.
这是官网对于 plop 的评价，实际上也确实是这样， plop 可以通过命令和用户配置的 hbs 模板文件来在指定的目录下生成对应的模板代码。想一想，我们只需要通过一个命令，就可以在我们需要的目录下生成对应的文件，里面有本来需要我们手写的结构代码，这该是一件多爽的事情，可以大大的节约我们的时间

# plop 体验

## 下载 plop(推荐全局下载)

```bash
pnpm install plop -g
```

## 编写模板代码

```
<!-- templates/sfc/index.vue.hbs -->

<script setup lang="ts">
{{> importVueRef}}
</script>

<template>
  <div class="{{componentName}}"></div>
</template>

<style scoped lang="less">
.{{componentName}}
{}
</style>
```

在根目录下新建一个文件夹存放我们对应的模板代码，plop 后面会根据这些模板代码来生成我们需要的代码

你可能已经发现了出现了`{{componentName}}`这样的代码 `{{}}`这个叫模板语法，你可能会说这不是和 vue 的模板语法是一样的嘛 是的 是一样的 因为 vue 的模板语法就是借鉴（抄）的这个 那么你应该会好奇 componentName 的值是什么呢 这个将会在命令里传递 请继续往下看

### 在项目根目录创建我们的 plopfile.js 文件

```
export default function (plop) {
  plop.setGenerator('createSFC', {
    description: 'create one SFC',
    prompts: [{ type: 'input', name: 'componentName', message: 'input componentName' }],
    actions: (data) => {
      return [
        {
          type: 'add',
          path: './src/{{camelCase componentName}}/index.vue',
          templateFile: './templates/sfc/index.vue.hbs',
        },
      ];
    },
  });
  plop.setPartial('importVueRef', `import {Ref} from 'vue' `);
}
```

简单讲解一下，我们用`plop`这个对象身上的`setGenerator`命令，配置了一个命令，`createSFC`是这个命令的名字（自己取），然后通过终端输入`plop createSFC`这个命令来使用。
`description`是对这个命令功能的描述，当我们输入`plop`命令的时候，终端会列出所有的 plop 命令和它的描述（就是我们这里 description 写的）.
`prompts`是一个数组，数组里面的元素是一个对象（用来描述终端提示语的）

- type
  这个提示的类型，这里是'input',
- name
  定义变量名(这里的变量名是 componentName)，用来存储用户将从终端输入的值(因为是 input 类型，所以是用户输入一个值)。
- message
  提示信息

`action`是这个命令具体将执行的操作，是一个数组，因为一个命令是可以执行多个操作的。

- type
  操作的类型，这里是'add', 也就是在指定目录下生成一个文件
- path
  生成的文件的路径，`'./src/{{camelCase componentName}}/index.vue'`， 这里又出现了插值语法，`componentName`是我们在`prompts`里面配置的用户输入的，我们假设输入的是'button-success'， 你可能注意到了前面还有一个`camelCase`，这个是一个`helper`，它的作用是把 componentName 的值变成驼峰形式，也就是说用户输入的'button-success'会变成'buttonSuccess'。也就是说实际上生成的文件路径会是`./src/buttonSuccess/index.vue`。
- templateFile
  我们写的模板代码，也就是 hbs 文件的路径，在这里就是生成的`index.vue`文件的代码将会是这个`'./templates/sfc/index.vue.hbs'`文件里的代码

## 在 package.json 文件的 script 里面配置脚本

```json
{
  "scripts": {
    "plop": "plop"
  }
}
```

## 执行命令

1. 执行`pnpm plop`命令
   ![plopLearn01.png](https://i0.imgs.ovh/2023/10/29/AVTPm.png)
2. 输入`componentName`
   ![plopLearn02.png](https://i0.imgs.ovh/2023/10/29/AVq6N.png)
3. src/buttonSuccess 目录下就生成了 index.vue 代码了
4. 查看 src/buttonSuccess/index.vue 的代码
   ![plop03.png](https://i0.imgs.ovh/2023/10/29/AVGGp.png)
   我们可以发现和我们的模板代码 hbs 文件几乎是一样的，并且`{{componentName}}`这种插值语法也已经被正确的替换为了用户输入的`buttonSucess`(用户输入的 button-success， 但是被我们设置的 camelCase 给转化成了驼峰形式)

# plop 的 api 讲解

## setHelper

用来自定义`helper`, `helper`用来对值做转换，例如:

```javascript
export default function (plop) {
  plop.setHelper('upperCase', function (text) {
    return text.toUpperCase();
  });
}
```

第一个参数为 helper 的名字，这里是"upperCase", 第二个参数为处理函数，这里是将值转换为大写

- 使用方式:

  ```
  {{upperCase componentName}}
  ```

- 效果：
  假设 componentName 本来的值是"button"， 那么它最终会被转换成'BUTTON'

- 自带的 helper（这些 helper 为 plop 自带的，可以直接使用）：
  - camelCase: changeFormatToThis
  - snakeCase: change_format_to_this
  - dashCase/kebabCase: change-format-to-this
  - dotCase: change.format.to.this
  - pathCase: change/format/to/this
  - properCase/pascalCase: ChangeFormatToThis
  - lowerCase: change format to this
  - sentenceCase: Change format to this,
  - constantCase: CHANGE_FORMAT_TO_THIS
  - titleCase: Change Format To This

## setPartial

用来自定义局部的模板, 例如：

```javascript
export default function (
  /** @type {import('plop').NodePlopAPI} */
  plop
) {
  plop.setPartial('importVueRef', `import {Ref} from 'vue' `);
}
```

第一个参数为局部模板的名字，这里是'importVueRef', 第二个人参数为模板, 这里是'import {Ref} from 'vue'

- 使用方式：

```
{{> importVueRef}}
```

- 效果：
  `{{> importVueRef}}`会变成 `import {Ref} from 'vue`

# 内置的 actions type

## add

用来在指定的路径生成文件

- 常用属性（其他属性请查官网）
  - path
    string 类型，生成的文件的路径
  - template
    string 类型，模板，以这个模板来生成文件（说白点，就是生成的文件的代码会和这个模板一摸一样）
  - templateFile
    string 类型，模板代码文件的路径，会以这个路径的模板文件来生成文件
  - force
    boolean 类型，当该文件已经存在的时候，是否覆盖
  - data
    对象或者函数类型（函数要返回一个对象），为模板提供额外的数据（就是为那些插值语法提供额外的数据）

## addMany

将多个目录和文件添加到指定目录

- 常用属性（其他属性请查官网）
  - destination
    string 类型，在该路径下添加文件
  - base
    string 类型， 在该路径下匹配目录和文件, 匹配到的目录和文件将会被添加到对应路径下面
  - templateFiles
    glob 类型， 以该模式匹配目录和文件的模式， 比如 templates/components/\*, 就是匹配 templates 目录下的 components 目录下的所有文件
  - stripExtensions
    添加到指定目录时自动删掉对应的后缀名，默认值为['hbs']， 也就是说添加文件的时候.hbs 后缀会被默认删掉。比如说文件名为 index.vue.hbs, 添加的时候就会变成 index.vue
  - force
    boolean 类型，当该文件已经存在的时候，是否覆盖
  - data
    对象或者函数类型（函数要返回一个对象），为模板提供额外的数据（就是为那些插值语法提供额外的数据）

## modify

用于替换指定文件中的匹配到的文本

- 常用属性（其他属性请查官网）
  - path
    同上
  - pattern
    正则表达式，用于匹配文本
  - template
    string 类型，用于将匹配到的文本替换成这个模板
  - templateFile
    string 类型， 模板的路径，以该路径的模板来替换匹配到的文本
  - transform
    函数类型，用来转换文件内容
  - data
    同上

## append

用于在指定文件中匹配到的文本后面添加内容

- 常用属性（其他属性请查官网）
  - path
    同上
  - pattern
    正则表达式，用于匹配文本
  - template
    string 类型，用于在匹配到的文本后面添加这个模板
  - templateFile
    string 类型， 模板的路径，将该路径的模板添加到匹配到的文本后面
  - transform
    函数类型，用来转换文件内容
  - data
    同上

## 内置的 promps type

和 inquire 的一样 详细的可以查看 inquire 的官网
https://github.com/SBoudrias/Inquirer.js
