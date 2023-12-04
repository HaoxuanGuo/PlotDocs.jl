```@setup backends
using StatsPlots, RecipesBase, Statistics; gr()
Plots.reset_defaults()

@userplot BackendPlot

@recipe function f(bp::BackendPlot; n = 4)
    t = range(0, 3π, length = 100)
    d = rand(3, 3)

    layout := n

    @series begin
        subplot := 1
        f = s -> -cos(s) * log(s)
        g = t -> sin(t) * log(t)
        [f g]
    end

    @series begin
        subplot := 2 + (n > 2)
        RecipesBase.recipetype(:groupedbar, d)
    end

    if n > 2
        @series begin
            subplot := 2
            line_z := t
            label := false
            seriescolor := :viridis
            seriestype := surface
            t, t, (x, y) -> x * sin(x) - y * cos(y)
        end

        @series begin
            subplot := 4
            seriestype := contourf
            t, t, (x, y) -> x * sin(x) - y * cos(y)
        end
    end
end
```

# [后端](@id backends)

后端是Plots的命脉，不同后端之间的特性、方法以及优点/缺点的多样性是我开始这个包的主要原因。

对于那些还没有在15种不同的绘图API上进行过编程的人：首先，你应该感到幸运。然而，你可能会在选择适合你手头任务的正确后端上有些困难。这份文档旨在作为一个指南和介绍，帮助你做出选择。

# 持久的后端选择

