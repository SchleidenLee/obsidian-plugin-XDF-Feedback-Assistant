# PENDING — XDF Feedback Assistant

> 待办清单。分区：**⚪ 待定**（方案未定）/ **🟡 已定待做**（方案已定，等动手）/ **🟢 进行中**。
> 完成一项后：从本文件**删除**该条 → 在 [DEVLOG.md](./DEVLOG.md) 追加一行 → 现状变了就更新 [ARCHITECTURE.md](./ARCHITECTURE.md)。

> 注：下方「代码审查整改」来自 2026-10-04 静态代码审查，行号基准为当日源码快照；动工时以实际代码为准。
> 该审查未发现 XSS / eval / 注入类安全风险（所有模板为静态字符串，最终经 `setText` 输出）。

---

## 🟢 进行中

（无）

---

## 🟡 已定待做

### A. 代码审查整改（严重 / 高）

- [ ] **纯条目小节的「分组标题」被错误丢弃（核心可用性缺陷）** — `src/SidePanelControlView.ts:133-160`
  - 现象：`drawContent` 用 `section.groups.every((g) => g.items && g.items.length > 0 && !g.subgroups && !g.dimensions)` 判定「纯条目小节」，一旦某小节所有 group 都只有 items，就进入扁平化分支把 items 直接铺进一个 flex 容器，**完全跳过 `group.header` 与 `typeName` 的渲染**（头部渲染只在 `drawGroup` 的 `hasSubgroups || hasDimensions` 分支里发生，见 `SidePanelControlView.ts:177-203`）。
  - 影响：`stu-performance`（`templates-student.ts:38-64`，3 组）、`stu-mastery`（`298-337`，4 组）、`stu-improve`（`338-364`，5 组）的分组标题全部不渲染，几十个无语义分隔的按钮混排，小节实际不可用。
  - 方案：扁平化判定改为「所有 group 都无 header 可显示时才扁平化」，或放弃扁平化、统一走 `drawGroup`；至少要在扁平化分支里为每个 group 渲染一次 header。

- [ ] **侧边栏插入依赖 `getMostRecentLeaf()`，取不到编辑器时静默失效** — `src/SidePanelControlView.ts:356-363`
  - 现象：`insertText` 取 `this.app.workspace.getMostRecentLeaf()`，仅当 `leaf.view instanceof MarkdownView` 才插入。点击侧边栏时「最近 leaf」很可能是侧边栏自身，`if` 直接跳过——**既不插入也不提示**。
  - 影响：按钮点了没反应，用户无法判断是没点中还是插件坏了。
  - 方案：改用 `getActiveViewOfType(MarkdownView)` 或缓存「最后一次活动的 MarkdownView」；取不到时 `new Notice('未找到可插入的编辑器')`。

- [ ] **版本号三处不一致** — `manifest.json:4` / `package.json:3` / `versions.json`
  - 状态：**已于 2026-10-16 修复**（见 DEVLOG）；保留此条仅作记录，若再次不一致可参照处理。
  - 方案：统一三处版本号并补 `versions.json` 条目。

### A. 代码审查整改（中）

- [ ] **Alt+Q 弹窗渲染超长模板文本且每次按键重算** — `src/CommandListView.ts:11-33`、`src/templates.ts:48-77`
  - 现象：每个候选项用 `textDiv.setText(item.text)` 渲染整段模板正文（单条可达上千字符）；`getSuggestions` 每次按键都 `getAllItems()` 全量重建再 `includes` 过滤。
  - 影响：弹窗卡顿、行高失控；`includes` 是子串匹配而非模糊搜索，跳字符输入搜不到。
  - 方案：建议行只渲染 `item.label`（正文改截断预览）；`getAllItems()` 结果模块级缓存；搜索换用子序列模糊匹配。

- [ ] **两个校验脚本已失效** — `verify_templates.js:3-7`、`verify_text.js:3-7`
  - 现象：两者仍读 `src/templates.ts`，但模板已拆分到 `templates-class/student/content.ts`，`templates.ts` 只剩聚合导出；正则也无法匹配含转义引号/换行的长文本。
  - 影响：读不到条目，比对形同虚设。
  - 方案：改读三个模板文件或 `import` 编译产物；正则支持转义与跨行，或解析 TS AST / `ts-node` 导入后遍历。

