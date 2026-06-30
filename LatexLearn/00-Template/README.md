# 深度强化学习笔记项目模板

这是一个项目化 LaTeX 模板，适合长期维护深度强化学习笔记。当前结构按 chapter 和 section 两级拆分：

- `main.tex`：总入口，只负责文档结构。
- `styles/preamble.tex`：宏包、颜色、数学命令、算法环境、代码环境。
- `chapters/*.tex`：每章入口，只保留 `\chapter{...}` 和本章 section 的 `\input{...}`。
- `chapters/<chapter-name>/*.tex`：每个 `\section{...}` 单独一个文件。
- `references.bib`：参考文献。
- `figures/`：图片资源。
- `tables/`：大型表格资源。

## 目录结构

```text
00-Template/
├── main.tex
├── references.bib
├── README.md
├── styles/
│   └── preamble.tex
├── chapters/
│   ├── 00-preface.tex
│   ├── 01-foundations.tex
│   ├── 01-foundations/
│   │   ├── 01-mdp.tex
│   │   ├── 02-value-functions.tex
│   │   └── 03-bellman-equations.tex
│   ├── 02-value-based.tex
│   ├── 02-value-based/
│   ├── 03-policy-gradient.tex
│   ├── 03-policy-gradient/
│   ├── 04-actor-critic.tex
│   ├── 04-actor-critic/
│   ├── 05-experiments.tex
│   ├── 05-experiments/
│   ├── 06-reading-template.tex
│   ├── 06-reading-template/
│   └── A-notation.tex
├── figures/
├── tables/
└── build/
```

## 编译

推荐使用：

```bash
latexmk -xelatex -output-directory=build main.tex
```

VS Code 中建议使用 LaTeX Workshop 的 `latexmk (xelatex) + copy PDF` recipe。

## 新增 Section

例如要在 `02-value-based` 中新增一节：

1. 新建文件：`chapters/02-value-based/04-rainbow-dqn.tex`。
2. 文件内容从 `\section{Rainbow DQN}` 开始。
3. 在 `chapters/02-value-based.tex` 中加入：

```latex
\input{chapters/02-value-based/04-rainbow-dqn.tex}
```

## 新增 Chapter

1. 新建章节入口：`chapters/07-offline-rl.tex`。
2. 新建 section 目录：`chapters/07-offline-rl/`。
3. 在章节入口中写：

```latex
\chapter{离线强化学习}

\input{chapters/07-offline-rl/01-problem-setting.tex}
```

4. 在 `main.tex` 中加入：

```latex
\input{chapters/07-offline-rl.tex}
```
