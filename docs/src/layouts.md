```@setup layouts
using Plots; gr()
Plots.reset_defaults()
```

# [布局](@id layouts)

从 v0.7.0 开始，Plots 接管了子图的定位，允许复杂、嵌套的子图和组件网格。我们已经尽力保持框架的灵活性和通用性，因此后端只需要支持精确定义子图的绝对位置的能力，就可以获得嵌套、绘图区域对齐等全部功能。只需在调用 `plot(...)` 时设置 `layout` 关键字即可。

此时，回顾一下术语是有帮助的：

- **Plot**：整个图/窗口
- **Subplot**：一个子图，包含标题、轴、颜色条、图例和绘图区域。
- **Axis**：子图的一个轴，包含轴指南（标签）、刻度标签和刻度线。
- **Plot Area**：数据显示的子图部分... 包含系列、网格线等。
- **Series**：数据的一个独特可视化。（例如：一条线或一组标记）

---

#### 简单布局

将整数传递给 `layout`，使其自动计算该许多子图的网格大小：

```@example layouts
# 创建一个 2x2 的网格，并将 4 个系列中的每一个映射到一个子图
plot(rand(100, 4), layout = 4)
```

将元组传递给 `layout` 以创建该大小的网格：

```@example layouts
# 创建一个 4x1 的网格，并将 4 个系列中的每一个映射到一个子图
plot(rand(100, 4), layout = (4, 1))
```

可以使用 `grid(...)` 构造函数创建更复杂的网格布局：

```@example layouts
plot(rand(100, 4), layout = grid(4, 1, heights=[0.1 ,0.4, 0.4, 0.1]))
```

可以轻松添加标题和标签：

```@example layouts
plot(rand(100,4), layout = 4, label=["a" "b" "c" "d"],
    title=["1" "2" "3" "4"])
```

---

#### 高级布局

`@layout` 宏是定义复杂布局的最简单方法，它使用 Julia 的[多维数组构造](https://docs.julialang.org/en/v1/manual/arrays/#man-array-concatenation)作为自定义布局语法的基础。可以使用花括号实现精确的尺寸，否则空闲空间将在子图的**绘图区域**之间平均分配。

符号本身（下面示例中的 `a` 和 `b`）可以是任何有效的标识符，并没有任何特殊含义。

```@example layouts
l = @layout [
    a{0.3w} [grid(3,3)
             b{0.2h}  ]
]
plot(
    rand(10, 11),
    layout = l, legend = false, seriestype = [:bar :scatter :path],
    title = ["($i)" for j in 1:1, i in 1:11], titleloc = :right, titlefont = font(8)
)
```

---

使用 `inset_subplots` 属性创建内嵌（浮动）子图。`inset_subplots` 接受一个 (parent_layout, BoundingBox) 元组的列表，其中边界框相对于父元素。

使用 `px`/`mm`/`inch` 表示绝对坐标，`w`/`h` 表示相对于父元素的百分比。原点在左上角。`h_anchor`/`v_anchor` 定义了边界框的 `x`/`y` 输入所指向的内容。

```@example layouts_2
# boxplot 在 StatsPlots 中定义
using StatsPlots, StatsPlots.PlotMeasures
gr(leg = false, bg = :lightgrey)

# 创建一个填充的等高线图和箱线图并排。
plot(contourf(randn(10, 20)), boxplot(rand(1:4, 1000), randn(1000)))

# 在热图上添加一个直方图插图。
# 我们将（可选的）位置相对于第一个子图的右下角设置。
# 调用是 `bbox(x, y, width, height, origin...)`，其中的数字被视为
# "父元素的百分比"。
histogram!(
    randn(1000),
    inset = (1, bbox(0.05, 0.05, 0.5, 0.25, :bottom, :right)),
    ticks = nothing,
    subplot = 3,
    bg_inside = nothing
)

# 添加在窗口中浮动的棒图（插图相对于窗口，而不是相对于子图）
sticks!(
    randn(100),
    inset = bbox(0, -0.2, 200px, 100px, :center),
    ticks = nothing,
    subplot = 4
)
```

### 增量添加子图
你也可以将多个图合并到一个图中。为了做到这一点，只需将保存先前图的变量传递给 `plot` 函数：

```julia
l = @layout [a ; b c]
p1 = plot(...)
p2 = plot(...)
p3 = plot(...)
plot(p1, p2, p3, layout = l)
```

### 在布局中忽略图
你可以使用 `_` 字符在布局中忽略图（空白图）：
```julia
plot((plot() for i in 1:7)..., layout=@layout([_ ° _; ° ° °; ° ° °]))
```
