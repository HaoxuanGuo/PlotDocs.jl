### 安装

首先，添加软件包：

```julia
import Pkg
Pkg.add("Plots")

# 如果你想要最新的功能：
Pkg.pkg"add Plots#master"
```

GR [后端](@ref backends) 默认就包含在内，但是如果你需要不同的后端，你可以安装额外的绘图包。

一级支持后端（按字母顺序）：
```julia
Pkg.add("GR")
# 你不需要添加这个包，因为它是默认的后端，
# 因此它会自动与Plots.jl一起安装。注意，如果你在Linux上，
# 你可能需要安装额外的系统包，
# 参见https://gr-framework.org/julia.html#installation

Pkg.add("PGFPlotsX")
# 你需要在你的系统上安装LaTeX

Pkg.add("PlotlyJS"); Pkg.add("PlotlyBase")
# 注意，只有在你需要Electron窗口和
# 额外的输出格式时，才需要添加这个，
# 否则`plotly()`会与Plots.jl一起提供。
# 为了在Jupyter上有好的体验，参考Plotly特定的
# Jupyter安装（https://github.com/plotly/plotly.py#installation）

Pkg.add("PythonPlot")
# 仅依赖PythonPlot包

Pkg.add("UnicodePlots")
```

二级支持后端：
```julia
Pkg.add("InspectDR")
Pkg.add("Gaston")
```

了解更多关于后端的信息 [点击这里](https://docs.juliaplots.org/latest/backends/).

最后，你可能希望从 [Plots 生态系统](@ref ecosystem) 中添加一些扩展：

```julia
Pkg.add("StatsPlots")
Pkg.add("GraphRecipes")
```

---

### 初始化

```julia
using Plots # 或者 StatsPlots
# using GraphRecipes  # 如果你希望使用GraphRecipes包
```

可选地，[选择一个后端](@ref backends) 并/或同时覆盖默认设置：

```julia
gr(size = (300, 300), legend = false)  # 提供可选的默认值
pgfplotsx()
plotly(ticks=:native)                  # plotlyjs 提供更丰富的保存选项
pythonplot()                           # 后端使用小写名称进行选择
unicodeplots()                         # 在终端中绘制
```

!!! 提示
    Plots 默认会使用 GR 后端。你可以通过在 `~/.julia/config/startup.jl` 文件中设置环境变量来覆盖这个选择（如果文件不存在，创建它）。例如，添加以下代码行：`ENV["PLOTS_DEFAULT_BACKEND"] = "PlotlyJS"`。

!!! 提示
    你可以在 `~/.julia/config/startup.jl` 文件中覆盖标准默认值，例如 `PLOTS_DEFAULTS = Dict(:markersize => 10, :legend => false, :warn_on_unsupported => false)`。
---
