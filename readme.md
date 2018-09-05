# didact [![explain]][source] [![translate-svg]][translate-list]

[explain]: http://llever.com/explain.svg
[source]: https://github.com/chinanf-boy/Source-Explain
[translate-svg]: http://llever.com/translate.svg
[translate-list]: https://github.com/chinanf-boy/chinese-translate-list\

「 一个DIY指南 > 建立你自己的React 翻译」

[中文](./readme.md) |  [english](https://engineering.hexacta.com/didact-learning-how-react-works-by-building-it-from-scratch-51007984e5c5)

---

### 更新 🀄

<!-- doc-templite START generated -->
<!-- repo = 'pomber/didact' -->
<!-- time = '2018 3.7' -->
<!-- commit = '6a86fc8bd58a56d65a720457d2b8be5e8282e99b' -->
翻译的原文 | 与日期 | 最新更新 | 更多
---|---|---|---
[commit] | ⏰ 2018 3.7 | ![last] | [中文翻译][translate-list]

[last]: https://img.shields.io/github/last-commit/pomber/didact.svg
[commit]: https://github.com/pomber/didact/tree/6a86fc8bd58a56d65a720457d2b8be5e8282e99b
<!-- doc-templite END generated -->

### 贡献

欢迎 👏 勘误/校对/更新贡献 😊 [具体贡献请看](https://github.com/chinanf-boy/chinese-translate-list#贡献)

## 生活

[help me live , live need money 💰](https://github.com/chinanf-boy/live-need-money)

---

<p align="center"><img src="https://cloud.githubusercontent.com/assets/1911623/26426031/5176c348-40ad-11e7-9f1a-1e2f8840b562.jpeg"></p>

# Didact [![Build Status](https://travis-ci.org/hexacta/didact.svg?branch=master)](https://travis-ci.org/hexacta/didact) [![npm version](https://img.shields.io/npm/v/didact.svg?style=flat)](https://www.npmjs.com/package/didact)


这个存储库与[系列帖子](https://engineering.hexacta.com/didact-learning-how-react-works-by-building-it-from-scratch-51007984e5c5)一起使用,这解释了如何从头开始逐步构建React. 

## 动机

Didact的目标是, 通过提供相同API的更简单实现, 以及如何构建它的逐步说明,使React的内部更容易理解. 一旦你理解了React如何在内部工作,使用它将更容易. 

## 分步指南

| Medium 博文 `en-翻墙`                                                                                                       |                             代码示例                             |                                                                                                                                       提交                                                                                                                                      |                                                            其他语言                                                           |
| -------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :-----------------------------------------------------------------------------------------------------------------------: |
| [介绍](https://engineering.hexacta.com/didact-learning-how-react-works-by-building-it-from-scratch-51007984e5c5) |                                                              |                                                                                                                                                                                                                                                                               |                                                                                                        [中文](#介绍)                    |
| [渲染DOM元素](https://engineering.hexacta.com/didact-rendering-dom-elements-91c9aa08323b)                          | [codepen](https://codepen.io/pomber/pen/eWbwBq?editors=0010) |                                                                                           [diff](https://github.com/hexacta/didact/commit/fc4d360d91a1e68f0442d39dbce5b9cca5a08f24)                                                                                           |               [中文](#1-%E6%B8%B2%E6%9F%93dom%E5%85%83%E7%B4%A0)               |
| [元素创建和JSX](https://engineering.hexacta.com/didact-element-creation-and-jsx-d05171c55c56)                       | [codepen](https://codepen.io/pomber/pen/xdmoWE?editors=0010) |                                                                                           [diff](https://github.com/hexacta/didact/commit/15010f8e7b8b54841d1e2dd9eacf7b3c06b1a24b)                                                                                           |           [中文](#2-%E5%85%83%E7%B4%A0%E5%88%9B%E5%BB%BA%E5%92%8Cjsx)          |
| [虚拟DOM和协调](https://engineering.hexacta.com/didact-instances-reconciliation-and-virtual-dom-9316d650f1d0)       | [codepen](https://codepen.io/pomber/pen/WjLqYW?editors=0010) | [diff](https://github.com/hexacta/didact/commit/8eb7ffd6f5e210526fb4c274c4f60d609fe2f810) [diff](https://github.com/hexacta/didact/commit/6f5fdb7331ed77ba497fa5917d920eafe1f4c8dc) [diff](https://github.com/hexacta/didact/commit/35619a039d48171a6e6c53bd433ed049f2d718cb) | [中文](#3-%E5%AE%9E%E4%BE%8B-%E5%AF%B9%E6%AF%94%E5%92%8C%E8%99%9A%E6%8B%9Fdom) |
| [组件和状态](https://engineering.hexacta.com/didact-components-and-state-53ab4c900e37)                              |        [codepen](https://codepen.io/pomber/pen/RVqBrx)       |                                                                                           [diff](https://github.com/hexacta/didact/commit/2e290ff5c486b8a3f361abcbc6e36e2c21db30b8)                                                                                           |            [中文](#4-%E7%BB%84%E4%BB%B6%E5%92%8C%E7%8A%B6%E6%80%81)            |
| [Fibre-递增对比](https://engineering.hexacta.com/didact-fiber-incremental-reconciliation-b2fe028dcaec)               |        [codepen](https://codepen.io/pomber/pen/veVOdd)       |                                              [diff](https://github.com/hexacta/didact/commit/6174a2289e69895acd8fc85abdc3aaff1ded9011) [diff](https://github.com/hexacta/didact/commit/accafb81e116a0569f8b7d70e5b233e14af999ad)                                              |              [中文](#5-fibre-%E9%80%92%E5%A2%9E%E5%AF%B9%E6%AF%94)             |

## 用法

> 🚧请勿将其用于生产代码!

用npm或yarn安装: 

    $ yarn add didact

然后像使用React一样使用它: 

```jsx
/** @jsx Didact.createElement */
import Didact from 'didact';

class HelloMessage extends Didact.Component {

  constructor(props) {
    super(props);
    this.state = {
      count: 1
    };
  }

  handleClick() {
    this.setState({
      count: this.state.count + 1
    });
  }

  render() {
    const name = this.props.name;
    const times = this.state.count;
    return (
      <div onClick={e => this.handleClick()}>
        Hello {name + "!".repeat(times)}
      </div>
    );
  }
}

Didact.render(
  <HelloMessage name="John" />,
  document.getElementById('container')
);
```

### 介绍

一步一步地来, 本文只是对作者-项目-解释-的中文翻译 「英文原版需要翻墙」

`我很想知道React`

幸运的是，如果我们不关心性能，可调试性，可移植性等等，React的三个或四个主要特性并不是很难重写。

实际上，它们非常简单，可以用不到200行代码编写。

我们将这样做。使用相同的API在不到200行的代码中编写React的工作版本。鉴于这个图书馆的教学性质，我们将其称为`Didact`。

[>>> 最后成果 codepen.io](https://codepen.io/pomber/pen/RVqBrx?editors=0010)

---


### 1. 渲染DOM元素

[>> 1.Rendering-DOM-elements.md](./1.Rendering-DOM-elements.md)

### 2. 元素创建和JSX

[>> 2.JSX.md](./2.JSX.md)

### 3. 实例-对比和虚拟DOM

[>> 3.Virtual.md](./3.Virtual.md)

### 4. 组件和状态

[>> 4.Components-and-State.md](./4.Components-and-State.md)

### 5. Fibre-递增对比

[>> 5.Fibre.readme.md](./5.Fibre.readme.md)

## 演示

[你好,世界](https://rawgit.com/hexacta/didact/master/examples/hello-world/index.html)\
[你好,世界JSX](https://rawgit.com/hexacta/didact/master/examples/hello-world-jsx/index.html)\
[todomvc](https://didact-todomvc.surge.sh)\
[递增渲染,演示](https://pomber.github.io/incremental-rendering-demo/didact.html)



## 执照

MIT©[Hexacta](https://www.hexacta.com)