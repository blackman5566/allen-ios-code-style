---
name: allen-ios-code-style
description: >-
  Use this skill when organizing or refactoring iOS Swift code in Allen's preferred style:
  split Swift code into responsibility-based extensions, classify code by responsibility,
  add Traditional Chinese intent comments for types, enum cases, properties, and functions,
  split function bodies into readable phases with concise Traditional Chinese block comments,
  preserve behavior, avoid over-engineering, and respect existing project/company
  patterns such as ToolsModel layout helpers.
---

# Allen iOS Code Style

Use this skill when the user asks to organize, clean up, annotate, refactor lightly, or "整理程式碼" for an iOS Swift project.

The goal is not to redesign the feature. The goal is to make the code easier to read, easier to maintain, and closer to the user's preferred style while preserving behavior.

## Core Rules

- Read the surrounding code first. Match the project style before changing structure.
- Preserve behavior unless the user explicitly asks for a functional change.
- Keep changes scoped to the files and feature being discussed.
- Treat responsibility-based extensions as a required output style for Swift files, not an optional cleanup.
- Prefer simple Swift, closures, callbacks, and existing project patterns over new abstractions.
- Do not introduce Combine, protocols, builders, or new helper layers just to make the code look clever.
- Avoid renaming public APIs, exposed callbacks, bridge names, JSON keys, route names, or config keys unless the user asks.
- If a layout uses company helpers such as `ADD`, `.貼著`, `.高度`, and `.OffSet`, prefer those helpers when they fit.
- If native `NSLayoutConstraint` is clearer or required for priorities, inequalities, or mutually exclusive constraints, keep it and add a short reason comment.
- Inside non-trivial functions, separate meaningful phases with short Traditional Chinese block comments, especially guard checks, data preparation, state updates, UI updates, callbacks, and side effects.
- After editing, run the smallest useful verification, usually build or targeted tests. If verification is not run, say why.

## Non-Negotiable Output Requirements

When organizing a Swift file, the output is incomplete if it does not satisfy these checks:

- The main type body contains only stored properties, computed properties when tightly tied to state, initializers, lifecycle overrides, and `deinit`.
- Private methods such as `setupViews()`, `setupLayout()`, `bindActions()`, `configure(...)`, `updateUI(...)`, `fetch...()`, `handle...()`, `buttonTapped(...)`, and helpers must live in responsibility-based extensions.
- Each extension must have a `//MARK:` that names its responsibility.
- Domain types, models, enums, enum cases, and non-obvious stored properties must have Traditional Chinese intent comments.
- Each non-trivial function must have a Traditional Chinese intent comment immediately above it.
- Each non-trivial function body must include short Traditional Chinese block comments before meaningful phases; prefer one concise line per phase instead of line-by-line comments.
- Long setup/layout functions must be split into small phase functions, or at minimum have clear block comments for each phase.
- A `setupViews()` function should usually orchestrate setup steps instead of containing every stack view, label style, text, alignment, and constraint in one large body.
- Do not leave `//MARK:` sections inside the main type body when those methods can be moved into extensions.
- Before finalizing, scan the file and move any private function still sitting in the main type body into the correct extension.

Allowed main type body shape:

```swift
final class SomeView: UIView {
    //MARK: property
    private let titleLabel = UILabel()

    //MARK: life cycle
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupViews()
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

Not allowed after organizing:

```swift
final class SomeView: UIView {
    private let titleLabel = UILabel()

    override init(frame: CGRect) {
        super.init(frame: frame)
        setupViews()
    }

    //MARK: setup
    private func setupViews() { ... }
}
```

Corrected output:

```swift
final class SomeView: UIView {
    //MARK: property
    private let titleLabel = UILabel()

    //MARK: life cycle
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupViews()
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}

//MARK: setup
private extension SomeView {
    //建立畫面元件與基本樣式。
    func setupViews() { ... }
}
```

## File Shape

Split Swift code into responsibility-based extensions by default. Keep stored properties, initializers, lifecycle overrides, and `deinit` in the main type body; move setup, bind, action, flow, api, web bridge, layout, update UI, delegate/data source, and helper functions into focused extensions.

Use `//MARK:` sections to group code by responsibility. Keep the order close to this when it fits the file:

