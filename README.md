# Note

这是 Note to WebNote 的本地笔记源仓库。

本仓库只用于提交适合长期版本管理的笔记源文件：

- Markdown 文档
- 图片资源
- 顶层 `.gitignore`
- 本说明文件

PDF、构建产物、第三方工程、缓存和嵌套 Git 仓库不进入 Note 仓库。PDF 可以保留在本地 `Note/` 目录中，供根仓库的 `Publish.py` 扫描并上传到 Cloudflare R2。

## 与发布流程的关系

发布入口在根仓库：

```powershell
cd "F:\Programme\Note to WebNote"
$env:WEBNOTE_R2_BUCKET="webnote-pdfs"
$env:WEBNOTE_R2_PUBLIC_BASE_URL="https://static.patfang.xyz"
python Publish.py
```

`Publish.py` 会扫描本地 `Note/` 下所有 PDF，并生成：

- WebNote 文章页
- `data/posts.json`
- R2 中的 `pdfs/<Note 相对路径>.pdf`

Note 仓库本身不会提交这些 PDF。

## Git 规则

当前 `.gitignore` 的策略是默认忽略所有文件，只放行：

- `*.md`
- 常见图片格式：`png`、`jpg`、`jpeg`、`gif`、`webp`、`svg`、`bmp`、`tif`、`tiff`、`avif`、`ico`
- `.gitignore`

如果某个文件需要进入 Note 仓库，应优先确认它是否属于 Markdown 或图片笔记资源。

## 注意

- 不要在 Note 内嵌套提交其他 Git 仓库。
- 不要把 PDF、模型文件、编译产物或下载的第三方工程提交到 Note 仓库。
- 移动或删除本地 PDF 后，重新运行根仓库的 `Publish.py`，R2 和 WebNote 会同步更新。
