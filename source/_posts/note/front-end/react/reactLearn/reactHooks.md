---
title: ReactHooks的原理解析
subtitle: 记录React学习过程中发现的ReactHooks的原理解析
link: reactHooks
catalog: true
date: 2023-03-04 12:25:19
tags:
  - 前端
  - React
  - ReactHooks
categories:
  - [笔记, 前端, React, React学习]
---

# hooks 使用规则

1. hooks 只能在函数组件 or 自定义 hook 的顶部使用
2. 不能在条件，循环，嵌套函数中使用 hooks

# 为什么会有这样的规则，hooks 究竟是怎么保存我们函数组件的状态的？

**Not Magic, Just Arrays**

# 基本原理

- 函数组件初次执行阶段

1. React 会维护一个数组`componentHooks`，和一个索引`componentHookIndex`
2. 函数组件中的`useState`被调用时，初始值和修改初始值的方法被存进`pair`数组，然后再将`pair`数组通过索引`componentHookIndex`存进`componentHooks`数组, 然后`componentHookIndex++`指向下一个空位。
3. 随着`useState`的顺序调用，一个个保存着初始值和修改初始值方法的`pair`被存进`componentHookIndex`。

- 函数组件的更新阶段

1. 随着修改初始值的方法被调用，函数组件会被重新执行，并且`componentHookIndex`会被重置为`0`, 然后因为我们是按照规则使用的 hooks，所以 hooks 的调用是按照固定的顺序的（**这个固定的顺序，就是为什么要遵守 hooks 的使用规则**），`useState`就可以通过`componentHooks`和`componentsHooksIndex`把我们保存的`pair`返回给我们，然后`componentHookIndex++`指向下一个保存的`pair`。以此来完成之前保存的状态和修改状态的方法的返回。

# 代码实现（此代码实现来自官方文档）

