```@setup colors
使用 Plots; gr()
Plots.reset_defaults()
```

# 颜色方案

Plots 支持来自 [ColorSchemes.jl](https://juliagraphics.github.io/ColorSchemes.jl/stable/basics/#Pre-defined-schemes-1) 的所有颜色方案。
它们可以作为渐变或调色板使用，并作为一个符号传递给 `cgrad` 或 `palette`。

```@example colors
plot(
    [x -> sin(x - a) for a in range(0, π / 2, length = 5)], 0, 2π;
    palette = :Dark2_5,
)
```

```@example colors
function f(x, y)
    r = sqrt(x^2 + y^2)
    return cos(r) / (1 + r)
end
x = range(0, 2π, length = 30)
heatmap(x, x, f, c = :thermal)
```

### ColorPalette

Plots 会自动从传递给 `color_palette` 属性的调色板中选择系列的颜色。
该属性接受颜色方案名称的符号或 `ColorPalette` 对象。
颜色调色板可以用 `palette(cs, [n])` 构建，其中 `cs` 可以是 `Symbol`，颜色向量，`ColorScheme`，`ColorPalette` 或 `ColorGradient`。
可选参数 `n` 决定从 `cs` 中选择多少种颜色。

```@example colors
palette(:tab10)
```

```@example colors
palette([:purple, :green], 7)
```

### ColorGradient

对于 `heatmap`，`surface`，`contour` 或 `line_z`，`marker_z` 和 `line_z`，Plots.jl 从 `ColorGradient` 中选择颜色。
如果未指定，将使用默认的 `ColorGradient` `:inferno`。
可以通过将颜色方案名称的符号传递给 `seriescolor` 属性来选择不同的渐变。
对于更详细的配置，颜色属性也接受 `ColorGradient` 对象。
颜色渐变可以用以下方式构建
```julia
cgrad(cs, [z], alpha = nothing, rev = false, scale = nothing, categorical = nothing)
```
其中 `cs` 可以是 `Symbol`，颜色向量，`ColorScheme`，`ColorPalette` 或 `ColorGradient`。

```@example colors
cgrad(:acton)
```
您可以将值在 0 和 1 之间的向量作为第二个参数传递，以指定颜色过渡的位置。
```@example colors
cgrad([:orange, :blue], [0.1, 0.3, 0.8])
```
使用 `rev = true` 可以反转颜色方案的颜色。
```@example colors
cgrad(:thermal, rev = true)
```
设置 `categorical = true` 返回一个 `CategoricalColorGradient`，它只从离散的颜色集中选择颜色，而不是连续插值。
可选的第二个参数决定从颜色方案中选择多少种颜色。
它们沿着颜色方案的颜色均匀分布。
```@example colors
cgrad(:matter, 5, categorical = true)
```
分类渐变也接受颜色过渡位置的向量，并且可以反转。
```@example colors
cgrad(:matter, [0.1, 0.3, 0.8], rev = true, categorical = true)
```
颜色选择的分布可以使用接受 `:log`，`:log10`，`:ln`，`:log2`，`:exp` 或应用于 0 和 1 之间的颜色位置值的函数的 `scale` 关键字参数进行缩放。
```@example colors
cgrad(:roma, scale = :log)
```
分类渐变也可以进行缩放。
```@example colors
cgrad(:roma, 10, categorical = true, scale = :exp)
```

# 预定义的颜色方案