```swift
//MARK: property
//MARK: life cycle
//MARK: setup
//MARK: bind
//MARK: action
//MARK: public method
//MARK: flow
//MARK: api
//MARK: web bridge
//MARK: layout
//MARK: update ui
//MARK: helper
```

Only include sections that actually help. Do not add empty or artificial sections.

Use extensions to make the file's responsibilities visible from a quick scan:

```swift
//MARK: setup
extension SomeViewController {
    //建立畫面層級與固定元件的 Auto Layout。
    private func setupViews() { ... }
}

//MARK: action
extension SomeViewController {
    //使用者點擊確認後，通知呼叫端繼續下一步流程。
    @objc private func confirmButtonTapped() { ... }
}
```

Avoid putting unrelated private methods into one large extension or the main type body:

```swift
//MARK: helper
extension SomeViewController {
    private func setupViews() { ... }
    private func fetchData() { ... }
    @objc private func confirmButtonTapped() { ... }
    private func applyLayout() { ... }
}
```

If a file is tiny, one setup extension may be enough. If a file owns several workflows, split extensions by responsibility even when all methods are private.

## Function Body Shape

Keep function bodies readable by separating phases. A function is too dense when it mixes several of these in one uninterrupted block:

- create subviews
- configure stack views
- apply labels/fonts/colors
- assign display text
- set alignment/compression/resistance
- add constraints
- update state
- trigger actions or callbacks

Use short block comments inside function bodies even when the function is not a long UI setup. The comment should mark the intent of the next small group of statements. Prefer one line per phase, not one comment per assignment.

Good ordinary helper shape:

```swift
//記住 Web 最近一次顯示的 PageTabs，讓無 payload 頁面可還原分頁脈絡。
func rememberVisiblePageTabs(from state: PageChromeState) {
    guard state.pageTabs.isVisible, !state.pageTabs.items.isEmpty else {
        return
    }

    //儲存最新的 PageTabs 資訊。
    latestVisiblePageTabItems = state.pageTabs.items
    latestVisiblePageTabsStyle = state.pageTabs.style

    //將目前選擇的 title 存起來。
    if let selectedId = state.pageTabs.selectedId,
       let selectedItem = state.pageTabs.items.first(where: { $0.id == selectedId }) {
        latestVisiblePageTabTitle = selectedItem.title
    }
}
```

Avoid leaving meaningful groups unexplained:

```swift
func rememberVisiblePageTabs(from state: PageChromeState) {
    guard state.pageTabs.isVisible, !state.pageTabs.items.isEmpty else {
        return
    }

    latestVisiblePageTabItems = state.pageTabs.items
    latestVisiblePageTabsStyle = state.pageTabs.style

    if let selectedId = state.pageTabs.selectedId,
       let selectedItem = state.pageTabs.items.first(where: { $0.id == selectedId }) {
        latestVisiblePageTabTitle = selectedItem.title
    }
}
```

For long UI setup, prefer this shape:

```swift
//MARK: setup
private extension SomeCell {
    //建立欄位標題列的主要畫面結構。
    func setupViews() {
        setupContentStyle()
        setupColumnStackView()
        setupSymbolStackView()
        setupLabelStyle()
        setupColumnText()
        setupColumnAlignment()
        setupColumnWidthConstraints()
        addActiveETFBottomLine()
    }

    //設定 cell 與 contentView 的背景樣式。
    func setupContentStyle() { ... }

    //建立與持股資料列一致的欄位排列。
    func setupColumnStackView() { ... }

    //建立代號與股名的水平排列。
    func setupSymbolStackView() { ... }
}
```

If splitting into helpers would make a very small file noisier, keep the code in one function but add block comments for each phase:

