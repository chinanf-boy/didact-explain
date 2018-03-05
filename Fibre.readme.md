## 5. Fibre-递增对比

未完成请勿点击

<details>

这个故事是一个部分系列赛里，我们写我们自己阵营的版本，但因为我们要重写大部分旧代码的，无论如何，我会文艺青年最爱的它为您提供：

>TL;到目前为止的系列DR：我们正在编写一个React克隆来了解React在底层做了什么。我们称之为Didact。为了简化代码，我们只关注React的主要功能。首先我们介绍如何渲染元素并使JSX工作。我们编写了协调算法来重新渲染仅更新之间更改的内容。然后我们添加了类组件和setState()。

现在React 16已经不复存在，并且有了一个新的内部架构，需要重写React的大部分代码。

这意味着一些期待已久的功能 - 旧的架构很难发展 - 被运送。

这也意味着我们在这个系列中编写的大部分代码现在都是毫无价值的。

在本文中，我们将重写didact系列中的大部分代码，以遵循React 16新架构。我们将尝试从React代码库中镜像结构，变量和函数名称。我们将跳过我们公共API所不需要的一切：

- Didact.createElement()

- Didact.render() （只有DOM渲染）

- Didact.Component（使用setState()但不是context生命周期方法）

如果你想跳到前面看代码工作，你可以去更新的演示或访问github存储库。

让我解释为什么我们需要重写旧代码。

### 5.1 为什么选择Fiber
这不会提供React Fiber的完整画面。如果您想了解更多信息，请查看此资源列表。
当浏览器的主线程忙于长时间运行时，关键的简短任务必须等待一段不可接受的时间才能完成。

为了展示这个问题，我做了一个小演示。为了保持行星的旋转，主线程需要每16ms左右一次即可使用。如果主线程被其他东西阻塞，让我们说200毫秒，你会注意到动画丢失帧和行星冻结，直到主线程再次释放。

是什么让主线程如此繁忙以至于无法在保持动画平滑和UI响应的情况下节省一些微秒？

记住对帐代码？一旦开始协调，它不会停止。如果主线程需要做其他任何事情，它将不得不等待。而且，`因为递归调用很大程度上取决于它，所以很难使它可以被使用`。这就是为什么我们要用一个新的数据结构来重写它，这将允许我们用循环替换递归调用。

### 5.2 调度微任务
我们需要将工作分解为更小的部分，短时间运行这些部分，让主线程执行更高优先级的任务，并且如果有任何待处理的事情回来完成工作。

我们会在requestIdleCallback()功能的帮助下做到这一点。它会在下一次浏览器闲置时调用一个回调deadline函数，并包含一个描述我们的代码有多少可用时间的参数：

```

```

真正的工作发生在performUnitOfWork函数内部。我们需要在那里编写我们的调节代码。该函数应该运行一部分工作，然后返回下一次需要恢复工作的所有信息。

为了跟踪这些工作，我们将使用纤维。

### 光纤数据结构
我们将为每个想要渲染的组件创建一个光纤。这nextUnitOfWork将是对我们想要工作的下一个光纤的参考。performUnitOfWork将在该光纤上工作，并返回一个新的直到所有工作完成。忍受我，我会稍后详细解释这一点。

纤维是怎样的？

```

```
这是一个普通的旧JavaScript对象。

我们将使用parent，child和sibling地产打造的纤维树描述部件的树。

这stateNode将是对组件实例的引用。它可以是DOM元素，也可以是用户定义的类组件的实例。

例如：

```

```

在这个例子中，我们可以看到我们将支持的三种不同类型的组件：

