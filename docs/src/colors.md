## 颜色

颜色属性有很多，包括线条、填充、标记、背景和前景的颜色。许多颜色遵循一个层级结构...例如，除非你覆盖这个值，否则`linecolor`会从`seriescolor`中获取其值。这让你能够精确地设置你想要的颜色，而不需要大量的样板代码。

颜色属性可以接受许多不同类型：

- `Symbol`或`String`会被传递给`Colors.parse(Colorant, c)`，因此`:red`等同于`colorant"red"`
- `false`或`nothing`会被转换为不可见的`RGBA(0,0,0,0)`
- 任何带有或不带有alpha/透明度的`Colors.Colorant`
- 任何`Plots.ColorScheme`，包括`ColorVector`，`ColorGradient`等
- 整数，会从`seriescolor`中选择相应的颜色

此外，还有一个广泛的设施用于选择和生成颜色映射/渐变。

- 有效的Symbol：`:inferno`（默认），`:heat`，`:blues`等
- 颜色列表（或可以转换为颜色的任何东西）
- 预构建的`ColorGradient`，可以使用`cgrad`辅助函数来构造。例如使用方法，请参见[这个简短的教程](https://github.com/tbreloff/ExamplePlots.jl/blob/master/notebooks/cgrad.ipynb)。

### 颜色名称
支持的颜色名称是[X11的](https://en.wikipedia.org/wiki/X11_color_names)和SVG的并集。
它们在[Colors.jl](https://github.com/JuliaGraphics/Colors.jl/blob/master/src/names_data.jl)中定义，如`blue`，`blue2`，`blue3`...等。

---

#### 系列颜色

对于系列，有几个需要知道的属性：

- **seriescolor**：不直接使用，但定义了系列的基本颜色
- **linecolor**：路径的颜色
- **fillcolor**：填充区域的颜色
- **markercolor**：标记和形状内部的颜色
- **markerstrokecolor**：标记和形状边界/描边的颜色

`seriescolor`默认为`:auto`，并根据其在子图中的索引从`color_palette`中分配颜色。默认情况下，其他颜色为`:match`。（参见下表）

!!! 提示
    通常，颜色渐变可以通过`*color`设置，对应的颜色值可以通过`*_z`在渐变中查找。

这个颜色... | 匹配这个颜色...
--- | ---
linecolor | seriescolor
fillcolor | seriescolor
markercolor | seriescolor
markerstrokecolor | foreground_color_subplot

!!! 注意
    这些属性每一个都有一个对应的alpha覆盖：`seriesalpha`，`linealpha`，`fillalpha`，`markeralpha`和`markerstrokealpha`。它们是可选的，你仍然可以作为`Colors.RGBA`的一部分提供alpha信息。

!!! 注意
    在某些情况下，如果用户没有设置值，`linecolor`或`markerstrokecolor`可能会被覆盖。

---

#### 前景/背景

前景和背景颜色的工作方式相似：

这个颜色... | 匹配这个颜色...
--- | ---
background\_color\_outside | background\_color
background\_color\_subplot | background\_color
background\_color\_legend  | background\_color\_subplot
background\_color\_inside  | background\_color\_subplot
foreground\_color\_subplot | foreground\_color
foreground\_color\_legend  | foreground\_color\_subplot
foreground\_color\_grid    | foreground\_color\_subplot
foreground\_color\_title   | foreground\_color\_subplot
foreground\_color\_axis    | foreground\_color\_subplot
foreground\_color\_border  | foreground\_color\_subplot
foreground\_color\_guide   | foreground\_color\_subplot
foreground\_color\_text    | foreground\_color\_subplot

---

#### 其他

- 默认主题下的`linecolor`并未在CSS中定义，但接近于`:steelblue`。
- `line_z`和`marker_z`参数会将数据值映射到`ColorGradient`值
- `color_palette`决定了`seriescolor == :auto`时分配的颜色：
    - 如果传递了颜色向量，它将强制循环这些颜色
    - 如果传递了渐变，它将无限地从该渐变中抽取唯一的颜色，试图将它们分散开