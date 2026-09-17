# AGENTS.md

## 项目定位

`vibecode` 是一个面向 Vibe Coding 的展示与转化网站，包含两类核心内容：

1. **免费组件库**：展示业务组件、App UI、动画组件，以及对应可直接用于 AI Coding 的 Prompt。
2. **完整模板库**：展示完整网站 / Web App / PWA 模板，并按不同技术路线给出参考报价，通过微信和邮箱进行人工沟通，不接入在线支付。

本项目优先采用静态化方案，追求低成本、低维护、SEO 友好、加载快，并复用 `zero-blog` 已验证的“数据源 -> 构建 -> 静态产物 -> GitHub Actions -> 又拍云”发布模式。

---

## 产品边界

### 免费组件库

只包含以下三类，不做 Button / Input / Select 等基础原子组件：

- **Business Components**：登录、搜索筛选、上传、结算、资料编辑、AI 对话等业务组件。
- **App UI**：预算、Todo、设置、聊天、统计、日历、笔记等完整业务页面或界面。
- **Motion Components**：Scroll Reveal、Card Stack、数字滚动、页面过渡、Bottom Sheet、3D Tilt、Marquee 等动画与交互效果。

每个组件至少包含：

- 效果预览
- Demo（如适用）
- Prompt
- 技术栈
- 功能 / 交互说明
- 标签与分类

组件内容永久免费。

### 完整模板库

Template 是可以直接作为完整产品交付的成品项目，例如：

- 企业官网
- Landing Page
- Blog
- 工具网站
- Web App
- PWA
- SaaS

每个模板至少展示：

- 完整效果图
- 在线 Demo
- 页面与功能列表
- 使用到的组件
- 技术栈
- 不同交付技术路线
- 各路线参考报价
- 微信 / 邮箱联系方式

不做在线支付、购物车、订单系统。

---

## 技术栈

### 核心框架

- **Next.js**：App Router
- **TypeScript**
- **React**
- **Tailwind CSS**
- **shadcn/ui**
- **Motion**：优先使用 `motion` / Motion for React
- **PWA**：Manifest + Service Worker + 可安装能力

### 重要约束

- **不使用 Vite 作为主构建工具。** Next.js 自带完整构建体系，本项目不维护 Next.js + Vite 双构建链。
- 网站首先是 PC / Mobile 响应式内容网站，PWA 是附加能力，不把整个站点强行设计成 iOS App。
- 默认黑色 / 深色视觉风格，强调作品展示、代码感和专业感。
- 优先支持桌面端浏览，同时保证移动端体验。

---

## 架构原则

### 1. 静态优先

V1 采用：

> Build Time Data + SSG + Client Interaction + PWA

最终输出静态文件并部署到又拍云。

Next.js 配置目标：

```ts
output: "export"
```

因此禁止依赖必须常驻 Node.js Server 才能工作的功能。

### 2. 不依赖 Next.js Server

除非项目未来明确升级部署方案，否则不要使用：

- Server Actions 作为核心业务能力
- 运行时 SSR
- 依赖服务端 Session 的认证体系
- 必须由 Next.js Server 执行的 API Route
- 动态图片处理依赖服务端运行时
- 需要数据库实时查询才能渲染的页面

需要交互时优先使用客户端能力、构建时生成数据或外部独立 API。

### 3. 数据驱动

页面禁止大量硬编码业务数据。

数据层参考 `zero-blog` 思路：

```text
内容源 / 静态数据
      ↓
build-site-data.ts
      ↓
标准化 JSON
      ↓
Next.js SSG
      ↓
out/
```

所有可展示内容尽可能由统一数据 Schema 驱动。

---

## 建议目录结构

```text
vibecode/
├── app/
│   ├── page.tsx
│   ├── components/
│   │   ├── page.tsx
│   │   └── [slug]/page.tsx
│   ├── templates/
│   │   ├── page.tsx
│   │   └── [slug]/page.tsx
│   ├── pricing/page.tsx
│   └── contact/page.tsx
│
├── components/
│   ├── ui/                 # shadcn/ui
│   ├── layout/
│   ├── gallery/
│   └── common/
│
├── data/
│   ├── components/
│   │   ├── business/
│   │   ├── app-ui/
│   │   └── motion/
│   └── templates/
│
├── generated/
│   ├── components.json
│   ├── templates.json
│   └── search-index.json
│
├── public/
│   ├── covers/
│   ├── previews/
│   ├── demos/
│   ├── icons/
│   └── manifest.webmanifest
│
├── scripts/
│   ├── build-site-data.ts
│   └── youpai-sync.*
│
├── lib/
│   ├── data/
│   ├── schema/
│   └── utils/
│
├── types/
├── AGENTS.md
└── ...
```

