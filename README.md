# OHSidian Latex Suite

Obsidian Latex Suite 的 OHSidian (HarmonyOS) 适配版，让 LaTeX 数学公式输入快如手写。

原始项目：[artisticat1/obsidian-latex-suite](https://github.com/artisticat1/obsidian-latex-suite)

## 功能

- **代码片段 (Snippets)** — 输入快捷词自动展开为 LaTeX 命令，例如 `sq` → `\sqrt{}`、`a/b` → `\frac{a}{b}`
- **自动分数** — 输入 `x/` 自动展开为 `\frac{x}{}`
- **矩阵快捷键** — Tab 插入 `&`，Enter 换行
- **符号隐藏 (Conceal)** — 隐藏 LaTeX 标记，显示为 Unicode 数学符号
- **括号着色与高亮** — 匹配括号同色显示
- **行内公式预览** — 光标在公式内时弹出渲染预览
- **Tab 跳出** — Tab 快速退出公式、跳转占位符
- **自动放大括号** — 包含 `\sum`、`\int`、`\frac` 时自动补 `\left` `\right`

## 安装

将 `main.js`、`manifest.json`、`styles.css` 放入 OHSidian 的插件目录，启用即可。

## 快速上手

在数学模式下输入：

| 输入 | 展开 |
|------|------|
| `dm` | `$$\n\n$$` (行间公式) |
| `mk` | `$ $` (行内公式) |
| `xsr` | `x^{2}` |
| `x/y` `Tab` | `\frac{x}{y}` |
| `sq` | `\sqrt{}` |
| `@a` | `\alpha` |
| `@b` | `\beta` |
| `pi` | `\pi` |

完整默认片段见 `src/default_snippets.js`。

## 构建

```bash
npm install
npm run build
# 产物: main.js, manifest.json, styles.css
```

## 适配说明

基于 obsidian-latex-suite v1.11.5，针对 HarmonyOS 做了以下修改：

1. 修复 HarmonyOS IME 框架下自动触发 (auto-snippet) 失效的问题

详见 `CHANGES.md`。

## 致谢

- 原始插件作者 [artisticat](https://github.com/artisticat1)
- 灵感来自 [Gilles Castel 的 UltiSnips 方案](https://castel.dev/post/lecture-notes-1/)

## 许可

MIT
