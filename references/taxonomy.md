# 18 类逐类详解

每类给：定义 → 用在哪 → 禁用在哪 → 各组件库/端的对应实现 → 无障碍与工程要点。
组件库缩写：**antd** = Ant Design（Web）、**MUI** = Material-UI、**M3** = Material 3 规范、**shadcn** = shadcn/ui + Radix、**iOS** = UIKit/SwiftUI、**Android** = Compose/Material、**MP** = 小程序（Taro/原生）。

---

## 第 0 级：被动浮层（用户发起）

### 1. Tooltip 文字提示
- **定义**：hover/聚焦才出现的短文本，解释**这个控件是什么**，不可点、不含动作。
- **用在哪**：图标按钮（无文字标签）、截断文本补全、术语释义、快捷键提示。
- **禁用**：任何「必须看到才能完成任务」的信息；任何需要点击的内容；正文级说明。
- **对应**：antd `Tooltip` · MUI `Tooltip`（触屏需 `describeChild`/常显替代）· shadcn `Tooltip` · iOS **无原生**（用长按菜单或常驻标签）· Android `Tooltip`/`RichTooltip`（M3 1.1+，长按触发）· MP 无原生。
- **要点**：延迟出现 300–500ms、离开即消；必须同时支持键盘聚焦触发（`aria-describedby`）；触屏产品**不能只靠 Tooltip 传达信息**；出现位置不得遮挡被解释的对象。

### 2. Hover Card 悬停卡片
- **定义**：hover 出现的富文本预览，解释**这个引用是谁/是什么**（人名、链接、术语）。
- **用在哪**：@提及的人卡、外链的目标预览、单号/ID 的摘要。
- **禁用**：需要用户在其中操作；移动端（无 hover）。
- **对应**：shadcn `HoverCard` · antd 用 `Popover trigger=hover` · MUI 用 `Popper` 自组 · iOS/Android/MP 无原生 → 降级为点击进详情。
- **要点**：卡内可聚焦内容时必须允许鼠标移入卡片而不消失（`openDelay`/`closeDelay`）；否则只放不可交互内容。

### 3. Popover 气泡卡片
- **定义**：锚定在某控件的轻量浮层，放**次级内容或工具**，不阻塞外部操作。
- **用在哪**：筛选器、日期选择、颜色选择、格式工具、行内详情速览。
- **禁用**：需要用户表态才能继续（那是 Modal/Popconfirm）；内容长到需要滚动（那是 Drawer/Sheet）。
- **对应**：antd `Popover` · MUI `Popover` · shadcn `Popover` · iOS `UIPopoverPresentationController`（iPad 适配，iPhone 常退化）· Android `Popup`/`DropdownMenu` · MP 自绘。
- **要点**：点外部关闭 + Esc 关闭是默认预期；焦点进入、归还触发元素；不得与触发元素脱开（滚动跟随或一起关闭）。

### 4. Menu / Dropdown 菜单
- **定义**：一组动作的收纳容器，**选择即关闭**。
- **用在哪**：行内更多操作、批量操作、导航下拉。
- **禁用**：动作超过 ~7 项（改 Drawer/新页面）；含表单输入（改 Popover）；破坏性动作混在常规项里（要么分区+destructive 样式，要么升级为确认）。
- **对应**：antd `Dropdown` · MUI `Menu` · shadcn `DropdownMenu` · iOS `UIMenu` · Android `DropdownMenu` · MP `wx.showActionSheet`。
- **要点**：键盘 ↑↓ + 首字母跳转；危险项与常规项之间要有分隔线且颜色区分；触屏目标 ≥44pt。

---

## 第 1 级：环境反馈（不请求决策）

### 17. Progress 进度反馈
- **定义**：**已知总量**的耗时操作，给百分比或阶段。
- **用在哪**：上传、导出、批量处理、生成文档。
- **禁用**：总量未知（用 Skeleton/Spinner，别造假百分比）；小于 1s 的操作（会闪）。
- **对应**：antd `Progress` · MUI `LinearProgress`（`variant=determinate`）· shadcn `Progress` · iOS `UIProgressView` · Android `LinearProgressIndicator` · MP `progress-bar`。
- **要点**：不确定态（indeterminate）只在真的不知道进度时用；给出**可取消**出口；`aria-valuenow` 或 `role=progressbar`。