目录允许随实际开发调整，但必须保持“数据源、生成数据、页面、展示组件、部署脚本”职责清晰。

---

## 数据模型

### Component

推荐核心字段：

```ts
export interface VibeComponent {
  slug: string
  name: string
  description: string

  type: "business" | "app-ui" | "motion"
  tags: string[]

  preview: {
    cover: string
    demo?: string
  }

  prompt: {
    content: string
    language?: "zh" | "en"
  }

  stack: string[]
  features?: string[]

  relatedTemplates?: string[]

  status?: "draft" | "published"
  sort?: number
}
```

### Template

```ts
export interface VibeTemplate {
  slug: string
  name: string
  description: string

  cover: string
  demo?: string

  tags: string[]
  pages?: string[]
  features: string[]
  stack: string[]

  relatedComponents?: string[]

  solutions: TemplateSolution[]

  status?: "draft" | "published"
  sort?: number
}

export interface TemplateSolution {
  type: "static" | "fullstack" | "admin" | string
  name: string
  price: string
  description?: string
  stack: string[]
  includes?: string[]
  suitableFor?: string
}
```

价格统一存字符串，例如：

```text
¥299 起
¥999 起
联系报价
```

不要把价格建模成强制数值，因为当前网站只展示参考报价，不做在线交易。

---

## 模板技术路线

同一个 Template 可以展示多套交付方案。

默认支持：

### Static / 纯静态版本

适合：

- 企业展示站
- Landing Page
- 工具站
- 无账号体系的小型 PWA

典型技术：

- React / Next.js Static Export
- LocalStorage / IndexedDB
- PWA
- 静态托管

### Full Stack / 全栈版本

适合：

- 需要账号体系
- 云端数据
- API
- 多端同步

典型技术：

- Next.js
- PostgreSQL
- Auth
- API
- 独立后端或 Serverless

### Admin / 后台管理版本

适合正式运营产品。

典型能力：

- 用户管理
- 内容管理
- 数据管理
- Dashboard
- RBAC
- 操作日志

Template 详情页需要明确解释不同路线的功能边界与报价差异。

---

## 组件与模板关联

免费组件与收费模板不能割裂。

推荐双向关联：

```text
Component
  ↓
Used in Templates

Template
  ↓
Built with Components
```

例如：

```text
Travel Budget Home UI
Expense Form
Number Animation
Bottom Sheet
        ↓
Travel Budget PWA Template
```

组件负责流量与 Prompt 传播，模板负责完整成品展示与商业转化。

---

## UI 规范

整体视觉方向：

- 黑色 / 深灰背景
- 大面积留白（深色负空间）
- 卡片式 Gallery
- 轻量渐变与高光
- 强调截图和 Demo，减少无意义装饰
- 大标题、大预览图、小面积品牌色
- 现代 SaaS / Developer Tool 风格

首页重点表达两件事情：

```text
Components
免费业务组件 / App UI / Motion + Prompt

Templates
完整网站模板 + 多种技术路线 + 参考报价
```

首页内容优先展示作品，不要堆砌大量营销文案。

---

## 页面规划

### `/`

首页：

- Hero
- Featured Components
- Business / App UI / Motion 分类入口
- Featured Templates
- 技术路线简要说明
- 联系 CTA

### `/components`

免费组件库。

支持：

- 分类
- 标签
- 搜索
- Gallery 浏览

### `/components/[slug]`

组件详情页：

- Preview
- Demo
- Prompt
- Copy Prompt
- Stack
- Features
- Related Templates

### `/templates`

完整模板 Gallery。

### `/templates/[slug]`

模板详情页：

- 完整效果
- Demo
- 页面列表
- 功能
- 技术栈
- Related Components
- Static / Full Stack / Admin 等交付方案
- 参考报价
- 微信 / Email 联系 CTA

### `/pricing`

不是套餐购买页，而是“技术路线与报价说明页”。

重点解释：

