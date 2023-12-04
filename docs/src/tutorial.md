```@setup tutorial
using Plots; gr()
Plots.reset_defaults()
```

# [教程](@id tutorial)

这是一个让你快速上手 Plots.jl 的指南。其主要目标是
向你介绍在这个包中使用的术语，如何在
常见的使用场景中使用 Plots.jl，并使你能够轻松理解其余部分
的手册。建议在
REPL 或交互式笔记本中跟随代码示例。

## 基础绘图：线图

在通过 `Pkg.add("Plots")` 安装了 Plots.jl 之后，第一步是
初始化这个包。根据你的电脑，这将需要几秒钟：

```@example tutorial
using Plots
```

首先，让我们绘制一些三角函数。对于 `x` 坐标，我们可以
创建一个从0到10的范围，例如，100个元素。对于 `y` 坐标，我们
可以通过逐元素方式评估 `sin(x)` 来创建一个向量。在 Julia 中，我们在函数调用后插入一个点来实现这个操作。最后，我们使用 `plot()`
来绘制线条。

```@example tutorial
x = range(0, 10, length=100)
y = sin.(x)
plot(x, y)
```

绘图会显示在一个绘图窗格、一个独立的窗口或浏览器中，
这取决于环境和后端(见[下面](@ref plotting-backends))。

如果这是你本次会话的第一次绘图，并且它需要一段时间才能显示出来，
这是正常的；这种延迟被称为 "首次绘图时间" 问题 (或 `TTFP`)，
后续的绘图将会快很多。由于 Julia 在底层的工作方式，
这是一个难以解决的问题，但是在过去的几年中已经取得了很大的进步，以减少这个编译时间。

在 Plots.jl 中，每一列都是一个**系列**，一组相关的点，形成线条、表面或其他绘图基元。我们可以通过绘制值的矩阵来绘制多条线，其中每一列被解释为一条单独的线。下面，`[y1 y2]` 形成一个 100x2 的矩阵 (100个元素，2列)。

```@example tutorial
x = range(0, 10, length=100)
y1 = sin.(x)
y2 = cos.(x)
plot(x, [y1 y2])
```

另外，我们可以通过改变绘图对象来添加更多的线。这是通过 `plot!` 命令完成的，其中 `!` 表示该命令正在修改当前的绘图。
你会注意到我们还使用了一个 `@.` 宏。这是一个便利的宏，
它为宏右边的每一个函数调用插入点，确保
整个表达式是以逐元素的方式进行评估的。
如果我们手动输入点，我们需要三个点，分别用于正弦、
指数和减法，结果的代码可读性会较差。

```@example tutorial
y3 = @. sin(x)^2 - 1/2   # equivalent to y3 = sin.(x).^2 .- 1/2
plot!(x, y3)
```

注意，我们可以使用明确的绘图变量来完成上面的操作，
我们称之为 `p`：

```@example tutorial
x = range(0, 10, length=100)
y1 = sin.(x)
y2 = cos.(x)
p = plot(x, [y1 y2])

y3 = @. sin(x)^2 - 1/2
plot!(p, x, y3)
```

在省略绘图变量的情况下，Plots.jl 自动使用全局
的 `Plots.CURRENT_PLOT`。

### 保存图形

保存图表是通过 `savefig` 命令完成的。例如：

```julia
savefig("myplot.png")      # 将CURRENT_PLOT保存为.png
savefig(p, "myplot.pdf")   # 将p的图表保存为.pdf矢量图形
```

还存在方便函数 `png`，`Plots.pdf` 和其他未导出的辅助函数。使用这些函数时，文件名省略了扩展名。以下代码等同于上述代码：

```julia
png("myplot")
Plots.pdf(p, "myplot")
```

关于输出图形的更多信息可以在手册的 [输出](@ref output) 部分找到。

## 图形属性

在上一节中，我们已经画出了图形...我们完成了吗？没有！我们需要对图形进行样式设计。在Plots.jl中，对图形的修改被称为**属性**，这些在[属性页面](@ref attributes)有详细的文档。Plots.jl对数据和属性遵循两个简单的规则：

