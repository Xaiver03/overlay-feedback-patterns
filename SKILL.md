---
name: overlay-feedback-patterns
description: 浮层与反馈组件的选型决策层。当要在 Modal/Dialog、AlertDialog、Drawer 抽屉、Bottom Sheet 底部面板、Popover、Tooltip、Toast 轻提示、Snackbar 消息条、Notification、Alert 警告条、Inline Alert 行内报错、Banner、Progress/Skeleton/Spinner 之间做选择、定关闭策略（点背景能不能关 / Esc / 自动消失）、设计删除等危险操作的确认与撤销、或评审一版界面里浮层用错用乱时使用。触发词：弹窗、弹层、浮层、遮罩、模态、对话框、确认框、抽屉、侧边栏、底部弹层、半屏、气泡、吐司、轻提示、消息条、撤销、回退、报错、提示、反馈。English: overlay, modal, dialog, alert dialog, drawer, side sheet, bottom sheet, snackbar, toast, popconfirm, tooltip, inline error, dismissal, undo.
metadata:
  short-description: 18 种浮层与反馈组件的选型判据、关闭策略与危险操作规范
---

# 浮层与反馈选型（Overlay & Feedback）

这份 Skill 管的是**该用哪一类、以及它该有什么行为**，不是视觉样式。
它补的是组件词典之上的决策层：组件词典告诉你 Modal 有哪些最佳实践，这里回答「这个场景到底该不该用 Modal」。

## 边界

- 视觉风格、配色、字体、圆角阴影 → 交给通用的 UI 风格类 Skill（若装了 `ui-ux-pro-max`、`design-taste-frontend` 之类）；没装也不影响本 Skill 的判定。
- 单个组件的 API 与结构细节 → 组件词典或该组件库文档（antd / MUI / Material 3 / shadcn-Radix / iOS / Android / 小程序各自的官方文档）。
- 弹层动效（进出时长、缓动、手势跟手）→ 交给动效类 Skill（如 `motion-design`）。这里只定**该不该出现、能不能关、留多久**。
- 本 Skill 不授权改代码。落到某个仓库前，先按 `references/audit-checklist.md` 取现状基线，再决定改哪一档。

## 一、五条判据轴

任何一条反馈/浮层，先答这五个问题，答案基本唯一。

| 轴 | 取值 | 决定什么 |
|---|---|---|
| **① 信息归属** | 控件 / 区域 / 页面 / 任务 / 系统 | 决定落点：控件级用 Tooltip·Popover·Inline Alert；区域级用 Alert·Card 内提示；任务级用 Modal·Drawer·Sheet；系统级用 Toast·Snackbar·Notification |
| **② 要不要用户决策** | 不需要 / 需要但不阻塞 / 必须表态 | 不需要 → 被动或轻反馈；需要不阻塞 → Popconfirm·Alert；必须表态 → Modal·AlertDialog |
| **③ 动作可逆性** | 可撤销 / 有代价 / 不可逆 | 可撤销 → Snackbar + 撤销，别弹确认；有代价 → Popconfirm；不可逆且不可撤销 → AlertDialog（且禁点背景、禁 Esc 确认） |
| **④ 时效性** | 错过无所谓 / 必须能看到 | 错过无所谓 → 自动消失（Toast/Snackbar）；必须能看到 → **永不自消**（Inline Alert·Alert·Modal） |
| **⑤ 端** | 移动 / 平板 / 桌面 | 移动优先 Bottom Sheet 与全屏；桌面可用 Drawer、右下 Snackbar、hover 类组件；Tooltip 在触屏上必须有替代 |

判据冲突时按 ③ > ④ > ① 让位：**不可逆的东西不能因为「信息属于系统级」就飘成一个 4 秒的 toast。**

## 二、打断强度阶梯（0–5）

**能低一级就别升一级。** 这是控制「弹窗疲劳」的唯一有效手段。

