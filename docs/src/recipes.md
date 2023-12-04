```@setup recipes
using Plots; gr()
Plots.reset_defaults()
```


# [配方](@id recipes)

配方是在你自己的包和代码中定义可视化的一种方式，无需依赖于 Plots。该功能依赖于 [RecipesBase](https://github.com/JuliaPlots/RecipesBase.jl)，这是一个超轻量级但功能强大的包，允许用户在没有 Plots 的情况下创建高级绘图逻辑。RecipesBase 中的 `@recipe` 宏会为 `RecipesBase.apply_recipe` 添加一个方法定义。Plots 也会增加并调用这个相同的函数，因此你的包和 Plots 可以在彼此不知道的情况下进行通信。魔法！

可视化自定义用户类型一直是一个令人困惑的问题。包开发者是否应该添加对绘图包的依赖（带来依赖的重大负担）？他们是否应该尝试条件依赖？他们是否应该提交 PR 到图形包来定义他们的自定义可视化？似乎每个选项都有许多的利弊，决定很难。有了配方，这些问题就消失了。一个微小的包（RecipesBase）提供了简单的挂钩到可视化管道，允许用户和包开发者专注于他们的可视化的具体细节。选择将代表你的数据的形状/线条/颜色，决定自定义的默认值，转换输入（如果你需要）。其他所有的事情都由 Plots 处理。在 Plots 和许多外部包中，都有许多配方的例子，包括 [GraphRecipes](https://github.com/JuliaPlots/GraphRecipes.jl)。

### 可视化用户类型

例子总是最好的。我们来探索[为分布创建可视化配方的实现](https://github.com/tbreloff/ExamplePlots.jl/tree/master/notebooks/usertype_recipes.ipynb)。

### 自定义处理输入组合

想要在第一个输入是时间序列时做一些特殊的事情吗？也许你想根据关键字标志预处理你的数据？通过创建具有独特分派签名的配方，这一切都是可能的。你可以卸载并使用 Plots 的预处理和后处理，只添加特定于你的部分。

### 类型配方：易于替换数据类型

许多时候，一个数据类型只是一个函数或数组的简单包装。例如：

```julia
mutable struct MyVec
    v::Vector{Int}
end
```

如果 `MyVec` 是 AbstractVector 的子类型，那么就没有什么要做的...它应该 "就能工作"。然而，这并不总是可取的，如果你能调用 `plot(10:20, myvec)` 而不必自己定义每一种可能的输入组合，那就太好了。在这种情况下，你会想要使用一种特殊类型的配方签名：

```julia
@recipe f(::Type{MyVec}, myvec::MyVec) = myvec.v
```

之后，所有适用于向量的绘图命令也将适用于你的数据类型。

### 系列配方

让我们快速讨论一下数据可视化的主要内容：直方图。Hadley Wickham 在他的 [Layered Grammar of Graphics](https://vita.had.co.nz/papers/layered-grammar.pdf) 中探讨了直方图的性质。在其中，他讨论了直方图实际上只不过是预先分箱的条形图。这是真的，而且可以进一步发展。条形图实际上是阶梯图的扩展，在其中，零值被编织在 x 值之间。阶梯图实际上只不过是一条只能水平或垂直行走的路径。当然，通过将条形图视为填充的多边形，也可以得到类似的分解。

要理解的一点是，图形包只需要能够绘制线条和多边形，就可以支持绘制直方图。从数据到直方图的路径通常非常复杂，但我们可以避免复杂性并定义一个配方将其转换为子组件。在几行可读的代码中，我们可以实现一个关键的统计可视化。查看[关于系列配方的教程](https://github.com/tbreloff/ExamplePlots.jl/tree/master/notebooks/series_recipes.ipynb)以更好地理解你可能如何使用它们。

## 配方类型

上面我们描述了 `类型配方` 和 `系列配方`。在 Plots 中，总共有四种主要类型的配方（按它们被处理的顺序列出）：

- 用户配方
- 类型配方
- 绘图配方
- 系列配方

**配方类型完全由分派签名确定。**每种配方类型都是从[绘图管道](https://docs.juliaplots.org/latest/pipeline/)的不同部分调用的，所以你会选择一种配方类型来匹配你希望在应用你的配方之前完成多少处理。

以下是每种类型的分派签名（注意，大多数都可以接受位置或关键字参数，用 `...` 表示）：

### 用户配方
```julia
@recipe function f(custom_arg_1::T, custom_arg_2::S, ...; ...) end
```
- 在流程的早期处理一组独特的类型。适用于用户定义的类型或基础类型的特殊组合。
- `@userplot` 宏是一种很好的便利，它既定义了一个新类型（以确保正确的调度），又导出了简写。
- 请参见 `graphplot` 作为示例。

### [类型配方](@id type-recipes)
```julia
@recipe function f(::Type{T}, val::T) where{T} end
```
- 对于包装或与 Plots 支持的某些东西一对一映射的用户定义类型，只需定义一个转换方法。
- 注意：这实际上是说“当你看到类型 T，将其替换为……”
- 请参见 `SymPy` 作为示例。

### 绘图配方
```julia
@recipe function f(::Type{Val{:myplotrecipename}}, plt::AbstractPlot; ...) end
```
- 这些在输入数据被处理之后，但是**在创建绘图之前**被调用。
- 构建布局，添加子图和其他全局绘图属性。
- 请参见 [StatsPlots](https://github.com/JuliaPlots/StatsPlots.jl) 中的 `marginalhist` 作为示例。

### [系列配方](@id series-recipes)
```julia
@recipe function f(::Type{Val{:myseriesrecipename}}, x, y, z; ...) end
```
- 这些是最后发生的调用。每个后端都会支持一小部分系列类型（`path`，`shape`，`histogram` 等）。如果一个系列类型是原生支持的，处理就会被传递（委托）给后端。如果一个系列类型**不是**由后端原生支持的，我们会尝试调用一个“系列配方”。
- 注意：如果没有定义系列配方，而后端也不支持它，你会看到像 `ERROR: The backend must not support the series type Val{:hi}, and there isn't a series recipe defined.` 这样的错误。
- 注意：你必须在签名中包含 `x, y, z`，否则它不会被处理为一个系列类型！

## 配方语法/规则

让我们分解配方宏内部发生的情况，从一个简单的配方开始：

```@example recipes
mutable struct MyType end

@recipe function f(::MyType, n::Integer = 10; add_marker = false)
    linecolor   --> :blue
    seriestype  :=  :path
    markershape --> (add_marker ? :circle : :none)
    delete!(plotattributes, :add_marker)
    rand(n)
end
```

我们创建一个新的类型 `MyType`，它是空的，并且纯粹用于分派。我们的目标是创建一个由 `n` 个点组成的随机路径。

有一些重要的事情需要知道，在此之后，配方就可以通过更新属性字典和返回输入数据来简化：

- 配方签名 `f(args...; kw...)` 被转换为 `apply_recipe(plotattributes::KW, args...)` 的定义，其中：
    - `plotattributes` 是类型为 `typealias KW Dict{Symbol,Any}` 的属性字典
    - 你的 `args` 必须足够独特，以便分派会调用你的定义（并且不会遮盖现有的定义）。使用自定义数据类型将确保正确的分派。
    - 函数 `f` 是无用的/无意义的... 你可以随意命名它。
- 特殊操作符 `-->` 将 `linecolor --> :blue` 转换为 `get!(plotattributes, :linecolor, :blue)`，只有当属性不存在时才设置属性。（提示：对于复杂的表达式，将右侧用括号包起来。）
- 特殊操作符 `:=` 将 `seriestype := :path` 转换为 `plotattributes[:seriestype] = :path`，强制该属性值。（提示：对于复杂的表达式，将右侧用括号包起来。）
- 在配方中不能使用别名（如 `colour` 或 `alpha`），只能使用完整的属性名。
- 配方的返回值是 `RecipeData` 对象的 `args`，该对象还引用了属性字典。
- 配方返回一个 Vector{RecipeData}。我们将在后面看到如何使用 `@series` 宏向这个列表添加内容。

!!! compat "RecipesBase 0.9"
    在配方中使用 `return` 关键字需要至少 RecipesBase 0.9。

分解上述例子：

在上述例子中，我们使用 `MyType` 进行分派，带有可选的位置参数 `n::Integer`：

```julia
@recipe function f(::MyType, n::Integer = 10; add_marker = false)
```

通过调用 `plot(MyType())` 或类似的方式，这个配方将被调用。如果 `linecolor` 尚未设置，它将被设置为 `:blue`：

```julia
    linecolor   --> :blue
```

强制 `seriestype` 为 `:path`：

```julia
    seriestype  :=  :path
```

`markershape` 稍微复杂一些；它检查 `add_marker` 自定义关键字，但只有在 `markershape` 尚未设置时才检查。（注意：`add_marker` 键是冗余的，用户可以直接设置标记形状... 我只是用它来演示）：

```julia
    markershape --> (add_marker ? :circle : :none)
```

然后返回要绘制的数据。
```julia
    rand(n)
end
```

我们的（大部分无用）配方的一些示例用法：

```@example recipes
mt = MyType()
plot(
    plot(mt),
    plot(mt, 100, linecolor = :red),
    plot(mt, marker = (:star,20), add_marker = false),
    plot(mt, add_marker = true)
)
```

---

### 用户配方

上面的例子是一个"用户配方"的例子，你在其中定义了完整的调度签名。用户配方（和其他配方一样）可以叠加和模块化。以下是有效的：

```julia
@recipe f(mt::MyType, n::Integer = 10) = (mt, rand(n))
@recipe f(mt::MyType, v::AbstractVector) = (seriestype := histogram; v)
```

在这里，对`plot(MyType())`的调用将按顺序应用这些配方；首先将`mt`映射到`(mt, rand(10))`，然后将seriestype设置为`:histogram`。

```@example recipes
plot(MyType())
```

---

### 类型配方

对于一些自定义数据类型，它们本质上是围绕内置容器的轻量级包装。例如，你可能有一个类型：

```julia
mutable struct MyWrapper
    v::Vector
end
```

在这种情况下，你希望你的`MyWrapper`对象就像向量一样被处理，但是不希望子类型化AbstractArray。没有问题！只需定义一个类型配方来进行转换：

```julia
@recipe f(::Type{MyWrapper}, mw::MyWrapper) = mw.v
```

当调度没有找到适合完整`args...`的配方时，会调用这个签名。所以`plot(rand(10), MyWrapper(rand(10)))`将"顺利工作"。

---

### 系列配方

这是魔法发生的地方。你可以为任意数据创建自己的自定义可视化。快速定义小提琴图，误差条，甚至标准类型的直方图和阶梯图。直方图是一个条形图：

```julia
@recipe function f(::Type{Val{:histogram}}, x, y, z)
    edges, counts = my_hist(y, plotattributes[:bins],
                               normed = plotattributes[:normalize],
                               weights = plotattributes[:weights])
    x := edges
    y := counts
    seriestype := :bar
    ()
end
```

而一个2D直方图实际上是一个热图：

```julia
@recipe function f(::Type{Val{:histogram2d}}, x, y, z)
    xedges, yedges, counts = my_hist_2d(x, y, plotattributes[:bins],
                                              normed = plotattributes[:normalize],
                                              weights = plotattributes[:weights])
    x := centers(xedges)
    y := centers(yedges)
    z := Surface(counts)
    seriestype := :heatmap
    ()
end
```

参数`y`总是被填充，参数`x`通过像`plot(x,y, seriestype =: histogram2d)`这样的调用被填充，对于`z`，`plot(x,y,z, seriestype =: histogram2d)`

下面我将通过一个系列配方来创建盒形图。许多这些"标准"配方在Plots中定义，尽管它们可以在任何地方定义**而不需要包依赖于Plots**。


---


# 案例研究

### 边际直方图

这里我们展示了一个用户配方版本的 `marginalhist` 图表配方，用于 [StatsPlots](https://github.com/JuliaPlots/StatsPlots.jl)。这是一个很好的例子，因为尽管它容易理解，但它利用了一些出色的Plots特性。

边际直方图是比较两个变量的可视化。主要的图是一个2D直方图，其中每个矩形是该桶中数据点的（可能经过归一化和加权的）计数。在主图上方是第一个变量的较小的直方图，主图右侧是第二个变量的直方图。完整的配方：

```@example recipes
@userplot MarginalHist

@recipe function f(h::MarginalHist)
    if length(h.args) != 2 || !(typeof(h.args[1]) <: AbstractVector) ||
        !(typeof(h.args[2]) <: AbstractVector)
        error("Marginal Histograms should be given two vectors.  Got: $(typeof(h.args))")
    end
    x, y = h.args

    # 设置子图
    legend := false
    link := :both
    framestyle := [:none :axes :none]
    grid := false
    layout := @layout [tophist           _
                       hist2d{0.9w,0.9h} righthist]

    # 主要的 histogram2d
    @series begin
        seriestype := :histogram2d
        subplot := 2
        x, y
    end

    # 这些是两个边际直方图的共同点
    fillcolor := :black
    fillalpha := 0.3
    linealpha := 0.3
    seriestype := :histogram

    # 上方的直方图
    @series begin
        subplot := 1
        x
    end

    # 右侧的直方图
    @series begin
        orientation := :h
        subplot := 3
        y
    end
end
```

使用方法：

```@example recipes
using Distributions
n = 1000
x = rand(Gamma(2), n)
y = -0.5x + randn(n)
marginalhist(x, y, fc = :plasma, bins = 40)
```

---

现在我将详细解释每一部分：

`@userplot` 宏是创建一个新的输入参数包装器的便利工具，这些参数在调度过程中可以是不同的。它还创建了小写的便利方法（`marginalhist` 和 `marginalhist!`）并导出它们。

```julia
@userplot MarginalHist
```

因此创建了一个用于调度的类型 `MarginalHist`。类型 `MarginalHist` 的对象有一个字段 `args`，这是调用绘图函数时的参数元组，可以是 `marginalhist(x,y,...)` 或 `plot(x,y, seriestype = :marginalhist)`。第一种语法是由 `@userplot` 宏创建的简写。

我们只调度生成的类型，因为真实的输入被包装在它内部：

```julia
@recipe function f(h::MarginalHist)
```

一些错误检查。注意我们将真实的输入（如在调用 `marginalhist(randn(100), randn(100))` 中）提取到 `x` 和 `y` 中：

```julia
    if length(h.args) != 2 || !(typeof(h.args[1]) <: AbstractVector) ||
        !(typeof(h.args[2]) <: AbstractVector)
        error("Marginal Histograms should be given two vectors.  Got: $(typeof(h.args))")
    end
    x, y = h.args
```

接下来我们构建子图布局并定义一些属性。有几点需要注意：

- 布局创建了三个子图（`_` 为空）
- 当以矩阵（行向量）形式传入时，属性会映射到每个子图
- 属性 `link := :both` 意味着每行的 y 轴（以及每列的 x 轴）将共享数据极值。其他值包括 `:x`，`:y`，`:all` 和 `:none`。

```julia
    # 设置子图
    legend := false
    link := :both
    framestyle := [:none :axes :none]
    grid := false
    layout := @layout [tophist           _
                       hist2d{0.9w,0.9h} righthist]
```

定义主图的系列。`@series` 宏使用 "let block" 复制属性字典 `plotattributes`。复制的字典和返回的参数被添加到从配方返回的 `Vector{RecipeData}` 中。这个块类似于调用 `histogram2d!(x, y; subplot = 2, plotattributes...)`（但实际上你不会想这样做）。

注意：这个 `@series` 块获取属性的 "快照"，所以它包含了这个块之前设置的任何东西，但不包含之后的。`@series` 块可以是独立的，就像这些一样，或者它们可以在循环中。

```julia
    # 主要的 histogram2d
    @series begin
        seriestype := :histogram2d
        subplot := 2
        x, y
    end
```

接下来我们转到边际图。我们首先设置两者都共享的属性：

```julia
    # 这些是两个边际直方图的共同点
    fillcolor := :black
    fillalpha := 0.3
    linealpha := 0.3
    seriestype := :histogram
```

现在我们创建两个更多的系列，每个直方图一个。

```julia
    # 上方的直方图
    @series begin
        subplot := 1
        x
    end

    # 右侧的直方图
    @series begin
        orientation := :h
        subplot := 3
        y
    end
end
```

需要注意的是：通常我们会从配方中返回参数，这些参数会被添加到一个 `RecipeData` 对象中，并推送到我们的 `Vector{RecipeData}` 上。然而，当使用 `@series` 宏创建系列时，你可以选择返回 `nothing`，这将绕过最后一步。

人们也可以在单个子图中有多个系列，并在需要的情况下重复多个子图。这将需要提供正确的子图 id/编号。

```julia
mutable struct SeriesRange
    range::UnitRange{Int64}
end
@recipe function f(m::SeriesRange)
    range = m.range
    layout := length(range)
    for i in range
        @series begin
            subplot := i
            seriestype := scatter
            rand(10)
        end
        @series begin
            subplot := i
            rand(10)
        end
    end
end
```
---

### 记录绘图函数

在配方定义上方添加的文档字符串不会产生任何效果，就像函数名一样没有意义。由于 Julia 中的所有内容都可以与 doc-string 关联，因此可以像这样将文档添加到绘图函数的名称中：
```julia
"""
我的文档字符串
"""
my_plotfunc
```
这可以放在代码的任何地方，并且会在调用 `?my_plotfunc` 时出现。

---

### 故障排除

在调试配方时，有时查看 `apply_recipe` 调用内部的调度顺序可能会有所帮助。用以下方式打开调试信息：

```julia
RecipesBase.debug()
```

你也可以传递一个 `Bool` 给 `debug` 方法来打开/关闭它。

以下是一些常见的错误，以及需要注意的事项：

#### convertToAnyVector

```
错误：在 convertToAnyVector 中，无法处理参数类型：<<某种类型>>
    [内联代码] 来自 ~/.julia/v0.4/Plots/src/series_new.jl:87
    在 apply_recipe 中，位于 ~/.julia/v0.4/RecipesBase/src/RecipesBase.jl:237
    在 _plot! 中，位于 ~/.julia/v0.4/Plots/src/plot.jl:312
    在 plot 中，位于 ~/.julia/v0.4/Plots/src/plot.jl:52
```

当输入类型无法被配方处理时，会发生这个错误。类型 `<<某种类型>>` 无法被处理。请记住，对于复杂的绘图，可能会递归调用多个配方。

#### MethodError: `start`没有与start(::Void)匹配的方法

```
错误：MethodError: `start`没有与start(::Void)匹配的方法
    在 collect 中，位于 ./array.jl:260
    在 collect 中，位于 ./array.jl:272
    在 plotly_series 中，位于 ~/.julia/v0.4/Plots/src/backends/plotly.jl:345
    在 _series_added 中，位于 ~/.julia/v0.4/Plots/src/backends/plotlyjs.jl:36
    在 _apply_series_recipe 中，位于 ~/.julia/v0.4/Plots/src/plot.jl:224
    在 _plot! 中，位于 ~/.julia/v0.4/Plots/src/plot.jl:537
```

当一个系列类型期望`x`，`y`或`z`的数据，但是却传递了`nothing`（其类型为`Void`）时，通常会遇到这个错误。检查你是否为3D图定义了`z`值，同样，检查你是否为`x`和`y`定义了有效的值。这也可能适用于像`fillrange`，`marker_z`，或`line_z`这样的属性，如果它们预期非空值。

#### MethodError: 无法将类型为Float64的对象`convert`为类型为RecipeData的对象

```
错误：MethodError: 无法将类型为Float64的对象`convert`为类型为RecipeData的对象
最接近的候选者是：
  convert(::Type{T}, ::T) where T at essentials.jl:171
  RecipeData(::Any, ::Any) at ~/.julia/packages/RecipesBase/G4s6f/src/RecipesBase.jl:57
```
!!! compat "RecipesBase 0.9"
    在配方中使用`return`关键字需要RecipesBase 0.9

如果你在配方中使用`return`关键字，这将会引发错误，这在RecipesBase最高到v0.8中是不支持的。

