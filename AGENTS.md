# AGENTS.md

## 项目定位

`vibecode` 是一个面向 Vibe Coding 的展示与转化网站，包含两类核心内容：

1. **免费组件库**：展示业务组件、App UI、动画组件，以及对应可直接用于 AI Coding 的 Prompt。
2. **完整模板库**：展示完整网站 / Web App / PWA 模板，并按不同技术路线给出参考报价，通过微信和邮箱人工沟通，不接入在线支付。

项目优先采用静态化方案，追求低成本、低维护、SEO 友好和高加载性能，并复用 `zero-blog` 已验证的“数据源 → 构建 → 静态产物 → GitHub Actions → 又拍云”发布模式。

---

## 产品边界

### 免费组件库

只包含以下三类，不做 Button / Input / Select 等基础原子组件：

- **Business Components**：登录、搜索筛选、上传、结算、资料编辑、AI 对话等业务组件。
- **App UI**：预算、Todo、设置、聊天、统计、日历、笔记等完整业务页面或界面。
- **Motion Components**：Scroll Reveal、Card Stack、数字滚动、页面过渡、Bottom Sheet、3D Tilt、Marquee 等动画与交互效果。

每个组件至少包含：效果预览、Demo（如适用）、Prompt、技术栈、功能 / 交互说明、标签与分类。组件内容永久免费。

### 完整模板库

Template 是可以直接作为完整产品交付的成品项目，例如企业官网、Landing Page、Blog、工具网站、Web App、PWA、SaaS。

每个模板至少展示：完整效果图、在线 Demo、页面与功能列表、使用到的组件、技术栈、不同交付技术路线、各路线参考报价、微信 / 邮箱联系方式。

不做在线支付、购物车、订单系统。

---

## 技术栈

- **Next.js**：App Router
- **TypeScript**
- **React**
- **Tailwind CSS**
- **shadcn/ui**
- **Motion**：优先使用 `motion` / Motion for React
- **PWA**：Manifest + Service Worker + 可安装能力

### 重要约束

- **不使用 Vite 作为主构建工具。** Next.js 自带完整构建体系，不维护 Next.js + Vite 双构建链。
- 网站首先是 PC / Mobile 响应式内容网站，PWA 是附加能力，不把整个站点强行设计成 iOS App。
- 默认黑色 / 深色视觉风格，强调作品展示、代码感和专业感。
- 优先支持桌面端浏览，同时保证移动端体验。

---

## 架构原则

### 1. 静态优先

V1 采用：

> Build Time Data + SSG + Client Interaction + PWA

最终输出静态文件并部署到又拍云。Next.js 配置目标：

```ts
output: "export"
```

因此禁止依赖必须常驻 Node.js Server 才能工作的能力。

### 2. 不依赖 Next.js Server

除非未来明确升级部署方案，否则不要把以下能力作为核心依赖：

- Server Actions
- 运行时 SSR
- 依赖服务端 Session 的认证体系
- 必须由 Next.js Server 执行的 API Route
- 依赖服务端运行时的动态图片处理
- 需要数据库实时查询才能渲染的页面

需要交互时优先使用客户端能力、构建时生成数据或外部独立 API。

### 3. 数据驱动

页面禁止大量硬编码业务数据。数据层参考 `zero-blog`：

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
├── components/
│   ├── ui/
│   ├── layout/
│   ├── gallery/
│   └── common/
├── data/
│   ├── components/
│   │   ├── business/
│   │   ├── app-ui/
│   │   └── motion/
│   └── templates/
├── generated/
│   ├── components.json
│   ├── templates.json
│   └── search-index.json
├── docs/
│   └── requirements/
│       ├── README.md
│       └── _template/
├── public/
│   ├── covers/
│   ├── previews/
│   ├── demos/
│   ├── icons/
│   └── manifest.webmanifest
├── scripts/
│   ├── build-site-data.ts
│   └── youpai-sync.*
├── lib/
│   ├── data/
│   ├── schema/
│   └── utils/
├── types/
└── AGENTS.md
```

目录允许随实际开发调整，但必须保持“数据源、生成数据、页面、展示组件、需求档案、部署脚本”职责清晰。

---

## 数据模型

### Component

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

价格统一存字符串，例如 `¥299 起`、`¥999 起`、`联系报价`。当前网站只展示参考报价，不做在线交易，因此不要强制建模为数值价格。

---

## 模板技术路线

同一个 Template 可以展示多套交付方案。

### Static / 纯静态版本

适合企业展示站、Landing Page、工具站、无账号体系的小型 PWA。典型技术：React / Next.js Static Export、LocalStorage / IndexedDB、PWA、静态托管。

### Full Stack / 全栈版本

适合需要账号体系、云端数据、API、多端同步的产品。典型技术：Next.js、PostgreSQL、Auth、API、独立后端或 Serverless。

### Admin / 后台管理版本

适合正式运营产品。典型能力：用户管理、内容管理、数据管理、Dashboard、RBAC、操作日志。

Template 详情页必须明确解释不同路线的功能边界与报价差异。

---

## 组件与模板关联

免费组件与收费模板不能割裂，采用双向关联：

```text
Component → Used in Templates
Template  → Built with Components
```

组件负责流量与 Prompt 传播，模板负责完整成品展示与商业转化。

---

## UI 规范

整体视觉方向：

- 黑色 / 深灰背景
- 大面积深色负空间
- 卡片式 Gallery
- 轻量渐变与高光
- 强调截图和 Demo，减少无意义装饰
- 大标题、大预览图、小面积品牌色
- 现代 SaaS / Developer Tool 风格

首页重点表达两件事：

```text
Components
免费业务组件 / App UI / Motion + Prompt