| 级 | 形态 | 谁发起 | 阻塞范围 | 成员 |
|---|---|---|---|---|
| 0 | 被动浮层 | 用户 hover/聚焦 | 无 | Tooltip、Hover Card、Popover |
| 1 | 环境反馈 | 系统，随进度 | 无 | Skeleton、Progress、Spinner、按钮 loading |
| 2 | 轻反馈 | 系统，事后 | 无（可错过） | Toast、Snackbar、Notification |
| 3 | 就地提示 | 系统，绑定字段/区域 | 该字段/该区域 | Inline Alert、Field Error、区域 Alert |
| 4 | 页面提示 | 系统 | 阅读，不阻塞操作 | Alert、Banner、Result、Empty State |
| 5 | 模态 | 系统或用户 | 页面其余一切 | Modal、AlertDialog、Drawer、Bottom Sheet、Command |

两条硬约束：

1. **同一时刻只允许一个 5 级层。** 禁止 Modal 套 Modal、Drawer 上叠 AlertDialog。第二个改用第 3/4 级就地承接。
2. **升到 5 级必须给出口**，且出口是「取消/不执行」。唯一的形似例外是危险确认：它不给关闭 X，但两个动作按钮都是显式表态，Esc 等价于其中安全的那一个。

## 三、18 类速查

逐类详解（定义 / 何时用 / 何时禁用 / 组件库对应 / 无障碍）见 `references/taxonomy.md`。

| # | 英文 | 中文 | 一句判据 |
|---|---|---|---|
| 1 | Tooltip | 文字提示 | 解释「这个控件是什么」，用户 hover 才出现，不可点 |
| 2 | Hover Card | 悬停卡片 | 预览「这个引用是什么」，可含富文本，仍不可点操作 |
| 3 | Popover | 气泡卡片 | 锚定控件的轻量内容/工具，不阻塞外部 |
| 4 | Menu / Dropdown | 菜单 | 一组动作的收纳，选择即关闭 |
| 5 | Modal / Dialog | 模态弹窗 | 一个**独立子任务**，完成或放弃才回主页 |
| 6 | AlertDialog | 确认型模态 | **不可逆或高代价**动作前的强制表态 |
| 7 | Popconfirm | 气泡确认框 | 就地、轻量、可反悔的二次确认（删除一行、退出填写） |
| 8 | Drawer / Side Sheet | 侧边抽屉 | 桌面/平板上的**从属详情或筛选**，与主页并存感 |
| 9 | Bottom Sheet | 底部面板 | 移动端的从属任务/详情，拇指可达，可拖拽换档 |
| 10 | Full-screen Overlay | 全屏浮层 | 移动端需要独占注意力的**重任务**（编辑器、图片查看、支付） |
| 11 | Toast | 轻提示 | 一次性结果回执，纯文本，自动消失 |
| 12 | Snackbar | 消息条 | 结果回执 **+ 一个动作**（撤销 / 重试 / 查看） |
| 13 | Notification | 通知提醒框 | 与当前操作**无关**的异步消息，可带标题/正文/跳转 |
| 14 | Alert | 警告提示 | 页面/区域内**持续成立**的状态说明，不自消 |
| 15 | Inline Alert / Field Error | 行内报错 | 绑定某个输入，**必须改对才能继续**，永不自消 |
| 16 | Banner | 横幅通告 | 整页级别的运营/系统/权限状态，可关闭且关过不再打扰 |
| 17 | Progress | 进度反馈 | 有**已知总量**的耗时操作，要给出百分比或阶段 |
| 18 | Skeleton / Spinner | 加载占位 | 无已知总量；有内容形状用 Skeleton，瞬时操作用按钮内 Spinner |

第 19 类 Command / 命令面板（⌘K）只在桌面键盘流里成立，按第 5 级模态对待、但**必须允许 Esc 直退**。

## 四、三张硬表

### 4.1 关闭策略矩阵

完整表（含理由与例外）在 `references/dismissal-and-timing.md`。核心两条不变量：

- **Esc 永远等价于「取消 / 不执行」。** 任何浮层里，Esc 不得触发确认、保存、提交、扣款、删除。
- **点背景（scrim）关闭 = 丢弃当前内容**，所以只有「无未保存内容」或「丢弃无代价」时才允许。表单已脏、危险确认、强制阅读（协议/告知）三类一律禁。

### 4.2 时长与自动消失

