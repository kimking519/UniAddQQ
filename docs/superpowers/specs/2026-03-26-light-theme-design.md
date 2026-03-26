# 白天模式主题设计

## 概述

为 QQ 临时会话工具添加白天模式主题，支持手动切换和 localStorage 持久化。

## 需求

- 手动切换深色/白天模式
- 记住用户偏好（localStorage）
- 保持现有深色主题不变

## 技术方案

采用 CSS 属性选择器 `[data-theme="light"]` 覆盖 CSS 变量，通过 `<html data-theme="light">` 切换。

## 实现细节

### 1. CSS 变量覆盖

在现有 `:root` 后添加：

```css
[data-theme="light"] {
  --bg: #ffffff;
  --bg2: #f8f9fa;
  --bg3: #f1f3f5;
  --bg4: #e9ecef;
  --border: rgba(0,0,0,0.06);
  --border2: rgba(0,0,0,0.10);
  --border3: rgba(0,0,0,0.15);
  --text: #1a1a2e;
  --text2: #6b7280;
  --text3: #71717a;  /* 调深以保证对比度 */
  --shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
  --shadow-md: 0 4px 12px rgba(0,0,0,0.08);
  --shadow-lg: 0 8px 24px rgba(0,0,0,0.12);
  --shadow-glow: 0 0 20px rgba(59,130,246,0.15);
  --primary-glow: rgba(59,130,246,0.08);
  --primary-glow-strong: rgba(59,130,246,0.12);
  --success-bg: rgba(34,197,94,0.08);
  --danger-bg: rgba(239,68,68,0.08);
  --warn-bg: rgba(245,158,11,0.08);
}
```

主色调 `--primary` 和语义色 `--success`、`--danger`、`--warn` 保持不变。

### 2. 过渡动画

在 `body` 样式中添加过渡效果，确保主题切换平滑：

```css
body {
  /* 现有样式保持不变 */
  transition: background-color 0.2s ease, color 0.2s ease;
}
```

为其他关键元素也添加过渡：

```css
.card, .tab, input, textarea, select, .btn, .qq-item, .stat-card {
  transition: background-color 0.2s ease, border-color 0.2s ease, color 0.2s ease;
}
```

### 3. HTML 结构变更

**当前结构（约第 76-81 行）：**
```html
<div class="header">
  <div class="header-badge">...</div>
  <h1>...</h1>
  <p>...</p>
</div>
```

**修改后结构：**
```html
<div class="header">
  <div class="header-content">
    <div class="header-badge">...</div>
    <h1>...</h1>
    <p>...</p>
  </div>
  <button class="theme-toggle" onclick="toggleTheme()" title="切换主题" aria-label="切换主题">
    <svg class="icon-sun" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
      <circle cx="12" cy="12" r="5"/>
      <line x1="12" y1="1" x2="12" y2="3"/>
      <line x1="12" y1="21" x2="12" y2="23"/>
      <line x1="4.22" y1="4.22" x2="5.64" y2="5.64"/>
      <line x1="18.36" y1="18.36" x2="19.78" y2="19.78"/>
      <line x1="1" y1="12" x2="3" y2="12"/>
      <line x1="21" y1="12" x2="23" y2="12"/>
      <line x1="4.22" y1="19.78" x2="5.64" y2="18.36"/>
      <line x1="18.36" y1="5.64" x2="19.78" y2="4.22"/>
    </svg>
    <svg class="icon-moon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
      <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"/>
    </svg>
  </button>
</div>
```

### 4. 切换按钮样式

**修改现有 `.header` 样式（约第 77-81 行）：**
```css
.header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 32px;
  padding-bottom: 24px;
  border-bottom: 1px solid var(--border);
}
```

**新增 `.header-content` 和按钮样式：**
```css
.header-content {
  flex: 1;
}

.theme-toggle {
  width: 38px;
  height: 38px;
  border-radius: 10px;
  background: var(--bg3);
  border: 1px solid var(--border2);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
  flex-shrink: 0;
  margin-left: 16px;
}

.theme-toggle:hover {
  background: var(--bg4);
  border-color: var(--border3);
  transform: scale(1.05);
}

.theme-toggle:focus-visible {
  outline: 2px solid var(--primary);
  outline-offset: 2px;
}

.theme-toggle svg {
  width: 18px;
  height: 18px;
  color: var(--text2);
  transition: color 0.2s ease;
}

.theme-toggle:hover svg {
  color: var(--text);
}

/* 深色模式显示太阳图标，白天模式显示月亮图标 */
.icon-moon { display: none; }
[data-theme="light"] .icon-sun { display: none; }
[data-theme="light"] .icon-moon { display: block; }
```

### 5. 特殊状态覆盖

为 `.status-skip` 添加白天模式覆盖：

```css
[data-theme="light"] .status-skip {
  background: rgba(0,0,0,0.03);
  border-color: rgba(0,0,0,0.08);
}
```

### 6. JavaScript 逻辑

在 `<script>` 标签开头添加以下代码：

```javascript
// 主题管理
function initTheme() {
  const saved = localStorage.getItem('qq_theme');
  if (saved === 'light') {
    document.documentElement.setAttribute('data-theme', 'light');
  }
}

function toggleTheme() {
  const html = document.documentElement;
  const isLight = html.getAttribute('data-theme') === 'light';

  if (isLight) {
    html.removeAttribute('data-theme');
    localStorage.removeItem('qq_theme');  // 不存储 'dark'，默认即为深色
  } else {
    html.setAttribute('data-theme', 'light');
    localStorage.setItem('qq_theme', 'light');
  }
}

// 页面加载时初始化主题
initTheme();
```

**说明：**
- 仅当用户选择白天模式时存储 `'light'`
- 切换到深色模式时移除存储，默认行为即为深色
- `initTheme()` 在脚本加载时立即执行

## 文件修改清单

| 位置 | 修改内容 |
|------|---------|
| CSS `:root` 后 | 添加 `[data-theme="light"]` 变量覆盖 |
| CSS `.header` | 修改为 flex 布局，添加 justify-content/align-items |
| CSS 新增 | `.header-content`、`.theme-toggle` 及相关样式 |
| CSS `.status-skip` 后 | 添加白天模式覆盖 |
| CSS `body` | 添加过渡动画 transition |
| HTML header | 包裹内容到 `.header-content`，添加按钮 |
| JS 开头 | 添加主题管理函数并调用 initTheme() |

## 验收标准

- [ ] 点击切换按钮可切换深色/白天模式
- [ ] 刷新页面后主题保持
- [ ] 白天模式配色协调，文字对比度符合 WCAG AA 标准
- [ ] 按钮图标正确切换（深色显示太阳，白天显示月亮）
- [ ] 切换时有平滑过渡动画
- [ ] 键盘 Tab 导航可访问切换按钮，焦点状态清晰