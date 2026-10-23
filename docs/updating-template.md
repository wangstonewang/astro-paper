# 模板更新与冲突避免指南

> 目标：从上游 AstroPaper 更新模板时，不覆盖、不丢失自己的文章、图片、配色和图标。
> 整理时间：2026-10-23。

## 核心结论

**只要用 Git 的"上游合并"流程，你的文章、图片不会被覆盖。**
Git 按文件和行做三方合并：

- 上游从没碰过的文件（你的新文章、新图标、新图片）→ 永远保留，**不可能冲突**
- 只有"双方都改过同一个文件"才需要手动解决

**唯一危险的做法**：下载新模板 zip 解压覆盖目录，或重新 clone 覆盖现有文件夹。这会绕过 git 合并，直接覆盖你的内容。**永远不要这样做。**

## 一、当前仓库状态（配置前）

- `origin` 指向自己的 fork：`wangstonewang/astro-paper`
- 完整保留了上游提交历史（提交哈希与上游一致）
- 尚未配置上游远程
- 历史完整，因此可以直接做三方合并（merge），这是最省事的方式

## 二、一次性配置

### 1. 先提交一个干净的基线

合并前工作区必须干净（`git status` 无未提交改动）。

如果 `package.json` 有意外改动（例如依赖版本被误改），先还原：

```bash
git checkout -- package.json
```

**本仓库统一使用 pnpm**（上游用 `pnpm-lock.yaml`，CI 也用 pnpm）。
不要提交 `package-lock.json`，也不要与 npm 混用。

提交基线：

```bash
git add -A
git commit -m "chore: local site baseline"
```

### 2. 添加上游远程

```bash
git remote add upstream https://github.com/satnaing/astro-paper.git
git fetch upstream
git remote -v      # 确认 upstream 出现
```

## 三、每次更新的固定流程

```bash
# 0. 确认自己的东西都提交了
git status

# 1. 打个备份标签，方便随时回滚
git tag pre-update-2026-10-22

# 2. 拉取上游
git fetch upstream

# 3. 看看上游改了哪些文件
git diff --stat HEAD upstream/main

# 4. 合并
git merge upstream/main
```

- **没有冲突**：`pnpm install && pnpm build` 验证后 `git push`
- **有冲突**：`git status` 列出冲突文件 → 逐个编辑 → `git add` → `git commit`
  - 你的文章、图片目录原则上**不会**出现在冲突列表里

合并后必须跑一次构建，因为模板可能有 Astro 大版本升级或配置项改名。

也可以改用 rebase（保持线性历史），但会逐个提交解决冲突，第一次操作不如 merge 直观：

```bash
git rebase upstream/main
```

**建议先用 merge。**

## 四、合并原理

| 情况 | 合并结果 |
|---|---|
| 文件只有你新增（你的文章、图片、新组件） | 直接保留，**不可能冲突** |
| 文件只有上游修改（模板源码） | 直接采用上游版本 |
| 文件双方都改了 | 产生冲突，需要手动选择 |

关键原则：**自己的东西尽量都做成"新文件"**，少去"改模板的旧文件"。

## 五、各类自定义的安全等级

| 想改的东西 | 推荐做法 | 冲突风险 |
|---|---|---|
| 新增文章 | 在 `src/content/posts/` 新建文件（不动模板文章） | 无 |
| 文章配图 | 放在文章同目录 `assets/` 或 `src/assets/images/`，用**新文件名** | 无 |
| 新图标 | 在 `src/assets/icons/` 新增自己的 SVG，**不要改模板已有图标** | 无 |
| 新页面 / 新组件 | 新建 `src/pages/xxx.astro`、`src/components/Xxx.astro` | 无 |
| 站名、作者、描述、时区、社交链接、分页数 | 只改 `astro-paper.config.ts`（官方预留的配置入口） | 低 |
| 配色 | **新建** `src/styles/custom.css`，在 `global.css` 末尾加一行 `@import` | 低（只有 1 行改动） |
| favicon / 默认分享图 | 替换 `public/favicon.svg`、`public/default-og.jpg` 的内容（文件名不变） | 低 |
| 导航栏加链接 | 小改 `src/components/Header.astro`，尽量只加几行 | 中 |
| 删除模板自带示例文章 | 一次性 `git rm`，冲突时保持删除即可 | 中 |
| 升级依赖 | 上游经常改 `package.json`，冲突集中在版本号 | 中高 |

### 配色隔离示例

新建 `src/styles/custom.css`：

```css
:root,
[data-theme="light"] {
  --accent: #0f766e;
  --background: #fdfdfd;
  --foreground: #1f2937;
}

[data-theme="dark"] {
  --accent: #2dd4bf;
  --background: #111827;
  --foreground: #e5e7eb;
}
```

然后在 `src/styles/global.css` 的三个 `@import` 之后追加：

```css
@import "./custom.css";
```

这样以后上游修改 `theme.css`，自己的配色仍然生效，冲突最多只是 `global.css` 里的那一行。

## 六、删除模板示例文章的冲突

`adding-new-post.mdx`、`examples/`、`_releases/` 等示例文章是上游**会持续修改**的文件。删除后，上游再修改它们时会出现 "deleted by us / modified by them" 冲突。

**解决方式永远是：保持删除**（`git rm` 这些文件后提交）。这类冲突不影响自己的内容，但可能反复出现。

应对方式：

1. 接受这种一次性、解决很简单的冲突；
2. 开启 `git rerere`，让 git 记住解决结果，下次同类冲突自动处理：

```bash
git config --global rerere.enabled true
```

如果希望完全没有这类噪音，也可以保留模板文档文章不动（它们会出现在文章列表里）。

## 七、`package.json` 冲突处理

上游用 Dependabot 频繁升级依赖，`package.json` 是最常见的冲突点。

解决原则：

- 保留上游的新版本号
- 保留自己额外添加的依赖
- 删除同一依赖重复出现的两行
- 重新安装依赖更新锁文件：`pnpm install`

## 八、更彻底的隔离方案（一般不必）

- **内容单独一个仓库**，用 git submodule 挂到 `src/content/posts`：模板与内容彻底解耦，但部署配置、图片路径都会复杂化，个人博客不建议
- **`git subtree`**：适合把模板当依赖嵌入，学习成本高

对个人站点来说，**"fork + upstream merge + 新文件优先"** 已足够。

## 九、更新后的检查清单

1. `git diff --stat HEAD~1` 查看合并带来了什么
2. 安装依赖：`pnpm install`
3. `pnpm build` —— 模板升级后类型检查会立刻暴露破坏性变更
4. `pnpm preview` —— 确认搜索、暗色模式、文章页正常
5. 查看上游 `CHANGELOG.md` 和 release notes，注意 Astro 主版本升级
6. 没问题再 `git push`；有问题回滚：

```bash
git reset --hard pre-update-2026-10-22
```