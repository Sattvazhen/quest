# ALOKA Quest — Apple Style 重构规范（唯一执行依据）

> 目标：在**不破坏任何现有功能与数据链路**的前提下，把 14 个静态页面统一到 Apple 设计语言，
> 消除外链字体/CDN、玻璃拟态、发光阴影、圆角混乱、动效过载与可访问性缺陷。
> 本文件是改造期间所有子任务必须遵守的基准；数值以 `design-tokens.md` 为准，不得另造一套。

## 0. 项目事实（已核实，禁止改动）

- 本地源码根目录：`C:\Users\Administrator\Doubao\chats\2026-10-29\new-chat-4\quest-main`
- 技术栈：纯静态多页站点，每页内联 `<style>` 与 `<script>`，无构建步骤、无框架。
- 后端：Supabase，URL `https://nhjkhtrgehznhdlqyybs.supabase.co`，anon publishable key
  `sb_publishable_BFkX70SHXwbSmNzE_h7AQg_a1OwKkO_`；localStorage 键 `aloka_sb_token`。
  **这些常量、URL、key、localStorage 键、hash 参数名（access_token / type / recovery / signup）一字不改。**
- 已本地化的第三方库：`daily/app/vendor/html2canvas.min.js`（1.4.1，MIT），供 `daily/app/index.html` 导出截图使用。
- 14 个页面（相对 `quest-main`）：

| 分组 | 文件 | 性质 |
|---|---|---|
| 根落地/通知 | `index.html` | 服务停用通知（中英双语） |
| 认证 | `verify.html` | 邮箱验证回调 |
| 认证 | `reset-password.html` | 重置密码回调 |
| 法务 | `privacy.html` | 隐私政策（繁体+英文，长文） |
| 法务 | `service.html` | 服务条款（繁体+英文，长文） |
| 应用主框架 | `daily/app/index.html` | 日记时间线 + 新建向导 + 导出 |
| 应用社交 | `daily/app/message.html` | 会话列表 + 聊天房间 |
| 应用社交 | `daily/app/moment.html` | Moment 动态流 + 新建向导 |
| 应用社交 | `daily/app/radar.html` | Radar 匹配雷达 |
| 应用个人 | `daily/app/my/index.html` | 个人主页 + 资料编辑弹窗 |
| 应用设置 | `daily/app/my/settings/index.html` | 设置列表 |
| 应用设置 | `daily/app/my/settings/safe/index.html` | 账户与安全列表 |
| 应用设置 | `daily/app/my/settings/safe/cancel.html` | 注销账户确认 |
| 应用设置 | `daily/app/my/settings/safe/change-email-address.html` | 更换邮箱 |

## 1. 不可触碰清单（功能保全契约）

以下内容**只允许保持原样**，视觉改造不得修改其行为、ID、类名、函数名、链接路径与文案语义：

1. 所有 `getElementById('...')` 引用的元素 ID（见 §6 钩子清单）。
2. 所有 `classList.add/remove/toggle` 操作的类名：`open`、`selected`、`view-hidden`、`active`、`liked`、`show-actions`、`idle`。
3. 所有内联 `onclick="..."` 调用的函数名与参数；不得删除 `triggerHaptic()` 调用。
4. Supabase 的 `fetch` 调用、请求方法、headers、body 字段名、跳转路径、`confirm()`/`alert()` 文案逻辑。
5. 页面之间的相对导航 `href`（`index.html`、`radar.html`、`moment.html`、`message.html`、`my/index.html`、`../`、`../../`、`../../../`）。
6. 法务页 `privacy.html` / `service.html` 的**正文条款文字逐字保留**，只允许改 `<title>`、样式与标题层级，不得改写、缩写、翻译或替换其中的法律表述。
7. SVG sprite 中 `id="icon-*"` 的定义与 `<use href="#icon-*">` 引用关系。
8. 温度状态色逻辑（`< 0 / <10 / <20 / <28 / <35 / 其他` 的文案与图标颜色分支）保留；仅把其中 `#007AFF` 替换为 `var(--accent)`，其余状态色（冷蓝/橙/红）作为 ≥24px 图形状态色保留。

