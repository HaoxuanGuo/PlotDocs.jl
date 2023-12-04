### 基本概念

使用 `plot` 创建一个新的绘图对象，使用 `plot!` 向现有的绘图对象添加内容：

```julia
plot(args...; kw...)                  # 创建一个新的 Plot，并将其设置为 `current`
plot!(args...; kw...)                 # 修改 `current()` 的 Plot
plot!(plt, args...; kw...)            # 修改 Plot `plt`
```

图形不会隐式显示，只有在"显示"时才会显示。当返回到 REPL 提示符或到 IJulia 单元格时，这将自动发生。还有[许多其他选项](@ref output)。

输入参数可以采取[许多形式](@ref input-data)。一些有效的例子：

```julia
plot()                                       # 空的 Plot 对象
plot(4)                                      # 初始化 4 个空的序列
plot(rand(10))                               # 1 个序列... x = 1:10
plot(rand(10,5))                             # 5 个序列... x = 1:10
plot(rand(10), rand(10))                     # 1 个序列
plot(rand(10,5), rand(10))                   # 5 个序列... 所有 y 值相同
plot(sin, rand(10))                          # y = sin.(x)
plot(rand(10), sin)                          # 同样... y = sin.(x)
plot([sin,cos], 0:0.1:π)                     # 2 个序列，sin.(x) 和 cos.(x)
plot([sin,cos], 0, π)                        # 在范围 [0, π] 上的 sin 和 cos
plot(1:10, Any[rand(10), sin])               # 2 个序列：rand(10) 和 map(sin,x)
@df dataset("Ecdat", "Airline") plot(:Cost)  # DataFrame 中的 :Cost 列... 必须导入 StatsPlots
```

[关键字参数](@ref attributes) 允许自定义绘图，子绘图，轴和序列。他们尽可能地遵循一致的规则，如果你仔细阅读这一部分，你将避免常见的陷阱：

- 许多参数有别名，它们在预处理期间被[替换](@ref step-1-replace-aliases)。`c` 和 `color` 是一样的，`m` 和 `marker` 是一样的，等等。你可以选择你感到舒适的详细程度。
- 有一些[特殊参数](@ref step-2-handle-magic-arguments)，它们可以一次性设置很多相关的内容。
- 如果参数是 "矩阵类型"，那么[每一列将映射到一个序列](@ref columns-are-series)，如果列数少于序列数，则循环通过列。在这个意义上，向量就像一个 "nx1 矩阵" 一样被处理。
- 许多参数接受许多不同的类型...例如，颜色（也包括 markercolor，fillcolor 等）参数将接受带有颜色名称的字符串或符号，或任何 Colors.Colorant，或 ColorScheme，或代表 ColorGradient 的符号，或颜色/符号等的 AbstractVector...

---

### 有用的提示

!!! tip
    一个常见的错误是传递一个 Vector，而你打算让每一项只应用于一个序列。而不是传递一个 n 长度的 Vector，传递一个 1xn 的 Matrix。

!!! tip
    你可以在创建绘图后更新某些绘图设置：
    ```julia
    plot!(title = "New Title", xlabel = "New xlabel", ylabel = "New ylabel")
    plot!(xlims = (0, 5.5), ylims = (-2.2, 6), xticks = 0:0.5:10, yticks = [0,1,5,10])

    # 或使用 magic:
    plot!(xaxis = ("mylabel", :log10, :flip))
    xaxis!("mylabel", :log10, :flip)
    ```

!!! tip
    使用[支持的后端](@ref supported)，你可以为 marker/markershape 参数传递一个 `Plots.Shape` 对象。`Shape` 在构造函数中接受一个 2 元组的向量，定义了多边形形状在单位缩放坐标空间中的点。例如，你可以这样创建一个正方形：`Shape([(1,1),(1,-1),(-1,-1),(-1,1)])`

!!! tip
    你可以使用 `default(arg::Symbol)` 查看给定参数的默认值，并使用 `default(arg::Symbol, value)` 或 `default(; kw...)` 设置默认值。例如，设置默认窗口大小和是否显示图例，可以使用 `default(size=(600,400), leg=false)`。

!!! tip
    调用 `gui()` 在窗口中显示绘图。交互性取决于后端。在 REPL 中绘图（没有分号）会隐式调用 `gui()`。

---
