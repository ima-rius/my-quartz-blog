# Quartz 部署 Obsidian 笔记教程

**Date**: May 31, 2026 at 07:56 PM
**Turns**: 1
**Source**: [Gemini Chat](https://gemini.google.com/app/b45a053f05e3a16c)

---

## Turn 1

### 👤 User

好了，终于完成了。
现在，你把具体方法和具体流程和内容写出来，做成一个文件，我进行下载备份。

### 🤖 Assistant

这是一份为您量身定制的完整备份指南。其中包含了我们最终跑通的**所有核心配置文件、网页端关键设置以及日常操作流程**。
我已将这些内容整理并打包成了一个可直接下载、阅读的 **Markdown 备份文件 (Quartz_Blog_Sync_Backup.md)**，方便您以后随时查阅或一键迁移。

```markdown
# 🪴 Quartz 4.0 + Obsidian 全自动即时同步博客系统备份指南

这份文件详细记录了 `ima-rius/obhub`（笔记私有库）与 `ima-rius/my-quartz-blog`（公开博客库）之间实现**秒级、无感、即时同步**的最终完美配置方案。

---

## 🛠️ 第一部分：核心架构与数据流向

为了完美绕过 GitHub 对私有仓库跨库读取的严格权限限制，系统采用了**“主动推送”**架构：

```
【本地 Obsidian 写作】
│
▼ (Obsidian Git 插件自动提交/推送)
【私有笔记库：ima-rius/obhub】
│
▼ (触发 trigger.yml 工作流：使用 QUARTZ_TOKEN 权限)
【强行轰炸推送至博客库：my-quartz-blog/content】
│
▼ (触发 deploy.yml 工作流：检测到 push 自动亮起黄灯)
【Quartz 4.0 云端自动编译编译】 ──> 【GitHub Pages 站点即时更新 🚀】

```

---

## 🔑 第二部分：网页端关键凭证设置 (GitHub Secrets)

为了让两个仓库之间能够合法通信，必须配置授权钥匙：

1. **生成全局令牌 (Personal Access Token)**
   * 进入 GitHub 个人设置 `Settings` -> `Developer settings` -> `Personal access tokens` -> `Tokens (classic)`。
   * 生成一个名为 `quartz-sync` 的永久有效（No expiration）令牌，**必须严格勾选 `repo` 权限大项**。
2. **在笔记库中绑定钥匙**
   * 进入 **`ima-rius/obhub`** 仓库的 `Settings` -> `Secrets and variables` -> `Actions`。
   * 点击 `New repository secret`，名称严格填写为 **`QUARTZ_TOKEN`**，内容填入生成的 `ghp_` 开头令牌。

---

## 📝 第三部分：两个仓库的自动化配置文件 (Workflow)

### 1. 笔记私有库中的“发报轰炸机”脚本
* **本地路径**：`你的Obsidian库/.github/workflows/trigger.yml`
* **文件内容**：
```yaml
name: Push Notes to Quartz Blog
on:
  push:
    branches:
      - main # 如果您的笔记仓库主分支叫 master，这里请改成 master

jobs:
  push-to-blog:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Notes Source
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Push content folder to Blog Repository
        uses: cpina/github-action-push-to-another-repository@main
        env:
          API_TOKEN_GITHUB: ${{ secrets.QUARTZ_TOKEN }}
        with:
          source-directory: '.' 
          destination-github-username: 'ima-rius'
          destination-repository-name: 'my-quartz-blog'
          target-branch: 'main'
          target-directory: 'content' 
          user-email: 'github-actions[bot]@users.noreply.github.com'
          user-name: 'github-actions[bot]'
          commit-message: 'Automated notes sync from obhub'

```

### 2. 博客主仓库中的“纯净编译”脚本

- **本地路径**：`D:\PG\my-quartz-blog\.github\workflows\deploy.yml`
- **文件内容**：

```yaml
name: Deploy Quartz site to GitHub Pages
on:
  push:
    branches:
      - main

permissions:
  contents: write
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Setup Pages
        uses: actions/configure-pages@v5
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20
      - name: Install Dependencies
        run: npm ci
      - name: Build Quartz
        run: npx quartz build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: public
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4

```

## 🎯 第四部分：日常极致流（无感日常）
从今以后，您**不需要**打开本地的电脑终端，**不需要**配置网络代理，甚至**不需要**双击任何脚本：

1. **写字**：和平常一样，在本地 **Obsidian** 里自由地增删笔记、修改文章。
  - *注意：请确保有一篇核心主页笔记命名为全小写的 index.md 作为博客首页。*
2. **发布**：Obsidian Git 插件会自动（或您手动在面板点一下）将内容推送到 `obhub`。
3. **生效**：剩下的全部交给云端。大约 1~2 分钟后，刷新您的专属网址即可看到全新内容：
👉 `https://ima-rius.github.io/my-quartz-blog/`

```

---

### 📥 备份保存方法
1. 您可以点击上方代码框右上角的 **“Copy”** 按钮。
2. 在您的电脑里或 Obsidian 中新建一个名为 `Quartz博客全自动同步备份.md` 的文件，把内容粘贴进去，作为永久的生存指南存档。

祝贺通关！您的个人高科技知识发布系统已彻底步入正轨。写字愉快！

```

---

*Exported from [Voyager](https://github.com/Nagi-ovo/gemini-voyager)*  
*Generated on May 31, 2026 at 07:56 PM*