## 2. 必须删除/替换的外部引用（fatal，0 容忍）

1. **删除全部 Google Fonts**：每个页面 `<head>` 里的三行 `preconnect` + `stylesheet`（`fonts.googleapis.com` / `fonts.gstatic.com`）。字体一律改用系统字体栈（见 §3）。
2. `daily/app/index.html` 第 11 行 CDN 脚本：
   ```html
   <script src="https://cdnjs.cloudflare.com/.../html2canvas.min.js"></script>
   ```
   改为相对本地路径：
   ```html
   <script src="./vendor/html2canvas.min.js"></script>
   ```
   并在 `downloadEntry()` 开头加防护：`if (typeof html2canvas === 'undefined') { alert('导出组件未加载'); return; }`。
3. `daily/app/my/index.html` 中 Unsplash 外链 `<img>`（`images.unsplash.com/...`）删除位图，改为纯 CSS 中性占位（`background: var(--surface-3)`，1px `--line`，无外链、无 `url(http...)`）；保留该 `<img>` 的 `id="display-banner"` 与 `alt`，JS 仍会设置其 `src`（用户上传后生效）。
4. 禁止新增任何 `https://` 字体、图片、图标、CDN、`@import`、`url(http...)`。
   允许保留的运行时网络调用仅限：Supabase 域名、`api.ipify.org`、`functions/v1/get-client-ip`、用户头像 `avatar_url`、`alokaquest.top` 等既有业务跳转。

## 3. 统一设计令牌（逐页复制到 `<style>` 顶部 `:root`）

把各页原有 `:root` 整段替换为下面这一套；**保留旧变量名作为别名**，使内联样式与 JS 生成的样式继续生效。

```css
:root {
    /* ---- Apple 设计令牌（唯一数值来源） ---- */
    --font-sans: -apple-system, BlinkMacSystemFont, "SF Pro Display", "SF Pro Text",
        "Helvetica Neue", "PingFang TC", "PingFang SC", "Hiragino Sans GB",
        "Microsoft YaHei", "Noto Sans TC", "Noto Sans SC", Arial, sans-serif;

    --ink-1: #1d1d1f; --ink-2: #424245; --ink-3: #6e6e73; --ink-4: #86868b;
    --line: rgba(0, 0, 0, 0.10);

    --surface-1: #ffffff; --surface-2: #fbfbfd; --surface-3: #f5f5f7;
    --surface-invert: #060607; --on-invert-1: #f5f5f7; --on-invert-2: #a1a1a6;
    --line-invert: rgba(255, 255, 255, 0.16);

    --accent: #0071e3; --accent-hover: #0077ed; --accent-press: #0068cf;
    --accent-wash: rgba(0, 113, 227, 0.06); --accent-on-dark: #2997ff;

    --error-color: #ff3b30; --success-color: #34c759;

    --radius-xs: 6px; --radius-sm: 10px; --radius-md: 18px;
    --radius-lg: 28px; --radius-pill: 999px;

    --ease-out: cubic-bezier(0.32, 0.72, 0, 1);
    --ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);
    --dur-fast: 200ms; --dur-base: 320ms; --dur-reveal: 640ms;
    --size-hit: 44px;

    --safe-top: env(safe-area-inset-top, 20px);
    --safe-bottom: env(safe-area-inset-bottom, 20px);

    /* ---- 旧变量别名（保留，禁止删除） ---- */
    --bg-paper: var(--surface-1);
    --bg-sub: var(--surface-3);
    --text-dark: var(--ink-1);
    --text-primary: var(--ink-1);
    --text-secondary: var(--ink-2);
    --text-muted: var(--ink-3);
    --line-light: var(--line);
    --apple-blue: var(--accent);
    --apple-blue-hover: var(--accent-hover);
    --apple-blue-light: var(--accent-wash);
    --radius: var(--radius-md);
    --ease-curve: var(--ease-out);
    --font-en: var(--font-sans);
    --font-tc: var(--font-sans);
    --glass-bg: rgba(255, 255, 255, 0.78);
}

[data-theme="dark"] {
    --bg-paper: #000000; --bg-sub: #1c1c1e;
    --text-dark: #f5f5f7; --text-muted: #8e8e93;
    --line-light: rgba(255, 255, 255, 0.12);
    --apple-blue-light: rgba(41, 151, 255, 0.14);
    --glass-bg: rgba(28, 28, 30, 0.78);
}
```