Templates
完整网站模板 + 多种技术路线 + 参考报价
```

首页优先展示作品，不堆砌营销文案。

---

## 页面规划

- `/`：Hero、Featured Components、分类入口、Featured Templates、技术路线说明、联系 CTA。
- `/components`：免费组件库，支持分类、标签、搜索和 Gallery 浏览。
- `/components/[slug]`：Preview、Demo、Prompt、Copy Prompt、Stack、Features、Related Templates。
- `/templates`：完整模板 Gallery。
- `/templates/[slug]`：完整效果、Demo、页面列表、功能、技术栈、Related Components、交付方案、参考报价、微信 / Email CTA。
- `/pricing`：技术路线与报价说明，不是套餐购买页。
- `/contact`：微信二维码、微信号（如需要）、Email、可承接范围。

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

PWA 不应破坏普通浏览器访问、SEO 或静态部署。离线策略优先简单可靠，不为了离线而缓存所有页面。

---

## SEO

每一个 Component 和 Template 都必须是独立可索引页面，至少提供：title、description、canonical、Open Graph、sitemap、robots.txt。

可进一步生成 RSS / Feed、`llms.txt`、Search Index。URL 应保持长期稳定。

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
Static Export → out/
   ↓
同步到又拍云
   ↓
刷新 CDN
```

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

部署阶段只上传 `out/`。

### 又拍云同步

优先复用 / 抽取 `zero-blog` 的同步思路，长期目标包括：manifest 对比、增量上传、并发上传、删除失效文件、合理缓存策略、必要时刷新 CDN。禁止每次无脑全量上传大量未变化资源。

---

## 开发原则

1. **先数据模型，后页面。** 新增组件和模板优先通过数据驱动，而不是复制页面代码。
2. **优先复用。** 重复出现两次以上的 UI 结构应评估抽成组件。
3. **shadcn/ui 优先。** 通用 UI 不重复造轮子。
4. **Motion 只服务交互。** 避免无意义、影响性能的动画。
5. **静态部署优先。** 引入任何服务端能力前，先确认静态方案是否真的无法满足。
6. **移动端必须可用，但设计不局限于移动端。**
7. **图片资源必须优化。** Gallery 图片多，关注尺寸、格式和加载策略。
8. **避免过度工程化。** V1 不引入数据库、登录、支付、CMS。
9. **保持内容可迁移。** 数据 Schema 不应强耦合 React Component 实现。
10. **商业转化保持简单。** Template 只展示报价和联系方式，人工沟通成交。

---

## Agent 开发工作流

以下规则适用于 Agent / AI 对本仓库进行需求实现、缺陷修复、架构调整和发布相关修改。

### 1. 开始修改前

任何代码或项目规则变更开始前，必须完成：

1. 阅读 `AGENTS.md`、`README.md`、`package.json`、`docs/requirements/README.md`，以及与本次需求直接相关的源码和配置；文件尚不存在时跳过，并在本次需求落地时按需要补齐。
2. 搜索 `docs/requirements/` 中已有需求，按“产品能力归属”判断本次请求是已有需求的延续，还是新的独立用户目标。
3. 执行 `git status -sb`，识别并保留用户已有修改；不得覆盖、回滚、格式化或顺带提交与本次任务无关的改动。
4. 修复可回归验证的缺陷或性能问题时，优先先补充能够稳定复现问题的测试或验证用例，再修改实现；无法自动化测试时，必须记录可重复的人工验证步骤。
5. 不修改、不提交构建和依赖产物，包括但不限于 `.next/`、`out/`、`dist/`、`node_modules/`、缓存目录以及本地临时文件；只有明确作为项目资产管理的生成文件除外。
6. 修改构建、数据生成、PWA 或部署链路时，必须确认不会破坏 `Next.js Static Export → out/ → 又拍云` 这一核心发布路径。

### 2. 需求档案与变动引用

所有会改变**产品行为、交互、数据结构、架构、构建发布方式或项目规则**的请求，都必须维护在 `docs/requirements/` 中。纯拼写修正、无行为变化的格式整理等机械修改可不单独建立需求档案。

#### 2.1 需求归属判断

