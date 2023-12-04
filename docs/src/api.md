# [参考文献](@id api)

## 内容
```@contents
Pages = ["api.md"]
Depth = 4
```

## 索引

```@index
Pages = ["api.md"]
```

## 公共接口

### 图表规范
```@docs
plot
bbox
grid
@layout
default
theme
with
```

```@autodocs
Modules = [Plots]
Pages   = ["components.jl"]
Order   = [:function]
```

```@autodocs
Modules = [Plots]
Pages   = ["shorthands.jl"]
```

### 动画
```@docs
animate
frame
gif
mov
mp4
webm
@animate
@gif
```

### 检索器

```@docs
current
Plots.xlims
Plots.ylims
Plots.zlims
backend_object
plotattr
```

### 输出
```@docs
display
```

```@autodocs
Modules = [Plots]
Pages   = ["output.jl"]
```