```javascript
let componentHooks = [];
let currentHookIndex = 0;

// How useState works inside React (simplified).
function useState(initialState) {
  let pair = componentHooks[currentHookIndex];
  if (pair) {
    // This is not the first render,
    // so the state pair already exists.
    // Return it and prepare for next Hook call.
    currentHookIndex++;
    return pair;
  }

  // This is the first time we're rendering,
  // so create a state pair and store it.
  pair = [initialState, setState];

  function setState(nextState) {
    // When the user requests a state change,
    // put the new value into the pair.
    pair[0] = nextState;
    updateDOM();
  }

  // Store the pair for future renders
  // and prepare for the next Hook call.
  componentHooks[currentHookIndex] = pair;
  currentHookIndex++;
  return pair;
}

function Gallery() {
  // Each useState() call will get the next pair.
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);

  function handleNextClick() {
    setIndex(index + 1);
  }

  function handleMoreClick() {
    setShowMore(!showMore);
  }

  let sculpture = sculptureList[index];
  // This example doesn't use React, so
  // return an output object instead of JSX.
  return {
    onNextClick: handleNextClick,
    onMoreClick: handleMoreClick,
    header: `${sculpture.name} by ${sculpture.artist}`,
    counter: `${index + 1} of ${sculptureList.length}`,
    more: `${showMore ? 'Hide' : 'Show'} details`,
    description: showMore ? sculpture.description : null,
    imageSrc: sculpture.url,
    imageAlt: sculpture.alt,
  };
}

function updateDOM() {
  // Reset the current Hook index
  // before rendering the component.
  currentHookIndex = 0;
  let output = Gallery();

  // Update the DOM to match the output.
  // This is the part React does for you.
  nextButton.onclick = output.onNextClick;
  header.textContent = output.header;
  moreButton.onclick = output.onMoreClick;
  moreButton.textContent = output.more;
  image.src = output.imageSrc;
  image.alt = output.imageAlt;
  if (output.description !== null) {
    description.textContent = output.description;
    description.style.display = '';
  } else {
    description.style.display = 'none';
  }
}

let nextButton = document.getElementById('nextButton');
let header = document.getElementById('header');
let moreButton = document.getElementById('moreButton');
let description = document.getElementById('description');
let image = document.getElementById('image');
let sculptureList = [
  {
    name: 'Homenaje a la Neurocirugía',
    artist: 'Marta Colvin Andrade',
    description:
      'Although Colvin is predominantly known for abstract themes that allude to pre-Hispanic symbols, this gigantic sculpture, an homage to neurosurgery, is one of her most recognizable public art pieces.',
    url: 'https://i.imgur.com/Mx7dA2Y.jpg',
    alt: 'A bronze statue of two crossed hands delicately holding a human brain in their fingertips.',
  },
  {
    name: 'Floralis Genérica',
    artist: 'Eduardo Catalano',
    description:
      'This enormous (75 ft. or 23m) silver flower is located in Buenos Aires. It is designed to move, closing its petals in the evening or when strong winds blow and opening them in the morning.',
    url: 'https://i.imgur.com/ZF6s192m.jpg',
    alt: 'A gigantic metallic flower sculpture with reflective mirror-like petals and strong stamens.',
  },
  {
    name: 'Eternal Presence',
    artist: 'John Woodrow Wilson',
    description:
      'Wilson was known for his preoccupation with equality, social justice, as well as the essential and spiritual qualities of humankind. This massive (7ft. or 2,13m) bronze represents what he described as "a symbolic Black presence infused with a sense of universal humanity."',
    url: 'https://i.imgur.com/aTtVpES.jpg',
    alt: 'The sculpture depicting a human head seems ever-present and solemn. It radiates calm and serenity.',
  },
  {
    name: 'Moai',
    artist: 'Unknown Artist',
    description:
      'Located on the Easter Island, there are 1,000 moai, or extant monumental statues, created by the early Rapa Nui people, which some believe represented deified ancestors.',
    url: 'https://i.imgur.com/RCwLEoQm.jpg',
    alt: 'Three monumental stone busts with the heads that are disproportionately large with somber faces.',
  },
  {
    name: 'Blue Nana',
    artist: 'Niki de Saint Phalle',
    description:
      'The Nanas are triumphant creatures, symbols of femininity and maternity. Initially, Saint Phalle used fabric and found objects for the Nanas, and later on introduced polyester to achieve a more vibrant effect.',
    url: 'https://i.imgur.com/Sd1AgUOm.jpg',
    alt: 'A large mosaic sculpture of a whimsical dancing female figure in a colorful costume emanating joy.',
  },
  {
    name: 'Ultimate Form',
    artist: 'Barbara Hepworth',
    description:
      'This abstract bronze sculpture is a part of The Family of Man series located at Yorkshire Sculpture Park. Hepworth chose not to create literal representations of the world but developed abstract forms inspired by people and landscapes.',
    url: 'https://i.imgur.com/2heNQDcm.jpg',
    alt: 'A tall sculpture made of three elements stacked on each other reminding of a human figure.',
  },
  {
    name: 'Cavaliere',
    artist: 'Lamidi Olonade Fakeye',
    description:
      "Descended from four generations of woodcarvers, Fakeye's work blended traditional and contemporary Yoruba themes.",
    url: 'https://i.imgur.com/wIdGuZwm.png',
    alt: 'An intricate wood sculpture of a warrior with a focused face on a horse adorned with patterns.',
  },
  {
    name: 'Big Bellies',
    artist: 'Alina Szapocznikow',
    description:
      'Szapocznikow is known for her sculptures of the fragmented body as a metaphor for the fragility and impermanence of youth and beauty. This sculpture depicts two very realistic large bellies stacked on top of each other, each around five feet (1,5m) tall.',
    url: 'https://i.imgur.com/AlHTAdDm.jpg',
    alt: 'The sculpture reminds a cascade of folds, quite different from bellies in classical sculptures.',
  },
  {
    name: 'Terracotta Army',
    artist: 'Unknown Artist',
    description:
      'The Terracotta Army is a collection of terracotta sculptures depicting the armies of Qin Shi Huang, the first Emperor of China. The army consisted of more than 8,000 soldiers, 130 chariots with 520 horses, and 150 cavalry horses.',
    url: 'https://i.imgur.com/HMFmH6m.jpg',
    alt: '12 terracotta sculptures of solemn warriors, each with a unique facial expression and armor.',
  },
  {
    name: 'Lunar Landscape',
    artist: 'Louise Nevelson',
    description:
      'Nevelson was known for scavenging objects from New York City debris, which she would later assemble into monumental constructions. In this one, she used disparate parts like a bedpost, juggling pin, and seat fragment, nailing and gluing them into boxes that reflect the influence of Cubism’s geometric abstraction of space and form.',
    url: 'https://i.imgur.com/rN7hY6om.jpg',
    alt: 'A black matte sculpture where the individual elements are initially indistinguishable.',
  },
  {
    name: 'Aureole',
    artist: 'Ranjani Shettar',
    description:
      'Shettar merges the traditional and the modern, the natural and the industrial. Her art focuses on the relationship between man and nature. Her work was described as compelling both abstractly and figuratively, gravity defying, and a "fine synthesis of unlikely materials."',
    url: 'https://i.imgur.com/okTpbHhm.jpg',
    alt: 'A pale wire-like sculpture mounted on concrete wall and descending on the floor. It appears light.',
  },
  {
    name: 'Hippos',
    artist: 'Taipei Zoo',
    description:
      'The Taipei Zoo commissioned a Hippo Square featuring submerged hippos at play.',
    url: 'https://i.imgur.com/6o5Vuyu.jpg',
    alt: 'A group of bronze hippo sculptures emerging from the sett sidewalk as if they were swimming.',
  },
];

// Make UI match the initial state.
updateDOM();
```

# 官方文档关于这部分的讲解

![](https://i.328888.xyz/2023/03/04/G4KFq.png)

[官方文档这部分解释的链接](https://beta.reactjs.org/learn/state-a-components-memory#giving-a-component-multiple-state-variables)