### 3.1 基础重置与全局样式（每页 `<style>` 内紧随令牌之后）

```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

html { -webkit-text-size-adjust: 100%; }

body {
    font-family: var(--font-sans);
    background: var(--surface-1);
    color: var(--ink-1);
    -webkit-font-smoothing: antialiased;
    text-align: left;
    letter-spacing: 0.01em;
    overflow-x: clip;
}

:lang(zh), :lang(zh-TW), :lang(zh-Hant), :lang(zh-CN) { letter-spacing: 0.01em; }

:focus-visible {
    outline: 2px solid var(--accent);
    outline-offset: 3px;
    border-radius: 2px;
}

@media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
        scroll-behavior: auto !important;
    }
}
```

## 4. 全局视觉规则（对照 audit.mjs 的 fatal/warning）

### 4.1 颜色
- 全页唯一饱和强调色 `--accent #0071e3`；现有 `#007AFF` 全部改为 `var(--accent)`。
- 底色最多 3 种：`--surface-1` 白、`--surface-3` 浅灰、`--surface-invert` 黑（本应用不新增黑场区块）。
- 禁止紫蓝/粉紫/霓虹色；禁止任何双饱和色标 `linear-gradient`；页面背景不使用渐变。
- 13px 及以下小字用 `--ink-2`，不得用 `--ink-4` 承载关键信息。

### 4.2 阴影与模糊（白名单）
- 允许 `box-shadow` 的只有：吸顶/吸底导航、模态层、浮层菜单。卡片、列表项、头像一律**无阴影**，层次靠 1px `--line` 与底色差。
- `backdrop-filter` **只允许**出现在选择器匹配 `header` 或 `nav` 的元素上（`.app-header`、`.room-header` 可以）。
  - `.bottom-tab-bar`：删除 `backdrop-filter`，背景用 `--glass-bg` 纯色半透明 + 顶部 1px `--line`。
  - `.modal-overlay` / 弹窗遮罩：删除 `backdrop-filter: blur(12px)`，保留 `background: rgba(0,0,0,0.4)`。
  - 其余任何卡片、气泡、浮层不得使用 `backdrop-filter`。
- 删除 `index.html` 主按钮的蓝色投影 `box-shadow: 0 4px 12px …` 与 hover 加深投影。

### 4.3 圆角（语义递增，尽量收敛到 3 种）
- 胶囊按钮 / 头像：`--radius-pill`（头像 `50%` 圆形作为语义特例保留）。
- 卡片 / 弹窗 sheet / 分组容器：`--radius-md`（18px）。
- 输入框 / 小控件 / 标签：`--radius-sm`（10px）。
- 清理现有 4 / 8 / 12 / 14 / 16 / 20 / 24 混用：16px 输入框 → 10px；20px 卡片 → 18px；24px sheet 顶部 → 18px；12px 小按钮 → 10px。

### 4.4 动效
- 只动 `transform` 与 `opacity`；悬停只允许换色或位移 ≤2px，**禁止** `scale(1.02)`、`scale(0.95)`、`scale(0.9)` 放大缩小。
- 删除 `animation … infinite`：
  - `radar.html` 的雷达扫掠/脉冲循环改为静态呈现（圆环 + 静态 blip 位置），不写无限循环；脉冲点用静态状态色。
  - `message.html` 的加载 spinner 循环改为静态占位（或一次性淡入），不写 `infinite`。
- 入场只保留一种 `translateY(24px) + opacity`，`--dur-reveal`；现有多套 `drawLine / iconSpring / appStageIn` 入场动画统一收敛为这一种，或直接移除装饰性入场。
- `transition: all 0.4s …` 改为明确属性（`background-color`、`color`、`border-color`、`transform`）。

