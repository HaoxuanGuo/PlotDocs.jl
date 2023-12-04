```@setup input_data
using Plots; gr()
Plots.reset_defaults()
```

# [输入数据](@id input-data)

Plots的强大之处在于它支持许多种输入数据的组合。
你不需要花费时间将你的数据转换和整理成特定的格式。
让Plots为你做这个工作。

只需记住几条规则，你很快就能成为高级用户。

## 输入是参数，不是关键字

`plot`函数有几种方法：
`plot(y)`：将输入视为`y`轴的值，并生成一个单位范围作为`x`值。
`plot(x, y)`：创建一个2D图
`plot(x, y, z)`：创建一个3D图

原因在于Julia的多重派发的灵活性，每种输入类型的组合
可以有独特的行为，如果需要的话。

## [列是系列](@id columns-are-series)

在大多数情况下，传入一个(`n` × `m`)的值矩阵（数字等）将创建`m`个系列，每个系列有`n`个数据点。这遵循一个一致的规则…向量应用于一个系列，矩阵应用于多个系列。这个规则也适用于关键字参数。`scatter(rand(10,4), markershape = [:circle, :rect])`将创建4个系列，每个系列都分配了markershape向量[:circle,:rect]。然而，`scatter(rand(10,4), markershape = [:circle :rect])`将创建4个系列，系列1和3的标记形状为`:circle`，系列2和4的标记形状为`:rect`（即正方形）。区别在于，在第一个例子中，它是一个长度为2的列向量，在第二个例子中，它是一个(1 × 2)的行向量（一个矩阵）。

这个的灵活性和强大可以通过以下代码片段来说明：
```@example input_data
using Plots

# 4个系列中的10个数据点
xs = range(0, 2π, length = 10)
data = [sin.(xs) cos.(xs) 2sin.(xs) 2cos.(xs)]

# 我们将标签放在一个行向量中：适用于每个系列
labels = ["Apples" "Oranges" "Hats" "Shoes"]

# Marker shapes in a column vector: applies to data points
markershapes = [:circle, :star5]

# Marker colors in a matrix: applies to series and data points
markercolors = [
    :green :orange :black :purple
    :red   :yellow :brown :white
]

plot(
    xs,
    data,
    label = labels,
    shape = markershapes,
    color = markercolors,
    markersize = 10
)
```
这个例子将四个系列用不同的标签、标记形状和标记颜色绘制出来，通过组合行向量和列向量来装饰数据。

以下示例说明了Plots.jl如何处理：一个矩阵数组，一个数组的数组数组，以及一个数组的数组元组。
```@example input_data
x1, x2 = [1, 0],  [2, 3]    # vectors
y1, y2 = [4, 5],  [6, 7]    # vectors
m1, m2 = [x1 y1], [x2 y2]   # 2x2 matrices

plot([m1, m2])              # array of matrices -> 4 series, plots each matrix column, x assumed to be integer count
plot([[x1,y1], [x2,y2]])    # array of array of arrays -> 4 series, plots each individual array, x assumed to be integer count 
plot([(x1,y1), (x2,y2)])    # array of tuples of arrays -> 2 series, plots each tuple as new series
```

## 同组中的非连续数据

如示例所示，你可以使用单个调用`plot`使用`:path`线类型绘制单个多边形。你可以使用多个调用`plot`绘制多个多边形。

现在，假设你正在绘制`n`个多边形分组到`g`个组，其中`n` > `g`。虽然你可以使用`plot`在每次调用时绘制单独的多边形，但你不能将两个单独的图形组合回一个单独的组。你将在图例中结束`n`个组，而不是`g`个组。

为了解决这个问题，你可以使用`NaN`作为路径分隔符。然后调用`plot`将绘制一个具有不连续的路径。以下代码绘制`n=4`个矩形在`g=2`个组。

```@example input_data
using Plots
plotlyjs()

function rectangle_from_coords(xb,yb,xt,yt)
    [
        xb  yb
        xt  yb
        xt  yt
        xb  yt
        xb  yb
        NaN NaN
    ]
end

some_rects=[
    rectangle_from_coords(1, 1, 5, 5)
    rectangle_from_coords(10, 10, 15, 15)
]
other_rects=[
    rectangle_from_coords(1, 10, 5, 15)
    rectangle_from_coords(10, 1, 15, 5)
]

plot(some_rects[:,1], some_rects[:,2], label = "some group")
plot!(other_rects[:,1], other_rects[:,2], label = "other group")
png("input_data_1") # hide
```
![](input_data_1.png)

## 支持DataFrames