* 位置参数对应输入数据
* 关键字参数对应属性

所以像`plot(x, y, z)`这样的是对没有属性的3D图的三维数据，而`plot(x, y, attribute=value)`是带有一个属性的二维数据，该属性被赋予了某个值。

举个例子，我们可以使用`linewidth`（或它的别名`lw`）来改变线宽，使用`label`来改变图例的标签，使用`title`来添加标题。注意`["sin(x)" "cos(x)"]`有与数据相同的列数。另外，因为线宽被赋予了`[y1 y2]`，所以两条线都会受到赋值的影响。让我们把所有这些应用到我们之前的图上：

```@example tutorial
x = range(0, 10, length=100)
y1 = sin.(x)
y2 = cos.(x)
plot(x, [y1 y2], title="Trigonometric functions", label=["sin(x)" "cos(x)"], linewidth=3)
```

每个属性也可以通过修改函数来改变图形。一些属性有自己的专门的修改函数，而其他的可以通过`plot!(attribute=value)`来访问。例如，`xlabel`属性为x轴添加一个标签。我们可以在绘图命令中使用`xlabel=...`来指定它，或者我们可以使用下面的修改函数在图形已经生成后添加它。这取决于你认为哪种方式更有利于代码的可读性。

```julia
xlabel!("x")
```

每个修改函数都是属性的名称后面加上`!`。这将隐式使用全局的`Plots.CURRENT_PLOT`。我们可以通过`attribute!(p, value)`将它应用到其他的图形对象上，其中`p`是想要被修改的图形对象的名称。

让我们使用关键字和修改函数来对我们的示例进行一些常见的修改，如下所示。你会注意到，对于`ls`和`legend`这两个属性，值包含一个冒号`:`。冒号在Julia中表示一个符号。它们通常用于Plots.jl中的属性的值，以及字符串和数字。

* 图例中各条线的标签
* 线宽（我们将使用别名`lw`代替`linewidth`）
* 线型（我们将使用别名`ls`代替`linestyle`）
* 图例位置（在图的外部，因为默认的会使图看起来混乱）
* 图例列数（3，以更好地利用水平空间）
* X轴的限制从`0`到`2pi`
* 图的标题和轴标签

```@example tutorial
x = range(0, 10, length=100)
y1 = sin.(x)
y2 = cos.(x)
y3 = @. sin(x)^2 - 1/2

plot(x, [y1 y2], label=["sin(x)" "cos(x)"], lw=[2 1])
plot!(x, y3, label="sin(x)^2 - 1/2", lw=3, ls=:dot)
plot!(legend=:outerbottom, legendcolumns=3)
xlims!(0, 2pi)
title!("Trigonometric functions") 
xlabel!("x")
ylabel!("y")
```

注意`y3`被绘制成了一条点线。这与数据的散点图是不同的。

### 对数尺度图

有时候，我们需要在数量级之间绘制数据。在这种情况下，可以将属性 `xscale` 和 `yscale` 设置为 `:log10`。它们也可以设置为 `:identity` 以保持线性尺度。我们需要确保数据和限制都是正数。

```@example tutorial
x = 10 .^ range(0, 4, length=100)
y = @. 1/(1+x)

plot(x, y, label="1/(1+x)")
plot!(xscale=:log10, yscale=:log10, minorgrid=true)
xlims!(1e+0, 1e+4)
ylims!(1e-5, 1e+0)
title!("对数-对数图") 
xlabel!("x")
ylabel!("y")
```

更多关于属性的信息可以在手册的
[属性](@ref attributes)部分找到。

### LaTeX 方程字符串

Plots.jl 可以与 LaTeXStrings.jl 一起使用，这是一个允许用户在字符串字面量中输入 LaTeX 方程的包。要安装它，请输入 `Pkg.add("LaTeXStrings")`。使用它的最简单方法是在 LaTeX 格式的字符串前添加 `L`。如果字符串是普通文本和 LaTeX 方程的混合，需要插入美元符号 `。

```@example tutorial
using LaTeXStrings

