# 移动端浮层：Bottom Sheet、Drawer 与全屏

## 1. Bottom Sheet 与 Drawer 到底差在哪

| 维度 | Drawer 侧边抽屉 | Bottom Sheet 底部面板 |
|---|---|---|
| 出身 | 桌面/平板（导航抽屉、详情面板） | 移动（拇指可达、单手操作） |
| 进入方向 | 左/右边缘滑入 | 底部升起 |
| 与主页关系 | 桌面可并存（persistent），主页仍可见 | 覆盖主页下半部，主页上半部仍可见（peek 档） |
| 高度/宽度 | 宽度按内容定，高度满屏 | **宽度满屏，高度分档** |
| 交互 | 点背景/Esc/关闭按钮 | **拖拽把手换档** + 下滑关闭 + 点背景 |
| 适合 | 详情、筛选、导航、辅助面板 | 半屏预览、动作列表、分享、筛选、短表单 |
| 窄屏手机 | 宽度失控或退化成全屏 → 不该用 | 原生形态 |
| 平板横屏 | 合适 | 偏小，考虑 Side Sheet 或 Split |
| 桌面 Web | 合适 | 不自然（无拖拽预期、鼠标远离） |

**判据一句话**：这个从属任务是「和主页并排看」还是「低头单手操作」？前者 Drawer，后者 Sheet。窄屏一律 Sheet 或全屏。

## 2. 三分屏：30% / 60% / 80–90%

这是抖音半屏评论区、地图路线卡、大众点评 POI 卡背后的通用规则。档位本身不是目的，**目的是让「瞄一眼」和「长时间用」不互相牺牲**。

| 档 | 高度（视口占比） | 用途 | 主页可见度 | 典型内容 |
|---|---|---|---|---|
| Peek / Collapsed | ≈30%（25–35%） | 只读预览、不打断当前任务 | 大部分可见可交互 | 地图路线摘要、歌曲条、下一站提示 |
| Half | ≈60%（55–65%） | 主任务：浏览列表、看详情、轻筛选 | 顶部标题与状态栏可见 | 评论区、商品规格、POI 详情 |
| Expanded | ≈80–95% | 长列表、需要滚动或输入的任务 | 只留状态栏/顶部条 | 长评论列表、带搜索的筛选、表单 |

规则：
1. **至少两档，且必须可拖回。** 单档全屏的 sheet 等于一个假页面。
2. **Peek 档不得阻塞主页操作**（地图可继续拖动）。若需要阻塞，用 Modal Bottom Sheet 而不是 Standard。
3. **Standard Bottom Sheet（非模态）**：主页仍可交互，无遮罩，点主页即收起 —— 适合 peek/half 的信息层叠。
   **Modal Bottom Sheet**：有遮罩、阻塞主页、Esc/下滑关闭 —— 适合动作列表、需要决策的半屏任务。
4. **记忆档位**：用户拖到 Expanded 后再次打开应停在他上次的位置（同一会话内），别每次从 peek 重来。
5. **Expanded 不要 100%**：留一条主页缝隙（或用安全区顶格），让用户感知「我还在原来那一页里」；确实需要独占时改用 Full-screen Overlay，别伪装成 sheet。
6. **内容超过 Expanded 一屏还多 → 那不该是 sheet，是页面**（有返回栈、有深链、有标题）。

## 3. 结构件

一个合格的 Sheet 有这些件，缺一项就是体验破口：

- **拖拽把手（grabber）**：顶部居中，≥32×4pt 可视、命中区 ≥44pt 高。它同时是「这里可以拖」的 affordance。
- **标题行**：说清这是什么任务；右侧放关闭（×）或完成（✓），**不要把关闭藏在只能下滑的手势里**。
- **内容区独立滚动**：把手/标题固定，内容滚动；到顶后继续下滑才收起 sheet（不是内容一滚就收）。
- **遮罩**：Modal 版有、Standard 版无。
- **安全区**：底部内容要避开 home indicator；输入框要随键盘抬起并在收起后回落。

