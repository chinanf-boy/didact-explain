# didact

「 一个DIY指南建立你自己的反应 」

[![explain](http://llever.com/explain.svg)](https://github.com/chinanf-boy/Source-Explain)
    


Explanation

> "version": "1.0.0"

[github source](https://github.com/hexacta/didact)

[english](https://engineering.hexacta.com/didact-learning-how-react-works-by-building-it-from-scratch-51007984e5c5)

---

### 一步一步地来, 本文只是对作者-项目-解释-的中文翻译

`我很想知道React`

幸运的是，如果我们不关心性能，可调试性，可移植性等等，React的三个或四个主要特性并不是很难重写。实际上，它们非常简单，可以用不到200行代码编写。

我们将这样做。使用相同的API在不到200行的代码中编写React的工作版本。鉴于这个图书馆的教学性质，我们将其称为Didact。

---

我们将在以下帖子中一次性向 `Didact` 添加一些功能：

- 渲染DOM元素

- 元素创建和JSX

- 实例，对帐和虚拟DOM

- 组件和状态

~~- 纤维：增量和解~~

---

[codepen.io](https://codepen.io/pomber/pen/RVqBrx?editors=0010)

---

## 渲染DOM元素

> 这个故事是我们一步一步构建自己版本的React的系列文章的一部分：

DOM审查

在我们开始之前，让我们回顾一下我们将使用的DOM API：