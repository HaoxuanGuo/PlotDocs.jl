# [输出](@id output)

**只有在返回时才会显示图表**（分号会抑制返回），或者如果明确地使用`display(plt)`、`gui()`或在你的绘图命令中添加`show = true`进行显示。

!!! 提示
    你可以通过设置默认值实现类似MATLAB的交互行为：default(show = true)

### 独立窗口

调用`gui(plt)`将打开一个独立的窗口。`gui()`，像`plot!(...)`一样，适用于"当前"的图表。将一个图表对象返回到REPL就像调用`gui(plt)`一样。

### Jupyter / IJulia

当返回到单元时，图表将内联显示。默认的输出格式是`svg`，适用于支持它的后端。
这可以通过`html_output_format`属性（别名为`fmt`）进行更改：

```julia
plot(rand(10), fmt = :png)
```

### Juno / Atom

当可能的时候，图表会在Atom的PlotPane中显示，无论是返回到控制台还是内联代码块。在任何时候，都可以使用`gui()`命令在独立窗口中打开图表。
可以在Juno的设置中禁用PlotPane。

### savefig / format

图表支持每个保存命令的2个不同版本。
命令`savefig`会根据文件扩展名自动选择文件类型。

```julia
savefig(filename_string) # 将最近的图表保存为filename_string（例如"output.png"）
savefig(plot_ref, filename_string) # 将由plot_ref引用的图表保存为filename_string（例如"output.png"）
```

此外，`Plots`导出了方便函数`png(filename::AbstractString)`。
其他函数，如`Plots.pdf`或`Plots.svg`保持未导出，因为它们可能
与其他包的导出冲突。
在这种情况下，包含文件名的字符串fn不需要文件扩展名。

```julia
png(filename_string) # 将当前图表保存为png，文件名为filename_string（例如"output.png"）
png(plot_ref, filename_string) # 将由plot_ref引用的图表保存为png，文件名为filename_string（例如"output.png"）
```

#### 大多数图形后端支持的文件格式

 - png（如果没有给出文件扩展名，`savefig`的默认输出格式）
 - svg
 - PDF

当不使用`savefig`时，默认的输出格式取决于环境（例如，当使用IJulia/Jupyter时）。

#### 支持的输出文件格式

注意：并非所有后端都支持每一种输出文件格式！
一个简单的表格显示哪种格式由哪个后端支持

| 格式 | 后端                                                                 |
| :----- | :------------------------------------------------------------------- |
| eps    | inspectdr, plotlyjs, pythonplot                                      |
| html   | plotly,  plotlyjs                                                    |
| json   | plotly, plotlyjs                                                     |
| pdf    | gr, plotlyjs, pythonplot, pgfplotsx, inspectdr, gaston               |
| png    | gr, plotlyjs, pythonplot, pgfplotsx, inspectdr, gaston, unicodeplots |
| ps     | gr, pythonplot                                                       |
| svg    | gr, inspectdr, pgfplotsx, plotlyjs, pythonplot, gaston               |
| tex    | pgfplotsx, pythonplot                                                |
| text   | hdf5, unicodeplots                                                   |

支持的文件格式可以通过例如`png(myplot, pipebuffer::IO)`将其写入到IO流，因此图像文件可以通过PipeBuffer传递给其他函数，例如`Cairo.read_from_png(pipebuffer::IO)`。