为纤维b，p和i表示主机组件。我们将用它来识别它们tag HOST_COMPONENT。在type这些纤维将是一个字符串（html元素的标签）。这props将是元素的属性和事件监听器。
所述Foo纤维表示类组分。其tag将CLASS_COMPONENT与type用户定义的类的引用延伸Didact.Component。
div代表主机根的光纤。它与主机组件相似，因为它具有DOM元素作为stateNode树，但是它将受到特殊处理。在tag该纤维会HOST_ROOT。请注意，stateNode这个光纤是传递给的DOM节点Didact.render()。
另一个重要的属性是alternate。我们需要它，因为大多数时候我们会有两根纤维树。一棵树将对应于我们已经呈现给DOM的东西，我们将它称为当前树或旧树。另一个是我们在创建新更新（调用setState()或）时构建的树Didact.render()，我们将此树称为正在进行中的树。

正在进行的工作树不会与旧树共享任何光纤。一旦我们完成建设工作正在进行树并取得所需的DOM突变，该工作正在进行树成为旧树。

因此，我们使用alternate链接正在工作的光纤与旧 树中相应的光纤。纤维和它的alternate份额相同tag，type而且stateNode。有时 - 当我们渲染新的东西时 - 光纤不会有alternate。

最后，我们有effects清单和effectTag。当我们发现在工作正在进行树的纤维，需要改变的DOM，我们将设置effectTag到PLACEMENT，UPDATE或DELETION。为了更容易将所有DOM突变一起提交，我们保留了列出的所有光纤（来自光纤子树）的effectTag列表effects。

这可能是一次太多的信息，如果你没有遵循，不要担心，我们很快就会看到光纤树在运行。

### Didact调用层次结构
要了解我们要编写的代码的流程，请查看此图表：

![5-pic](./imgs/5-pic.png)

### 旧代码
我告诉过你，我们将重写大部分代码，但我们首先回顾一下我们不重写的代码。

在元素创建和JSX中，我们编写了用于编译JSX的函数的代码createElement()。我们不需要改变它，我们将继续使用相同的元素。如果你不知道的元素，type，props和children，请查看旧帖子。

在实例中，我们编写了updateDomProperties()用于更新节点DOM属性的调节和虚拟DOM。我还提取了用于创建DOM元素的代码createDomElement()。你可以在这个dom-utils.js的要点中看到这两个函数。

在组件和状态中，我们编写了Component基类。让我们改变它以便setState()调用scheduleUpdate()，并createInstance()保存对实例中光纤的引用：

```

```

从这段代码开始，没有别的让我们从头开始重写其余部分。

![5-pic1](./imgs/5-pic1.png)

除了Component课程之外createElement()，我们还会有两个公共职能：render()而且setState()，我们看到这setState()只是调用scheduleUpdate()。

render()并且scheduleUpdate()是相似的，他们会收到一个新的更新并对其进行排队：

```

```

我们将使用该updateQueue数组来跟踪待处理的更新。每次打电话render()或scheduleUpdate()推送一个新的更新updateQueue。每个更新中的更新信息都不同，我们将看到我们以后如何使用它resetNextUnitOfWork()。

将更新推送到队列后，我们触发延迟呼叫performWork()。

![5-pic2](./imgs/5-pic2.png)

```

```

这是我们使用performUnitOfWork()之前看到的模式的地方。

requestIdleCallback()以截止日期为参数调用目标函数。performWork()将截止日期传给它workLoop()。后workLoop()返回时，performWork()检查是否还有待审批工作。如果有的话，它会为它自己安排一个新的延期呼叫。

workLoop()是关注时间的功能。如果截止日期太近，它会停止工作循环并保持nextUnitOfWork更新状态，以便下次恢复。

我们使用ENOUGH_TIME（1ms常数，与React相同）来检查是否deadline.timeRemaining()足以运行另一个工作单元。如果performUnitOfWork()超过这一点，我们将超过最后期限。截止日期只是来自浏览器的建议，所以将其超过几毫秒并不是那么糟糕。
performUnitOfWork()将为其正在进行的更新构建正在进行的工作树，并找出我们需要对DOM应用哪些更改。这将逐步完成，每次一根光纤。

