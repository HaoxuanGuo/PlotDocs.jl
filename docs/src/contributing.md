这是一份关于如何为Plots及其周边生态系统做出贡献的指南。Plots是一套复杂且覆盖广泛的软件组件，因此，当社区贡献他们自己的专业知识、知识、观点和努力时，它会更加有效。本文档大致分为以下几个类别，阅读完这个介绍后，你应该能够自由跳转到你最感兴趣的部分：

- [JuliaPlots 组织](#JuliaPlots-组织)：包和依赖项
- [选择一个项目](#选择一个项目)：修复错误，添加功能，创建配方
- [关键设计原则](#关键设计原则)：设计目标和考虑因素
- [代码组织](#代码组织)：在实现新功能时应该查看哪些内容
- [Git-fu (或者说...贡献的机制)](#Git-fu-(或者说...贡献的机制))：Git（如何提交/推送），Github（如何提交PR），测试（VisualRegressionTests，Travis）

如果有疑问，可以使用由[开源大牛](https://github.com/tbreloff)设计的这个方便的逻辑...

![](https://cloud.githubusercontent.com/assets/933338/23193321/4cd1d578-f876-11e6-92dc-222b52598054.png)

---

## JuliaPlots 组织

[JuliaPlots](https://github.com/JuliaPlots)是所有关于Plots的事情的家。它由[Tom Breloff](https://www.breloff.com)创立，并通过许多来自[成员](https://github.com/orgs/JuliaPlots/people)和其他人的贡献进行扩展。第一步贡献将是了解哪个包或哪些包是你的代码的合适去处。

### Plots

这是以下核心包：

- `plot`/`plot!`的定义
- [核心处理管道](@ref pipeline)
- `path`、`scatter`、`bar`等的基本[配方](@ref recipes)
- 通用的[输出](@ref output)方法
- 通用的[布局](@ref layouts)方法
- 通用的[动画](@ref animations)方法
- 通用类型：Plot，Subplot，Axis，Series，...
- 便利方法：`getindex`/`setindex`，`push!`/`append!`，`unzip`，`cycle`，...

此包依赖于RecipesBase，PlotUtils和PlotThemes。在贡献新的功能/特性时，你应该尽力找到一个更合适的家（比如StatsPlots，PlotUtils等），而不是直接贡献给核心的Plots。一般来说，我们一直在努力减小Plots的大小和范围，并将特性移动到其他包中。

### 后端

后端代码（例如链接Plots和GR的代码）位于`Plots/src/backends`目录中。因此，后端代码应贡献给核心的Plots。GR和Plotly是默认安装的唯一后端。所有其他后端代码都是使用[Requires.jl](https://github.com/JuliaPackaging/Requires.jl)在`Plots/src/init.jl`中有条件地加载的。

### PlotDocs

PlotDocs是此文档的主页。文档是使用[Documenter.jl](https://github.com/JuliaDocs/Documenter.jl)构建的。

### RecipesBase

更新不频繁，但是必不可少。这是你创建第三方配方时需要依赖的包。它包含定义新配方所需的最基本内容。

### PlotUtils

可以用于其他（非Plots）包的组件。任何足够通用且有用的东西都可以贡献到这里。

- 颜色（转换，构造，便利方法）
- 颜色渐变/映射
- 刻度计算

### PlotThemes

视觉主题（即属性默认值），例如"dark"，"orange"等。

### StatsPlots

Plots的扩展：统计图形和表格数据。复杂的直方图和密度图，相关图，以及对DataFrames的支持。任何与统计相关或对表格数据的特殊处理都应在此处。

### GraphRecipes

StatsPlots的扩展：图形，地图等。

---

## 选择一个项目

对于新接触Plots的人来说，第一步应该是阅读（和重读）文档。编写一些示例，玩玩属性，试试多种后端。如果你不知道如何使用一个项目，那么为这个项目做贡献就很困难。

### 初级项目想法

- **创建一个新的配方**：最好是你关心的东西。也许你想要热图和散点的自定义覆盖？也许你有一个目前不支持的输入格式？为此创建一个配方，这样你可以只用`plot(thing)`来画图。
- **修复错误**：有很多"错误"是特定于一个后端的，或者是错误实现了一些不常用的特性。一些想法可以在[标记为easy的问题](https://github.com/JuliaPlots/Plots.jl/issues?q=is%3Aissue+is%3Aopen+label%3A%22easy+-+up+for+grabs%22)中找到。
- **向外部包添加配方**：通过依赖RecipesBase，一个包可以为其自定义类型定义一个配方。向你关心的包提交一个PR，为该包添加一个配方。例如，看看[这个为TimeSeries.jl添加OHLC图的PR](https://github.com/JuliaStats/TimeSeries.jl/pull/303)。

### 中级项目想法

- **改进你最喜欢的后端**：可以对各个后端进行许多缺失的特性和其他改进。大多数特定于一个后端的问题都有一个[特殊标签](https://github.com/JuliaPlots/Plots.jl/issues?q=is%3Aissue+is%3Aopen+label%3APlotly)。
- **帮助文档**：这可能以改进描述、额外的示例或完整的教程的形式出现。请向[PlotDocs](https://github.com/JuliaPlots/PlotDocs.jl)贡献改进内容。
- **扩展StatsPlots的功能**：qqplot，DataStreams，或你能想到的其他任何东西。

### 高级项目想法

- **ColorBar重新设计**：Colorbars [需要大量的照顾](https://github.com/JuliaPlots/Plots.jl/issues?utf8=%E2%9C%93&q=is%3Aissue%20is%3Aopen%20colorbar)...这可能需要一个新的Colorbar类型，该类型与相应的Series对象链接，并在subplot布局时独立。我们希望允许许多系列（可能来自多个subplot）使用相同的clims并共享一个colorbar，或者有多个colorbar可以灵活地定位。
- **PlotSpec重新设计**：这个[长期存在的重新设计提案](https://github.com/JuliaPlots/Plots.jl/issues/390)可以允许通用的序列化/反序列化Plot数据和属性，以及在突变图表时的一些改进/优化。例如，我们可以延迟计算属性值，并在它们改变时智能地标记它们为"dirty"，允许后端跳过大部分浪费的处理和当前发生的不必要的重建。
- **改进图形配方**：这里有很多事情要做：清理视觉效果，改进边缘绘制，实现[布局算法](https://github.com/JuliaGraphs/NetworkLayout.jl)，等等。

---

## 关键设计原则

灵活和通用...这些是Plots开发的核心原则，也往往会在用户过分关注他们特定的用例时引起混淆。

我（Tom）已经精心设计了核心逻辑，以支持几乎所有存在的或可能存在的用例。我不假设你想如何使用Plots，或你可能传入什么类型的数据，或你可能想应用什么类型的配方。因此，我尽量避免不必要的类型限制，或强制转换，或许多其他有限的可视化框架的陷阱。结果是一个高度模块化的框架，只受你的想象力的限制。

当为Plots（或周围的生态系统）贡献新特性时，你也应该追求这种心态。新功能应尽可能保持通用，同时避免明显的特性冲突。

例如，你可能想要一个新的配方，当传入Float64数字时显示直方图，但对于字符串，则显示每个唯一值的计数。所以你制作了一个完美适合你目的的配方：

```@example contributing
using Plots, StatsBase
gr(size = (300, 300), leg = false)

@userplot MyCount
@recipe function f(mc::MyCount)
    # get the array from the args field
    arr = mc.args[1]

    T = typeof(arr)
    if T.parameters[1] == Float64
        seriestype := :histogram
        arr
    else
        seriestype := :bar
        cm = countmap(arr)
        x = sort!(collect(keys(cm)))
        y = [cm[xi] for xi ∈ x]
        x, y
    end
end
```

上面定义的配方是一个"用户配方"，它为Float64数组构建直方图，否则显示排序的唯一值和它们观察到的计数的"countmap"。你只关心Float64和String，所以你的结果是好的：

```@example contributing
mycount(rand(500))
```

```@example contributing
mycount(rand(["A","B","C"],100))
```

但是你没有考虑到未来可能想要将整数传递给这个配方的人：

```@example contributing
mycount(rand(1:500, 500))
```

这个用户期望整数被视为数字并输出直方图，但是它们被视为字符串。一个简单的解决方案是将`if T.parameters[1] == Float64`替换为`if T.parameters[1] <: Number`。然而，我们甚至应该依赖`T`的第一个参数是元素类型吗？（不）所以更好的是`if eltype(arr) <: Number`，这现在允许任何包含任何数字类型的容器触发"直方图"逻辑。

这个简单的例子概述了在开发Plots（或真正任何其他Julia包）时的一个常见主题。尝试创建你能想到的最通用的实现，同时保持正确性。你不知道其他人会使用什么疯狂的类型来尝试访问你的功能。

---

## 代码组织

一般来说，相类似的功能被保留在同一个文件中。在`src`目录中，大多数文件应该是不言自明的（例如，你会在`animation.jl`文件中找到动画方法/宏），但有些可能需要一个内容概要：

- `Plots.jl`：导入，导出，简写和初始化
- `args.jl`：默认值，别名和属性处理
- `components.jl`：形状，字体和其他各种好东西
- `pipeline.jl`：通过递归应用配方来构建图表和子图的代码
- `recipes.jl`：主要是核心系列配方
- `series.jl`：核心输入数据处理和处理
- `utils.jl`：许多没有家的功能... `getindex`/`setindex!`用于`Plot`/`Subplot`/`Axis`/`Series`，`push!`/`append!`用于向系列添加数据，`cycle`/`unzip`和类似的实用程序函数，`Segments`/`SegmentsIterator`等。

这些文件可能需要重新组织，但在此之前...

### 创建新的后端

根据`Plots/src/backends/template.jl`模型创建新的后端。实现适当的回调，尤其是`_display`和`_show`，分别用于GUI和图像输出。

### 样式/设计指南

- 尽一切努力最小化外部依赖和导出。要求新的依赖关系是使你的PR"无法合并"的最可能的方式。
- 在添加Base类型（Array等）的现有方法的方法签名时要小心，因为你可能会覆盖关键功能。这对于配方尤其正确。考虑将输入包装在新类型中（如在"用户配方"中）。
- 简洁的代码是可以的，冗长的代码也是可以的。重要的是理解和上下文。读你的代码的人会知道你的意思吗？如果不是，考虑写注释来描述你的设计理由，或用清晰的散文描述你刚刚实现的黑客。有时候[你的注释比你的代码更长是没问题的](https://github.com/JuliaPlots/Plots.jl/blob/master/src/pipeline.jl#L62-L67)。
- 为自己选择项目，但为他人编写代码。它应该超出你的需要而具有通用性和实用性，你永远不应该**因为你无法弄清楚如何实现某件事而破坏功能**。花更多的时间在它上面...总有一个更好的方法。

---

## Git-fu (或者说...贡献的机制)

许多人对Git有困扰。更多的人对Github有困扰。我认为大部分的混乱发生在你运行命令但不理解它们做什么时。我们都有这个问题，但恢复通常意味着"重新开始"。在这一节中，我将尽量保持对创建PR的简单、实用的方法。这对我来说效果很好，尽管你的情况可能会有所不同。

### 指南

以下是开发工作流程的一些指南（注意：即使你过去已经为Plots做过20个PR，也请阅读这个，因为它可能与过去的指南有所不同）：

- **提交到你自己的分支。** 通常这意味着你应该给你的分支一个对你来说是唯一的名字，可能包括你正在开发的特性的信息。例如，当我开始处理字体时，我可能会选择`git checkout -b tb-fonts`。
- **对master开放一个PR。** `master`是"前沿"。 （注意：我过去建议对`dev`进行PR）
- **只有在绝对必要时才合并其他人的更改。** 你应该更喜欢使用`git rebase origin/master`而不是`git merge origin/master`。一个rebase将你的最近提交重播到最新的`master`上，避免复杂和混乱的合并提交，并通常避免混乱。如果你遵循第一个规则，那么你可能不会给自己带来麻烦。Rebase的恐怖故事通常是当许多人在同一个分支上工作时发生的。我发现[这个资源](https://git-scm.com/book/en/v2/Git-Branching-Rebasing)非常适合理解`git rebase`的重要部分。

---

### 开发工作流程

我的建议是一个顺畅的开发工作流程：

#### Fork仓库

导航到仓库网站（https://github.com/JuliaPlots/Plots.jl）并点击"Fork"按钮。你可能会得到一个选择将fork放在哪个账户或组织的选择。接下来我将假设你fork到了Github用户名为`user123`的地方。

#### 设置git远程

导航到本地仓库。注意：我假设你在Julia目录中进行开发，并使用Mac/Linux。根据需要进行调整。

```
cd ~/.julia/v0.5/Plots
git remote add forked git@github.com:user123/Plots.jl.git
```

运行这些命令后，`git remote -v`应显示两个远程：`origin`（主仓库）和`forked`（你的fork）。一个远程仅仅是指向托管仓库的github网站的引用/指针，而一个fork就是任何其他与原始仓库有特殊链接的git仓库。

#### 创建一个新的分支

如果你刚开始为一个新特性工作：

```
git fetch origin
git checkout master
git merge --ff-only origin/master
git checkout -b user123-myfeature
git push -u forked user123-myfeature
```

前三行是确保你从主仓库的master分支开始。`--ff-only`标志确保你只会"快进"到较新的提交，并避免在你不打算这样做时创建一个新的合并提交。`git checkout`行既创建一个新的分支（`-b`），也指向当前提交，并使该分支为当前。`git push`行将这个分支添加到你的Github fork，并设置本地分支来"跟踪"（`-u`）远程分支，以便后续的`git push`和`git pull`调用。

#### 或者...重用一个旧的分支

如果你有一个正在进行的开发分支（比如，`user123-dev`），你更愿意使用（并且已经被合并到master！）那么你可以用以下方式将其更新：

```
git fetch origin
git checkout user123-dev
git merge --ff-only origin/master
git push forked user123-dev
```

我们更新我们的origin的本地副本，检出dev分支，然后尝试"快进"到当前的master。如果成功，我们将分支推回到我们fork的仓库。

#### 编写并格式化代码

启动你最喜欢的编辑器（也许是 [Juno](https://junolab.org/)？）并对仓库进行一些代码更改。

使用以下命令格式化你的更改（保持代码风格的一致性）：
```bash
$ julia -e 'using JuliaFormatter; format(["src", "test"])'
```

#### 提交

在应用更改后，你会想要 "提交" 或保存你所做的所有更改的快照。提交后，你可以将这些更改 "推送" 到你在 Github 上的 forked 仓库：

```
git add src/my_new_file.jl
git commit -am "my commit message"
git push forked user123-dev
```

第一行是可选的，当向仓库添加新文件时使用。`-a` 表示 "提交我所有的更改"，`-m` 允许你写一个关于提交的注释（你应该总是这样做，并希望让它具有描述性）。

#### 提交 PR

你快到了！浏览你的 fork (https://github.com/user123/Plots.jl)。最有可能的是，在代码上方会有一个区域询问你是否希望从 `user123-dev` 分支创建 PR。如果没有，你可以点击 "New pull request" 按钮。

确保 "base" 分支是 JuliaPlots 的 `master`，"compare" 分支是 `user123-dev`。添加一个信息丰富的标题和描述，并链接到相关的问题或讨论，然后点击 "Create pull request"。你可能会收到一些关于它的问题，可能还有一些关于如何修复它以便 "合并准备就绪" 的建议。然后，希望它能被合并...感谢你的贡献！！

#### 清理

在所有这些之后，你可能会想要回到使用 `master`（或可能在你的特性被标记后使用一个标记的发布）。清理如下：

```
git fetch origin
git checkout master
git merge --ff-only origin/master
git branch -d user123-dev
```

这将你的本地 master 分支与远程 master 分支同步，然后删除 dev 分支。如果你想要返回到标记的发布，从 Julia REPL 运行 `Pkg.free("Plots")`。

---

### 标签

新的标签应代表 "稳定的发布"...那些你愿意分发给终端用户的。在创建新标签之前，应尽力确保测试通过，理想情况下，应添加新的测试以测试你的新功能。当然，对于可视化库来说，这比其他软件更棘手。请参阅下面的 [测试部分](#testing)。

只有 JuliaPlots 成员才能创建新的标签。为了创建一个新的标签，我们将在 Github 上创建一个新的发布，并使用 [attobot](https://github.com/attobot/attobot) 来生成 PR 到 METADATA。在 https://github.com/JuliaPlots/Plots.jl/releases/new 创建一个新的发布（当然要将仓库名替换为你正在标记的包名）。

版本号（vMAJOR.MINOR.PATCH）应使用 [semver](https://semver.org/) 进行增加，这通常意味着破坏性的更改应增加主要的数字，向后兼容的更改应增加次要的数字，而错误修复应增加补丁的数字。对于 "v0.x.y" 版本，这个要求被放宽。次要版本可以因为破坏性的更改而增加。

---

### 测试

#### VisualRegressionTests

在 Plots 中，我们使用 [VisualRegressionTests](https://github.com/JuliaPlots/VisualRegressionTests.jl) 进行测试。参考图像存储在 [PlotReferenceImages](https://github.com/JuliaPlots/PlotReferenceImages.jl) 中。有时候需要更新参考图像（如果特性改变，或者底层后端改变）。VisualRegressionTests 使得更新参考图像相当简单：

在 Julia REPL 中，运行 `Pkg.test(name="Plots")`。这将尝试绘制测试，然后将结果与存储的参考图像进行比较。如果测试输出与参考输出有足够的不同（使用 Tim Holy 的优秀算法进行比较），那么一个 GTK 窗口将弹出，显示并排比较。你可以选择替换参考图像，或者不替换，这取决于是否发现了真正的错误。

在参考图像被更新后，导航到 PlotReferenceImages 并将更改推送到 Github：

```
cd ~/.julia/v0.5/PlotReferenceImages
git add Plots/*
git commit -am "a useful message"
git push
```

如果由于错误存在不匹配，**不要更新参考图像**。

#### CI

在 `git push` 上，测试将作为我们连续集成设置的一部分自动运行。这将运行与上面相同的测试，下载并比较参考图像，尽管对差异的容忍度更大。当这些错误时，可能是由于超时、过时的参考图像或其他一系列原因。检查日志以确定原因。如果测试因新的提交而破坏，考虑回滚。
