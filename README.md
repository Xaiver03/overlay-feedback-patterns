# overlay-feedback-patterns

一份给 AI 编码代理和人用的**浮层与反馈选型规范**：18 种 overlay / feedback 组件之间到底怎么选、选完之后该有什么行为。

它要解决的是一句很贵的话——**「加个弹窗」**。这句话把 18 种可能压成 1 种，实现者只能猜，于是要么给高频可撤销操作套上确认框，要么把必须读到的表单错误飘成一个 4 秒就消失的 toast。这个仓库把「猜」换成「算」：先答五条判据轴，落点基本唯一。

内容综合了主流组件库与平台规范的公开约定（Ant Design、MUI、Material 3、shadcn/ui + Radix、iOS `UISheetPresentationController`、Android `ModalBottomSheet`、微信小程序），加上逐条可判的行为规则。**不依赖任何特定组件库**，换栈照样用。

## 里面有什么

```
SKILL.md                          决策层主体：五条判据轴、打断强度 0–5 阶梯、18 类速查、三张硬表、八条常见误用
references/
  taxonomy.md                     18 类逐类详解：定义 / 用在哪 / 禁用在哪 / 各库各端对应 / 无障碍要点
  dismissal-and-timing.md         关闭策略矩阵（20 行）、时长与自动消失规则、堆叠去重、无障碍与 z-index
  destructive-actions.md          危险操作三档判定、撤销成立的 7 条工程前提、不可逆动作的界面义务
  mobile-sheets.md                Bottom Sheet 30/60/80 分档、与 Drawer 的分界、键盘遮挡、Web 套壳的分端策略
  audit-checklist.md              把一个已有产品对表整改：基线探针、整改分档 T1–T8、评审清单、多端核对
  glossary-zh-en.md               中英对照词表 + 提示词模板 + 「模糊说法 → 精确说法」
```

## 核心机器

**五条判据轴** — 信息归属 / 要不要用户决策 / 动作可逆性 / 时效性 / 端。冲突时按 可逆性 > 时效性 > 归属 让位：不可逆的事不能因为「这是系统级回执」就飘成一个 4 秒的 toast。

**打断强度 0–5 阶梯** — 从 Tooltip（0 级，用户发起）到模态（5 级，阻塞一切）。硬约束两条：能低一级就别升一级；同一时刻只允许一个 5 级层。

**关闭策略矩阵** — 逐类规定点背景能不能关、Esc 干什么、会不会自动消失。两条不变量：
- **Esc 永远等价于「取消 / 不执行」**，任何浮层里都不许被绑成确认；
- **点背景关闭 = 丢弃当前内容**，所以已脏表单、危险确认、强制阅读三类一律禁止。

**危险操作三档** — 判据是「做错了要花多少代价恢复」，不是「文案里有没有删除二字」。可撤销 → Snackbar + 撤销；单条不可撤销 → Popconfirm；不可逆且影响面大 → AlertDialog；删主体/工作区/密钥 → 再加输入名称确认。

**撤销的 7 条工程前提** — 软删除、保留期、幂等、无外部副作用、审计留痕、下游可重算、权限一致。**缺一条就不许出现「撤销」二字**，否则就是给用户一个点了什么也恢复不了的假按钮。

## 安装

这是一个 Agent Skill 目录，克隆到你的 skills 发现根即可（目录名保持 `overlay-feedback-patterns`）：

```sh
# Claude Code
git clone https://github.com/Xaiver03/overlay-feedback-patterns.git ~/.claude/skills/overlay-feedback-patterns

# 通用 Agents 目录（Codex / 其他支持 skills 的客户端）
git clone https://github.com/Xaiver03/overlay-feedback-patterns.git ~/.agents/skills/overlay-feedback-patterns
```

没有 skills 机制也能用：把 `SKILL.md` 贴进对话，或让代理直接读 `references/` 下的对应文件。

## 怎么用

**选型**——把场景说清楚，四要素齐了就不会被猜错：

> 这里用 Bottom Sheet，半屏档（约 60%），可上拖到 90%，点背景关闭，下滑关闭；里面是只读详情，不需要确认。

**评审现有界面**：

> 按「打断强度 0–5」给这个页面上每条反馈定级，指出：有没有用 toast 承载必须看到的错误；有没有可撤销操作被做成确认框、或不可逆操作只给了 toast；每个浮层的 scrim / Esc / 自动消失策略分别是什么；移动端从属任务有没有被做成居中弹窗或窄屏抽屉；行内报错会不会自动消失。只报缺陷与证据位置，不要顺手改风格。

**整改一个仓库**：走 `references/audit-checklist.md`——先取基线读数，再按 T1–T8 一档一个纵切做完，别一次改全仓。

## 边界

这个 Skill 只管**该用哪一类、以及它该有什么行为**。以下不归它：

- 视觉风格、配色、字体、圆角阴影
- 单个组件的 API 与结构细节（去看组件库文档）
- 动效时长、缓动曲线、手势跟手
- 授权改代码——它给判据，不给许可

## License

MIT
