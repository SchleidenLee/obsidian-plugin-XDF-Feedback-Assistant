# Obsidian Feedback Assistant

一个用于**教学反馈撰写**的 Obsidian 插件。通过侧边栏面板或快捷键命令选择器，一键插入预定义的反馈文本片段，大幅减少重复输入，让老师可以专注于反馈内容本身。

> 本插件基于 [obsidian-markdown-formatting-assistant](https://github.com/Reocin/obsidian-markdown-formatting-assistant-plugin) 改写而来，仅保留其"点击插入文本"的核心交互，感谢原作者的工作。

## 功能特性

### 侧边栏面板

通过左侧 Ribbon 图标打开面板，面板按三大类组织反馈模板：

| 分类 | 内容 |
| ---- | ---- |
| **班级** | 出勤、纪律、课堂氛围、课堂表现、作业情况、当堂练习（区分整体 / 个人维度） |
| **学员** | 出勤、入门测（速度 / 质量）、课堂表现（参与度 / 认真程度 / 独立练习）、作业（提交状态 / 做题习惯 / 作业态度）、题型（填空、判断、选择等细分） |
| **教学内容** | 初级教材、初级讲义、中级教材、中级讲义（按单元组织的课程内容条目） |

- 顶部 Tab 切换分类，子 Tab 切换小节
- 分组标题、维度标签（如"整体 / 个人"、"速度 / 质量"）均可点击插入对应文字
- 点击任意条目按钮，将对应的完整反馈语句插入到当前编辑器光标处

### 快捷键命令选择器

默认快捷键 `ALT+Q` 打开搜索弹窗。匹配方式是名称与正文的**大小写不敏感子串**（`includes`），不是跳字模糊搜索；回车插入到光标位置。

可在 Obsidian 的快捷键设置中自定义快捷键。

## 安装

1. 下载最新的 Release 中的 `main.js`、`manifest.json`、`styles.css`
2. 在你的 Vault 中创建文件夹：`.obsidian/plugins/xdf-feedback-assistant/`
3. 将三个文件放入该文件夹
4. 在 Obsidian 设置 → 第三方插件中启用 **XDF Feedback Assistant**

## 开发

```bash
# 安装依赖（仓库使用 pnpm-lock.yaml，没有 npm lockfile）
pnpm install

# 开发模式（监听文件变化，直接输出到本地 Vault 的 .obsidian/plugins/ 目录）
pnpm run dev

# 生产构建（输出到 dist/：main.js、manifest.json、styles.css）
pnpm run build
```

## 项目结构

```
├── src/                     # 插件源码
│   ├── main.ts              # 入口：视图注册、快捷键、设置
│   ├── SidePanelControlView.ts  # 侧边栏面板 UI
│   ├── CommandListView.ts   # Alt+Q 子串搜索弹窗
│   ├── templates.ts         # 模板数据模型与聚合
│   ├── templates-class.ts   # 班级类模板
│   ├── templates-student.ts # 学员类模板
│   ├── templates-content.ts # 教学内容类模板
│   ├── generalFunctions.ts  # 通用工具函数
│   └── icons.ts             # Ribbon 图标
├── dist/                    # 生产构建输出（main.js / manifest.json / styles.css）
├── docs/                    # 参考文档
│   ├── feedback_templates.md    # 反馈模板清单（与 src 模板对照校验）
│   ├── xmind_templates.md       # 模板结构（从 XMind 还原）
│   ├── 初级教材.md / 初级讲义.md
│   └── 中级教材.md / 中级讲义.md
├── verify_templates.js      # 校验 TS 模板与 MD 文档条目一致
└── verify_text.js           # 校验 TS 模板与 MD 文档文本一致
```

## 设置

- **Side Pane Side**：界面仍是英文文本框，填写 `left` 才把侧边栏放到左侧，其他值都当右侧。默认右侧。
- **持久化有缺陷，尚未修复**：`loadSettings` 把读到的数据写进模块级 `DEFAULT_SETTINGS`，不是复制到新对象。文本框显示 `left` / `right`，但保存值是布尔 `sidePaneSideLeft`，重开设置页时文本框不会按布尔值回写成 `left` 或 `right`。版本号三处不一致已在 2026-10-16 修复，与本项无关。

## License

[MIT](LICENSE)