### 4.5 布局与视口
- 现有 `height: 100vh` 全部改为 `100dvh`；`min-height: 100vh` 改为 `min-height: 100dvh`。
- 应用壳保持移动端优先、`max-width: 600px` 单列居中；≥600px 两侧 1px `--line` 描边保留。
- 正文左对齐；法务长文 `max-width: 692px`，行高 1.6，不居中段落墙。

### 4.6 可访问性
- 每个页面必须有且仅有一个 `<h1>`；标题层级不跳级（h1 → h2 → h3）。现有 `div.brand-title` / `div.brand-subtitle` 改为 `<h1 class="brand-title">` + `<p class="brand-subtitle">`。
- 主内容包一层 `<main id="main">`；页面顶部加 skip link：
  ```html
  <a class="skip-link" href="#main">跳到主要内容</a>
  ```
  ```css
  .skip-link { position: absolute; left: -9999px; top: 0; z-index: 30; padding: 12px 16px; background: var(--surface-invert); color: var(--on-invert-1); }
  .skip-link:focus { left: 16px; top: 16px; }
  ```
- 所有图标按钮（无文字）必须带 `aria-label` 或 `title`。
- 表单输入框必须有可见 `<label for="...">`（现有仅 placeholder 的补上可见标签，视觉上可用小号 `--text-caption`）。
- 触控命中区 ≥ 44px 高；现有 40px 圆形按钮补到 44px。
- 所有 `<img>` 必须有 `alt`。
- 页面 `<title>` 格式：`产品名 — 一句话`，去掉营销词与多余装饰。

### 4.7 文案
- 删除营销腔与感叹号；不使用 audit 禁用词清单（赋能/极致/引领/全新/无缝/一站式/重新定义等）。
- 法务正文例外：逐字保留，不改写；如 audit 在 `service.html` 正文命中 `旨在` 等法律用语，在审计报告中登记为「法务原文，豁免改写」。
- `index.html` 修正繁简混杂：「您進入」改为通顺简体中文；英文副标去掉营销腔，保持陈述句。

## 5. 逐文件改造清单

> 每个文件改完必须：删除字体外链、替换 §3 令牌、按 §4 规则清理样式、补齐 §4.6 可访问性、运行 `audit.mjs` 到 **0 fatal**。

### 5.1 `index.html`（服务通知页）
- 品牌图标保留内联 SVG；删除按钮阴影与 hover `scale(1.02)`。
- 一个实心主按钮（`.btn-primary` 胶囊，`--accent`，44px，白字）；次按钮由描边实心胶囊降级为**文字链接 + chevron SVG**（`.link-more`），不再是第二个等权按钮。
- `#86868b` 的 16px 英文正文提升到 `--ink-2`（对比度达标）。
- 标题用 `<h1>`；补 `<main>`、skip link、`:focus-visible`、reduced-motion；`100vh` → `100dvh`。
- 修正 `您進入` 病句与 `title` 中的 "Minimalist Service Update" 营销后缀。

### 5.2 `verify.html` / `reset-password.html` / `change-email-address.html`（认证页同构）
- 删除字体外链；替换令牌；`.brand-title` 升级为 `<h1>`；补 `<main>`、skip link、focus-visible、reduced-motion。
- 输入框：补可见 `<label>`；`border-radius:16px` → `--radius-sm`；`outline:none` 移除，用 `:focus-visible` 与 `:focus` 边框 `--accent`。
- 主按钮 `border-radius:16px` → `--radius-pill`；删除 `:active { transform: scale(0.95) }`，改为 `:active { background: var(--accent-press) }`。
- `height:100vh; overflow:hidden` → `100dvh`；保留 `[data-theme="dark"]` 别名映射。
- `reset-password.html` / `change-email-address.html` 保留全部 Supabase 调用与状态切换逻辑。

### 5.3 `privacy.html` / `service.html`（法务长文）
- 删除字体外链；替换令牌；正文条款逐字保留。
- `.header` 标题不再强制居中（单行 h1 可居中，段落不居中）；容器 `max-width: 692px`；正文 `--ink-1/2`，15px。
- 现有 `prefers-color-scheme: dark` 与 `[data-theme="dark"]` 映射统一到 §3 别名。
- 补 `<main id="main">`、skip link、`:focus-visible`、reduced-motion、`<h1>` 唯一。
- `service.html` 正文命中的禁用词属法务原文，登记豁免，不改动句子。

