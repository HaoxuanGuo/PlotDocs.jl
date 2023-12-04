# [属性](@id attributes)

```@setup attr
使用 Plots
```

### 属性简介

在 Plots 中，输入数据是位置传递的（例如，`plot(y)`中的`y`），属性则作为关键字传递（例如，`plot(y, color = :blue)`）。这个页面上的大部分信息都可以从你的 Julia REPL 中获取。在 REPL 中执行 `using Plots` 后，你可以使用 `plotattr()` 函数打印出所有属性的列表，无论是系列、图表、子图，还是轴。

```julia
# 有效操作
plotattr(:Plot)
plotattr(:Series)
plotattr(:Subplot)
plotattr(:Axis)
```

获取属性列表后，你可以使用特定属性的别名，或者调查特定属性以打印出该属性的别名和描述。

```@repl attr
# 特定属性示例
plotattr("size")
```

!!! note
    不要忘记用双引号括起你试图使用的属性！

---

### [别名](@id aliases)

关键字可以通过**别名机制**接收一系列值。例如，`plot(y, color = :blue)` 实际上被解释为 `plot(y, seriescolor = :blue)`。每个属性都有一些别名（见下面的图表），这可以避免因为忘记参数名而不断查阅绘图 API 文档的痛苦。`c`、`color` 和 `seriescolor` 都表示同一件事，实际上这些最终会被转换为更精确的属性 `linecolor`、`markercolor`、`markerstrokecolor` 和 `fillcolor`（你可以在需要时覆盖这些属性）。

!!! tip
    对于一次性的分析和可视化，使用别名，但对于长期的库代码，使用真正的关键字名称以避免混淆。

---

### [魔术参数](@id magic-arguments)

有些参数包含了设置多个相关参数的智能简写。Plots 使用类型检查和多重分派来智能地"弄清楚"哪些值适用于哪个参数。传入一个值元组。单个值将首先被包装在一个元组中进行处理。

##### axis (和 xaxis/yaxis/zaxis)

将一组设置以元组的形式传递给 `xaxis` 参数将允许快速定义 `xlabel`、`xlims`、`xticks`、`xscale`、`xflip` 和 `xtickfont`。以下两种方式是等价的：

```julia
plot(y, xaxis = ("my label", (0,10), 0:0.5:10, :log, :flip, font(20, "Courier")))

plot(y,
    xlabel = "my label",
    xlims = (0,10),
    xticks = 0:0.5:10,
    xscale = :log,
    xflip = true,
    xtickfont = font(20, "Courier")
)
```

注意，`yaxis` 和 `zaxis` 的工作方式类似，`axis` 会应用到所有的轴。

将一个元组传递给 `xticks`（以及类似地传递给 `yticks` 和 `zticks`）会改变刻度的位置和标签：

```julia
plot!(xticks = ([0:π:3*π;], ["0", "\\pi", "2\\pi"]))
yticks!([-1:1:1;], ["min", "zero", "max"])
```

##### line

设置与系列线相关的属性。别名：`l`。以下两种方式是等价的：

```julia
plot(y, line = (:steppre, :dot, :arrow, 0.5, 4, :red))

plot(y,
    seriestype = :steppre,
    linestyle = :dot,
    arrow = :arrow,
    linealpha = 0.5,
    linewidth = 4,
    linecolor = :red
)
```

##### fill

设置与系列填充区域相关的属性。别名：`f`，`area`。以下两种方式是等价的：

```julia
plot(y, fill = (0, 0.5, :red))

plot(y,
    fillrange = 0,
    fillalpha = 0.5,
    fillcolor = :red
)
```

##### marker

设置与系列标记相关的属性。别名：`m`，`mark`。以下两种方式是等价的：

```julia
scatter(y, marker = (:hexagon, 20, 0.6, :green, stroke(3, 0.2, :black, :dot)))

scatter(y,
    markershape = :hexagon,
    markersize = 20,
    markeralpha = 0.6,
    markercolor = :green,
    markerstrokewidth = 3,
    markerstrokealpha = 0.2,
    markerstrokecolor = :black,
    markerstrokestyle = :dot
)
```

### [值得注意的参数](@id notable-arguments)
这是一些不太为人知的参数的集合：

```julia
scatter(y, thickness_scaling = 2)  # 字体大小和线宽增加2倍
# 适用于演示和海报
# 如果后端不支持这个功能，使用函数 `scalefontsizes(2)` 来缩放
# 默认的字体大小。

scatter(y, ticks=:native)  # 告诉后端自己计算刻度。
# 如果你使用的是交互式后端并进行鼠标缩放，这是个好主意

scatter(rand(100), smooth=true)  # 在你的图表中添加一条回归线
```
