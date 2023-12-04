```@setup animations
using Plots; gr()
Plots.reset_defaults()
```

### [动画](@id animations)

动画的创建分为3个步骤:

- 初始化一个 `Animation` 对象。
- 使用 `frame(anim)` 保存动画的每一帧。
- 使用 `gif(anim, filename, fps=15)` 将帧转换为动画gif。

!!! tip
    便捷宏 `@gif` 和 `@animate` 极大地简化了这段代码。请参阅 [主页](@ref simple-is-beautiful) 上的简短版本示例，或者 [gr 示例](@ref gr_demo_2) 的长版本。

---

### 便捷宏

有两种宏可以在不同级别的便利性中创建动画：`@animate` 和 `@gif`。主要的区别是 `@animate` 将返回一个 `Animation` 对象以供稍后处理，而 `@gif` 将创建一个动画gif文件（并在返回到 IJulia 单元时显示它）。

对于您想要立即查看的简单的一次性动画，请使用 `@gif`。对于任何更复杂的内容，请使用 `@animate`。当您需要完全控制动画的生命周期时（虽然通常不必要），可以构造 `Animation` 对象。

示例：

```@example animations
using Plots

@userplot CirclePlot
@recipe function f(cp::CirclePlot)
    x, y, i = cp.args
    n = length(x)
    inds = circshift(1:n, 1 - i)
    linewidth --> range(0, 10, length = n)
    seriesalpha --> range(0, 1, length = n)
    aspect_ratio --> 1
    label --> false
    x[inds], y[inds]
end

n = 150
t = range(0, 2π, length = n)
x = sin.(t)
y = cos.(t)

anim = @animate for i ∈ 1:n
    circleplot(x, y, i)
end
gif(anim, "anim_fps15.gif", fps = 15)
```

```@example animations
gif(anim, "anim_fps30.gif", fps = 30)
```

`every` 标志只会在 "每N次迭代" 时保存一帧:

```@example animations
@gif for i ∈ 1:n
    circleplot(x, y, i, line_z = 1:n, cbar = false, framestyle = :zerolines)
end every 5
```

`when` 标志只会在 "表达式为真时" 保存一帧

```@example animations
n = 400
t = range(0, 2π, length = n)
x = 16sin.(t).^3
y = 13cos.(t) .- 5cos.(2t) .- 2cos.(3t) .- cos.(4t)

@gif for i ∈ 1:n
    circleplot(x, y, i, line_z = 1:n, cbar = false, c = :reds, framestyle = :none)
end when i > 40 && mod1(i, 10) == 5
```