当performUnitOfWork()完成当前更新的所有工作时，它将返回null并将待处理的更改留在DOM中pendingCommit。最后，commitAllWork()将采取effects从pendingCommit和变更DOM。

请注意，commitAllWork()在循环之外调用。完成的工作performUnitOfWork()不会改变DOM，因此可以将其分开。另一方面，commitAllWork()将改变DOM，它应该一次完成，以避免不一致的UI。

我们还没有看到第一个nextUnitOfWork来自哪里。

![5-pic3](./imgs/5-pic3.png)

需要更新并将其转换为第一个的函数nextUnitOfWork是resetNextUnitOfWork()：

```

```

resetNextUnitOfWork() 首先从队列中提取第一个更新。

如果更新有，partialState我们将它存储在属于组件实例的光纤上，以便稍后在调用组件时使用它render()。

然后我们找到老纤维树的根。如果更新来自于第一次render()被调用时，我们会不会有一个根纤维所以root会null。如果它来自后续调用render()，我们可以_rootContainerFiber在DOM节点的属性上找到根。如果更新来自a setState()，我们需要从实例光纤上去，直到找到没有光纤parent。

然后我们分配nextUnitOfWork一个新的光纤。这种光纤是一个新的工作进行中的树的根。

如果我们没有一个旧的根，那么stateNode这个DOM节点就会在render()调用中作为参数接收。这props将是newProps从更新：一个children属性的对象具有元素 - 的另一个参数render()。该alternate会null。

如果我们有一个旧的根，那么stateNode将会是前一个根的DOM节点。该props会又newProps如果不是null，否则我们复制props从老根。这alternate将是老根。

我们现在拥有正在进行中的树的根，让我们开始构建剩余的树。

![5-pic4](./imgs/5-pic4.png)

```

```
performUnitOfWork() 走在进行中的工作树。

我们称之为beginWork() - 创造一种纤维的新生儿 - 然后让第一个孩子回到原来的状态nextUnitOfWork。

如果没有任何的孩子，我们呼吁completeWork()并返回sibling的nextUnitOfWork。

如果没有sibling，我们会去找父母打电话，completeWork()直到我们找到sibling（我们将成为nextUnitOfWork）或直到我们到达根部。

performUnitOfWork()多次呼叫会沿着树木向下，造成每根纤维的第一个孩子的孩子，直到找到没有孩子的纤维。然后它就像兄弟姐妹一样向右移动。而且它也跟叔叔一样。（为了更加生动的描述，尝试在光纤调试器上渲染一些组件）


```

```

beginWork() 做两件事：

创造stateNode如果我们没有一个
获取组件子项并将它们传递给 reconcileChildrenArray()
因为两者都取决于我们处理的组件的类型，所以我们将它分成两部分：updateHostComponent()和updateClassComponent()。

updateHostComponent()处理主机组件以及根组件。如果需要的话，它会创建一个新的DOM节点（只有一个节点，没有子节点，并且不会将其附加到DOM）。然后它reconcileChildrenArray()使用光纤中的子元素进行调用props。

updateClassComponent()处理类组件实例。如果需要的话，它会创建一个调用组件构造函数的新实例。它更新实例props，state因此它可以调用该render()函数来获取新的子项。

updateClassComponent()也验证是否有意义调用render()。这是一个简单的版本shouldComponentUpdate()。如果看起来我们不需要重新渲染，那么我们只是将当前的子树克隆到正在进行的工作树中，而不进行任何调整。

现在，我们已经newChildElements准备好为正在进行中的光纤创建子光纤。


这是图书馆的核心，正在进行的工作树在不断增长，我们决定在提交阶段对DOM做什么更改。


开始之前，我们确保newChildElements是一个数组。（与之前的协调算法不同，这个算法总是与子数组一起工作，这意味着我们现在可以在组件的render()函数上返回数组）

