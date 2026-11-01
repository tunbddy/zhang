# 张家诚 · 全球科技与 AI 导航门户（NEXUS）

面向国际用户的科技网站导航门户：把国外主流的 AI 工具、硬件厂商、软件平台与开发者资源，
收进一个可搜索、可筛选的沉浸式单页站点。React + Vite 实现，无后端依赖，无重型 UI 库。

品牌与服务信息集中在 `src/data/profile.js`（姓名、英文名、首字标记、简介、统计、社交链接），
导航栏、首屏、关于我、页脚与 `index.html` 标题都从该文件取数。

## 快速开始

```bash
npm install
npm run dev
```

打开终端提示的地址（默认 http://localhost:5173）即可预览。

生产构建与本地预览：

```bash
npm run build
npm run preview
```

> 本项目同样支持 pnpm：`pnpm install && pnpm dev`。

## 发布部署

1. 把 `.env` 里的 `VITE_SITE_URL` 改成正式域名（带协议、结尾不带斜杠），社交分享卡片会使用绝对地址；
   留空时退回相对路径 `/og-cover.png`。
2. 执行构建，产物在 `dist/`：

```bash
pnpm build      # 或 npm run build
pnpm preview    # 本地预览生产产物（默认 http://localhost:4173）
```

3. 把 `dist/` 作为静态站点发布即可。各平台配置：

| 平台 | 构建命令 | 输出目录 | 备注 |
| --- | --- | --- | --- |
| Vercel | `pnpm build` | `dist` | 自动识别 Vite，无需额外配置 |
| Netlify | `pnpm build` | `dist` | 同上 |
| Cloudflare Pages | `pnpm build` | `dist` | 同上 |
| GitHub Pages | `pnpm build` | `dist` | 仓库子路径部署时需在 `vite.config.js` 设置 `base: '/<仓库名>/'` |

页面内的跳转全部是 `#锚点`，没有前端路由，因此**不需要配置 SPA 回退重写规则**。

发布前已做的优化：vendor 按 react / motion / gsap 分块（首屏并行下载、二次访问命中缓存）、
生产构建剔除 `console`/`debugger`、首屏 poster 预加载、补齐 Open Graph 与 Twitter 卡片、
`apple-touch-icon`、`robots.txt`。分享图与主屏图标由 `scripts/generate-social-assets.py` 生成，
品牌信息变更后可重新执行该脚本。

## 目录结构

```
public/assets/         首屏 poster、项目封面、头像等静态视觉资源（SVG）
src/
  App.jsx              整体布局与全局滚动渐显（IntersectionObserver）
  index.css            设计令牌（色板 / 模糊值 / 间距 / 圆角）与通用类
  components/          Navbar、Hero、Sites、About、Projects、Strengths、Footer
  data/                静态数据：sites、categories、projects、strengths、profile、site-config
  hooks/               useScrollReveal、useCountUp、useActiveSection
```

## 五个核心模块

| 模块 | 文件 | 说明 |
| --- | --- | --- |
| 顶栏导航 | `components/Navbar.jsx` | 固定顶部毛玻璃导航，滚动高亮当前板块，窄屏折叠菜单 |
| 首屏 Hero | `components/Hero.jsx` | 全屏视频背景 + 极光遮罩，SplitText 标题入场、RotatingText 关键词轮播 |
| 网站导航 | `components/Sites.jsx` | 53 个国外站点，四类筛选 + 实时搜索，卡片 ⇄ 紧凑列表视图切换 |
| 关于我 | `components/About.jsx` | 左图右文，头像环形动效、统计数字滚动、邮箱与社交入口 |
| 精选项目 | `components/Projects.jsx` | 双列大卡片，封面缩放 + 卡片上浮 + 悬停详情 |
| 核心优势 | `components/Strengths.jsx` | 四列卡片，进入视口依次渐显 |
| 联系方式 | `components/Footer.jsx` | 整屏深色渐变，大号欢迎语、邮箱按钮与社交图标阵列 |

## React Bits 组件与动效

`src/components/reactbits/` 下是从 [reactbits.dev](https://reactbits.dev) 引入的 JS + CSS 变体源码，
按官方 registry 原样落盘（仅 `SquishSwitch` 增加了 track `children`、`SpotlightCard` 增加 rAF 节流与
透传 props 两处最小改动）：

| 组件 | 依赖 | 用在哪里 |
| --- | --- | --- |
| SquishSwitch | motion | 网站导航区的「卡片 ⇄ 紧凑列表」视图开关（品牌渐变轨道 + 轨道内图标） |
| SplitText | gsap + @gsap/react | Hero 主标题第一行逐字入场 |
| RotatingText | motion | Hero「覆盖 … 四大方向」关键词轮播 |
| ShinyText | motion | 各章节 eyebrow 英文标签的扫光（经 `components/ShinyLabel.jsx` 统一封装） |
| SpotlightCard | 无 | 站点卡片与项目卡片的指针跟随聚光（rAF 节流，触屏不绑定） |
| ClickSpark | 无 | 全站点击火花：`canvasMode="fixed"` 让画布只占视口尺寸，页面上万像素高也不会爆显存 |
| Noise | 无 | 全站颗粒层，固定定位、`mix-blend-mode: overlay`、`pointer-events: none` |

视图开关默认关闭（卡片视图），不跨访问记忆；
`prefers-reduced-motion: reduce` 下 SplitText 不拆分、关键词轮播停止、扫光与颗粒静止、视图切换无动画。

## 设计约束

- 版心宽度 1700px（`--container`），在 1280px–1920px 之间保持稳定显示。
- 毛玻璃统一走令牌：`--glass-blur: 30px` + `--glass-saturate: 185%` 做磨砂与提色，
  `--glass-bg`（内部高光渐变 + 深色底衬）保证文字对比度，`--glass-sheen` + `.glass::after`
  负责 1px 镜面描边；`.ambient-glass-field` 是玻璃背后那层固定的彩色光场。
- 交互动画只使用 `transform` 与 `opacity`，时长 300ms–600ms，避免触发重排。
- 首屏视频背景通过 `src/data/site-config.js` 的 `heroVideo` 配置；
  留空时使用 poster 图片 + CSS 极光图层，视频加载失败也不会影响首屏完整性。

## 如何增删站点

编辑 `src/data/sites.js`，按现有结构追加对象即可（分类取自 `src/data/categories.js`）：

```js
{
  id: 'example',
  name: 'Example',
  url: 'https://example.com',
  initial: 'EX',
  category: 'ai',            // ai | hardware | software | dev
  tags: ['标签一', '标签二'],
  description: '一句话介绍。',
  accent: ['#7AA2FF', '#4FE3C1'],
  featured: true,            // 可选：标记为精选
}
```