x = 10 .^ range(0, 4, length=100)
y = @. 1/(1+x)

plot(x, y, label=L"\frac{1}{1+x}")
plot!(xscale=:log10, yscale=:log10, minorgrid=true)
xlims!(1e+0, 1e+4)
ylims!(1e-5, 1e+0)
title!(L"对数-对数图 of $\frac{1}{1+x}$") 
xlabel!(L"x")
ylabel!(L"y")
```

## 更改序列类型：散点图

至此，你已经了解了线形图，但是你不想以其他方式绘制你的数据吗？在Plots.jl中，这些其他的绘图方式被称为**序列类型**。线形图是一种序列类型。然而，散点图是另一种常用的序列类型。

让我们再次从正弦函数开始，但这次，我们将定义一个名为`y_noisy`的向量，其中添加了一些随机性。我们可以使用`seriestype`属性来改变序列类型。

```@example tutorial
x = range(0, 10, length=100)
y = sin.(x)
y_noisy = @. sin(x) + 0.1*randn()

plot(x, y, label="sin(x)")
plot!(x, y_noisy, seriestype=:scatter, label="data")
```

对于每种内置的序列类型，都有一个与其名称匹配的直接调用该序列类型的简写函数。它处理属性的方式与`plot`命令相同，并且它有一个以`!`结束的变异形式。例如，我们可以将最后一行写为：

```julia
scatter!(x, y_noisy, label="data")
```

可用的序列类型取决于后端，并在[支持的属性页面](@ref supported)上有文档记录。如我们稍后将描述的，其他库可以使用**配方**添加新的序列类型。

散点图将具有一些与标记相关的常见属性。下面是相同图表的一个示例，但是其中添加了一些属性以使图表更具表现力。为了简洁，使用了许多别名，以下列表并非详尽无遗。

* `lc`代表`linecolor`
* `lw`代表`linewidth`
* `mc`代表`markercolor`
* `ms`代表`markersize`
* `ma`代表`markeralpha`

```@example tutorial
x = range(0, 10, length=100)
y = sin.(x)
y_noisy = @. sin(x) + 0.1*randn()

plot(x, y, label="sin(x)", lc=:black, lw=2)
scatter!(x, y_noisy, label="data", mc=:red, ms=2, ma=0.5)
plot!(legend=:bottomleft)
title!("Sine with noise")
xlabel!("x")
ylabel!("y")
```

## [绘图后端](@id plotting-backends)

Plots.jl是一个绘图元包：它是对许多不同绘图库的接口。
Plots.jl实际上在做的是解释你的命令，然后
使用另一个名为**后端**的绘图库来生成图表。
这样做的好处是你可以使用许多不同的绘图库
都使用Plots.jl的语法，我们将在稍后看到Plots.jl
为每个库增加了新的功能！

当我们开始上面的绘图时，我们的图表使用了默认的后端GR。
然而，假设我们想要一个不同的绘图后端，它可以绘制到
一个漂亮的图形用户界面或VS Code的绘图窗格。为此，我们需要一个后端
与这些功能兼容。一些常见的后端包括
PythonPlot和Plotly。例如，要安装PythonPlot，只需在REPL中输入命令
`Pkg.add("PythonPlot")`；要安装Plotly，输入
`Pkg.add("PlotlyJS")`。

我们可以通过使用后端的名称（全部小写）作为函数来特别选择我们要绘制的后端。让我们用Plotly和GR来绘制上面的例子：

```@example tutorial
plotlyjs()   # 设置后端为Plotly

x = range(0, 10, length=100)
y = sin.(x)
y_noisy = @. sin(x) + 0.1*randn()

# 这将通过Plotly在一个独立的窗口中绘制
plot(x, y, label="sin(x)", lc=:black, lw=2)
scatter!(x, y_noisy, label="data", mc=:red, ms=2, ma=0.5)
plot!(legend=:bottomleft)
title!("用Plotly绘制的带噪声的正弦")
xlabel!("x")
ylabel!("y")
png("plotlyjs_tutorial")  #hide
```
![](plotlyjs_tutorial.png)

```@example tutorial
gr()   # 设置后端为GR