```swift
//建立表格欄位標題列，欄寬需和持股資料列一致。
func setupViews() {
    //設定 cell 基本樣式。
    selectionStyle = .none
    backgroundColor = ActiveETFColor.tableCellBackground
    contentView.backgroundColor = backgroundColor

    //建立外層欄位排列，欄位順序需對齊持股資料列。
    let stackView = UIStackView(arrangedSubviews: [symbolContainer, changeLabel, holdingLabel, trendLabel, rankLabel])
    ...

    //建立代號與股名的內層排列。
    let symbolStackView = UIStackView(arrangedSubviews: [codeLabel, nameLabel])
    ...

    //統一欄位文字樣式，避免各 label 字重或縮放比例不一致。
    [codeLabel, nameLabel, changeLabel, holdingLabel, trendLabel, rankLabel].forEach { label in
        ...
    }

    //設定欄位顯示文字與對齊方式。
    codeLabel.text = "代號"
    ...

    //固定每個欄位寬度，確保 header 與資料列對齊。
    symbolWidthConstraint = symbolContainer.widthAnchor.constraint(...)
    ...
}
```

Do not leave long setup functions as an unannotated stream of statements.

## Comment Style

Write comments in Traditional Chinese. Comments should explain intent, flow, constraint, or risk. Do not translate every line of code.

Comments are required for domain-facing declarations, not only functions:

- Type comments for `class`, `struct`, `enum`, and `protocol` that represent app concepts, view models, API models, section models, cell types, bridge payloads, or config.
- Case comments for enums that drive UI, routing, table sections, collection sections, cell rendering, API states, or bridge actions.
- Property comments for model fields, config values, callback closures, state flags, and anything whose meaning is not obvious from the name alone.
- Function comments for every non-trivial `func`.

Every non-trivial `func` should have a concise intent comment immediately above it. The comment should answer "why this function exists" or "what workflow this function owns", not repeat the function name in Chinese.

Good model and enum comments:

```swift
//主題子選單中主動式 ETF 頁面的資料區塊。
enum ActiveETFSectionType {
    //法人觀點與投資風險說明。
    case report

    //主動式 ETF 商品列表。
    case etf

    //市場觀察或投資洞察內容。
    case insight
}

//投資洞察區塊中的一則文字內容。
struct ActiveETFInsightItem {
    //顯示於段落上方的主題文字。
    let topic: String

    //已套用字型、行距與顏色的段落內容。
    let content: NSAttributedString
}

//主動式 ETF 頁面 TableView 使用的 cell 類型。
enum ActiveETFCellType {
    //報告區塊標題與日期。
    case report(title: String, dateText: String)

    //ETF 區塊標題列。
    case etfTitle(ActiveETFSection)

    //欄位名稱 header。
    case columnHeader

    //ETF 持股異動資料列。
    case holding(ItemSCActiveEtfHoldingChangesHolding)

    //投資洞察文字列，showsHeader 控制是否顯示段落標題。
    case insight(ActiveETFInsightItem, showsHeader: Bool)
}
```

Avoid comment-free model declarations:

```swift
enum ActiveETFSectionType {
    case report
    case etf
    case insight
}

struct ActiveETFInsightItem {
    let topic: String
    let content: NSAttributedString
}
```

Good function comments:

```swift
//建立畫面層級，讓 header、WebView、TabBar 的上下關係固定。
private func setupViews() { ... }

//依照 Web 傳來的 chrome 狀態，統一更新原生 header、page tabs、tab bar 顯示。
private func applyNativeChromeState(_ state: PageChromeState) { ... }

//使用者點擊原生 TabBar 時，只回報事件給 Web，實際選中狀態由 Web 再送回。
@objc private func tabButtonTapped(_ sender: UIButton) { ... }
```

Avoid weak function comments:

```swift
//設定畫面
private func setupViews() { ... }

//按鈕點擊
@objc private func buttonTapped() { ... }

//更新 state
private func updateState() { ... }
```

Inside a function, add block comments whenever the body has meaningful phases, not only when the function is long. This includes guard/failure branches, data preparation, state memory, layout calculation, UI update, callback, navigation, API, or WebView bridge side effects. Keep comments close to the code they explain.

Good block comments describe what the next block is trying to accomplish:

```swift
//Auto Layout 尚未完成時，先用估算寬度計算內容高度。
let contentWidth = contentContainerView.bounds.width > 0
    ? contentContainerView.bounds.width
    : estimatedContentWidth()

//內容高度不可小於最小值，也不可超過畫面可用高度。
let maxContentHeight = availableContentHeight()
let preferredHeight = currentContentView.preferredHeight(
    for: contentWidth,
    maxHeight: maxContentHeight
)
```

