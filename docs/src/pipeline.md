```@setup pipeline
using Plots; gr()
Plots.reset_defaults()
```

# [处理管道](@id pipeline)

绘图命令将通过一系列预处理步骤发送输入，以便转换、简化和概括。其目的在于，终端用户需要在调用方式（以及如何调用）上有极大的灵活性。他们可能希望完全控制绘图属性，或者完全不控制。可能有8个属性是常数，但有一个属性会根据数据系列变化。我们需要能够轻松地在彼此之上叠加复杂的图表，并轻松定义它们应该是什么样子。输入数据可能以任何形式出现。

我将介绍在调用 `plot()` 或 `plot!()` 后发生的步骤，并暗示由此产生的强大和灵活性。

### 一个例子

假设我们有数据：

```@example pipeline; continued = true
n = 100
x, y = range(0, 1, length = n), randn(n, 3)
```

我们想要将 `x` 与 `y` 的每一列进行可视化。这是在 Plots 中的一个样本命令：

```@example pipeline
使用 Plots; pythonplot(size = (400, 300))
plot(
    x, y,
    line = (0.5, [4 1 0], [:path :scatter :histogram]),
    normalize = true,
    bins = 30,
    marker = (10, 0.5, [:none :+ :none]),
    color = [:steelblue :orangered :green],
    fill = 0.5,
    orientation = [:v :v :h],
    title = "My title",
)
```

在这个例子中，我们有一个输入矩阵，我们希望在其上绘制三个系列，每个系列对应数据的一列。我们创建一个符号的行向量（1x3 矩阵）来为每个系列分配不同的可视化类型，设置直方图的方向，并设置 alpha 值。

为了比较，这与 PythonPlot 中的以下调用有些类似：

```@example pipeline
import PythonPlot
fig = PythonPlot.gcf()
fig.set_size_inches(4, 3, forward = true)
fig.set_dpi(100)
PythonPlot.clf()

n = 100
x, y = range(0, 1, length = n), randn(n, 3)

PythonPlot.plot(x, y[:,1], alpha = 0.5, "steelblue", linewidth = 4)
PythonPlot.scatter(x, y[:,2], alpha = 0.5, marker = "+", s = 100, c="orangered")
PythonPlot.hist(
    y[:,3],
    orientation = "horizontal",
    alpha = 0.5,
    density = true,
    bins=30,
    color="green",
    linewidth = 0
)

ax = PythonPlot.gca()
ax.xaxis.grid(true)
ax.yaxis.grid(true)
PythonPlot.title("My title")
PythonPlot.legend(["y1","y2"])
PythonPlot.savefig("pythonplot.svg")
nothing  #hide
```
![](pythonplot.svg)

---



### [步骤1：预处理属性](@id step-1-replace-aliases)

详见 [替换别名](@ref aliases) 和 [魔法参数](@ref magic-arguments)。

然后，有一些参数会被简化和压缩，例如将布尔设置 `colorbar = false` 转换为内部描述 `colorbar = :none`，以便在不复杂的接口下允许复杂的行为，用不可见的 `RGBA(0,0,0,0)` 替换 `nothing`，等等。

---



### [步骤2：处理输入数据：用户配方，分组，等等](@id step-2-handle-magic-arguments)

Plots 很少会要求你预处理自己的输入。你有 Julia 数组？很好。DataFrame？没问题。Surface 函数？你搞定了。

在这一步中，Plots 将在绘图类型和其他输入的上下文中转换你的输入数据，生成一个切片和/或扩展表示的列表，其中每个项目代表一个绘图系列的数据。在底层，它大量使用 [多重分派](https://docs.julialang.org/en/release-0.4/manual/methods/) 和 [配方](@ref recipes)。

输入会递归处理，直到找到匹配的配方。这意味着你可以制作模块化和分层的配方，就像处理 Plots 内置的任何东西一样。

```@example pipeline
Plots.reset_defaults() # hide
mutable struct MyVecWrapper
  v::Vector{Float64}
end
mv = MyVecWrapper(rand(10))

@recipe function f(mv::MyVecWrapper)
    markershape --> :circle
    markersize  --> 8
    mv.v
end

plot(
    plot(mv.v),
    plot(mv)
)
```

注意，如果分派没有找到输入组合的配方，那么它将尝试将 [类型配方](@ref type-recipes) 应用到每个单独的参数。

这个钩子给了我们一个很好的方式来替换输入数据，并为用户类型添加自定义可视化属性。像误差条、回归线、带状图和分组过滤等也在这个递归过程中处理。

分组：当你想要将一个数据系列分割成多个绘图系列时，你可以使用 `group` 关键字。属性可以应用到结果系列，就像你的数据已经被分割成不同的输入数据一样。`group` 变量决定如何分割数据，并分配图例标签。

在这个例子中，我们随机地将数据点分割成3组，并给它们不同的标记形状（`[:s :o :x]` 是 `:star5`、`:octagon` 和 `:xcross` 的别名）。其他属性（`:markersize` 和 `:markeralpha`）是共享的。

```@example pipeline
scatter(rand(100), group = rand(1:3, 100), marker = (10,0.3, [:s :o :x]))
```

---



### 步骤3：初始化和更新 Plot 和 Subplots

应用于 Plot、Subplot 或 Axis 对象的属性被提取出来并处理。触发初始化图形/窗口的后端方法，并构建 [布局](@ref layouts)。

---



### 步骤4：系列配方

这部分有些神奇。在前三步之后，我们有一个关键字字典列表（类型 `KW`），其中既包含数据也包含属性。现在我们将递归应用 [系列配方](@ref series-recipes)，首先检查后端是否原生支持一个系列类型，如果不支持，应用一个系列配方并重新处理。

结果是，可以创建通用的配方（例如，将直方图转换为条形图），这将把系列降低到后端支持的最高级别类型。由于配方很容易创建，我们可以在原生支持的功能很少的后端中进行复杂的可视化。

---



### 步骤5：准备输出

大部分的重处理都被推迟到需要的时候。Plots会尽量避免昂贵的图形更新，直到你实际选择[显示](@ref output)绘图。就在显示之前，我们将计算子图和其他绘图组件的布局细节和边界框，然后触发回调到后端代码以绘制/更新绘图。

---

### 步骤6：显示

打开/刷新一个 GUI 窗口，写入到一个文件，或者在 IJulia 中内联显示。请记住，在 IJulia 或 REPL 中，**只有当返回一个 Plot 时才会显示**（分号会抑制返回），或者如果用 `display()`、`gui()` 显示，或者在你的绘图命令中添加 `show = true`。

!!! 提示
    你可以通过设置默认值：default(show = true) 来实现类似 MATLAB 的交互行为
---