## 4. 键盘与输入

Sheet 里放输入框是最容易翻车的一格：
- 键盘弹起时 sheet 会顶到 Expanded 或被遮挡 —— **需要键盘的任务优先全屏页/独立路由**，别硬塞在 60% 档。
- 若必须放：输入框获得焦点时自动升到 Expanded；失焦/键盘收起时回到原档。
- 主操作按钮（保存/完成）放在**键盘上方**或标题行右侧，不要放在会被键盘埋住的 sheet 底部。
- 提交失败时错误要留在字段下方（Inline Alert），**不能只弹一个会被键盘挡住的 Toast**。

## 5. 各端实现映射

| 端 | 分档能力 | 备注 |
|---|---|---|
| iOS 原生 | `UISheetPresentationController.detents` = `.medium`(≈半屏) / `.large` / `.custom { identifier }` | 自定义 detent 才能做出 30%/80%；`prefersGrabberVisible=true` 给把手；`shouldDismissAtPageSize` 控制 peek 是否可下滑关 |
| SwiftUI | `.presentationDetents([.fraction(0.3), .medium, .large])` + `.presentationDragIndicator(.visible)` | iOS 16.4+ 支持 `selection` 双向绑定档位 |
| Android/M3 | `ModalBottomSheet` + `BottomSheetScaffold`；`SheetState(peek)`；`BottomSheetBehavior` 的 `STATE_HIDDEN/COLLAPSED/HALF_EXPANDED/EXPANDED` | 有现成 HALF_EXPANDED 态；`skipCollapsed` 会吃掉中间档 |
| Web（React） | Radix 无官方 sheet；常用 **vaul**（`snapPoints`/`detents`，支持拖拽与多档）；MUI `SwipeableDrawer anchor="bottom"`；antd-mobile `Popup` | 用 `Sheet`（Radix Dialog 换 side）做的「侧边 sheet」和「底部 sheet」是两件事，别共用一套高度逻辑 |
| 小程序 | 原生只有 `wx.showActionSheet`（固定动作列表，无分档）；`page-container`（半屏页容器，支持自定义高度与下滑关闭）；否则自绘 | 自绘要自己处理滚动穿透（`catchtouchmove`）与安全区 |
| Web 套壳（Capacitor/Cordova） | **默认全部是 Web modal，没有原生 sheet** | 见 §6 |

## 6. 混合应用（Web 套壳）的分端策略

这是多数「一个 React 代码包出 iOS/Android」产品的真实处境：

1. **没有原生适配时，移动端的 Dialog 必须改成底部对齐的 sheet 形态**。居中模态在窄屏上会：吃掉拇指热区、键盘弹起后遮挡主内容、与系统返回手势打架。做法：同一组件按视口切换 `center`(桌面) / `bottom`(移动)，而不是两套组件。
2. **系统返回必须接管**。原生 `history`/`popstate` 或 Capacitor 的 `App.addListener('backButton')` 要能关浮层，优先级：Popconfirm/Sheet > Modal > 页面返回。做不到就至少保证浮层有常驻关闭按钮。
3. **不要用浏览器原生 `confirm()`/`alert()`**。在 WKWebView/Android WebView 里它们弹出系统 JS 对话框：无样式、无本地化、文案截断、无法放后果说明、无法测焦点。
4. **要不要上原生 sheet 插件？** 只有当「拖拽换档 + 跟手动画」成为核心体验时才值得引入原生层（成本是双端维护与焦点/滚动同步）。多数后台/工具型产品用 Web 侧 vaul 级别的实现就够了 —— 但**分档语义（30/60/80、把手、记忆）要在 Web 层做出来**，不能因为「是套壳」就退化成居中弹窗。
5. **视口判定用能力不用 UA**：`(pointer: coarse)` + 宽度断点，比 `isNativePlatform()` 更贴近「该不该用 sheet」这个真实问题；原生壳检测只用于接管返回键、状态栏这类真原生能力。