Avoid empty comments like:

```swift
//設定 title
titleLabel.text = title

//新增 view
view.addSubview(contentView)
```

Prefer comments for:

- Function entry points, especially private functions that own a UI step, API step, or bridge event.
- Internal function phases, especially guard checks followed by state updates or side effects.
- Lifecycle timing, especially app foreground/background and biometric prompts.
- WebView bridge direction and message ownership.
- API request flow, token/encryption, and environment config.
- Auto Layout choices that are not obvious.
- State machines, modal/alert flows, and ordering rules.
- Workarounds for platform or Web behavior.

Do not force comments on tiny computed properties or one-line passthrough helpers when the name already makes the behavior obvious. If a `func` exists only as a selector or callback entry, still add one intent comment because the call site is often far away.

## Naming

- Prefer names that describe responsibility rather than implementation detail.
- Use `applyNativeChromeState` over vague names like `applyChrome` when the method controls native UI chrome.
- Use `showPrivacyCover`, `hidePrivacyCover`, `sendBiometricResultToWeb`, or similar direct names.
- Keep existing domain names if the project already uses them.
- Do not rename files or types broadly unless the user asks.

## Layout Style

When using company helpers, keep the layout readable and grouped by view:

```swift
view.ADD(目標: titleHeaderView) { headerView in
    headerView.topAnchor.貼著(目標Anchor: self.view.topAnchor)
    headerView.leftAnchor.貼著(目標Anchor: self.view.leftAnchor)
    headerView.rightAnchor.貼著(目標Anchor: self.view.rightAnchor)
    headerView.bottomAnchor
        .貼著(目標Anchor: self.view.safeAreaLayoutGuide.topAnchor)
        .OffSet(value: Metrics.navigationBarHeight)
}
```

Use native constraints when the helper cannot express the intent clearly:

```swift
//此處需要 priority 與 lessThanOrEqualTo，使用原生 constraint 會比公司 helper 清楚。
cardWidthConstraint = cardView.widthAnchor.constraint(equalToConstant: Metrics.preferredCardWidth)
cardWidthConstraint?.priority = .defaultHigh
```

## WebView And Native Bridge

For hybrid apps, keep ownership clear:

- Web owns page state and decides which native UI should show.
- Native owns the actual native view rendering and native button tap events.
- Native button taps should send events to Web.
- Web decides route/API/state, then sends a complete UI state back to Native.
- Keep bridge payloads stable and explicitly named.

Comment the bridge direction when useful:

```swift
//Native 只回報使用者點了哪個按鈕，實際頁面狀態仍由 Web 決定。
callWebEvent([
    "type": "nativeButtonTapped",
    "payload": ["id": "cart"]
])
```

## API And Config

- Keep environment-specific values in config files, not hardcoded in feature code.
- Keep API structure close to the existing project pattern.
- Prefer dependency injection if the project already uses it.
- Do not log secrets, AES keys, tokens, or sensitive response bodies.
- When encryption/token generation exists, comment the required protocol but do not expose secret values.

## Alert And Flow Code

For alerts, launch notices, and onboarding flows:

- Separate data preparation, presentation, dismiss action, and next-step navigation.
- Keep display order explicit.
- Avoid making one view controller handle unrelated alert types unless they share the same lifecycle.
- If WebView content is used inside an alert, avoid showing blank content; preload or delay display until content is ready when feasible.

## Example Code

Use examples like these as the target rhythm when organizing Swift files.

Before:

```swift
extension SplashViewController {
    private func showLaunchNoticesOrEnterApp(_ notices: [LaunchNoticeItem]) {
        guard !notices.isEmpty else {
            enterMainApp()
            return
        }

        let alertController = LaunchNoticeAlertController()
        alertController.parameter = ["noticeItems": notices] as NSDictionary
        alertController.dismissBlock = { [weak self] _ in
            self?.enterMainApp()
        }

        present(alertController, animated: true)
    }
}
```

