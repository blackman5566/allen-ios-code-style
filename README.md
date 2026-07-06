# Allen iOS Code Style

A practical iOS Swift code style playbook for AI-assisted refactoring, readable structure, and production review.

This repository is not a general Swift style guide.

It focuses on one specific problem:

> When AI helps organize or refactor Swift code, the result should still look like production code that a human iOS engineer can review, maintain, and ship.

## Why This Exists

AI can generate Swift code quickly, but code that compiles is not always code that should be merged.

In real iOS projects, maintainability depends on more than syntax:

- responsibilities should be visible
- existing behavior should be preserved
- public contracts should not change casually
- layout code should match project conventions
- comments should explain intent, not repeat syntax
- refactoring should stay scoped and reviewable

This playbook gives AI a repeatable structure for cleaning up Swift code without turning a cleanup task into a rewrite.

## Philosophy

- Preserve behavior first.
- Prefer readability over cleverness.
- Match the existing project style.
- Keep changes scoped.
- Make responsibilities visible.
- Avoid over-engineering.
- Use comments to explain intent, flow, constraints, and risk.

## When To Use

Use this style when working on iOS Swift code and the task is about:

- organizing a large Swift file
- lightly refactoring a UIKit or SwiftUI screen
- splitting setup, actions, update logic, and helpers by responsibility
- preparing AI-generated Swift code for production review
- adding meaningful Traditional Chinese intent comments
- improving readability without changing behavior

Do not use this as an excuse to redesign the feature, rename public APIs, or introduce a new architecture.

## Core Rules

- Read the surrounding code first.
- Preserve existing behavior unless a functional change is explicitly requested.
- Keep changes scoped to the requested file or feature.
- Keep the main type body focused on properties, initializers, lifecycle methods, and `deinit`.
- Move private setup, bind, action, update, layout, API, bridge, and helper methods into responsibility-based extensions.
- Add a useful `//MARK:` for each extension.
- Prefer existing project patterns over new abstractions.
- Avoid introducing protocols, builders, coordinators, Combine, or MVVM only to make the code look more advanced.
- Avoid renaming public APIs, callbacks, JSON keys, route names, bridge event names, or config keys unless requested.
- Run the smallest useful verification after editing.

## File Shape

Preferred structure:

```swift
//登入頁面，負責呈現帳號密碼輸入與登入入口。
final class LoginViewController: UIViewController {
    //MARK: property
    private let accountTextField = UITextField()
    private let passwordTextField = UITextField()
    private let loginButton = UIButton(type: .system)

    //登入資料驗證完成後，交由外部流程處理實際登入。
    var onLogin: ((String, String) -> Void)?

    //MARK: life cycle
    override func viewDidLoad() {
        super.viewDidLoad()
        setupViews()
        bindActions()
    }
}

//MARK: setup
private extension LoginViewController {
    //建立登入頁面的畫面層級與基本樣式。
    func setupViews() { ... }
}

//MARK: action
private extension LoginViewController {
    //使用者點擊登入後，先驗證輸入再交給登入流程。
    @objc func loginButtonTapped() { ... }
}
```

Avoid this after cleanup:

```swift
final class LoginViewController: UIViewController {
    private let loginButton = UIButton(type: .system)

    override func viewDidLoad() {
        super.viewDidLoad()
        setupViews()
    }

    //MARK: setup
    private func setupViews() { ... }
}
```

## Comment Style

Swift comments should use Traditional Chinese.

Good comments explain intent:

```swift
//使用者點擊登入後，先驗證輸入再交給登入流程。
@objc func loginButtonTapped() { ... }

//Web 回傳完整導覽狀態後，Native 才更新原生 UI 顯示。
func handleNavigationState(_ payload: [String: Any]) { ... }
```

Weak comments only repeat syntax:

```swift
//設定畫面
func setupViews() { ... }

//按鈕點擊
@objc func buttonTapped() { ... }
```

## Function Body Shape

Non-trivial functions should be split into readable phases.

```swift
//使用者點擊登入後，先驗證輸入再交給登入流程。
@objc func loginButtonTapped() {
    //確認帳號密碼都有輸入。
    guard let account = accountTextField.text, !account.isEmpty,
          let password = passwordTextField.text, !password.isEmpty else {
        showInputError()
        return
    }

    //鎖住按鈕，避免重複送出登入請求。
    loginButton.isEnabled = false

    //把輸入資料交給外部登入流程。
    onLogin?(account, password)
}
```

For long setup code, prefer orchestration:

```swift
//建立登入頁面的畫面層級與基本樣式。
func setupViews() {
    setupBackground()
    setupTextFields()
    setupLoginButton()
    setupLayout()
}
```

## Anti-Patterns

Avoid:

- huge ViewControllers
- one giant `setupViews()`
- protocol for everything
- MVVM just because
- casual public API renaming
- over-abstracting small features
- comments on every obvious line
- hidden behavior changes inside a refactor
- unrelated cleanup while touching a file
- new helper layers used by only one call site

## Example Prompt

```text
Use Allen iOS Code Style.

Refactor this Swift file for readability.
Preserve behavior.
Keep changes scoped to this file.
Do not rename public callbacks, route names, JSON keys, bridge event names, or config keys.
Move private methods into responsibility-based extensions with //MARK sections.
Add Traditional Chinese intent comments for non-trivial functions and meaningful code phases.
Avoid new architecture unless necessary.
After editing, report changed files, verification result, and remaining risk.
```

## Checklist

Before finishing:

- Main type body only contains properties, initializers, lifecycle methods, and `deinit`.
- Private methods are grouped into responsibility-based extensions.
- Each extension has a useful `//MARK:`.
- Non-trivial functions have Traditional Chinese intent comments.
- Function bodies have short phase comments where useful.
- Existing behavior is preserved.
- Public API, JSON keys, route names, bridge event names, and config keys are unchanged.
- No unrelated refactors are included.
- Build or targeted verification has been attempted.
- Final report includes changed files, verification result, and remaining risk.

## Status

This is the first public version of the playbook.

It currently focuses on iOS Swift code organization and AI-assisted refactoring style.

Future versions may include more examples, UIKit-specific patterns, SwiftUI-specific patterns, and visual review workflows.