### 18. Skeleton / Spinner 加载占位
- **定义**：内容尚未到达的形状占位（Skeleton）或纯忙碌指示（Spinner）。
- **用在哪**：首屏/局部刷新用 Skeleton（保留布局，防跳动）；按钮内提交中用 Spinner + 文案变化（「保存中…」）。
- **禁用**：整页 Spinner 遮罩（除非真的全局阻塞）；用 Skeleton 假装加载而内容结构未知。
- **对应**：antd `Skeleton`/`Spin` · MUI `Skeleton`/`CircularProgress` · shadcn `Skeleton` · iOS `UIActivityIndicatorView`/红杉 shimmer · Android 同 · MP `skeleton`。
- **要点**：按钮的 loading 态必须**同时禁用该按钮**并保留宽度（防文字跳动）；`prefers-reduced-motion` 下换静态样式；`aria-busy` 标在容器上。

---

## 第 2 级：轻反馈（可错过）

### 11. Toast 轻提示
- **定义**：一次性结果回执，纯文本（可带图标），自动消失，不打断操作。
- **用在哪**：保存成功、复制成功、已同步、离线已入队。
- **禁用**：需要用户行动；失败原因需要被读到（→ Inline Alert/Alert）；含动作（→ Snackbar）；多条同类堆叠。
- **对应**：antd `message` · MUI 无（用 Snackbar）· shadcn 用 `sonner` · iOS 无原生（自绘 HUD）· Android `Toast`（系统级，不可靠）· MP `wx.showToast`（**只支持文本/图标，不支持动作**）。
- **要点**：成功 4s / 失败 6–8s；同屏桌面 ≤3、移动 ≤2；同类合并计数；位置固定（移动顶中，桌面右上或右下，**不要随页面变**）；`role=status` + `aria-live=polite`。

### 12. Snackbar 消息条
- **定义**：轻反馈 **+ 一个动作**（撤销 / 重试 / 查看）。Material 的核心贡献。
- **用在哪**：删除后给撤销、失败后给重试、完成后给「查看」。
- **禁用**：两个以上动作（→ 页面/Notification）；动作是「确认」而非「反悔」；不可逆动作的事后回执（撤销是假的）。
- **对应**：MUI `Snackbar`（`action`）· Android `Snackbar`/`SnackbarHost` · antd 用 `notification` 或 `message` 的 `duration:0 + btn`（形似而语义不同）· shadcn `sonner` 的 `action` · iOS 自绘底部条 · MP 需自绘。
- **要点**：**有动作时 ≥7s 且 hover/聚焦暂停计时**；出现在拇指可达处（移动底部、桌面左下）；不得遮挡输入框与键盘；撤销必须真的可撤销（见 `destructive-actions.md`）。

### 13. Notification 通知提醒框
- **定义**：与当前操作**无关**的异步消息，带标题/正文/跳转，通常停留更久或需手动关。
- **用在哪**：后台任务完成、他人评论/审批到达、系统维护公告、跨页面事件。
- **禁用**：用户刚点的那个按钮的直接结果（那是 Toast）；必须立即处理（那是 AlertDialog）。
- **对应**：antd `notification` · MUI 无 · shadcn 无官方 · 桌面 Web `Notification API` · iOS `UNUserNotificationCenter` · Android 通知栏 · MP 订阅消息。
- **要点**：可折叠成中心（列表 + 未读数），避免堆满屏幕；点击要有明确落点；关闭后按用户意图不再打扰（持久化「已读」）。

---

## 第 3–4 级：就地与页面提示（不自消）

