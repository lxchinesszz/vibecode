# AGENTS.md

## 项目说明

`vibecode` 是一个面向 Vibe Coding 的展示与转化网站：

- **Components**：永久免费，只收录业务组件、App UI、动画组件，并提供可直接用于 AI Coding 的 Prompt。
- **Templates**：展示可交付的完整网站 / Web App / PWA 模板，按技术路线提供参考报价，通过微信和邮箱人工沟通，不接入在线支付。

项目采用静态优先架构，复用 `zero-blog` 的“内容源 → 构建 → 静态产物 → GitHub Actions → 又拍云”发布模式。

## 开发入口

开始任何修改前，必须先阅读与本次任务相关的 `.codex/` 规则文件。规则按职责拆分如下：

- [`.codex/产品与界面.md`](.codex/产品与界面.md)：产品边界、组件与模板定义、UI 风格、页面规划。
- [`.codex/技术架构.md`](.codex/技术架构.md)：技术栈、静态架构、数据模型、PWA、SEO、目录结构。
- [`.codex/开发规范.md`](.codex/开发规范.md)：修改前检查、编码原则、测试验证、构建产物约束。
- [`.codex/需求管理.md`](.codex/需求管理.md)：`docs/requirements/` 需求档案、编号、变更记录、状态与完成门禁。
- [`.codex/构建与发布.md`](.codex/构建与发布.md)：GitHub Actions、Static Export、又拍云同步与发布规则。

如果一次任务同时涉及多个领域，必须同时遵守对应规则文件。

## 核心技术方案

```text
Next.js App Router
+ TypeScript
+ React
+ Tailwind CSS
+ shadcn/ui
+ Motion
+ PWA
+ 构建时静态数据
+ Next.js Static Export
+ GitHub Actions
+ 又拍云对象存储 / CDN
```

核心约束：

1. 不使用 Vite 作为主构建工具，不维护 Next.js + Vite 双构建链。
2. V1 不引入数据库、登录、支付、CMS 或常驻 Node.js 服务。
3. 页面和内容优先数据驱动，最终必须能够通过 `output: "export"` 生成 `out/`。
4. GitHub 是代码与内容源；GitHub Actions 负责构建；最终静态产物部署到又拍云。
5. 所有会改变产品行为、架构、发布方式或项目规则的请求，都必须按照 `.codex/需求管理.md` 建立或更新需求档案。