然后我们开始比较旧纤维树的孩子和新元素（我们将纤维与元素进行比较）。来自老纤维树的孩子们都是孩子们的孩子wipFiber.alternate。新元素是我们wipFiber.props.children从调用或从调用中获得的元素wipFiber.stateNode.render()。

我们和解算法通过第一古老纤维（匹配wipFiber.alternate.child）与第一子元素（elements[0]），第二旧纤维（wipFiber.alternate.child.sibling）的第二子元素（elements[1]）等。对于每oldFiber- element对：

如果oldFiber和element同样的type好消息，这意味着我们可以保留旧的stateNode。我们创建一个基于旧的纤维的新纤维。我们添加UPDATE effectTag。我们将新光纤添加到正在进行的工作树中。
如果我们element与我们有一个不同type的oldFiber或者我们没有oldFiber（因为我们有比新的孩子更多的新孩子），我们创建一个新的纤维与我们的信息element。请注意，这种新光纤没有alternate，也不会有stateNode（stateNode我们将创建beginWork()）。的effectTag该纤维是PLACEMENT。
如果oldFiber和element有不同的type或没有任何element这oldFiber（因为我们有更多的老孩子比新来的孩子），我们将标记oldFiber为DELETION。鉴于这种光纤不是正在进行工作的树的一部分，我们需要将它添加到wipFiber.effects列表中，以便我们不会丢失它的踪迹。
与React不同的是，我们没有使用键来进行和解，所以我们不知道一个孩子是否从之前的位置移动过来。

updateClassComponent() 有一个特殊情况，我们采取快捷方式，将旧的光纤子树克隆到正在进行中的工作树，而不是进行调整。


cloneChildFibers()克隆每个wipFiber.alternate孩子并将其附加到正在进行中的树中。我们不需要添加任何内容，effectTag因为我们确信没有任何变化。


在performUnitOfWork()a wipFiber没有新的孩子或者我们已经完成了所有孩子的工作时，我们打电话给他completeWork()。


completeWork()首先更新与类组件实例相关的光纤引用。（说实话，这并不是真的需要在这里，但它必须在某个地方）

然后它建立一个列表effects。该列表将包含来自正在进行中的子树的所有光纤effectTag（它也包含来自旧子树的光纤DELETION effectTag）。这个想法是在根effects表中累积所有的纤维effectTag。

最后，如果光纤没有parent，我们就是正在进行工作的树的根。所以我们完成了这次更新的所有工作并收集了所有的效果。我们分配根，pendingCommit以便workLoop()可以调用commitAllWork()。


我们需要做的最后一件事是：改变DOM。


commitAllWork()首先迭代每个根上effects调用的所有根commitWork()。commitWork()检查effectTag每根光纤：

如果是，PLACEMENT我们查找父DOM节点，然后简单地追加光纤stateNode。
如果是这样的话，UPDATE我们会把stateNode旧道具和新道具放在一起，然后updateDomProperties()决定要更新什么。
如果它是a DELETION并且光纤是主机组件，那很简单，我们只是打电话removeChild()。但是如果光纤是类组件，在调用之前，removeChild()我们需要从光纤子树中找到需要删除的所有主机组件。
一旦我们完成了所有的效果，我们可以重置nextUnitOfWork和pendingCommit。正在进行的工作树不再是正在进行中的工作树，并成为旧树，因此我们将其根指定给_rootContainerFiber。之后，我们完成当前的更新，我们准备开始下一个🚀。

正在运行Didact
如果你想把所有的东西放在一起，只公开API，你可以这样做：


或者你可以使用之前发布的演示更新版本。所有这些代码也可以在Didact的仓库和npm中获得。

下一步是什么？
Didact缺少很多React的功能，但我特别有兴趣根据优先级安排更新：


反应源
所以下一篇文章可能会这样。

就这样！如果你喜欢它，不要忘记拍手👏，在Twitter上关注我，发表评论和所有这些内容。

谢谢阅读。
</details>