### 5.4 `daily/app/index.html`（日记主框架）
- 替换 CDN 为 `./vendor/html2canvas.min.js`，加 `typeof html2canvas === 'undefined'` 防护。
- 删除字体外链；统一令牌；`.bottom-tab-bar` 去 blur；`.modal-overlay` 去 blur(12px)。
- 向导表单：`.form-input` 圆角收敛到 `--radius-sm`；`.option-btn` 选中态用 `--accent-wash` 底 + `--accent` 描边；按钮组保持 1 个 `.btn-primary` 实心 + `.btn-secondary` 文字/弱描边。
- 多套入场动画（`drawLine/iconSpring/appStageIn`）统一删除或收敛为一种 `translateY+opacity`。
- 浮动新建按钮 56px 圆形保留，`active` 不做 scale，只换 `--accent-press`。
- 保留全部 ID（见 §6）、Supabase 读写、`export-render-card` 导出结构与内联 SVG sprite。
- 补 `<main id="main">`、skip link、图标按钮 `aria-label`、reduced-motion、focus-visible、`100vh`→`100dvh`。

### 5.5 `daily/app/message.html`
- 删除字体外链；统一令牌；`.app-header` / `.room-header` 保留 blur；`.bottom-tab-bar` 去 blur。
- 删除加载 spinner 的 `infinite` 动画；聊天气泡无阴影、1px `--line` 或 `--surface-3` 底色；发送态 `.sent` 用 `--accent` 白字，接收态 `--surface-3`。
- 替换正文中的 `⚠` emoji 为 12px 内联 SVG 警示图标（或删除该字符）。
- 保留 `chat-list / chat-room / reply-banner / send-btn / chat-input / room-avatar / room-name / reply-text-content / message-container` 等 ID 与 `show-actions`、`view-hidden`、`open` 类。
- 补 `<main>`、skip link、aria-label、focus-visible、reduced-motion、`100vh`→`100dvh`。

### 5.6 `daily/app/moment.html`
- 删除字体外链；统一令牌；移除两张 moment 卡片的 `box-shadow`；`.modal-overlay` 去 blur。
- 其余与 5.4 向导同构规则一致；保留 `feed-container`、`like-btn`/`liked`、`moment-*` 类与全部 ID。
- 补 `<main>`、skip link、aria-label、focus-visible、reduced-motion、`100vh`→`100dvh`。

### 5.7 `daily/app/radar.html`
- 删除字体外链；统一令牌；删除所有 `animation … infinite`（扫掠、脉冲循环），改为静态圆环与静态 blip。
- 删除 `radar-blip` 的发光 `box-shadow: 0 0 10px 4px rgba(0,122,255,.2)`；用静态 `--accent` 小圆点。
- `.bottom-tab-bar` 去 blur；保留 `match-container`、`radar-scan-view`、`scan-status`、`status-dot`、`idle`、`view-hidden` 钩子。
- 补 `<main>`、skip link、aria-label、focus-visible、reduced-motion。

### 5.8 `daily/app/my/index.html`
- 删除 Unsplash 外链位图；banner 改为 CSS 中性占位，保留 `display-banner` ID 与 alt。
- 删除字体外链；统一令牌；`.modal-overlay` 去 blur；资料卡片无阴影、1px `--line`。
- 补 `<h1>`（个人主页标题，可放在 `.profile-name` 之上的语义标题）、`<main>`、skip link、`<img>` alt、aria-label、focus-visible、reduced-motion、`100vh`→`100dvh`。
- 保留全部 `display-*`、`input-*`、`avatar-preview-img`、`banner-preview-img`、`edit-modal`、`save-btn`、`tag-option`/`selected` 钩子。

### 5.9 `daily/app/my/settings/index.html` 与 `safe/index.html`
- 删除字体外链；统一令牌；列表项 1px 分隔线、无阴影；卡片圆角 `--radius-md`；`100vh`→`100dvh`。
- 退出/危险操作按钮红色 `--error-color` 保留语义；补 skip link、focus-visible、reduced-motion、图标 aria-label。

