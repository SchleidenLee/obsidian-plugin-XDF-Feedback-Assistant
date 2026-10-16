# XDF Feedback Assistant UI 改造方案

## 状态：📋 待改造

## 现状问题

- ❌ 没有 Banner，直接 `h2` 标题（英文）
- ❌ 只有一个设置项（Side Pane Side）
-  没有版本/作者信息
- ❌ 界面是英文

## 改造方案

### 1. 添加 Banner

```html
<div class="xdf-banner">
  <div class="xdf-banner-icon">📝</div>
  <div>
    <h1 class="xdf-banner-title">XDF Feedback Assistant</h1>
    <p class="xdf-banner-desc">反馈速写助手</p>
  </div>
</div>
```

### 2. 翻译为中文

- "Side Pane Side" → "侧边栏位置"
- "Choose on which side the Side Pane appears." → "选择侧边栏出现在左侧还是右侧"
- "Enter left or right" → "输入 left 或 right"

### 3. 添加 Footer

```html
<div class="xdf-footer">
  <span class="xdf-version">v0.1.2</span>
  <span class="xdf-author">· Schleiden</span>
</div>
```

## 改动文件

- `src/main.ts` — 添加 Banner/Footer，翻译为中文

## 预估工作量

- Banner/Footer：~20 行 HTML
- 翻译：~5 行
