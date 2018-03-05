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

- [`对帐和虚拟DOM`](#实例，对帐和虚拟DOM)

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

> 这个故事是我们一步一步构建自己版本的React的系列文章的一部分：

### 2.1 JSX

上次我们介绍了[Didact Elements](#1.2-Didact元素)，它是一种描述我们想要呈现给DOM的非常详细的方式。

在这篇文章中，我们将看到如何使用JSX来简化元素的创建。

JSX提供了一些语法糖来创建元素。代替：

``` js
const element = {
  type: "div",
  props: {
    id: "container",
    children: [
      { type: "input", props: { value: "foo", type: "text" } },
      {
        type: "a",
        props: {
          href: "/bar",
          children: [{ type: "TEXT ELEMENT", props: { nodeValue: "bar" } }]
        }
      },
      {
        type: "span",
        props: {
          onClick: e => alert("Hi"),
          children: [{ type: "TEXT ELEMENT", props: { nodeValue: "click me" } }]
        }
      }
    ]
  }
};
```

我们得到

``` js
const element = (
  <div id="container">
    <input value="foo" type="text" />
    <a href="/bar">bar</a>
    <span onClick={e => alert("Hi")}>click me</span>
  </div>
);
```

如果你对JSX不熟悉，你可能会想知道最后一个片段是否是有效的javascript：它不是。

为了使浏览器的理解，需要的代码由预处理转化为有效的JS，像巴贝尔（了解更多关于JSX阅读这篇文章由贾森·米勒）。

例如，babel从上面将JSX转换为：

``` js
const element = createElement(
  "div",
  { id: "container" },
  createElement("input", { value: "foo", type: "text" }),
  createElement(
    "a",
    { href: "/bar" },
    "bar"
  ),
  createElement(
    "span",
    { onClick: e => alert("Hi") },
    "click me"
  )
);
```

[>>> babel repl ](https://babeljs.io/repl/#?babili=false&evaluate=true&lineWrap=false&presets=react&targets=&browsers=&builtIns=false&debug=false&code=%2F**%20%40jsx%20createElement%20*%2F%0A%0Aconst%20element%20%3D%20%28%0A%20%20%3Cdiv%20id%3D%22container%22%3E%0A%20%20%20%20%3Cinput%20value%3D%22foo%22%20type%3D%22text%22%20%2F%3E%0A%20%20%20%20%3Ca%20href%3D%22%2Fbar%22%3Ebar%3C%2Fa%3E%0A%20%20%20%20%3Cspan%20onClick%3D%7Be%20%3D%3E%20alert%28%22Hi%22%29%7D%3Eclick%20me%3C%2Fspan%3E%0A%20%20%3C%2Fdiv%3E%0A%29%3B)

我们需要添加到Didact中来支持JSX是一个createElement功能，这就是其余部分工作由预处理器完成的。

函数的第一个参数是type元素的第一个参数，第二个参数是元素的对象props，以及所有下面的参数children。

createElement需要创建一个props对象，将其分配给第二个参数中的所有值，将该children属性设置为第二个参数后面的所有参数，然后使用typeand 返回一个对象props。把它放到代码中更容易：

``` js
function createElement(type, config, ...args) {
  const props = Object.assign({}, config);
  const hasChildren = args.length > 0;
  props.children = hasChildren ? [].concat(...args) : [];
  return { type, props };
}
```

除了一件事情之外，这个函数运行良好：文本元素。

文本儿童作为字符串传递给createElement函数，Didact需要文本元素type以及props其余元素。

所以我们将每个arg转换为一个文本元素，这个元素还不是一个元素：

``` js
const TEXT_ELEMENT = "TEXT ELEMENT";

function createElement(type, config, ...args) {
  const props = Object.assign({}, config);
  const hasChildren = args.length > 0;
  const rawChildren = hasChildren ? [].concat(...args) : [];
  props.children = rawChildren
    .filter(c => c != null && c !== false)
    .map(c => c instanceof Object ? c : createTextElement(c));
  return { type, props };
}

function createTextElement(value) {
  return createElement(TEXT_ELEMENT, { nodeValue: value });
}
```

我还筛选了要排除的子项列表null，undefined并指出false，我们不会呈现这些子项，因此不需要添加它们props.children。
### 2.2 概要

在这篇文章中我们没有给Didact增加任何实际的权力，但是我们现在有了改进的开发者体验，因为我们可以使用JSX来定义元素。我已经[更新了上次的codepen](https://codepen.io/pomber/pen/xdmoWE?editors=0010)以包含来自这篇文章的代码。请注意，codepen使用babel来传输JSX，开头的注释/** @jsx createElement */告诉babel使用函数。

您还可以检查[Github提交的更改。](https://github.com/hexacta/didact/commit/15010f8e7b8b54841d1e2dd9eacf7b3c06b1a24b)

在下一篇文章中，[Didact: Instances, reconciliation and virtual DOM](https://engineering.hexacta.com/didact-instances-reconciliation-and-virtual-dom-9316d650f1d0) | [我们介绍了Didact的虚拟DOM和协调算法以支持DOM更新](#实例-对比和虚拟dom)

## 实例-对比和虚拟DOM

> 这个故事是我们一步一步构建自己版本的React的系列文章的一部分：

到目前为止，我们实现了一个基于JSX描述创建dom元素的机制。在这篇文章中，我们将重点介绍如何更新DOM。

直到我们setState在后面的文章中介绍时，更新dom的唯一方法是使用不同的元素再次调用render函数（[从第一篇文章开始](#1.3-渲染-DOM-元素)）。例如，如果我们想渲染一个时钟，代码将是：

``` js
const rootDom = document.getElementById("root");

function tick() {
  const time = new Date().toLocaleTimeString();
  const clockElement = <h1>{time}</h1>;
  render(clockElement, rootDom);
}

tick();
setInterval(tick, 1000);
```

[>>> codepen.io](https://codepen.io/pomber/pen/KmXeXr?editors=0010)

使用该函数的当前版本，这不起作用。而不是更新每个它相同的div 它会追加一个新的。

解决这个问题的第一种方法是替换每个更新的div。

在函数结束时，我们检查父项是否有任何子项，如果有，我们用新元素生成的dom替换它：rendertickrender

``` js
function render(element, parentDom) {  
  
  // ...
  // Create dom from element
  // ...
  
  // Append or replace dom
  if (!parentDom.lastChild) {
    parentDom.appendChild(dom);     
  } else {
    parentDom.replaceChild(dom, parentDom.lastChild);    
  }
}  
```

对于这个小例子，这个解决方案运行良好，但对于更复杂的情况，重新创建所有子节点的性能成本是不可接受的。所以我们需要一种方法来比较当前和前一次调用生成的元素树render，并只更新差异。


### 3. 虚拟DOM和对比

React称这种“差异化”[进程调节](https://facebook.github.io/react/docs/reconciliation.html)。

对于我们来说，首先我们需要保留先前渲染的树，以便我们可以将它与新树进行比较。

换句话说，我们将维护我们自己的DOM版本，一个虚拟的DOM。

什么应该是这个虚拟DOM中的“节点”？

一种选择是只使用Didact Elements，它们已经有一个props.children属性，允许我们以树的形式导航它们。

但是有两个问题，

一个是我们需要在虚拟DOM的每个节点上保留一个对真实DOM节点的引用，以便使对账更容易，我们更愿意保持这些元素不变。

第二个问题是（稍后）我们将需要支持具有自己状态的组件，并且元素将无法处理它。

### 3.1 实例

所以我们需要引入一个新的术语：实例。

一个实例表示已呈现给DOM的元素。

它是具有三个属性的纯JS对象：element，dom，和childInstances。

childInstances是一个包含元素子元素实例的数组。

> 请注意，我们在这里引用的实例与[Dan Abramov在React Components，Elements和Instances中使用的实例](https://medium.com/@dan_abramov/react-components-elements-and-instances-90800811f8ca)不同。他引用了公共实例，这是React在调用继承自类的构造函数时得到的React.Component。我们将在未来的帖子中将公开实例添加到Didact。

每个DOM节点都会有一个匹配的实例。协调算法的一个目标是尽可能避免创建或删除实例。创建和删除实例意味着我们也将修改DOM树，所以我们重新利用实例的次数越多，修改DOM树的次数越少。

### 3.2 重构
让我们重写我们的render函数，保持同样的暴力协调算法，并添加一个instantiate函数来创建一个给定元素的实例（及其子元素）：

``` js
let rootInstance = null;

function render(element, container) {
  const prevInstance = rootInstance;
  const nextInstance = reconcile(container, prevInstance, element);
  rootInstance = nextInstance;
}

function reconcile(parentDom, instance, element) {
  if (instance == null) {
    const newInstance = instantiate(element);
    parentDom.appendChild(newInstance.dom);
    return newInstance;
  } else {
    const newInstance = instantiate(element);
    parentDom.replaceChild(newInstance.dom, instance.dom);
    return newInstance;
  }
}

function instantiate(element) {
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

  // Instantiate and append children
  const childElements = props.children || [];
  const childInstances = childElements.map(instantiate);
  const childDoms = childInstances.map(childInstance => childInstance.dom);
  childDoms.forEach(childDom => dom.appendChild(childDom));

  const instance = { dom, element, childInstances };
  return instance;
}
```
代码和以前一样，但是我们现在正在将最后一次调用的实例存储起来。render.我们还将实例化中的调节功能分开。

为了重新使用DOM节点，我们需要一种方法来更新DOM属性（className，style，onClick而无需创建一个新的DOM节点等）。因此，让我们将当前设置属性的代码部分提取为更新它们的更通用的函数：
 
``` js
function instantiate(element) {
  const { type, props } = element;

  // Create DOM element
  const isTextElement = type === "TEXT ELEMENT";
  const dom = isTextElement
    ? document.createTextNode("")
    : document.createElement(type);

  updateDomProperties(dom, [], props);

  // Instantiate and append children
  const childElements = props.children || [];
  const childInstances = childElements.map(instantiate);
  const childDoms = childInstances.map(childInstance => childInstance.dom);
  childDoms.forEach(childDom => dom.appendChild(childDom));

  const instance = { dom, element, childInstances };
  return instance;
}

function updateDomProperties(dom, prevProps, nextProps) {
  const isEvent = name => name.startsWith("on");
  const isAttribute = name => !isEvent(name) && name != "children";

  // Remove event listeners
  Object.keys(prevProps).filter(isEvent).forEach(name => {
    const eventType = name.toLowerCase().substring(2);
    dom.removeEventListener(eventType, prevProps[name]);
  });

  // Remove attributes
  Object.keys(prevProps).filter(isAttribute).forEach(name => {
    dom[name] = null;
  });

  // Set attributes
  Object.keys(nextProps).filter(isAttribute).forEach(name => {
    dom[name] = nextProps[name];
  });

  // Add event listeners
  Object.keys(nextProps).filter(isEvent).forEach(name => {
    const eventType = name.toLowerCase().substring(2);
    dom.addEventListener(eventType, nextProps[name]);
  });
}
```

updateDomProperties从dom节点中删除所有旧属性，然后添加所有新属性。如果属性发生了变化，它不会改变，所以它会进行大量不必要的更新，但为了简单起见，现在就让它保持原样。

重用DOM节点
我们说调和算法需要尽可能多地重用DOM节点。让我们为该reconcile函数添加一个验证，以检查之前渲染的元素是否与type我们当前正在渲染的元素相同。如果type相同，我们将重新使用它（更新属性以匹配新的属性）：

``` js
function reconcile(parentDom, instance, element) {
  if (instance == null) {
    // Create instance
    const newInstance = instantiate(element);
    parentDom.appendChild(newInstance.dom);
    return newInstance;
  } else if (instance.element.type === element.type) {
    // Update instance
    updateDomProperties(instance.dom, instance.element.props, element.props);
    instance.element = element;
    return instance;
  } else {
    // Replace instance
    const newInstance = instantiate(element);
    parentDom.replaceChild(newInstance.dom, instance.dom);
    return newInstance;
  }
}
```

### 3.3 儿童和解
该reconcile功能缺少一个关键步骤，它使孩子不受影响。儿童和解是React的一个关键方面，它需要元素（key）中的额外属性来匹配先前和当前树中的孩子。我们将使用这种算法的天真版本，它只比较儿童数组中相同位置的孩子。这种方法的成本是，我们失去了重用DOM节点的机会，当他们改变渲染之间的子数组的顺序时。

为了实现这一点，我们将先前的子实例instance.childInstances与子元素进行匹配element.props.children，然后reconcile逐个调用。我们还保留所有返回的实例，reconcile以便我们可以更新childInstances：

``` js
function reconcile(parentDom, instance, element) {
  if (instance == null) {
    // Create instance
    const newInstance = instantiate(element);
    parentDom.appendChild(newInstance.dom);
    return newInstance;
  } else if (instance.element.type === element.type) {
    // Update instance
    updateDomProperties(instance.dom, instance.element.props, element.props);
    instance.childInstances = reconcileChildren(instance, element);
    instance.element = element;
    return instance;
  } else {
    // Replace instance
    const newInstance = instantiate(element);
    parentDom.replaceChild(newInstance.dom, instance.dom);
    return newInstance;
  }
}

function reconcileChildren(instance, element) {
  const dom = instance.dom;
  const childInstances = instance.childInstances;
  const nextChildElements = element.props.children || [];
  const newChildInstances = [];
  const count = Math.max(childInstances.length, nextChildElements.length);
  for (let i = 0; i < count; i++) {
    const childInstance = childInstances[i];
    const childElement = nextChildElements[i];
    const newChildInstance = reconcile(dom, childInstance, childElement);
    newChildInstances.push(newChildInstance);
  }
  return newChildInstances;
}
```

### 3.4 删除DOM节点

如果nextChildElements长于childInstances，reconcileChildren将为所有额外的子元素调用reconcile一个undefined实例。这不应该是一个问题，因为它if (instance == null)会照顾它并创建新的实例。但是反过来呢？当childInstances它比nextChildElements传递undefined元素的时间长，reconcile并试图获取时抛出错误element.type。

这是因为当我们需要从DOM中删除元素时，我们没有考虑过这种情况。因此，我们需要做两件事情，检查element == null在reconcile功能和过滤childInstances的reconcileChildren功能：

``` js
function reconcile(parentDom, instance, element) {
  if (instance == null) {
    // Create instance
    const newInstance = instantiate(element);
    parentDom.appendChild(newInstance.dom);
    return newInstance;
  } else if (element == null) {
    // Remove instance
    parentDom.removeChild(instance.dom);
    return null;
  } else if (instance.element.type === element.type) {
    // Update instance
    updateDomProperties(instance.dom, instance.element.props, element.props);
    instance.childInstances = reconcileChildren(instance, element);
    instance.element = element;
    return instance;
  } else {
    // Replace instance
    const newInstance = instantiate(element);
    parentDom.replaceChild(newInstance.dom, instance.dom);
    return newInstance;
  }
}

function reconcileChildren(instance, element) {
  const dom = instance.dom;
  const childInstances = instance.childInstances;
  const nextChildElements = element.props.children || [];
  const newChildInstances = [];
  const count = Math.max(childInstances.length, nextChildElements.length);
  for (let i = 0; i < count; i++) {
    const childInstance = childInstances[i];
    const childElement = nextChildElements[i];
    const newChildInstance = reconcile(dom, childInstance, childElement);
    newChildInstances.push(newChildInstance);
  }
  return newChildInstances.filter(instance => instance != null);
}
```

### 3.5 概要

在这篇文章中，我们增强了Didact以允许更新DOM。我们还提高了效率，通过重用DOM节点来避免对DOM树的大部分更改。这也具有保持一些DOM内部状态（如滚动位置或焦点）的良好副作用。

我[更新了以前的codepen](https://codepen.io/pomber/pen/WjLqYW?editors=0010)。它调用render状态（stories数组）中的每个更改。如果DOM节点重新创建，您可以检查开发工具。



[>>> codepen.io](https://codepen.io/pomber/pen/WjLqYW?editors=0010)

当我们调用render树的根时，调和适用于整棵树。在接下来的文章中，我们将介绍组件，这将使我们能够和解算法适用于只是受影响的子树：

[Didact: Component and State](https://engineering.hexacta.com/didact-components-and-state-53ab4c900e37)|[Didact：组件和状态](#组件和状态)

在GitHub上检查[这 三个 提交](https://github.com/hexacta/didact/commit/6f5fdb7331ed77ba497fa5917d920eafe1f4c8dc)，以查看代码如何从前一篇文章中更改。