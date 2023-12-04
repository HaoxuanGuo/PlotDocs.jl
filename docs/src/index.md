```@setup index
using Plots; gr()
Plots.reset_defaults()
```

# Plots - Julia中强大且方便的可视化工具

**作者：Thomas Breloff (@tbreloff)**

要开始，请[参见教程](@ref tutorial)。

在Plots中，几乎所有的操作都是通过指定绘图[属性](@ref attributes)来完成的。

利用[Plots生态系统](@ref ecosystem)提供的广泛的可视化功能，以及使用[配方](@ref recipes)轻松构建自己的复杂图形组件。

## Julia中的Plots简介

数据可视化有着复杂的历史。绘图软件在功能和简单性、速度和美观、静态和动态界面之间进行权衡。有些包创建一个显示并且永不更改，而其他包则实时更新。

Plots是一个可视化界面和工具集。它位于其他后端之上，如GR, PythonPlot, PGFPlotsX或Plotly，将命令与实现连接起来。如果一个后端不支持您所需的功能或没有做出正确的权衡，您可以只通过一个命令切换到另一个后端。无需更改您的代码。无需学习新的语法。Plots可能是你最后需要学习的绘图包。

该包的目标是：

- **强大**。用更少做更多。复杂的可视化变得简单。
- **直观**。在不阅读大量文档的情况下开始生成图表。命令应该"就能工作"。
- **简洁**。代码越少意味着错误越少，开发和分析更高效。
- **灵活**。从你最喜欢的包中生成你最喜欢的图表，只是更快更简单。
- **一致**。不要承诺使用一个图形包。使用相同的代码并利用所有[后端](@ref backends)的优点。
- **轻量级**。由于后端是动态加载和初始化的，所以依赖性非常少。
- **智能**。它不完全是AGI，但Plots应该能够弄清楚你希望它做什么，而不仅仅是你告诉它。

使用Plots中的[预处理管道](@ref pipeline)在调用后端代码之前完全描述您的可视化。这种预处理保持了模块性，并允许有效地分离前端代码、算法和后端图形。

请将愿望清单项、错误或任何其他评论/问题添加到[问题列表](https://github.com/tbreloff/Plots.jl/issues)，并[在zulip上参与讨论](https://julialang.zulipchat.com/#streams/236493/plots.jl)。

尽管如此，极度可配置性并不是Plots的目标。如果你需要一个相当特定的绘图功能，请随意[请求它](https://github.com/JuliaPlots/Plots.jl/issues?q=is%3Aissue+is%3Aopen+label%3Aextension)。然而，请理解，Plots需要在所有的后端上实现该功能，这可能因为某些后端的限制而具有挑战性。

---

### [简单就是美](@id simple-is-beautiful)

洛伦兹吸引子

```@example index
using Plots
# 定义洛伦兹吸引子
Base.@kwdef mutable struct Lorenz
    dt::Float64 = 0.02
    σ::Float64 = 10
    ρ::Float64 = 28
    β::Float64 = 8/3
    x::Float64 = 1
    y::Float64 = 1
    z::Float64 = 1
end

function step!(l::Lorenz)
    dx = l.σ * (l.y - l.x)
    dy = l.x * (l.ρ - l.z) - l.y
    dz = l.x * l.y - l.β * l.z
    l.x += l.dt * dx
    l.y += l.dt * dy
    l.z += l.dt * dz
end

attractor = Lorenz()


# 初始化一个3D图表，包含1个空的系列
plt = plot3d(
    1,
    xlim = (-30, 30),
    ylim = (-30, 30),
    zlim = (0, 60),
    title = "Lorenz Attractor",
    legend = false,
    marker = 2,
)

# 通过向图表推送新点来构建一个动画gif，每10帧保存一次
@gif for i=1:1500
    step!(attractor)
    push!(plt, attractor.x, attractor.y, attractor.z)
end every 10
```

制造一些波浪

```@example index
using Plots
default(legend = false)
x = y = range(-5, 5, length = 40)
zs = zeros(0, 40)
n = 100

@gif for i in range(0, stop = 2π, length = n)
    f(x, y) = sin(x + 10sin(i)) + cos(y)

    # 创建一个包含3个子图和自定义布局的图表
    l = @layout [a{0.7w} b; c{0.2h}]
    p = plot(x, y, f, st = [:surface, :contourf], layout = l)

    # 引入一个轻微的摇摆相机角度扫描，以度为单位（方位角，高度）
    plot!(p[1], camera = (10 * (1 + cos(i)), 40))

    # 添加一个跟踪线
    fixed_x = zeros(40)
    z = map(f, fixed_x, y)
    plot!(p[1], fixed_x, y, z, line = (:black, 5, 0.2))
    vline!(p[2], [0], line = (:black, 5))

    # 添加到并随时间显示跟踪值
    global zs = vcat(zs, z')
    plot!(p[3], zs, alpha = 0.2, palette = cgrad(:blues).colors)
end
```


鸢尾花数据集

```@example index
# 加载数据集
using RDatasets
iris = dataset("datasets", "iris");

# 通过以下方式加载StatsPlots配方（用于DataFrames）：
# Pkg.add("StatsPlots")
using StatsPlots

# 带有一些自定义设置的散点图
@df iris scatter(
    :SepalLength,
    :SepalWidth,
    group = :Species,
    title = "My awesome plot",
    xlabel = "Length",
    ylabel = "Width",
    m = (0.5, [:cross :hex :star7], 12),
    bg = RGB(0.2, 0.2, 0.2)
)
```
