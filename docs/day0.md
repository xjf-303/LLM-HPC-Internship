# 实习笔记 Day 0｜MkDocs + GitHub Pages 搭建记录

> 关键词：MkDocs / GitHub Pages / gh-pages / 工程化部署 / 踩坑记录

在正式开始大模型高性能计算方向的实习之前，我决定先搭建一个**长期维护的个人技术笔记网站**，用于记录实习期间每天的学习内容、问题与思考。

这篇笔记完整记录了：

* 我为什么选择 MkDocs + GitHub Pages
* 在实际搭建过程中遇到的问题
* 每个问题背后的工程原因
* 最终形成的一套**稳定、可复用的发布流程**

这也是本仓库的 **第一篇正式笔记（Day 0）**。

---

## 一、为什么要在实习前搭建一个笔记网站？

我即将入职的大模型「高性能计算方向」实习，工作内容涉及：

* GPU 计算与资源调度
* NCCL / MPI 等分布式通信
* 大模型训练框架（DeepSpeed / Megatron-LM）

这些内容具有几个特点：

* 信息密度高
* 问题高度工程化
* 很多坑只在**真实实践中出现**

因此我希望：

> 把“每天遇到的问题 + 解决过程”系统性沉淀下来，而不是零散地记在本地。

选择「静态网站 + GitHub Pages」有几个优势：

* 不需要服务器
* 版本可追溯
* 内容天然工程化

---

## 二、整体技术方案

最终采用的方案是：

```text
Markdown（源内容）
        ↓
MkDocs（静态站点生成）
        ↓
gh-pages 分支（HTML 产物）
        ↓
GitHub Pages（对外访问）
```

核心工具：

* **MkDocs**：将 Markdown 转为静态站点
* **GitHub Pages**：托管静态网页
* **gh-pages 分支**：专门存放发布产物

---

## 三、仓库为什么需要两个分支？

这是我在搭建过程中第一个真正「工程化」的问题。

### 1. main 分支：源内容（Source）

```text
mkdocs.yml
docs/
  index.md
  day0.md
```

* 人写的 Markdown
* 可维护、可 review
* 是唯一的“事实来源（source of truth）”

### 2. gh-pages 分支：发布产物（Artifact）

```text
index.html
assets/
search/
sitemap.xml
```

* MkDocs 自动生成
* 给浏览器 / CDN 用
* **不手动维护**

> 这和 CUDA 工程里 `.cu` 与 `.cubin` 的关系是完全一致的。

---

## 四、为什么 `site/` 目录不需要 track？

在本地构建时，会生成一个 `site/` 目录：

```text
site/
  index.html
  assets/
```

但这个目录：

* 每次构建都会变
* 内容完全由 `docs/` 推导
* 无法反向维护

因此它的工程地位等同于：

```text
build/
dist/
__pycache__/
```

**结论：`site/` 不应该加入 git 管理。**

---

## 五、遇到的核心问题与排查过程

### 问题 1：mkdocs gh-deploy 成功，但页面 404

```text
Your documentation should shortly be available at:
https://xjf-303.github.io/LLM-HPC-Internship/
```

但访问后却是：

```
404 File not found
```

### 排查思路

1. 确认 `docs/index.md` 存在
2. 确认 `mkdocs.yml` 配置正确
3. 切换到 gh-pages 分支查看内容

```bash
git checkout gh-pages
ls
```

发现：

```text
index.html
assets/
```

👉 **说明构建和 push 都是成功的**

---

## 六、真正的原因：GitHub Pages 没指向 gh-pages

GitHub Pages 并不会自动识别 MkDocs。

它只认：

* 哪个分支？
* 哪个目录？
* 是否存在 `index.html`

最终正确配置是：

```text
Settings → Pages
Source: Deploy from a branch
Branch: gh-pages
Folder: / (root)
```

> 注意：这里的 `/ (root)` 指的是 **gh-pages 分支的根目录**，
> 而不是 main 分支中的某个文件夹。

---

## 七、最终形成的稳定发布流程

### 日常写作（服务器 / 本地都可以）

```bash
# 编辑 Markdown
docs/day0.md
```

### 构建 + 发布

```bash
mkdocs gh-deploy
```

### 页面访问

```
https://xjf-303.github.io/LLM-HPC-Internship/
```

---

## 八、这件事带给我的工程认知

这次搭建网站，其实提前帮我踩了一次**真实工程问题**：

* 构建成功 ≠ 服务可用
* 产物存在 ≠ 路由正确
* 工具没错 ≠ 配置没错

这和未来在大模型 / HPC 工程中会遇到的：

* NCCL 拓扑配置
* GPU 资源调度
* 分布式训练启动

本质是同一类问题。

---

## 九、结语

这是我的第一篇实习前笔记。
希望在实习结束时，这个仓库能成为一份**完整的工程成长记录**。
