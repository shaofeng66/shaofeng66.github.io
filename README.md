This is a [Hugo](https://github.com/gohugoio/hugo) generated site.

## 初始化

克隆仓库后需要初始化 submodule（主题）：

```bash
git submodule update --init --recursive
```

## 构建站点

需要使用 Hugo **Extended** 版本来构建（Ananke 主题需要 SCSS/SASS 支持）：

```bash
# 构建
hugo

# 本地预览
hugo server -D
```

## 新增文章

使用 `hugo new` 命令自动生成带有 front matter 的文件：

```bash
hugo new articles/文章标题.md
```

这会创建文件并自动添加日期和标题。编辑内容后，将 `draft = true` 改为 `draft = false` 即可发布。

```bash
# 重新构建
hugo

# 提交更改
git add content/articles/文章标题.md
git commit -m "Add new article"
```