### 14. Alert 警告提示
- **定义**：页面或区域内**持续成立**的状态说明（信息/警告/错误/成功四态）。
- **用在哪**：额度将满、数据不同步、某功能受限、批量操作结果汇总、表单区顶部错误汇总。
- **禁用**：一次性回执（→ Toast）；需要强制决策（→ AlertDialog）；每个字段级错误（→ Inline Alert）。
- **对应**：antd `Alert` · MUI `Alert`（+`AlertTitle`）· shadcn `Alert` · MP 自绘 view。
- **要点**：`role=alert` 只在**错误且需立即知晓**时用，其余用 `role=status`；可关闭的 Alert 要记住关闭状态；图标+文字双通道（不能只靠颜色区分严重度，色盲）。

### 15. Inline Alert / Field Error 行内报错
- **定义**：绑定某个输入控件的错误/帮助文本，**必须改对才能继续**。
- **用在哪**：表单校验失败、字段格式错、服务端返回的字段级错误、必填未填。
- **禁用**：跨字段/全局错误（→ 顶部 Alert 汇总 + 滚动到第一个错误）；用 Toast 代替（**这是最常见的严重误用**）。
- **对应**：antd `Form.Item` 的 `help`/`validateStatus` · MUI `FormHelperText error` · shadcn `FormMessage` · iOS 自绘 label · Android supporting text · MP 自绘。
- **要点**：**永不自消**，直到该字段被改动并重新校验通过；错误文案给「怎么改」不是「不合法」（「税号应为 18 位，当前 15 位」优于「格式错误」）；`aria-invalid=true` + `aria-describedby` 指到错误文本；提交失败时滚动并聚焦第一个出错字段；成功态不要长期占用（改了就去掉）。

### 16. Banner 横幅通告
- **定义**：整页级、跨模块的状态条（运营、系统维护、权限、合规告知）。
- **用在哪**：需要所有用户看到且持续成立的事。
- **禁用**：单个操作的结果；用户无法处理的事（要么给出口要么别放）；不可关闭且长期驻留（会形成盲区）。
- **对应**：M3 `Banner` · antd/MUI 用 `Alert` + 全宽样式 · 自绘。
- **要点**：可关闭 + 关闭后持久化不再出现（除非状态变化）；一条页面同时最多一条 Banner；不得挤掉主内容首屏。

---

## 第 5 级：模态（阻塞一切）

### 5. Modal / Dialog 模态弹窗
- **定义**：一个**独立子任务**，用户完成或放弃后才回到主页上下文。
- **用在哪**：短表单（新建/编辑一条记录）、需要聚焦阅读的选择、预览确认。
- **禁用**：内容需要滚动很久（→ 页面或 Drawer）；只是补充信息（→ Popover/Alert）；高频且可撤销的操作（别加模态摩擦）。
- **对应**：antd `Modal` · MUI `Dialog` · shadcn `Dialog` · iOS `sheet`/`formSheet` · Android `AlertDialog`/`Dialog` · MP `wx.showModal`（仅文案+两按钮）或自绘。
- **要点**：宽度用档位不用自由值（移动满屏减边距，桌面 400/560/720/960）；标题是动词短语；焦点陷阱 + 关闭后归还焦点；**有未保存改动时 scrim/Esc 必须先问「放弃修改？」**；`aria-modal=true` + `role=dialog`；背景滚动锁定；禁止 Modal 套 Modal。

### 6. AlertDialog 确认型模态
- **定义**：**必须表态**才能继续的模态，用于不可逆或高代价动作。
- **用在哪**：永久删除、扣款、清空、覆盖、对外发出、批量不可逆、注销工作区。
- **禁用**：可撤销的动作（→ Snackbar + 撤销）；常规二次确认（→ Popconfirm）；用原生 `confirm()`/`alert()`（样式失控、移动端体验破口、无法放后果说明）。
- **对应**：MUI `AlertDialog` · shadcn `AlertDialog`（Radix，**不响应 scrim**）· antd `Modal.confirm({okType:'danger'})` · iOS `UIAlertController` + `.destructive` 动作 · Android `AlertDialog` with confirm/dismiss · MP 自绘。
- **要点**：标题写**后果**（「永久删除这 12 条记录？」），正文写**范围与不可逆性**；主按钮用 destructive 样式并复述动词，禁用「确定/OK」这种空洞文案；**初始焦点落在「取消」**；禁 scrim 关闭、禁 X；Esc = 取消；超危对象要求**输入名称确认**（见 `destructive-actions.md`）。

