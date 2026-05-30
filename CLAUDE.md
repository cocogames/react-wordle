# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

**React Wordle** — 流行的 Wordle 单词猜谜游戏的克隆版，使用 React 17、TypeScript 和 Tailwind CSS 构建。这是一个 Create React App (CRA) 项目，每日谜题答案基于时间纪元索引生成。

## 常用命令

```bash
npm run start       # 启动开发服务器 (localhost:3000)
npm run build       # 生产环境构建
npm run test        # 运行测试（CRA 监听模式，Jest）
npm run lint        # 检查代码格式（Prettier）
npm run fix         # 自动修复代码格式（Prettier --write）
```

## 架构

### 入口文件

- `src/index.tsx` — 渲染 `<App>`，外层包裹 `<AlertProvider>`（唯一的 React Context）

### 主应用 (`src/App.tsx`)

单页应用，所有游戏状态在顶层管理：
- **游戏状态**: `guesses`（已提交的单词数组）、`currentGuess`、`isGameWon`、`isGameLost`
- **设置**: `isHardMode`、`isDarkMode`、`isHighContrastMode`（持久化到 localStorage）
- **UI**: 弹窗开关状态、当前行动画类名、翻牌动画锁（`isRevealing`）
- **持久化**: 每次猜测后将游戏状态保存到 localStorage；加载时恢复（仅当答案与当天一致时）

### 组件结构

```
src/components/
├── alerts/          — AlertContainer + Alert（通过 AlertContext 实现的 toast 通知）
├── grid/            — Grid, Cell, CompletedRow, CurrentRow, EmptyRow（猜测结果展示）
├── keyboard/        — Keyboard + Key（带颜色状态标记的虚拟键盘）
├── modals/          — BaseModal（基于 @headlessui 的 Dialog）、InfoModal、StatsModal、SettingsModal、SettingsToggle
├── navbar/          — Navbar（标题 + 弹窗图标按钮）
└── stats/           — StatBar, Histogram, Progress（统计数据展示）
```

### 游戏逻辑 (`src/lib/`)

| 文件 | 用途 |
|------|------|
| `words.ts` | 单词验证、困难模式校验、Unicode 分词（grapheme-splitter）、每日答案 `getWordOfDay()` |
| `statuses.ts` | 计算猜测字母状态（正确/位置不对/不存在） |
| `stats.ts` | 加载/保存游戏统计数据（胜负、连胜、分布） |
| `share.ts` | 生成 emoji 网格分享文本 |
| `localStorage.ts` | 加载/保存游戏状态和设置到 localStorage |

### 常量 (`src/constants/`)

| 文件 | 用途 |
|------|------|
| `settings.ts` | `MAX_WORD_LENGTH`（5）、`MAX_CHALLENGES`（6）、时间常量 |
| `strings.ts` | 所有用户界面文本字符串（用于 i18n） |
| `wordlist.ts` | `WORDS` — 有效答案列表 |
| `validGuesses.ts` | `VALID_GUESSES` — 更广泛的合法猜测列表 |

### 每日答案逻辑

答案根据自 2022 年 1 月 1 日以来的天数对 `WORDS` 数组长度取模确定。这意味着同一份词表对所有玩家每天产生相同的答案。

### 环境变量 (`.env`)

- `REACT_APP_GAME_NAME` — 导航栏显示的标题
- `REACT_APP_GAME_DESCRIPTION` — meta 描述
- `REACT_APP_LOCALE_STRING` — 区域设置，用于语言敏感的大小写转换（支持 i18n）
- `REACT_APP_GOOGLE_MEASUREMENT_ID` — Google Analytics（可选）

### 样式

- **Tailwind CSS** 实用类
- **暗色模式** 通过 `<html>` 元素上的 `dark` 类切换
- **高对比度模式** 通过 `<html>` 元素上的 `high-contrast` 类切换
- 自定义 CSS 在 `src/App.css` 和 `src/index.css` 中定义动画（jiggle、bounce）

### 核心依赖

- `@headlessui/react` — 弹窗/对话框组件
- `@heroicons/react` — 图标库
- `grapheme-splitter` — 正确处理非 ASCII 字符的 Unicode 字素分割
- `react-countdown` — 倒计时组件（距离下一题的时间）
- `classnames` — 条件 CSS 类名组合

### 测试

使用 CRA 内置的 Jest + React Testing Library。测试文件为 `src/App.test.tsx`。运行 `npm run test`（监听模式）或 `CI=true npm run test`（单次运行）。