使用[StatsPlots](https://github.com/JuliaPlots/StatsPlots.jl)扩展包，你可以将`DataFrame`作为第一个参数传入（类似于Gadfly或R的ggplot2）对于数据字段或某些属性（如`group`），一个符号将被替换为`DataFrame`的相应列。此外，列名可能会被用作一个例子：

```@example input_data
using StatsPlots, RDatasets
gr()
iris = dataset("datasets", "iris")
@df iris scatter(
    :SepalLength,
    :SepalWidth,
    group = :Species,
    m = (0.5, [:+ :h :star7], 12),
    bg = RGB(0.2, 0.2, 0.2)
)
```

## 函数

函数通常可以替代输入数据，并且它们将按需要进行映射。还可以创建2D和3D的参数图，范围可以给出为向量或最小/最大值。例如，这里有几种创建相同图的替代方法：

```@example input_data
using Plots
tmin = 0
tmax = 4π
tvec = range(tmin, tmax, length = 100)

plot(sin.(tvec), cos.(tvec))
```
```@example input_data
plot(sin, cos, tvec)
```
```@example input_data
plot(sin, cos, tmin, tmax)
```

函数的向量也是允许的（每个函数一个系列）。

## 图像

可以直接使用[Images.jl](https://github.com/timholy/Images.jl)库将图像添加到图中。例如，可以导入一个栅格图像并通过以下命令将其与Plots一起绘制：

```julia
using Plots, Images
img = load("image.png")
plot(img)
```

PDF图形也可以使用`load("image.pdf")`添加到Plots.jl图中。请注意，Images.jl要求PDF的颜色方案是RGB。

## 形状

*Save Gotham*

```@example input_data
using Plots

function make_batman()
    p = [(0, 0), (0.5, 0.2), (1, 0), (1, 2),  (0.3, 1.2), (0.2, 2), (0, 1.7)]
    s = [(0.2, 1), (0.4, 1), (2, 0), (0.5, -0.6), (0, 0), (0, -0.15)]
    m = [(p[i] .+ p[i + 1]) ./ 2 .+ s[i] for i in 1:length(p) - 1]

    pts = similar(m, 0)
    for (i, mi) in enumerate(m)
        append!(
            pts,
            map(BezierCurve([p[i], m[i], p[i + 1]]), range(0, 1, length = 30))
        )
    end
    x, y = Plots.unzip(Tuple.(pts))
    Shape(vcat(x, -reverse(x)), vcat(y, reverse(y)))
end

# background and limits
plt = plot(
    bg = :black,
    xlim = (0.1, 0.9),
    ylim = (0.2, 1.5),
    framestyle = :none,
    size = (400, 400),
    legend = false,
)
```

```@example input_data
# create an ellipse in the sky
pts = Plots.partialcircle(0, 2π, 100, 0.1)
x, y = Plots.unzip(pts)
x = 1.5x .+ 0.7
y .+= 1.3
pts = collect(zip(x, y))

# beam
beam = Shape([(0.3, 0.0), pts[95], pts[50], (0.3, 0.0)])
plot!(beam, fillcolor = plot_color(:yellow, 0.3))
```

```@example input_data
# spotlight
plot!(Shape(x, y), c = :yellow)
```

```@example input_data
# buildings
rect(w, h, x, y) = Shape(x .+ [0, w, w, 0, 0], y .+ [0, 0, h, h, 0])
gray(pct) = RGB(pct, pct, pct)
function windowrange(dim, denom)
    range(0, 1, length = max(3, round(Int, dim/denom)))[2:end - 1]
end

for k in 1:50
    local w, h, x, y = 0.1rand() + 0.05, 0.8rand() + 0.3, rand(), 0.0
    shape = rect(w, h, x, y)
    graypct = 0.3rand() + 0.3
    plot!(shape, c = gray(graypct))

    # windows
    I = windowrange(w, 0.015)
    J = windowrange(h, 0.04)
    local pts = vec([(Float64(x + w * i), Float64(y + h * j)) for i in I, j in J])
    windowcolors = Symbol[rand() < 0.2 ? :yellow : :black for i in 1:length(pts)]
    scatter!(pts, marker = (stroke(0), :rect, windowcolors))
end
plt
```

```@example input_data
# Holy plotting, Batman!
batman = Plots.scale(make_batman(), 0.07, 0.07, (0, 0))
batman = translate(batman, 0.7, 1.23)
plot!(batman, fillcolor = :black)
```

## [额外的关键字](@id extra_kwargs)

有些特性非常特定于某个后端或者还没有在Plots中实现。
对于这些情况，可以将额外的关键字转发给后端。
每个不是Plots关键字的关键字将被收集在一个`extra_kwargs`字典中。

这个字典有三层：`:plot`，`:subplot`和`:series`（默认）。
关键字被收集到哪一层可以由`extra_kwargs`关键字指定。
如果在同一次调用中需要在多个层次传递参数，或者关键字已经是一个有效的Plots关键字，那么必须在调用处构造`extra_kwargs`字典。
```julia
plot(1:5, series_keyword = 5)
# results in extra_kwargs = Dict( :series => Dict( series_keyword => 5 ) )
plot(1:5, colormap_width = 6, extra_kwargs = :subplot)
# results in extra_kwargs = Dict( :subplot => Dict( colormap_width = 6 ) )
plot(1:5, extra_kwargs = Dict( :series => Dict( series_keyword => 5 ), :subplot => Dict( colormap_width => 6 ) ) )
```

参考[追踪问题](https://github.com/JuliaPlots/Plots.jl/issues/2648)来查看这个特性在哪些后端中实现了。
后端实际处理的额外关键字应该在后端文档中有所记录。

!!! warning
    使用额外关键字机制会使你的代码依赖于后端。
    只在最后调整时使用它。在配方中使用显然是一个坏主意。
