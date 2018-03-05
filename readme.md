# didact

「 一个DIY指南建立你自己的反应 」

[![explain](http://llever.com/explain.svg)](https://github.com/chinanf-boy/Source-Explain)
    


Explanation

> "version": "1.0.0"

[github source](https://github.com/hexacta/didact)

[english](https://engineering.hexacta.com/didact-learning-how-react-works-by-building-it-from-scratch-51007984e5c5)

---

### 一步一步地来, 本文只是对作者-项目-解释-的中文翻译 「英文原版需要翻墙」

`我很想知道React`

幸运的是，如果我们不关心性能，可调试性，可移植性等等，React的三个或四个主要特性并不是很难重写。

实际上，它们非常简单，可以用不到200行代码编写。

我们将这样做。使用相同的API在不到200行的代码中编写React的工作版本。鉴于这个图书馆的教学性质，我们将其称为Didact。

---

我们将在以下帖子中一次性向 `Didact` 添加一些功能：

- [`渲染DOM元素`](#渲染DOM元素)

- [`元素创建和JSX`](#元素创建和JSX)

- [`实例，对帐和虚拟DOM`](#实例，对帐和虚拟DOM)

- [`组件和状态`](#组件和状态)

~~- [`Fibre：增量和解`](#纤维：增量和解)~~

---

[>>> 最后成果 codepen.io](https://codepen.io/pomber/pen/RVqBrx?editors=0010)

---

## 1. 渲染DOM元素

> 这个故事是我们一步一步构建自己版本的React的系列文章的一部分：

### 1.1 DOM审查

在我们开始之前，让我们回顾一下我们将使用的DOM API：

``` js
// Get an element by id
const domRoot = document.getElementById("root");
// Create a new element given a tag name
const domInput = document.createElement("input");
// Set properties
domInput["type"] = "text";
domInput["value"] = "Hi world";
domInput["className"] = "my-class";
// Listen to events
domInput.addEventListener("change", e => alert(e.target.value));
// Create a text node
const domText = document.createTextNode("");
// Set text node content
domText["nodeValue"] = "Foo";
// Append an element
domRoot.appendChild(domInput);
// Append a text node (same as previous)
domRoot.appendChild(domText);
```

[>>> codepen.io](https://codepen.io/pomber/pen/aWBLJR)

请注意，我们正在设置[元素属性而不是属性](http://stackoverflow.com/questions/6003819/properties-and-attributes-in-html)。这意味着只允许有效的属性。

### 1.2 Didact元素

我们将使用普通的JS对象来描述需要渲染的东西。我们将它们称为Didact Elements。

这些元素有两个必需的属性：`type`和`props`。

- `type`可以是一个字符串或一个函数，但我们将只使用字符串，直到我们在稍后的帖子中引入组件。

- `props`是可以为空的对象（但不为空）。props可能有一个children属性，它应该是一个Didact元素的数组。

> 我们会很多地使用Didact Elements，所以从现在开始我们只会称它们为元素。不要与HTML元素混淆，我们会调用DOM元素，或者只是dom在命名变量时（如preact一样）。

例如，像这样的一个元素：

``` js
const element = {
  type: "div",
  props: {
    id: "container",
    children: [
      { type: "input", props: { value: "foo", type: "text" } },
      { type: "a", props: { href: "/bar" } },
      { type: "span", props: {} }
    ]
  }
};
```

描述这个dom:

``` jsx
<div id="container">
  <input value="foo" type="text">
  <a href="/bar"></a>
  <span></span>
</div>
```

---

Didact元素与React元素非常相似。

但是通常你在使用React时不会创建React元素作为JS对象，

你可能使用JSX或者甚至是createElement。

我们将在Didact中做同样的事情，但我们将留下系列中下一篇文章的元素创建代码。

---

### 1.3 渲染-DOM-元素

下一步是将元素及其子元素呈现给dom。

我们将使用一个render函数（相当于ReactDOM.render）接收一个元素和一个dom容器。

该函数应该创建由元素定义的dom子树并将其附加到容器中：

``` js
function render(element, parentDom) {
  const { type, props } = element;
  const dom = document.createElement(type);
  const childElements = props.children || [];
  childElements.forEach(childElement => render(childElement, dom));
  parentDom.appendChild(dom);
}
```
我们仍然缺少属性和事件监听器。让我们props用Object.keys函数迭代属性名称并相应地设置它们：

``` js
function render(element, parentDom) {
  const { type, props } = element;
  const dom = document.createElement(type);

  const isListener = name => name.startsWith("on");
  Object.keys(props).filter(isListener).forEach(name => {
    const eventType = name.toLowerCase().substring(2);
    dom.addEventListener(eventType, props[name]);
  });

  const isAttribute = name => !isListener(name) && name != "children";
  Object.keys(props).filter(isAttribute).forEach(name => {
    dom[name] = props[name];
  });

  const childElements = props.children || [];
  childElements.forEach(childElement => render(childElement, dom));

  parentDom.appendChild(dom);
}
```

### 1.4 渲染DOM文本节点

render函数不支持的一件事是文本节点。首先，我们需要定义文本元素的外观。例如，`<span>Foo</span>`在React中描述的元素如下所示：

``` js
const reactElement = {
  type: "span",
  props: {
    children: ["Foo"]
  }
};
```

请注意，孩子，而不是另一个元素对象，只是一个字符串。

这违背了我们如何定义Didact元素：children应该是元素的数组和所有元素应该有type和props。

如果我们遵循这些规则，我们将来会少一些if。

因此，Didact Text Elements将type与“TEXT ELEMENT”相等，实际文本将位于nodeValue属性中。

像这个：

``` js
const textElement = {
  type: "span",
  props: {
    children: [
      {
        type: "TEXT ELEMENT",
        props: { nodeValue: "Foo" }
      }
    ]
  }
};
```

现在我们已经定义了文本元素的外观，就像我们可以呈现它 与其他元素的区别是文本元素，

而不是使用createElement应该使用createTextNode。

就是这样，nodeValue将会像其他属性一样设置。

``` js
function render(element, parentDom) {
  const { type, props } = element;

  // Create DOM element
  const isTextElement = type === "TEXT ELEMENT";
  const dom = isTextElement
    ? document.createTextNode("")
    : document.createElement(type);

  // Add event listeners
  const isListener = name => name.startsWith("on");
  Object.keys(props).filter(isListener).forEach(name => {
    const eventType = name.toLowerCase().substring(2);
    dom.addEventListener(eventType, props[name]);
  });

  // Set properties
  const isAttribute = name => !isListener(name) && name != "children";
  Object.keys(props).filter(isAttribute).forEach(name => {
    dom[name] = props[name];
  });

  // Render children
  const childElements = props.children || [];
  childElements.forEach(childElement => render(childElement, dom));

  // Append to parent
  parentDom.appendChild(dom);
}
```

### 1.5 概要

我们创建了一个render函数，允许我们将一个元素及其子元素呈现给DOM。

接下来我们需要的是创建元素的简单方法。

我们将在下一篇文章中做到这一点，在那里我们将让JSX与Didact一起工作。

如果您想尝试我们迄今为止编写的代码，请检查[codepen](https://codepen.io/pomber/pen/eWbwBq?editors=0010)。你也可以从[github回购中检查这个差异](https://github.com/hexacta/didact/commit/fc4d360d91a1e68f0442d39dbce5b9cca5a08f24)。

下一篇文章：[Didact: Element creation and JSX {en}](https://engineering.hexacta.com/didact-element-creation-and-jsx-d05171c55c56) | [Didact：元素创建和JSX {zh}](#2-元素创建和JSX)

---

## 2. 元素创建和JSX