- Toast 成功 4s；失败 6–8s；**带任何动作按钮 → 不自消或 ≥10s，且用户 hover/聚焦时暂停计时**。
- Snackbar 带撤销：≥7s，撤销动作在计时结束前一直可点；**若这条消息是失败原因，改成不自消的 Alert 或 Inline Alert**。
- Inline Alert / Alert / Banner / Modal / AlertDialog / Popconfirm / Drawer / Sheet：**自动消失 = 缺陷**。
- 同屏并发：桌面 ≤3 条、移动 ≤2 条；同类消息合并并计数（「3 条保存失败」），不要叠 3 条同样文案。

### 4.3 危险操作三档

判据是**「做错了要花多少代价恢复」**，不是「文案里有没有『删除』二字」。完整决策树与撤销的工程前提（软删除、幂等、审计留痕、外部副作用不可撤销）在 `references/destructive-actions.md`。

| 档 | 成立条件 | 该用什么 |
|---|---|---|
| 轻 | 可撤销、代价 ≤ 一次点击 | **Snackbar + 撤销**，不弹确认 |
| 中 | 不可撤销但影响范围只有这一条记录 | **Popconfirm**，或输入名称确认（高危对象） |
| 重 | 不可逆 + 影响面大（批量、跨工作区、对外发出、扣钱、删库、已提交的申报） | **AlertDialog**：标题写清后果、正文写清范围、主按钮用 destructive 样式并复述动词（「永久删除 12 条」），禁点背景、禁 X、Esc = 取消 |

## 五、常见误用（这八条覆盖了绝大多数真实返工）

1. **把「确认删除」当默认安全垫。** 能用软删除 + 撤销解决的，弹确认框只是把成本转嫁给每次正常操作。高频操作尤其亏。
2. **让行内报错自动消失。** 表单提交失败只弹一个 5 秒 toast，用户抬头时错误已经没了、表单还在——这是最容易做错的第 4 个灵魂问题。错误必须留在出错的字段下方，直到该字段被改动。
3. **危险确认可点背景关闭。** 点背景 = 取消 = 安全，看似没问题；但一旦某个实现把 scrim 关闭绑成「确认」或「提交」，就成了静默扣款路径。规范上直接禁止危险确认响应 scrim。
4. **Esc 绑到主按钮。** 键盘用户的「取消」反射会被执行成「确认」。
5. **移动端用桌面 Drawer。** 侧边抽屉在窄屏上要么压成全屏、要么宽度失控；移动端从属任务的正确形态是 Bottom Sheet（或全屏浮层），见 `references/mobile-sheets.md`。
6. **Bottom Sheet 只有一个高度。** 没有分档的半屏面板在「瞄一眼」和「长时间操作」之间来回全屏，是抖音/地图那类列表详情最容易被做坏的地方。
7. **Tooltip 承载必须看到的信息。** 触屏没有 hover；读屏器不一定播报；用户截图时它不在画面里。
8. **一个时刻叠两层模态。** 第二层出现时第一层的焦点、滚动锁定、Esc 归属全部失控。

## 六、向 AI 描述需求

用户侧最常见的痛点是「沟通时瞎比划」。中英对照词表 + 可直接粘贴的提示词模板（含关闭策略、时长、危险档位的显式写法）在 `references/glossary-zh-en.md`。一句可用的形状：

> 这里用 Bottom Sheet，半屏档（约 60%），可上拖到 90%，点背景关闭，Esc 关闭；里面是只读详情，不需要确认。

## 七、落地到某个仓库

1. 跑 `references/audit-checklist.md` §1 的探针，取现状基线（有几套 toast、多少处原生 `confirm()`、有没有统一危险入口、行内报错走不走 toast、遮罩/Esc 有没有约定）。
2. 按 §2 的整改分档表选**一档**做完，不要一次改全仓：一档 = 一个可验证的纵切（例如「全部原生 confirm() 收敛到统一危险入口」）。
3. §3 是可打勾的评审清单，改完逐条自查；能机械化的判据写成契约测试，别停在文档里。
4. 多端产品（Web + iOS/Android 壳 + 小程序）按 §4 分端核对：同一句话在触屏上必须有非 hover 的替代。