### 7. Popconfirm 气泡确认框
- **定义**：就地、轻量、单点的二次确认，锚定在触发它的元素上。
- **用在哪**：删除一行、移除成员、退出未保存的填写、开关一个有风险的功能。
- **禁用**：需要说明复杂后果（→ AlertDialog）；含多个选项或输入；触屏窄屏（气泡无处安放，降级为 AlertDialog 或 Snackbar 撤销）。
- **对应**：antd `Popconfirm` · MUI 无官方（Popover + 两个 TextButton 组合）· shadcn 无官方（Popover/AlertDialog 组合）· iOS/Android 无原生等价物 · MP 无。
- **要点**：两个动作都是显式的（「删除」/「取消」）；点外部 = 取消；标题问句 + 一句后果说明；同一时刻只允许一个 Popconfirm。

### 8. Drawer / Side Sheet 侧边抽屉
- **定义**：从屏幕边缘滑入的**从属面板**，桌面/平板上可与主内容并存。
- **用在哪**：详情查看（列表 → 详情不丢列表位置）、筛选与排序、导航、辅助编辑（字段面板）。
- **禁用**：需要强制决策（→ AlertDialog）；主任务本身（那是页面）；窄屏手机（→ Bottom Sheet 或全屏）。
- **对应**：MUI `Drawer`（`temporary`/`persistent`/`permanent`）· antd `Drawer` · shadcn `Sheet` · M3 `Side Sheet` · iOS iPad `UISplitViewController` · Android `ModalNavigationDrawer` · MP 自绘。
- **要点**：宽度按内容定（详情 320–480，表单 480–640，再宽就该是页面）；`temporary` 才遮罩，`persistent` 在桌面可保留背景可操作；Esc + scrim 关闭是默认预期；关闭后焦点回列表行；**详情型抽屉里不要放不可逆动作而不给确认**。

### 9. Bottom Sheet 底部面板
- **定义**：从底部升起的移动优先面板，可拖拽换档，拇指可达。
- **用在哪**：移动端筛选、分享、动作列表、地图路线卡、评论半屏、详情预览。
- **禁用**：桌面（宽度失控、无拖拽预期）；需要长时间输入且键盘会遮挡（→ 全屏页）；需要强制表态（→ AlertDialog）。
- **对应**：M3 `Modal Bottom Sheet` / `Standard Bottom Sheet` · MUI `SwipeableDrawer anchor=bottom` · iOS `UISheetPresentationController`（`.medium`/`.large`/`.custom` detents）· Android `ModalBottomSheet` / `BottomSheetScaffold` · antd-mobile `Popup` · MP `page-container` / 自绘。
- **要点**：**分档 30% / 60% / 80–90%**（详见 `mobile-sheets.md`）；必须有拖拽把手与明确关闭方式；键盘弹起时不得把主内容顶出可视区；档位切换要记忆。

### 10. Full-screen Overlay 全屏浮层
- **定义**：覆盖整个视口的独占层，用于移动端的重任务。
- **用在哪**：图片/文件查看器、富文本编辑、支付与实名流程、需要键盘的长表单。
- **禁用**：只是补充信息（浪费）；无退出路径（必须有显式关闭 + 支持系统返回手势）。
- **对应**：iOS `fullScreenCover` / push · Android 全屏 Activity / `DialogFragment` · Web `inset-0` 自绘 · MP 独立页面（优先真页面，保留返回栈）。
- **要点**：必须有常驻关闭/返回；有未保存内容时返回要确认；保留系统返回手势；标题栏明确「你在做哪件事」。

### 附：19. Command / 命令面板（⌘K）
桌面键盘流的可选加速器，按第 5 级模态对待但**必须允许 Esc 直退、不做任何强制**。移动端不提供。它不能是唯一入口。
