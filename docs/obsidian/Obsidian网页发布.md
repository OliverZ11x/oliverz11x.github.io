# Obsidian+Github Page搭建个人网页

## Webpage HTML Export

从单个文件、画布页面或整个库导出 HTML。通过直接访问导出的 HTML 文件，您可以在任何地方发布您的数字花园。注重灵活性、功能和风格的一致性。

Demo / docs: [docs.obsidianweb.net](https://docs.obsidianweb.net/)

![image](https://github.com/KosmosisDire/obsidian-webpage-export/assets/39423700/b8e227e4-b12c-47fb-b341-5c5c2f092ffa)

![image](https://github.com/KosmosisDire/obsidian-webpage-export/assets/39423700/06f29e1a-c067-45e7-9882-f9d6aa83776f)

> [!NOTE]
> 虽然该插件功能齐全，但仍处于开发阶段，因此更新之间可能会经常出现较大的变动，从而影响您的工作流程！Bug 也不少见，请报告您发现的任何问题，我正在努力使插件更加稳定。

### 功能：

- 全文搜索
- 文件导航树
- 文件大纲
- 图表视图
- 主题切换
- 针对网页和移动设备进行了优化
- 支持大多数插件（数据视图、任务等）
- 可将 HTML 和依赖关系导出到一个文件中

### Initiate an Export

There are three main ways to initiate an export.
Using the [ribbon icon](https://docs.obsidianweb.net/getting-started/1.-ribbon-icon.html), [commands](https://docs.obsidianweb.net/getting-started/3.-commands.html), or [context menus](https://docs.obsidianweb.net/getting-started/2.-context-menus.html)

#### Modifying Exports

Once you have opened the [export modal](https://docs.obsidianweb.net/getting-started/4.-export-modal.html)

1. Select which files you want to export using the [file picker](https://docs.obsidianweb.net/getting-started/6.-file-picker.html).
2. Select your desired [export mode](https://docs.obsidianweb.net/getting-started/5.-export-modes.html)
3. Select the directory to export to
4. Export!

#### Hosting your HTML

If you have selected the [Online Web Server](https://docs.obsidianweb.net/getting-started/5.-export-modes.html#Online_Web_Server_Mode) mode, you may be wanting to host your HTML somewhere.

##### Github Pages

[Github Pages](https://pages.github.com/) is a service which allows you to host your own website for free.

## GitHub Pages 快速入门

您可以使用 GitHub Pages 来展示一些开源项目、主持博客甚或分享您的简历。 本指南将帮助您开始创建下一个网站。

### 简介

GitHub Pages 是通过 GitHub 托管和发布的公共网页。 启动和运行的最快方法是使用 Jekyll 主题选择器加载预置主题。 然后，您可以修改 GitHub Pages 的内容和样式。

本指南将引导你在 `username.github.io` 创建用户站点。

### 创建网站

1. 在任何页面的右上角，选择 ，然后单击“新建存储库”****。

	![GitHub 下拉菜单的屏幕截图，其中显示了用于创建新项的选项。 菜单项“新建存储库”用深橙色框标出。](https://docs.github.com/assets/cb-29762/images/help/repository/repo-create-global-nav-update.png)

2. 输入 `username.github.io` 作为存储库名称。 将 `username` 替换为你的 GitHub 用户名。 例如，如果用户名为 `octocat`，则存储库名称应为 `octocat.github.io`。

	![存储库中 GitHub Pages 设置的屏幕截图。 存储库名称字段包含文本“octocat.github.io”，并用深橙色框出。](https://docs.github.com/assets/cb-48480/images/help/pages/create-repository-name-pages.png)

3. 在存储库名称下，单击 “设置”。 如果看不到“设置”选项卡，请选择“”下拉菜单，然后单击“设置”********。

	![存储库标头的屏幕截图，其中显示了选项卡。 “设置”选项卡以深橙色边框突出显示。](https://docs.github.com/assets/cb-28260/images/help/repository/repo-actions-settings.png)

4. 在边栏的“代码和自动化”部分中，单击“ Pages”。
5. 在“生成和部署”的“源”下，选择“从分支进行部署”。
6. 在“生成和部署”的“分支”下，使用分支下拉菜单并选择发布源。

	![GitHub 存储库中 Pages 设置的屏幕截图。 用于为发布源选择分支的菜单（标有“无”）用深橙色框出。](https://docs.github.com/assets/cb-47265/images/help/pages/publishing-source-drop-down.png)

7. （可选）打开存储库的 `README.md` 文件。 `README.md` 文件是你将为站点编写内容的位置。 您可以编辑文件或暂时保留默认内容。
8. 访问 `username.github.io` 以查看新网站。 请注意，对站点的更改在推送到 GitHub 后，最长可能需要 10 分钟才会发布。

### 更改标题和说明

默认情况下，站点的标题为 `username.github.io`。 可通过编辑存储库中的 `_config.yml` 文件来更改标题。 您还可以为您的网站添加说明。

1. 单击存储库的“代码”选项卡。
2. 在文件列表中，单击 `_config.yml` 以打开该文件。
3. 单击  编辑文件。
4. `_config.yml` 文件已包含指定站点主题的行。 添加一个新行，其中 `title:` 后跟所需的标题。 添加一个新行，其中 `description:` 后跟所需的描述。 例如：

	```yaml
    theme: jekyll-theme-minimal
    title: Octocat's homepage
    description: Bookmark this to keep an eye on my project updates!
    ```

5. 编辑完文件后，单击“提交更改”。

### [后续步骤](https://docs.github.com/zh/pages/quickstart#next-steps)

若要详细了解如何向网站添加其他页面，请参阅“[使用 Jekyll 向 GitHub Pages 站点添加内容](https://docs.github.com/zh/pages/setting-up-a-github-pages-site-with-jekyll/adding-content-to-your-github-pages-site-using-jekyll#about-content-in-jekyll-sites)”。

若要详细了解如何使用 Jekyll 设置 GitHub Pages 站点，请参阅“[关于 GitHub 页面和 Jekyll](https://docs.github.com/zh/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll)”。

# Publish your Obsidian Notes

MkDocs template [![Built with Material for MkDocs](https://img.shields.io/badge/Material_for_MkDocs-526CFE?style=for-the-badge&logo=MaterialForMkDocs&logoColor=white)](https://squidfunk.github.io/mkdocs-material/)

Would you like to take _some_ of your notes in [Obsidian](https://obsidian.md/) and make it public?

This template gives you an easy (and automated) way to publish your Obsidian notes (or blog!) on your Github pages.

With this template, you get these **out-of-the-box**:

- an awesome website based on Material theme, complete with a search bar (Checkout this template repo published [here](https://jobindjohn.github.io/obsidian-publish-mkdocs/))

![[IMG-2024-07-29-18-26.png]]
- get the Obsidian/Roam style `[[wikilinks]]` from your vault in your published notes
- Toggle between light and dark mode
- Blog folder

## Quick start

1. Create a **new github repository using this template**. Click the green button at the top or use [this link](https://github.com/jobindjohn/obsidian-publish-mkdocs/generate).
![[IMG-2024-07-29-18-26-1.png]]

2. **Give a name** to your repository. By default your notes will be published at `<https://username.github.io/repo-name/>`
	 - Copy only the `main` branch while creating the repo from the template
3. **Clone** the repository you generated **into your Obsidian folder/vault.**
4. **Move your notes** that you would like to make public to the `repo-name/docs` folder.
	- Easiest way to do this would be using drag and drop within Obsidian
5. Commit and **push** the changes. Github actions will take care of the rest, publishing your notes using [MkDocs](https://www.mkdocs.org/), with the [Material theme](https://squidfunk.github.io/mkdocs-material/).
6. Go to `Settings > Pages` and select the select the **Source** as your `gh-pages` branch.

![[IMG-2024-07-29-18-26-2.png]] ![[2021-11-22-22-52-49.png]]

**Not working for you?** Open an [issue](https://github.com/jobindjohn/obsidian-publish-mkdocs/issues/new/choose) and let me know what went wrong.

## Configuring your website

### How do I arrange notes as sections and pages?

By default, the sections and pages will follow the folder structure within `/docs`. The folders and sub-folders will show up as sections. Try not to have white spaces in your folder and file names, as these will be converted to HTML links. The webpage heading will be the same as the first-level heading in the markdown note.

- If you would like to arrange the pages manually, then use the `nav` option in the `mkdocs.yml` [configuration file](https://www.mkdocs.org/#adding-pages) at the root of this repo to set custom page navigation.
	- For example, see the setup for [the Blue Book](https://lyz-code.github.io/blue-book/) at [github](https://github.com/lyz-code/blue-book/blob/master/mkdocs.yml). Managing each page using `nav` can become cumbersome as the number of notes increase though!
- The Materials theme provides multiple options to arrange [sections](https://squidfunk.github.io/mkdocs-material/setup/setting-up-navigation/#navigation-sections), use [navigation tabs](https://squidfunk.github.io/mkdocs-material/setup/setting-up-navigation/#navigation-tabs), and many other helpful [navigation setups](https://squidfunk.github.io/mkdocs-material/setup/setting-up-navigation/)

## Alternatives

- [datopian/obsidian-flowershow](https://github.com/datopian/obsidian-flowershow): plugin for publishing with flowershow direct from your obsidian vault.
- [kmaasrud/oboe](https://github.com/kmaasrud/oboe): tool to convert an Obsidian vault into a static directory of HTML files.
- [Jackiexiao/foam-mkdocs-template](https://github.com/Jackiexiao/foam-mkdocs-template): template for Obsidian/Foam using mkdocs/mkdocs-material/mkdocs-roamlinks-plugin
- [foambubble/foam-template](https://github.com/foambubble/foam-template): Foam workpace template
- [ObsidianPublisher/obsidian-mkdocs-publisher-template](https://github.com/ObsidianPublisher/obsidian-mkdocs-publisher-template): Obsidian Mkdocs Publisher, a free obsidian publish alternative throught Mkdocs
- [KosmosisDire/obsidian-webpage-export](https://github.com/KosmosisDire/obsidian-webpage-export): Webpage HTML Export lets you export single files or whole vaults as HTML websites or documents. It is similar to publish, but you get direct access to the exported HTML.
- [Enveloppe/obsidian-enveloppe: publish your notes on a GitHub repository from Obsidian Vault](https://github.com/Enveloppe/obsidian-enveloppe)

## Other interesting projects

- [mathieudutour/gatsby-digital-garden: digital garden with Gatsby](https://github.com/mathieudutour/gatsby-digital-garden)
- [TuanManhCao/digital-garden: Free Obisidian Publish alternative](https://github.com/TuanManhCao/digital-garden)