# 这将使用GR进行绘制
plot(x, y, label="sin(x)", lc=:black, lw=2)
scatter!(x, y_noisy, label="data", mc=:red, ms=2, ma=0.5)
plot!(legend=:bottomleft)
title!("用GR绘制的带噪声的正弦")
xlabel!("x")
ylabel!("y")
```

每个绘图后端都有非常不同的感觉。有些具有交互性，有些
更快，可以处理大量的数据点，有些可以做
3D绘图。像GR这样的后端可以保存为矢量图形和PDF，而
像Plotly这样的其他后端只能保存为PNG。

有关后端的更多信息，请参见[后端页面](@ref backends)。
有关来自各种后端的绘图示例，请参见示例部分。

## 在脚本中绘图

在教程开始时，我们建议你在交互式会话中跟随代码示例，原因如下：尝试将同样的绘图命令添加到脚本中。现在调用脚本...图像没有显示出来？这是因为Julia在通过REPL进行交互使用时，会对每个没有分号`;`的命令返回的变量调用`display`。在上述每种情况中，交互式使用都会自动在返回的绘图对象上调用`display`。

在脚本中，Julia不会自动显示，这就是为什么`;`不是必需的。然而，如果我们想在脚本中显示我们的绘图，这意味着我们只需要添加`display`调用。例如：

```julia
display(plot(x, y))
```

或者，我们可以在最后调用`gui()`来做同样的事情。
最后，如果我们有一个绘图对象`p`，我们可以输入`display(p)`来显示绘图。

## 将多个绘图组合为子图

我们可以使用**布局**将多个绘图组合为子图。有许多方法可以做到这一点，我们将展示两种简单的方法来生成简单的布局。在[布局页面](@ref layouts)中显示了更高级的布局。

第一种方法是定义一个布局，该布局将分割一个系列。`layout`命令接受一个2元组`layout=(N, M)`，它构建一个NxM的绘图网格，并自动将一个系列分割到每个绘图中。例如，如果我们在一个有三个系列的绘图上输入`layout=(3, 1)`，那么我们将得到三行绘图，每个绘图中都有一个系列。

让我们定义一些函数并将它们绘制在单独的绘图中。由于每个绘图中只有一个系列，我们还将使用`legend=false`移除每个绘图中的图例。

```@example tutorial
x = range(0, 10, length=100)
y1 = @. exp(-0.1x) * cos(4x)
y2 = @. exp(-0.3x) * cos(4x)
y3 = @. exp(-0.5x) * cos(4x)
plot(x, [y1 y2 y3], layout=(3, 1), legend=false)
```

我们也可以在绘图对象的绘图上使用布局。例如，我们可以生成四个单独的绘图，并生成一个将它们组合成2x2网格的单一绘图。

```@example tutorial
x = range(0, 10, length=100)
y1 = @. exp(-0.1x) * cos(4x)
y2 = @. exp(-0.3x) * cos(4x)
y3 = @. exp(-0.1x)
y4 = @. exp(-0.3x)
y = [y1 y2 y3 y4]

