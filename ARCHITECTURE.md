# ARCHITECTURE — XDF Feedback Assistant

> 现状架构说明。定位与使用见 [README.md](./README.md)；待办见 [PENDING.md](./PENDING.md)；变更史见 [DEVLOG.md](./DEVLOG.md)。

## 1. 定位

Obsidian 教学反馈撰写辅助插件（id `xdf-feedback-assistant`，`isDesktopOnly: false`）。提供两类入口——**侧边栏面板**与 **Alt+Q 命令选择器**——把预定义的教学反馈模板一键插入到当前编辑器光标处，减少重复输入。

上游蓝本为 [obsidian-markdown-formatting-assistant](https://github.com/Reocin/obsidian-markdown-formatting-assistant-plugin)，仅保留其「点击插入文本」的核心交互。

## 2. 目录结构

```
├── src/
│   ├── main.ts                  插件入口 + 设置页（SettingsTab）
│   ├── SidePanelControlView.ts  侧边栏面板视图（ItemView）
│   ├── CommandListView.ts       Alt+Q 模糊搜索弹窗（SuggestModal）
│   ├── generalFunctions.ts      通用工具函数（当前为死代码）
│   ├── icons.ts                 Ribbon 图标注册
│   ├── templates.ts             数据模型与聚合（feedbackTemplates / getAllItems）
│   ├── templates-class.ts       班级类模板
│   ├── templates-student.ts     学员类模板
│   └── templates-content.ts     教学内容类模板
├── dist/                        构建输出（main.js / manifest.json / styles.css）
├── docs/                        参考文档（模板清单、XMind 结构、教材/讲义）
├── references/                  上游参考（已 gitignore）
├── verify_templates.js          模板与 docs 一致性校验（已失效，见 PENDING）
└── verify_text.js               模板文本校验（已失效，见 PENDING）
```

## 3. 模块职责

| 模块 | 职责 | 关键点 |
|------|------|--------|
| `src/main.ts` | 加载/保存设置；`registerView` 注册侧边栏视图；Ribbon 图标；`Open Command Selector` 命令（Alt+Q）；设置页 | `onunload()` 为空（未 detach 视图，见 PENDING） |
| `src/SidePanelControlView.ts` | 侧边栏面板：顶部 Tab 切分类、子 Tab 切小节；递归渲染 group / subgroup / dimension / item 按钮；点击插入文本 | 含分组标题渲染分支与扁平化分支（当前存在缺陷，见 PENDING） |
| `src/CommandListView.ts` | `CodeSuggestionModal`：全量模板模糊筛选 + 回车插入 | 当前每次按键 `getAllItems()` 全量重建 + `includes` 匹配（见 PENDING） |
| `src/templates.ts` | 定义 `Category / Section / DisplayGroup / SubGroup / EvalDimension / TemplateItem`；聚合三大类模板；`getAllItems()` 展平 | 数据模型入口 |
| `src/templates-class.ts` | 班级类模板（出勤、纪律、课堂氛围、课堂表现、作业、当堂练习） | 区分「整体 / 个人」维度 |
| `src/templates-student.ts` | 学员类模板（出勤、入门测、课堂表现、作业、题型） | 多个小节为「纯分组、无维度」结构 |
| `src/templates-content.ts` | 教学内容类模板（初级/中级教材与讲义，按单元组织） | 长文本字面量，单条可达上千字符 |
| `src/icons.ts` | `addIcons()` 注册 Ribbon 图标 | — |
| `src/generalFunctions.ts` | `checkIfMarkdownSource` | 全项目无引用，死代码 |

## 4. 数据流

```
模板源（templates-*.ts: Category → Section → DisplayGroup → items/dimensions/subgroups）
   │
   ├─ templates.ts 聚合 → feedbackTemplates[] / getAllItems()
   ▼
两类入口
   ├─ SidePanelControlView（侧边栏）—— 按 Tab/Section 逐层渲染按钮，点击 insertText
   └─ CodeSuggestionModal（Alt+Q）—— 按查询过滤 getAllItems()，回车 replaceSelection
   ▼
当前编辑器（MarkdownView.editor）光标处
```

- **插入路径**：侧边栏走 `SidePanelControlView.insertText`（取当前活动编辑器后 `editor.replaceSelection`）；命令选择器走 `Editor.replaceSelection`。
- **无持久化**：插件不产生数据文件；模板全部硬编码在 `templates-*.ts` 中。

## 5. 模板数据模型

```
Category  { id, label, sections: Section[] }
└─ Section  { id, label, groups: DisplayGroup[] }
   └─ DisplayGroup { header, typeName?, items?, dimensions?, subgroups? }
      ├─ items: TemplateItem[]              —— 直接可插入的条目
      ├─ dimensions: EvalDimension[]         —— 「整体/个人」「速度/质量」等维度，各含 items
      └─ subgroups: SubGroup[]               —— 子分组，同样可含 items / dimensions
```

`getAllItems()` 递归展平以上四层，供命令选择器搜索。模板结构调整时**务必对齐层级**，否则会触发侧边栏扁平化分支的渲染缺陷（见 PENDING）。

## 6. 构建与发布

- 构建：`npm run build` = `rollup --config rollup.production.config.js` → `dist/`（`main.js` / `manifest.json` / `styles.css`）。
- 开发：`npm run dev`（rollup watch，输出到本地 Vault 插件目录）。
- 发布：`.github/workflows/release.yml` 由 tag `v*` 触发，校验 `package.json` / `manifest.json` / tag 三者版本一致后构建发布，release notes 自动生成。
- 版本号需一致：`package.json` / `manifest.json` / `versions.json` / release tag。

## 7. 约定

- 模板一律以静态字符串声明在 `templates-*.ts`，最终经 `setText` 输出（无 XSS 风险）。
- 新增/调整模板时同步维护 `docs/feedback_templates.md`，并修复校验脚本后跑一遍。
- 设置项文案与本插件中文定位一致；界面文案随 `UI-REFACTOR` 一并中文化（见 PENDING）。

## 8. 已知技术债（详见 PENDING）

- 侧边栏「纯条目小节」判定过宽 → 分组标题被丢弃（核心可用性缺陷）。
- 侧边栏插入依赖 `getMostRecentLeaf()`，取不到 MarkdownView 时**静默失效**。
- `manifest.json` / `package.json` / `versions.json` 版本号曾不一致（已在 2026-10-16 修复）。
- `verify_templates.js` / `verify_text.js` 因模板拆分后路径与正则失效，形同虚设。
- 按钮/样式构建代码在 `SidePanelControlView` 中四处重复。