- 按“产品能力 / 用户目标”判断是否属于同一需求，不按本次请求的措辞判断。
- 同一能力的增强、缺陷修复、性能优化、交互调整、验收变化和边界补充，都归入原需求目录。
- 只有引入新的独立用户目标或新的产品能力时，才创建新需求目录。
- 一次请求同时影响多个既有能力时，可以关联多个需求，但必须分别记录实际影响。
- 无法可靠判断归属时，不得自行新建重复需求；先说明拟归属的需求和理由，再向用户确认。
- 已创建需求的编号和目录是永久标识，后续即使产品名称或需求标题变化也不得重新编号。

#### 2.2 目录与编号

新需求目录统一使用：

```text
docs/requirements/REQ-YYYYMMDD-NNN-short-name/
```

规则：

- 日期使用该需求首次提出的日期。
- `NNN` 为当天从 `001` 开始递增的三位序号。
- `short-name` 使用稳定、简短、可读的英文 kebab-case，不随文案调整频繁改名。
- 新目录必须从 `docs/requirements/_template/` 创建，并维护：
  - `requirement.md`：用户目标、范围、非目标、验收标准。
  - `solution.md`：技术方案、数据结构、关键决策与取舍。
  - `implementation.md`：实际实现、测试、构建、发布状态。
  - `changes.md`：该需求后续所有变动的追加式时间线。

#### 2.3 已有需求的后续改动

重复请求或已有能力的延续不得创建新需求目录。每次修改必须完成对应文档闭环：

1. 在原需求 `changes.md` 中追加一条变动记录，保留用户请求、归属判断、处理结果和验证结果。
2. 产品目标、范围、边界或验收标准变化时，同步更新 `requirement.md`。
3. 技术方案、数据结构、关键依赖或架构取舍变化时，同步更新 `solution.md`，并记录为什么改变。
4. 实际代码、测试、构建或发布状态变化时，同步更新 `implementation.md`。
5. 在 `docs/requirements/README.md` 的“变动时间线”顶部追加本次事件。

需求文档描述的是当前真实状态，`changes.md` 保留历史；不要为了让文档看起来整洁而删除已经发生的关键决策和变更记录。

#### 2.4 变动点引用

- `docs/requirements/README.md` 总时间线中的每条事件，必须链接到对应需求 `changes.md` 的**本次变动标题锚点**，不能只链接到需求目录或 `changes.md` 文件顶部。
- `changes.md` 每条记录使用唯一、可读的二级标题，推荐格式：

```md
## 2026-09-17 调整模板技术路线报价展示
```

- 同一天同一需求发生多次不同变更时，标题必须能够区分，不依赖序号猜测内容。
- 一次请求涉及多个需求时，分别写入各自 `changes.md`，并在总时间线分别建立引用。
- 总时间线按事件时间倒序排列；同一个需求允许出现多次。
- 存在 commit、PR、部署地址、Demo 地址或可下载产物时，应记录在对应变动条目或 `implementation.md` 中。

#### 2.5 状态与完成门禁

需求状态统一使用：

```text
待分析 → 待确认 → 已确认 → 开发中 → 待发布 → 已发布
```

异常状态允许：`已阻塞`、`已取消`。

状态规则：

- 用户明确要求直接实现时，可以从“已确认”进入“开发中”，但仍必须先建立或更新需求档案。
- “代码完成”“测试通过”“构建成功”“已提交”“已推送”“已部署”是不同事实，必须分别如实记录，不得互相替代。
- 代码实现完成但尚未部署时，最多进入“待发布”，不得标记为“已发布”。
- 未实际推送远程仓库，不得记录为“已推送”；未实际部署到可访问环境，不得记录为“已发布”。
- 遇到阻塞时记录阻塞原因、已完成部分和恢复条件，不得用“已完成”掩盖未完成工作。
- **需求文档闭环属于完成标准的一部分。** 代码、测试、构建完成后必须同步维护需求档案，不得留到后续补写。

### 3. 修改后的最低验证要求

根据变更范围执行最小但充分的验证：

- 数据 / Schema：运行对应数据生成和校验脚本。
- UI / 页面：至少验证受影响页面的桌面端和移动端关键路径。
- Motion：验证动画进入、退出、重复触发和 `prefers-reduced-motion` 场景。
- PWA：验证 manifest、Service Worker、安装能力及缓存更新不影响正常网页访问。
- 构建 / 路由 / SEO：至少执行生产构建，确认 Static Export 成功生成 `out/`，并检查受影响静态路由。
- 部署脚本：优先使用 dry-run / 测试目标验证，不得在未确认影响范围时直接执行破坏性同步或删除。

能够自动化的验证优先自动化；无法执行的验证必须在 `implementation.md` 和最终交付说明中明确写出原因，不得默认视为通过。

---

## V1 明确不做

- 用户注册 / 登录
- 收藏同步
- 在线支付
- 订单系统
- 自建 CMS
- 必须常驻的 Next.js / Node.js 服务端
- 为基础原子 UI 重复建设组件库

---

## 最终技术方案

```text
Next.js App Router
+ TypeScript
+ React
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
