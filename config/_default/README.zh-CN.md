# config/_default 说明

本目录是 Hugo 的默认配置目录，Hugo 会自动读取这里的配置文件并合并生效（同名配置项后读的会覆盖先读的）。

## 文件一览

- [config.toml](file:///Users/hanhan/Projects/PersonalOpenSource/Han-GR.github.io/config/_default/config.toml)
  - 站点基础配置：`baseurl`、`languageCode`、站点标题、分页等。
  - `defaultContentLanguage`：主题 UI 文案语言（会影响暗色模式、归档、搜索等内置文案的显示语言）。
  - `hasCJKLanguage`：中文/日文/韩文建议开启，摘要与字数统计更准确。

- [_languages.toml](file:///Users/hanhan/Projects/PersonalOpenSource/Han-GR.github.io/config/_default/_languages.toml)
  - 多语言配置模板。
  - 将它重命名为 `languages.toml` 才会启用 Hugo 的多语言模式（多语言站点/URL 前缀/每种语言的菜单等）。

- [menu.toml](file:///Users/hanhan/Projects/PersonalOpenSource/Han-GR.github.io/config/_default/menu.toml)
  - 菜单配置：
    - `main`：侧边栏主导航菜单（首页/分类/归档/标签/友链/搜索/关于等）。
    - `social`：侧边栏头像下方的社交链接（GitHub/邮箱/微信等）。
  - `weight` 用于排序，数字越小越靠前。
  - `params.icon` 会去 `assets/icons/<icon>.svg` 找同名 SVG（你项目里已用这个规则自定义图标）。

- [params.toml](file:///Users/hanhan/Projects/PersonalOpenSource/Han-GR.github.io/config/_default/params.toml)
  - 主题参数（hugo-theme-stack 的核心配置入口）：侧边栏文案与头像、首页组件 widgets、文章行为、暗色模式开关等。
  - 常改项：
    - `[sidebar]`：头像、签名、emoji
    - `[widgets]`：首页/页面的组件
    - `[colorScheme]`：暗色模式默认与是否显示切换按钮

- [permalinks.toml](file:///Users/hanhan/Projects/PersonalOpenSource/Han-GR.github.io/config/_default/permalinks.toml)
  - 永久链接格式：决定不同内容类型（post/page）的 URL 结构，比如文章 `/p/:slug/`。

- [markup.toml](file:///Users/hanhan/Projects/PersonalOpenSource/Han-GR.github.io/config/_default/markup.toml)
  - Markdown 渲染与代码高亮配置：
    - Goldmark（是否允许 HTML、扩展等）
    - TOC（目录）层级
    - highlight（代码高亮、行号等）

- [related.toml](file:///Users/hanhan/Projects/PersonalOpenSource/Han-GR.github.io/config/_default/related.toml)
  - 相关文章推荐配置：基于 tags/categories 等进行相似度计算。

- [module.toml](file:///Users/hanhan/Projects/PersonalOpenSource/Han-GR.github.io/config/_default/module.toml)
  - Hugo Modules 配置：通过 Go Modules 引入主题（此项目引入 `hugo-theme-stack/v4`）。