- 为什么静态站成本低
- 什么情况下需要后端
- 什么情况下需要 Admin
- 不同技术路线的能力边界

### `/contact`

展示：

- 微信二维码
- 微信号（如需要）
- Email
- 可承接范围

---

## PWA 规范

至少包含：

- `manifest.webmanifest`
- App icons
- `display: standalone`
- theme color
- background color
- 基础 Service Worker
- 静态资源缓存

PWA 不应破坏普通浏览器访问、SEO 或静态部署。

离线策略优先简单可靠，不要为了离线而缓存所有动态页面。

---

## SEO

每一个 Component 和 Template 都应该是独立可索引页面。

至少提供：

- title
- description
- canonical
- Open Graph
- sitemap
- robots.txt

可进一步生成：

- RSS / Feed（如后续内容规模需要）
- `llms.txt`
- Search Index

URL 保持长期稳定。

---

## 构建与发布

发布流程复用 `zero-blog` 模式：

```text
本地开发
   ↓
git push
   ↓
GitHub Actions
   ↓
pnpm install
   ↓
数据生成脚本
   ↓
Next.js build
   ↓
Static Export -> out/
   ↓
同步到又拍云
   ↓
刷新 CDN
```

### GitHub Actions 基本要求

CI 至少执行：

```bash
pnpm install --frozen-lockfile
pnpm build
```

如数据生成独立于 build：

```bash
pnpm generate:data
pnpm build
```

部署阶段上传 `out/`。

### 又拍云同步

优先复用 / 抽取现有 `zero-blog` 的同步思路。

同步脚本长期目标：

- manifest 对比
- 增量上传
- 并发上传
- 删除失效文件
- 合理设置缓存策略
- 必要时刷新 CDN

禁止每次无脑全量上传大量未变化资源。

---

## 开发原则

1. **先数据模型，后页面。** 新增组件和模板优先通过数据驱动，而不是复制页面代码。
2. **优先复用。** 重复出现两次以上的 UI 结构应评估抽成组件。
3. **shadcn/ui 优先。** 通用 UI 不重复造轮子。
4. **Motion 只服务交互。** 避免无意义、影响性能的动画。
5. **静态部署优先。** 引入任何服务端能力前，先确认静态方案是否真的无法满足。
6. **移动端必须可用，但设计不局限于移动端。**
7. **图片资源必须优化。** Gallery 项目图片多，关注尺寸、格式和加载策略。
8. **避免过度工程化。** V1 不引入数据库、登录、支付、CMS。
9. **保持内容可迁移。** 数据 Schema 不应强耦合 React Component 实现。
10. **商业转化保持简单。** Template 只展示报价和联系方式，人工沟通成交。

---

## V1 明确不做

- 用户注册 / 登录
- 收藏同步
- 在线支付
- 订单
- 购物车
- 在线购买源码
- 评论系统
- 用户投稿
- CMS 后台
- 自建数据库
- 实时服务端搜索

这些能力只有在真实需求出现后再增加。

---

## Agent 执行要求

AI Agent 在本仓库开发时必须遵循：

1. 修改前先理解现有目录和数据 Schema，禁止未经检查大范围重构。
2. 新页面默认考虑 Static Export 兼容性。
3. 新增依赖前先确认现有技术栈无法解决。
4. 不得擅自引入 Vite、独立 Node Server、数据库、认证服务或支付系统。
5. 不得将大量业务内容硬编码在 JSX 中；优先放入数据层。
6. 组件详情与模板详情必须能够通过 slug 静态生成。
7. 新增视觉效果时保持深色设计语言一致。
8. 新功能不能破坏 GitHub Actions -> `out/` -> 又拍云的静态发布链路。
9. 涉及架构性变更时，优先更新本文件后再实施。
10. 如果需求与本文件冲突，以用户最新明确指令为准，并同步修订本文件。

---

## 当前最终技术方案

```text
Next.js App Router
+ React
+ TypeScript
+ Tailwind CSS
+ shadcn/ui
+ Motion
+ PWA
+ 静态数据 / 构建时数据生成
+ Next.js Static Export
+ GitHub Actions
+ 又拍云对象存储 / CDN
```

核心原则：

> GitHub 是代码与内容源，构建阶段生成网站数据和静态页面，最终将 `out/` 部署到又拍云。V1 不需要数据库、不需要在线支付、不需要常驻服务端。