Plots使用[Preferences](https://github.com/JuliaPackaging/Preferences.jl)机制，使默认后端选择在julia重启后保持不变。

```julia
$ JULIA_PKG_PRECOMPILE_AUTO=0 julia -e 'import Plots; Plots.set_default_backend!(:pythonplot)'
$ julia  # 重启，显示持久模式
julia> using Plots
[ Info: Precompiling Plots [91a5bcdd-55d7-5caf-9e0b-520d859cae80]
[ Info: PythonPlot  # 为此后端预编译
julia> plot(1:2) |> display  # 默认使用`PythonPlot`
```

你可以使用`Plots.set_default_backend!()`清除偏好设置。或者，可以使用环境变量`PLOTS_DEFAULT_BACKEND`选择默认后端（但这需要使用`Base.compilecache(Plots)`手动触发预编译）。

# 一瞥

我的最爱：`GR`对于速度，`Plotly(JS)`对于交互性，`UnicodePlots`对于REPL/SSH，`PythonPlot`对于其他情况。

| 如果你需要...         | 那么使用...                                 |
| :------------------------ | :------------------------------------------ |
| 功能                  | GR, PythonPlot, Plotly(JS), Gaston          |
| 速度                     | GR, UnicodePlots, InspectDR, Gaston         |
| 交互性             | PythonPlot, Plotly(JS), InspectDR           |
| 美观                    | GR, Plotly(JS), PGFPlots/ PGFPlotsX         |
| REPL绘图             | UnicodePlots                                |
| 3D绘图                  | GR, PythonPlot, Plotly(JS), Gaston          |
| GUI窗口              | GR, PythonPlot, PlotlyJS, Gaston, InspectDR |
| 小内存占用         | UnicodePlots, Plotly                        |
| 后端稳定性         | PythonPlot, Gaston                          |
| 绘图+数据 -> `.hdf5` 文件 | HDF5                                        |

当然，这个列表相当主观，生活中没有那么简单。可能存在微妙的后端之间的权衡，长期隐藏的bug，以及更多的兴奋。不要害羞尝试新的东西！

---

## [GR](https://github.com/jheinen/GR.jl)

默认的后端。非常快，有很多种绘图类型。仍在积极开发并且每天都在改进。

```@example backends
gr(); backendplot()  #hide
```

优点：

- 速度
- 2D和3D
- 独立或内联

缺点：

- 交互性有限

主要作者：Josef Heinen (@jheinen)

### 精细调整
可以通过[`extra_kwargs`](@ref extra_kwargs)机制使用`GR`的更多功能。

```@example backends
using Plots; gr()

x = range(-3, 3, length=30)
surface(
  x, x, (x, y)->exp(-x^2 - y^2), c=:viridis, legend=:none,
  nx=50, ny=50, display_option=Plots.GR.OPTION_SHADED_MESH,  # <-- series[:extra_kwargs]
)
```

#### 支持的`:subplot` `:extra_kwargs`

| 关键字        | 描述                         |
| :------------- | :---------------------------------- |
| legend_hfactor | 图例的垂直间距因子 |
| legend_wfactor | 影响图例宽度的乘法因子 |

#### 支持的`:series` `:extra_kwargs`

| 系列类型              | 关键字        | 描述                                                                                      |
| :----------------------- | :------------- | :----------------------------------------------------------------------------------------------- |
| `:surface`               | nx             | x方向的插值点数量                                                |
| `:surface`               | ny             | y方向的插值点数量                                                |
| `:surface`, `:wireframe` | display_option | 参见[GR文档](https://gr-framework.org/julia-gr.html#GR.surface-e3e6f234cc6cd4713b8727c874a5f331) |


## [Plotly / PlotlyJS](https://github.com/spencerlyon2/PlotlyJS.jl)

这些被视为独立的后端，尽管它们共享大部分代码并使用Plotly JavaScript API。
`plotly()`是唯一不需要依赖的绘图选项，因为所需的JavaScript已与Plots捆绑在一起。
它可以在IJulia中创建内联图，或者在从Julia REPL运行时打开独立的浏览器窗口。

`plotlyjs()`是首选选项，它利用了Spencer Lyon的PlotlyJS.jl的强大功能。
内联IJulia图可以从任何单元格更新...这是这个后端的突出之处。
从Julia REPL，它利用Blink.jl和Electron在独立的GUI窗口内绘图...也非常酷。
此外，PlotlyJS支持将输出保存为比Plotly更多的格式，如EPS和PDF，因此是开发出版质量图形的Plotly推荐版本。

```@example backends
plotlyjs(); backendplot(n = 2)  #hide
png("backends_plotlyjs.png")  #hide
```
![](backends_plotlyjs.png)

优点：

- [大量的功能](https://plot.ly/javascript/)
- 2D和3D
- 成熟的库
- 交互性（即使在内联）
- 独立或内联

缺点：

- 没有自定义形状
- JSON可能限制性能

主要PlotlyJS.jl作者：Spencer Lyon (@spencerlyon2)

### MathJax

Plotly需要加载MathJax以渲染LaTeX字符串，因此通过`extra_kwargs = :plot`实现了传递额外关键字。
有了它，就可以将头部传递给额外的`include_mathjax`关键字。
它有以下选项：

- `include_mathjax = ""` (默认): 没有mathjax头部
- `include_mathjax = "cdn"` 包含标准的在线版本的头部
- `include_mathjax = "<filename?config=xyz>"` 包含用户定义的文件

这些也可以使用`extra_plot_kwargs`关键字传递。

```@example backends
using LaTeXStrings
plotlyjs()
plot(
    1:4,
    [[1,4,9,16]*10000, [0.5, 2, 4.5, 8]],
    labels = [L"\alpha_{1c} = 352 \pm 11 \text{ km s}^{-1}";
              L"\beta_{1c} = 25 \pm 11 \text{ km s}^{-1}"] |> permutedims,
    xlabel = L"\sqrt{(n_\text{c}(t|{T_\text{early}}))}",
    ylabel = L"d, r \text{ (solar radius)}",
    yformatter = :plain,
    extra_plot_kwargs = KW(
        :include_mathjax => "cdn",
        :yaxis => KW(:automargin => true),
        :xaxis => KW(:domain => "auto")
   ),
)
Plots.html("plotly_mathjax")  #hide
```
```@raw html
<object type="text/html" data="plotly_mathjax.html" style="width:100%;height:450px;"></object>
```

### 精细调整
可以通过[`extra_kwargs`](@ref extra_kwargs)机制向plotly系列和布局字典添加额外的参数。
支持任意参数，但需要小心，因为没有进行检查，因此可能无意中覆盖现有条目。

例如，添加[customdata](https://plotly.com/javascript/reference/scatter/#scatter-customdata)可以通过以下方式完成`scatter(1:3, customdata=["a", "b", "c"])`。
还可以向plotly传递多个额外参数。
```
pl = scatter(
    1:3,
    rand(3),
    extra_kwargs = KW(
        :series => KW(:customdata => ["a", "b", "c"]),
        :plot => KW(:legend => KW(:itemsizing => "constant"))
    )
)
```

## [PythonPlot](https://github.com/stevengj/PythonPlot.jl)

围绕流行的Python包`Matplotlib`的Julia包装器。它使用`PythonCall.jl`传递数据，开销最小。

```@example backends
pythonplot(); backendplot()  #hide
```

优点：

- 大量的功能
- 2D和3D
- 成熟的库
- 独立或内联
- 在Plots中得到了良好的支持

缺点：

- 使用Python
- 依赖性经常导致设置问题

主要作者：Steven G Johnson (@stevengj)

### 精细调整
可以通过[`extra_kwargs`](@ref extra_kwargs)机制使用`matplotlib`的更多功能。
例如，对于一个3D图，以下例子应该生成一个位于适当位置的colorbar；如果没有下面的`extra_kwargs`，colorbar将显示得过于右侧，无法看到其刻度和数字。下面的例子中的四个坐标，即`[0.9, 0.05, 0.05, 0.9]`指定了colorbar的位置`[ left, bottom, width, height ]`。注意对于2D图，这种精细调整是不必要的。

```@example backends
using Plots; pythonplot()

x = y = collect(range(-π, π; length = 100))
fn(x, y) = 3 * exp(-(3x^2 + y^2)/5) * (sin(x+2y))+0.1randn(1)[1]
surface(x, y, fn, c=:viridis, extra_kwargs=Dict(:subplot=>Dict("3d_colorbar_axis" => [0.9, 0.05, 0.05, 0.9])))
```

#### 支持的`:subplot` `:extra_kwargs`

| 关键字          | 描述| :--------------- | :------------------------------------------------------------------------------- |
| 3d_colorbar_axis | 指定3D图的colorbar位置`[ left, bottom, width, height ]` |


## [PGFPlotsX](https://github.com/KristofferC/PGFPlotsX.jl)

基于`PGF/TikZ`的LaTeX绘图。

```@example backends
pgfplotsx(); backendplot()  #hide
```

是PGFPlots后端的继任者。

具有更多的功能并且仍在开发中，否则相同。

!!! 提示
    要添加保存一个包含前导词的独立.tex文件，请在您的`plot`命令中使用属性`tex_output_standalone = true`。

优点：

- 美观的图表
- 大量的功能（尽管代码仍在开发中）

缺点：

- 安装困难
- 重量级依赖项

作者：

- PGFPlots: Christian Feuersanger
- PGFPlotsX.jl: Kristoffer Carlsson (@KristofferC89), Tamas K. Papp (@tpapp)
- Plots <--> PGFPlotsX链接代码：Simon Christ (@BeastyBlacksmith)，基于Patrick Kofod Mogensen (@pkofod)的代码

### LaTeX工作流

要使用`pgfplotsx`后端的原生LaTeX输出，你可以将你的图保存为`.tex`或`.tikz`文件。
```julia
using Plots; pgfplotsx()
pl  = plot(1:5)
pl2 = plot((1:5).^2, tex_output_standalone = true)
savefig(pl,  "myline.tikz")    # 生成一个tikzpicture环境，可以包含在其他文档中
savefig(pl2, "myparabola.tex") # 生成一个独立的文档，可以自行编译，包括前导词
```
保存为`.tikz`文件的优点是，你可以使用`\includegraphics`重新缩放你的图，而不改变字体的大小。
默认的LaTeX输出旨在作为另一个文档的图形包含，它本身无法编译。
如果你在另一个LaTeX文档中包含这些图形，你需要有正确的前导词。
图的前导词可以使用`Plots.pgfx_preamble(pl)`显示，或者从独立输出中复制。

#### 精细调整

可以通过[`extra_kwargs`](@ref extra_kwargs)机制使用`PGFPlotsX`的更多功能。
默认情况下，它将每个额外的关键字解释为`plot`命令的选项。
设置`extra_kwargs = :subplot`将把它们视为`axis`命令的选项，`extra_kwargs = :plot`将被视为`tikzpicture`环境的选项。

例如，更改为pgfplots本地的colormap可以用以下方式实现。像这样可以保持latex文档的前导词清洁。

```@example backends
using Plots; pgfplotsx()
surface(range(-3,3, length=30), range(-3,3, length=30),
        (x, y) -> exp(-x^2-y^2),
        label="",
        colormap_name = "viridis",
        extra_kwargs =:subplot)
```

此外，可以通过特殊的`add`关键字添加额外的命令或字符串。
这将一个正方形添加到普通的线图中：

```@example backends
plot(1:5, add = raw"\draw (1,2) rectangle (2,3);", extra_kwargs = :subplot)
```

## [UnicodePlots](https://github.com/JuliaPlots/UnicodePlots.jl)

简单和轻量级。直接在你的终端中绘图。你不会产生任何出版质量的东西，但对于快速查看你的数据来说，它是非常棒的。允许在无头节点（SSH）上绘图。

```@example backends
import FileIO, FreeType  #hide
unicodeplots(); backendplot()  #hide
```

优点：

- 最小的依赖项
- REPL绘图
- 轻量级
- 快速

缺点：

- 有限的精度，密度

主要作者：Christof Stocker (@Evizero)

### 精细调整
可以通过[`extra_kwargs`](@ref extra_kwargs)机制使用`UnicodePlots`的更多功能。

```@example backends
using Plots; unicodeplots()

extra_kwargs = Dict(:subplot=>(; border = :bold, blend = false))
p = plot(1:4, 1:4, c = :yellow; extra_kwargs)
plot!(p, 2:3, 2:3, c = :red)
```

#### 支持的`:subplot` `:extra_kwargs`

| 关键字    | 描述                                                                                                |
| :--------- | :--------------------------------------------------------------------------------------------------------- |
| width      | 绘图宽度                                                                                               |
| height     | 绘图高度                                                                                                |
| projection | 3D投影（`:orthographic`, `perspective`）                                                         |
| zoom       | 3D缩放级别                                                                                              |
| up         | 3D上向量（方位角和仰角使用`Plots.jl`的`camera`控制）                            |
| canvas     | 画布类型 (参见 [Low-level Interface](https://github.com/JuliaPlots/UnicodePlots.jl#low-level-interface)) |
| border     | 边框类型 (`:solid`, `:bold`, `:dashed`, `:dotted`, `:ascii`, `:none`)                                   |
| blend      | 切换画布颜色混合 (`true` / `false`)                                                            |

#### 支持的`:series` `:extra_kwargs`

| 系列类型      | 关键字  | 描述                                                                     |
| :--------------- | :------- | :------------------------------------------------------------------------------ |
| `all`            | colormap | 色图 (参见 [Options](https://github.com/JuliaPlots/UnicodePlots.jl#options)) |
| `heatmap`, `spy` | fix_ar   | 切换固定终端纵横比 (`true` / `false`)                          |
| `surfaceplot`    | zscale   | `z`轴缩放                                                                |
| `surfaceplot`    | lines    | 使用`lineplot`代替`scatterplot` (单调数据)                        |

## [Gaston](https://github.com/mbaz/Gaston.jl)

`Gaston`是对[gnuplot](https://gnuplot.info)，一个跨平台的命令行驱动的绘图工具的直接接口。`Gaston`在`Plots`中的集成是最近的（2021年），但支持很多功能。

```@example backends
gaston(); backendplot()  #hide
```

## [InspectDR](https://github.com/ma-laforge/InspectDR.jl)

快速绘图，有一个响应式的GUI（可选）。目标：快速识别设计/模拟问题和故障，以缩短设计迭代。

```@example backends
inspectdr(); backendplot(n = 2)  #hide
```

优点：

- 相对较短的加载时间/首次绘图时间。
- 交互式鼠标/键绑定。
  - 快速简单的方式来平移/缩放数据。
- 拖放&Delta;-标记（测量/显示&Delta;x, &Delta;y和斜率）。
- 设计考虑到了较大的数据集。
  - 即使对于中等大小（>200k点）的数据集也很反应灵敏。
  - 已经证实可以在运行Windows 7的较旧的台式机上以合理的速度处理2GB的数据集（强烈建议不要拖动+平移数据区域）。

缺点：

- 主要限于2D线/散点图

主要作者：MA Laforge (@ma-laforge)

## [HDF5](https://github.com/JuliaIO/HDF5.jl) (HDF5-Plots)

将图 + 数据写入*单个* `HDF5` 文件，使用人类可读的结构，可以轻松地进行逆向工程。

![](assets/hdf5_samplestruct.png)

**写入.hdf5文件**
```julia
hdf5() # 选择HDF5-Plots "后端"
p = plot(...) # 像往常一样构造图
Plots.hdf5plot_write(p, "plotsave.hdf5")
```

**从.hdf5文件读取**
```julia
pythonplot() # 首先必须选择某个后端
pread = Plots.hdf5plot_read("plotsave.hdf5")
display(pread)
```

优点：

- 复杂数据集的开放、标准文件格式。
- 人类可读（使用[HDF5view](https://support.hdfgroup.org/products/java/hdfview/)）。
- 将图 + 数据保存到单个二进制文件中。
- 在以后的时间使用你最喜欢的后端（重新）渲染图。

缺点：

- 目前缺少对`SeriesAnnotations`和`GridLayout`的支持。
  - （如果你有需要，请开启一个“issue”）。
- 目前还没有设计用于向后兼容性（没有适当的版本控制）。
  - 因此目前还不真正适合存档目的。
- 目前作为“后端”实现，以避免向`Plots.jl`添加依赖项。

主要作者：MA Laforge (@ma-laforge)

---

# 已弃用的后端

### [PyPlot](https://github.com/stevengj/PyPlot.jl)

基于`matplotlib`的后端，使用`PyCall.jl`和`PyPlot.jl`。被`PythonCall.jl`和`PythonPlot.jl`取代。
虽然在`Plots 1.X`中仍然支持，但建议用户过渡到`pythonplot`后端。

### [PGFPlots](https://github.com/sisl/PGFPlots.jl)

基于PGF/TikZ的LaTeX绘图。

!!! 提示
    要添加保存一个包含前导词的独立.tex文件，请在您的`plot`命令中使用属性`tex_output_standalone = true`。

优点：

- 美观的图表
- 大量的功能（尽管代码仍在开发中）

缺点：

- 安装困难
- 重量级依赖项

作者：

- PGFPlots: Christian Feuersanger
- PGFPlots.jl: Mykel Kochenderfer (@mykelk)，Louis Dressel (@dressel)，以及其他人
- Plots <--> PGFPlots链接代码：Patrick Kofod Mogensen (@pkofod)


### [Gadfly](https://github.com/dcjones/Gadfly.jl)

受"Grammar of Graphics"启发的Julia实现。

优点：

- 清晰的外观
- 大量的功能
- 与Compose.jl结合时灵活（内嵌图等）

缺点：

- 不支持3D
- 首次绘图时间较长
- 大量的依赖项
- 没有交互性

主要作者：Daniel C Jones

### [Immerse](https://github.com/JuliaGraphics/Immerse.jl)

基于Gadfly，Immerse添加了一些交互性和独立的GUI窗口，包括缩放/平移和一个很酷的"点套索"工具，用来保存选择的数据点的Julia向量。

优点：

- 与Gadfly相同
- 交互性
- 独立或内联
- 套索功能

缺点：

- 与Gadfly相同

主要作者：Tim Holy

### [Qwt](https://github.com/tbreloff/Qwt.jl)

我的包，包装PyQwt。与PyPlot类似，它使用PyCall将调用转换为python。虽然Qwt.jl是Plots的"初稿"，但其功能被其他后端超越，而且我没有时间去维护。

主要作者：Thomas Breloff

### [Bokeh](https://github.com/bokeh/Bokeh.jl)

未完成，但非常类似于PlotlyJS... 使用那个代替。

### [Winston](https://github.com/nolta/Winston.jl)

功能不完整...我从未完成过它的包装，我不认为它提供了超越其他后端的任何东西。然而，图表看起来很干净，而且相对较快。

---