After:

```swift
//MARK: notice flow
extension SplashViewController {
    //有公告就顯示啟動公告彈窗，沒有公告就直接進入 Web 容器。
    private func showLaunchNoticesOrEnterApp(_ notices: [LaunchNoticeItem]) {
        guard !notices.isEmpty else {
            enterMainApp()
            return
        }

        //公告彈窗關閉後才進主畫面，避免轉場跟 dismiss 動畫互相搶畫面。
        let alertController = CustomAlertControllerFactory.presentFormSheetType(
            alertType: .LaunchNotice,
            parameter: ["noticeItems": notices] as NSDictionary,
            dismissBlock: { [weak self] _ in
                guard let self else { return }
                self.dismissAlertController(dismissStyleType: .Dismiss) { [weak self] in
                    self?.enterMainApp()
                }
            },
            submitBlock: nil
        )

        presentFormAlertController(
            customAlertController: alertController,
            presentStyleType: .FadeIn,
            completion: nil
        )
    }
}
```

Layout example:

```swift
//MARK: layout
private extension LaunchNoticeAlertController {
    //依照目前內容重新計算卡片高度，讓長文可捲動、短文維持置中。
    func updateCardLayout() {
        let contentWidth = contentContainerView.bounds.width > 0
            ? contentContainerView.bounds.width
            : estimatedContentWidth()
        guard contentWidth > 0 else { return }

        //內容高度不可小於最小值，也不可超過畫面可用高度。
        let maxContentHeight = availableContentHeight()
        let preferredHeight = currentContentView.preferredHeight(
            for: contentWidth,
            maxHeight: maxContentHeight
        )

        contentHeightConstraint?.constant = preferredHeight
        view.layoutIfNeeded()
    }
}
```

Web bridge example:

```swift
//MARK: web bridge
private extension MainWebContainerViewController {
    //Native 只回報使用者點了哪個按鈕，實際 route 與 selected state 仍由 Web 決定。
    func sendNativeButtonTapToWeb(_ buttonId: String) {
        callWebEvent([
            "type": "nativeButtonTapped",
            "payload": ["id": buttonId]
        ])
    }

    //Web 回傳完整 chrome 狀態後，Native 才更新原生 UI 顯示與選取狀態。
    func handleSetChromeMessage(_ payload: [String: Any]) {
        guard let state = PageChromeState(payload: payload) else { return }
        applyNativeChromeState(state)
    }
}
```

UITableViewCell example:

```swift
final class ActiveETFDisclaimerCell: UITableViewCell {
    //MARK: property
    static let reuseIdentifier = "ActiveETFDisclaimerCell"

    private let titleLabel = UILabel()

    //MARK: life cycle
    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        setupViews()
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}

//MARK: setup
private extension ActiveETFDisclaimerCell {
    //建立法人視角底部來源與投資風險文字。
    func setupViews() {
        selectionStyle = .none
        contentView.ADD(目標: titleLabel) { label in
            label.topAnchor.貼著(目標Anchor: self.contentView.topAnchor)
            label.leftAnchor.貼著(目標Anchor: self.contentView.leftAnchor)
            label.rightAnchor.貼著(目標Anchor: self.contentView.rightAnchor)
            label.bottomAnchor.貼著(目標Anchor: self.contentView.bottomAnchor)
        }
    }
}

//MARK: update ui
extension ActiveETFDisclaimerCell {
    //套用列表資料，讓 cell 重用時能完整刷新文字。
    func configure(text: String) {
        titleLabel.text = text
    }
}
```

## Cleanup Checklist

Before finishing a code organization task:

- Code is grouped by responsibility.
- Private methods are split into responsibility-based extensions instead of staying in the main type body.
- Non-trivial function bodies have short Traditional Chinese block comments before meaningful phases.
- Long setup/layout functions are split into smaller phase functions, or each phase has a clear block comment.
- Comments explain intent, not obvious syntax.
- Public behavior and bridge contracts are preserved.
- Layout constraints do not introduce conflicts.
- No unrelated refactors are included.
- Build or targeted verification has been attempted.
- Final response mentions what changed and whether verification passed.
