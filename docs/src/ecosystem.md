```@setup ecosystem
using StatsPlots, Plots, RDatasets, Distributions; gr()
Plots.reset_defaults()

iris = dataset("datasets", "iris")
singers = dataset("lattice","singer")
dist = Gamma(2)
a = [randn(100); randn(100) .+ 3; randn(100) ./ 2 .+ 3]
```

Plots本身就很出色，但真正的强大之处来自于围绕它形成的生态系统。Plots（更具体地说，[RecipesBase](https://github.com/JuliaPlots/RecipesBase.jl)）的设计目标是将不同的功能整合在一起，为用户提供一致且连贯的使用体验。一些包可能选择实现配方来可视化他们的自定义类型。其他的可能扩展了Plots对于基本类型的功能。在此页面，我将尝试收集和展示你可以使用围绕Plots核心开发的生态系统所能做的许多事情。

---

# [JuliaPlots](@id ecosystem)

[JuliaPlots](https://github.com/JuliaPlots)组织构建并维护了许多常用的、独立于核心Plots的功能，以及RecipesBase、PlotUtils、文档等等。

# 社区包

## [AtariAlgos](https://github.com/tbreloff/AtariAlgos.jl)

`AtariAlgos.jl`将ArcadeLearningEnvironment封装为Reinforce接口的一个抽象环境的实现。这使得它可以作为一个插件模块与一般的强化学习代理一起使用。

游戏也可以使用Plots.jl进行"绘制"，使其成为用于跟踪学习进度和更多复杂可视化的组件，同时也使得创建动画变得简单。

![](https://cloud.githubusercontent.com/assets/933338/17670982/8923a2f6-62e2-11e6-943f-bd0a2a7b5c1f.gif)

## [Reinforce](https://github.com/tbreloff/Reinforce.jl)

`Reinforce.jl`是一个强化学习的接口。它旨在将模块化的环境、策略和解决方案连接起来，提供一个简单的接口。

![](https://cloud.githubusercontent.com/assets/933338/17703784/f3e18414-63a0-11e6-9f9e-f531278216f9.gif)


## [JuliaML](https://github.com/JuliaML)

Julia中与机器学习相关的工具、模型和数学。

![](https://cloud.githubusercontent.com/assets/933338/18800737/93b71b42-81ac-11e6-9c7a-0cddf6d083ab.png)

## [Augmentor](https://github.com/Evizero/Augmentor.jl)

`Augmentor.jl`是一个图像增强库，旨在使人工数据集扩大的过程更加方便、减少错误，并易于复制。这是通过使用概率转换管道实现的。

![](https://cloud.githubusercontent.com/assets/10854026/17645973/3894d2b0-61b6-11e6-8b10-1cb5139bfb6d.gif)

## [DifferentialEquations](https://github.com/ChrisRackauckas/DifferentialEquations.jl)

`DifferentialEquations.jl`是一个用于在Julia中数值求解微分方程的包，由Chris Rackauckas创建。这个包的目的是提供各种微分方程的高效Julia实现。这个包包括的方程有普通微分方程(ODEs)、随机普通微分方程(SODEs或SDEs)、随机偏微分方程(SPDEs)、偏微分方程（使用有限差分和有限元方法）、微分代数方程和微分延迟方程。它包括了对经典算法和最近研究的算法的优化实现，包括为高精度和HPC应用优化的算法。

所有的求解器都返回设置了绘图配方的解决方案对象，以提供有信息的默认绘图。

![diffeq](https://cloud.githubusercontent.com/assets/1814174/17526562/9daa2d1e-5e1c-11e6-9f21-fda6f49f6833.png)

## [PhyloTrees](https://github.com/jangevaare/PhyloTrees.jl)

`PhyloTrees.jl`包提供了系统树类型的表示。还为系统树提供了模拟、推断和可视化功能。绘图配方允许用户按照他们喜欢的方式绘制系统树的结构。

![](https://cloud.githubusercontent.com/assets/5422422/17630286/a25374fc-608c-11e6-9160-32466b094f0b.png)

## [EEG](https://github.com/codles/EEG.jl)

处理EEG文件并可视化脑活动。

![](https://cloud.githubusercontent.com/assets/748691/17362167/210f9c28-5974-11e6-8a05-62fa399d32d1.png)

![](https://cloud.githubusercontent.com/assets/748691/17363374/523373a0-597a-11e6-94d9-826381617756.png)

## [ImplicitEquations](https://github.com/jverzani/ImplicitEquations.jl)

在一篇论文中，Tupper提出了一种绘制二维隐函数和不等式的方法。这个包提供了论文的基本算法的实现，使得Julia用户可以自然地表示和轻松地绘制隐函数和方程的图形。

![](https://camo.githubusercontent.com/950ef704a0601ed9429addb35e6b7246ca5da149/687474703a2f2f692e696d6775722e636f6d2f4c4368547a43312e706e67)



## [ControlSystems](https://github.com/JuliaControl/ControlSystems.jl)

一个用于Julia的控制系统设计工具箱。这个工具箱的工作方式类似于其他主要的计算机辅助控制系统设计(CACSD)工具箱。系统可以在传递函数或者状态空间表示中创建。然后，这些系统可以组合成更大的架构，在时间和频率域中模拟，并分析稳定性/性能属性。

![](https://juliacontrol.github.io/ControlSystems.jl/latest/plots/pidgofplot2.svg)

## [ValueHistories](https://github.com/JuliaML/ValueHistories.jl)

用于高效跟踪优化历史、训练曲线或者其他任意类型信息和任意间隔采样时间的实用程序包。

![](https://cloud.githubusercontent.com/assets/10854026/17512899/58461c20-5e2a-11e6-94d4-b4699c63ab1a.png)


## [ApproxFun](https://github.com/ApproxFun/ApproxFun.jl)

`ApproxFun.jl`是一个用于近似函数的包。它深受Matlab包Chebfun和Mathematica包RHPackage的影响。

![](https://raw.githubusercontent.com/ApproxFun/ApproxFun.jl/master/images/extrema.png)


## [AverageShiftedHistograms](https://github.com/joshday/AverageShiftedHistograms.jl)

使用平均移动直方图进行密度估计。

![](https://cloud.githubusercontent.com/assets/933338/17702262/3bfc9a96-639b-11e6-8976-aa8bb8fabfc8.gif)

## [MLPlots](https://github.com/JuliaML/MLPlots.jl)

统计和机器学习的常用绘图配方。

![](https://cloud.githubusercontent.com/assets/933338/17702652/bca0158c-639c-11e6-8e36-4bfc7b36727e.png)

![](https://cloud.githubusercontent.com/assets/933338/17702662/cdc08752-639c-11e6-8c3c-e186456630e2.png)


## [LazySets](https://github.com/JuliaReach/LazySets.jl)

`LazySets.jl`是一个用于凸集运算的Julia包。LazySets背后的原理是将集合运算包装到专门的类型中，延迟表达式结果的计算，直到需要为止。通过在高维度中组合懒惰操作和在低维度中进行显式计算，该库可以应用于解决复杂的高维问题。

[两模式混合系统](https://juliareach.github.io/LazySets.jl/dev/man/reach_zonotopes_hybrid/#Example)的可达性绘图：

![](https://raw.githubusercontent.com/JuliaReach/JuliaReach-website/master/src/images/hybrid2d.png)

---

还有更多：

- `Losses.jl`
- `IterativeSolvers.jl`
- `SymPy.jl`
- `OnlineStats.jl`
- `Robotlib.jl`
- `JWAS.jl`
- `QuantEcon.jl`
- `Reinforce.jl`
- `Optim.jl`
- `Transformations.jl` / `Flow.jl`
- ...