### 5.10 `safe/cancel.html`
- 删除字体外链；输入框补可见 label；危险按钮圆角 `--radius-pill`，去掉 `scale(0.98)`；保留 Supabase DELETE 逻辑与 `confirm-input / btn-cancel / status-msg`。

## 6. JS 钩子保全清单（改造前后逐一核对存在）

| 文件 | 必须保留的 ID / 类 |
|---|---|
| daily/app/index.html | IDs: btn-save-story, custom-mood-input, custom-temp-input, custom-weather-input, detail-content-area, detail-modal, dynamic-thermometer, ex-date, ex-loc, ex-loc-wrapper, ex-mood, ex-mood-wrapper, ex-story, ex-temp, ex-temp-wrapper, ex-weather, ex-weather-wrapper, export-render-card, live-clock, location-input, nickname-input, story-blocks, story-main-title, temp-custom-toggle, temp-slider, temp-text-status, temp-val, timeline-container, wizard-modal；类: open/selected/view-hidden, loc-btn |
| daily/app/message.html | IDs: chat-input, chat-list, chat-room, message-container, reply-banner, reply-text-content, room-avatar, room-name, send-btn；类: empty-state, msg-container-outer, show-actions, open, view-hidden |
| daily/app/moment.html | IDs: btn-save-story, custom-mood-input, custom-temp-input, custom-weather-input, dynamic-thermometer, feed-container, live-clock, location-input, nickname-input, story-blocks, story-main-title, temp-custom-toggle, temp-slider, temp-text-status, temp-val, wizard-modal；类: active, liked, open, selected, view-hidden, loc-btn, modal-sheet, wizard-step |
| daily/app/my/index.html | IDs: avatar-preview-img, banner-preview-img, display-avatar, display-banner, display-bio, display-ip, display-location, display-name, display-tags-str, edit-modal, input-avatar-file, input-banner-file, input-bio, input-custom-tags, input-location, input-name, save-btn；类: tag-option, open, selected |
| daily/app/radar.html | IDs: match-container, radar-scan-view, scan-status, status-dot；类: idle, view-hidden |
| settings/index.html | 函数 handleLogout；类 settings-group/settings-item/settings-label/settings-label-sub/chevron-icon |
| safe/cancel.html | IDs: btn-cancel, confirm-input, status-msg |
| change-email-address.html | IDs: btn-update, confirm-email, new-email, status-msg |
| reset-password.html | IDs: btn-reset, new-password, status-msg |
| verify.html | IDs: btn-home, status-message |

## 7. 验收标准（DoD）

1. 每个 HTML 文件运行：
   ```
   node <skill>/scripts/audit.mjs <file.html>
   ```
   **fatal = 0**；warning 尽可能清零，无法消除的（头像圆形半径、法务原文禁用词、业务状态色）在报告中逐条登记原因。
2. 全仓库 `grep`：不再出现 `fonts.googleapis.com`、`fonts.gstatic.com`、`cdnjs.cloudflare.com`、`images.unsplash.com`。
3. 所有既有 ID、函数名、Supabase 常量、导航相对路径与 §6 清单逐项仍存在且拼写一致。
4. 无 `backdrop-filter` 出现在非 header/nav 选择器；无 `animation … infinite`；无 `scale()` 悬停/按下放大；无卡片 `box-shadow`。
5. 375px / 768px / 1440px 宽度下布局不破；`prefers-reduced-motion` 生效；Tab 键可走完主要操作，焦点可见。
6. 不新增任何构建步骤、框架、运行时依赖；产出仍为可直接双击打开的静态文件。

## 8. 已知豁免（在审计报告中如实登记，不算失败）

- `--error-color` 红、`--success-color` 绿用于危险/成功语义按钮与状态，属第二功能色，仅限控件级。
- 温度图标状态色（冷蓝/橙/红）为图形状态色，保留分支逻辑。
- 法务页正文中 audit 识别到的法律惯用词（如「旨在」）属合同原文，不做营销化改写。
- 头像 `border-radius: 50%` 为圆形语义，计入圆角特例。
- `daily/app/vendor/html2canvas.min.js` 为本地第三方运行时库（已脱离 CDN），不计为外链。