- [ ] **按钮/样式构建代码大量重复** — `src/SidePanelControlView.ts:143-159、229-249、312-325、328-354`
  - 现象：「创建按钮 div → 逐条设置样式 → appendText → 绑定 `insertText`」四处近乎逐字复制。
  - 影响：样式/交互调整需同步改 4 处，易漏改。
  - 方案：抽 `createActionButton(container, item, styleVariant?)` 统一复用。

- [ ] **生命周期清理缺失 + 设置项体验/本地化不一致** — `src/main.ts:57`、`src/main.ts:105-117`
  - 现象：`onunload() {}` 为空（未 `detachLeavesOfType`）；设置项用 `addText` 手输 `left`/`right`，标题/名称仍英文，与中文定位不一致。
  - 方案：`onunload` 中 `detachLeavesOfType(SidePanelControlViewType)`；设置项改 `addDropdown`（左侧/右侧）、文案中文化、使用设置面板推荐标题写法。

- [ ] **`loadSettings` 直接改写默认常量对象** — `src/main.ts:22-24`、`src/main.ts:60`
  - 现象：`Object.assign(DEFAULT_SETTINGS, await this.loadData())` 把读到的数据写进了模块级常量 `DEFAULT_SETTINGS`。
  - 方案：改为 `Object.assign({}, DEFAULT_SETTINGS, await this.loadData())`。

- [ ] **依赖归类不当与无用依赖** — `package.json:17-36`
  - 现象：`obsidian` 放在 `dependencies`（应属 `devDependencies`，rollup 已 external）；`postcss`、`rollup-plugin-import-css`、`rollup-plugin-postcss` 在配置与源码中均未被引用。
  - 方案：`obsidian` 移入 `devDependencies`；删除未使用的三个依赖。

### A. 代码审查整改（低）

- [ ] **`checkIfMarkdownSource` 缺 null 保护且已成死代码** — `src/generalFunctions.ts:3-7` → 补 `leaf?.view` 判空，或直接删除该函数及文件。
- [ ] **`drawItemButton` 靠全局类名查找容器，布局脆弱** — `src/SidePanelControlView.ts:328-354` → 由 `drawSubGroup` 显式传入/创建行容器并按需换行。
- [ ] **视图 toggle 未判空** — `src/main.ts:71-84` → 对 leaf 与数组首元素判空后再调用。
- [ ] **弹窗选择后未回收焦点** — `src/CommandListView.ts:35-37` → 插入后 `this.editor.focus()`。
- [ ] **数据结构可读性** — `src/templates.ts:15-34` 及相关模板文件 → 收敛结构，或为长文本抽独立字符串常量。
- [ ] **`nodemon.json` 硬编码 Linux 路径** — `nodemon.json:14-15` → 移除或改跨平台脚本，或标注仅 Linux 使用。
- [ ] **缺少 LICENSE 文件** — 根目录 → 补充与 `package.json` 一致的 MIT LICENSE 文件。

### B. 设置页 UI 改造

现状问题：无 Banner（直接 `h2` 英文标题）、只有一个设置项、无版本/作者信息、界面英文。

- [ ] 添加 Banner（图标 📝 + 标题 `XDF Feedback Assistant` + 副标题 `反馈速写助手`）。
- [ ] 界面中文化：`Side Pane Side` → `侧边栏位置`，`Choose on which side the Side Pane appears.` → `选择侧边栏出现在左侧还是右侧`。
- [ ] 添加 Footer（`v0.1.2 · Schleiden`）。
- [ ] 侧边栏位置改为下拉选择（与审查中「设置项体验」条目合并处理）。
- 改动文件：`src/main.ts`（Banner/Footer/中文化）、`styles.css`（`.xdf-banner` / `.xdf-footer` 样式）。

---

## ⚪ 待定

（无）