p1 = plot(x, y)
p2 = plot(x, y, title="Title 2", lw=3)
p3 = scatter(x, y, ms=2, ma=0.5, xlabel="xlabel 3")
p4 = scatter(x, y, title="Title 4", ms=2, ma=0.2)
plot(p1, p2, p3, p4, layout=(2,2), legend=false)
```

注意，单个绘图中的属性应用于这些单个绘图，而最后`plot`调用中的属性`legend=false`应用于所有的子图。

## 图表配方和配方库

你现在已经了解了Plots.jl的所有基本术语，并可以自由地浏览文档，成为绘图大师。然而，还有一件事情需要了解：**配方**。绘图配方是对Plots.jl框架的扩展。它们添加了：

1. 通过**用户配方**新增的`plot`命令。
2. 通过**类型配方**默认解释Julia类型作为绘图数据。
3. 通过**绘图配方**新增的生成图表的函数。
4. 通过**系列配方**新增的系列类型。

编写你自己的配方是一个高级主题，在[配方页面](@ref recipes)上进行了描述。相反，我们将介绍如何使用配方。

配方包含在许多配方库中。两个基本的配方库是[PlotRecipes.jl](https://github.com/JuliaPlots/PlotRecipes.jl) 和 [StatsPlots.jl](https://github.com/JuliaPlots/StatsPlots.jl)。让我们看一下StatsPlots.jl。StatsPlots.jl添加了一堆配方，但我们将关注的是：

1. 它为`Distribution`s添加了一个类型配方。
2. 它为marginal直方图添加了一个绘图配方。
3. 它添加了一堆新的统计绘图系列。

除了配方，StatsPlots.jl还提供了一个专用宏`@df`，用于直接从数据表进行绘图。

### 使用用户配方

用户配方说明了如何解释新数据类型上的绘图命令。在这种情况下，StatsPlots.jl有一个宏`@df`，它允许你直接使用列名绘制`DataFrame`。让我们构建一个包含列`a`、`b`和`c`的`DataFrame`，并告诉Plots.jl使用`a`作为`x`轴，并绘制由列`b`和`c`定义的系列：

```@example tutorial
# Pkg.add("StatsPlots")
# 需要 dataframe 用户配方
using StatsPlots 

# 现在让我们创建 dataframe
using DataFrames
df = DataFrame(a=1:10, b=10*rand(10), c=10*rand(10))

# 通过声明列名来绘制 dataframe
# x = :a, y = [:b :c] (注意 y 有两列！)
@df df plot(:a, [:b :c])
```

这里你不需要做太多：所有以前的命令（属性，系列类型等）仍然适用于这些数据：

```@example tutorial
# x = :a, y = :b
@df df scatter(:a, :b, title="My DataFrame Scatter Plot!") 
```

### 使用类型配方

此外，StatsPlots.jl通过为其分布类型添加类型配方，扩展了Distributions.jl，因此它们可以直接解释为绘图数据：

```@example tutorial
using Distributions
plot(Normal(3, 5), lw=3)
```

类型配方是绘制需要更多干预的专门类型的非常方便的方式！

### 使用绘图配方

StatsPlots.jl通过绘图配方添加了`marginhist`多图。对于我们的数据，我们将从RDatasets中获取著名的`iris`数据集：

```@example tutorial
# Pkg.add("RDatasets")
using RDatasets, StatsPlots
iris = dataset("datasets", "iris")
@df iris marginalhist(:PetalLength, :PetalWidth)
```

这里，`iris`是一个DataFrame；使用上述的`DataFrame`上的`@df`宏，我们给`marginalhist(x, y)`提供了`PetalLength`和`PetalWidth`列的数据。

注意，这不仅仅是一个系列，因为它生成了多个系列（即，由于顶部和右侧的直方图，有多个绘图）。因此，绘图配方不仅仅是一个系列，而且像新的`plot`命令。

### 使用系列配方

StatsPlots.jl还引入了新的系列配方。关键是你不必做任何不同的事情。在`using StatsPlots`之后，你可以简单地使用那些新的系列配方，就像它们被内置到绘图库中一样。让我们在一些随机数据上使用小提琴图：

```@example tutorial
y = rand(100, 4)
violin(["Series 1" "Series 2" "Series 3" "Series 4"], y, legend=false)
```

我们可以使用之前的相同的变异命令在顶部添加一个`boxplot`：

```@example tutorial
boxplot!(["Series 1" "Series 2" "Series 3" "Series 4"], y, legend=false)
```

## 尝试的附加插件

考虑到Plots.jl的易扩展性，你可以尝试许多其他的事情。这里是一份非常实用的插件清单供你查看：

- [PlotThemes.jl](https://github.com/JuliaPlots/PlotThemes.jl)允许你更改你的绘图的颜色方案。例如，`theme(:dark)`添加一个深色主题。
- [StatsPlots.jl](https://github.com/JuliaPlots/StatsPlots.jl)为统计分析的可视化添加了功能
- [生态系统页面](@ref ecosystem)展示了许多其他具有配方并扩展了Plots.jl功能的包。
