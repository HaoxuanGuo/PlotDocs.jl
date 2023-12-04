# PlotDocs.jl

[Plots.jl](https://github.com/JuliaPlots/Plots.jl)的文档

## 编辑文档

要编辑文档，只需编辑[docs/src](https://github.com/JuliaPlots/PlotDocs.jl/tree/master/docs/src)中的 Markdown 文件。[Documenter.jl](https://github.com/JuliaDocs/Documenter.jl)将在更改合并到 master 时自动重建文档。

新的文档文件将被推送到此存储库中的 [`gh-pages`](https://github.com/JuliaPlots/PlotDocs.jl/tree/gh-pages) 分支（对于通过 CI 调试 `Plots.jl` 文档构建很有用）。

在为新版本构建新文档（例如 `1.Y.Z`）时，不要忘记手动删除 `gh-pages` 中的上一个补丁版本（`1.Y.(Z - 1)`），从而避免填满 `github` 强加的 `10Gb` 部署配额。

## 构建文档

在安装了适当的依赖项后，在您的本地终端上运行以下命令：
```bash
$ CI=true julia --project=docs docs/make.jl
```

设置环境变量 `CI=true` 是可选的，但结果将更接近远程文档构建过程。

要仅构建文档的一部分（更快的开发），请使用以下环境变量：
- `PLOTDOCS_BACKENDS='GR PyPlot'` 用于选择后端；
- `PLOTDOCS_EXAMPLES='1 22 60'` 用于选择画廊示例。

## 贡献演示

演示是使用 [Literate 标记语法][literate_syntax] 编写的有效的 julia 脚本，并由 [DemoCards.jl][democards_jl] 管理。以下步骤显示了添加演示的常见工作流：

1. 在 `docs/user_gallery/` 的任何子文件夹中创建您的 julia 脚本。例如，`docs/user_gallery/misc/gr_lorenz_attractor.jl`；
2. 使用 [DemoCards YAML frontmatter][yaml_frontmatter] 配置演示。您也可以查看其他演示的配置作为参考；
3. 使用 Literate 标记语法在 Julia 中编写演示；
3. 激活文档环境并通过 `import Pkg; Pkg.activate("docs")` 将 `PlotDocs.jl` 添加到环境中；
4. 使用 [`DemoCards.preview_demos` 功能][democards_preview] 预览演示。例如，您可以通过 `preview_demos("docs/user_gallery/misc/gr_lorenz_attractor.jl")` 来部分构建一个单独的文件，或者通过 `preview_demos("docs/user_gallery/misc")` 来预览整个部分，甚至通过 `preview_demos("docs/user_gallery")` 来预览整个页面。

[literate_syntax]: https://fredrikekre.github.io/Literate.jl/v2/fileformat/
[yaml_frontmatter]: https://juliadocs.github.io/DemoCards.jl/stable/quickstart/usage_example/julia_demos/1.julia_demo/#juliademocard_example
[democards_jl]: https://github.com/johnnychen94/DemoCards.jl
[democards_preview]: https://juliadocs.github.io/DemoCards.jl/stable/preview/
