# iOS UI Tests Skill — Robot Pattern with XCTest

This guide tells how to structure iOS UI tests using the **Robot Pattern** with **XCTest**.  
Goal: make UI tests **readable, reusable, and stable** by moving UI interactions and assertions into Robots.

---

## Principles

### Why Robot Pattern
- Tests read like scenarios: `robot.login().openSettings().logout()`
- UI selectors are centralized (less duplication)
- Assertions live close to the UI action that requires them
- Easier maintenance when UI changes

### Rules (mandatory)
1. **No direct `XCUIElement` queries in test methods**  
   All element lookup must be inside Robots.
2. **Robots return `Self` for chaining**  
   Every action method should be `@discardableResult -> Self`.
3. **Every action must validate preconditions**  
   Use `XCTAssertTrue(element.waitForExistence(timeout: ...))` before tapping.
4. **Prefer accessibility identifiers**  
   Robots must query using `accessibilityIdentifier` constants (not labels).
5. **Robots own assertions for their screen**  
   Assertions about UI state (labels, visibility, enabled/disabled) go in Robots.
6. **Tests only express intent**  
   A test should orchestrate Robots and validate high-level outcomes.

Test methods must never contain:
- `app.buttons[...]`
- `app.staticTexts[...]`
- `app.otherElements[...]`

All queries must live in Robots.---

### 3.2 Robots must be chainable
All public Robot action methods must:
- Be annotated with `@discardableResult`
- Return `Self`

```swift
@discardableResult
func tapConfirm() -> Self { ...; return self }
```

---

### 3.3 Validate existence before interaction
Every interaction must:
- Use `waitForExistence(timeout:)`
- Assert success before acting

```swift
XCTAssertTrue(button.waitForExistence(timeout: 2))
button.tap()
```

---

### 3.4 Prefer accessibility identifiers
Robots must select elements using:
- `accessibilityIdentifier`
- Centralized constants

Avoid visible labels (localization-safe).

---

### 3.5 Assertions belong in Robots
UI assertions must be placed in Robots, near the related action:
- labels
- visibility
- enabled/disabled state
- error messages

---

### 3.6 Use @MainActor
All Robots and UI test classes must be annotated with `@MainActor`.

---

### 3.7 Never use sleep
`sleep()` is forbidden.  
Use explicit waits only.

---

## 4. Project Structure

```
UITests/
  Onboarding/
    OnboardingPage1Robot.swift
    OnboardingPage2Robot.swift
    OnboardingTests.swift
  SignIn/
    SignInPage.swift
    SignInUITests.swift
  Robot/
    UIRobot.swift
```

---

## 5. Base Robot

All Robots must inherit from a common base class.

```swift
import XCTest

class UIRobot {

    let app: XCUIApplication

    init(app: XCUIApplication) {
        self.app = app
    }

    func localized(_ key: String) -> String {
        let bundle = Bundle(for: type(of: self))
        return NSLocalizedString(key, bundle: bundle, comment: "")
    }
}
```

---

## 6. Page Identifiers (Mandatory)

All selectors must be centralized.

```swift
enum RecordingPage {
    enum Identifiers {
        static let recordButton = "recordButton"
        static let playerStatusInfoText = "playerStatusInfoText"
    }
}
```

---

## 7. Screen Robot Example

```swift
import XCTest

@MainActor
final class RecordingRobot: UIRobot {

    @discardableResult
    func tapRecordButton() -> Self {
        let button = recordButton()
        XCTAssertTrue(button.waitForExistence(timeout: 2))
        button.tap()

        let status = infoStatus()
        XCTAssertTrue(status.waitForExistence(timeout: 4))
        XCTAssertEqual(status.label, localized("recording_info_recording"))

        return self
    }

    @discardableResult
    func longPressRecordButton() -> Self {
        let button = recordButton()
        XCTAssertTrue(button.waitForExistence(timeout: 2))
        button.press(forDuration: 1.2)

        let status = infoStatus()
        XCTAssertTrue(status.waitForExistence(timeout: 2))
        XCTAssertEqual(status.label, "")

        return self
    }

    private func recordButton() -> XCUIElement {
        app.otherElements[RecordingPage.Identifiers.recordButton]
    }

    private func infoStatus() -> XCUIElement {
        app.staticTexts[RecordingPage.Identifiers.playerStatusInfoText]
    }
}
```

---

## 8. Navigation Strategy

### Preferred
Robots do not create other Robots.  
Tests instantiate the next Robot explicitly.

```swift
recording.tapRecordButton()
let detail = RecordDetailRobot(app: app)
detail.tapCancelButton()
```

---

## 9. XCTestCase Template

```swift
import XCTest

@MainActor
final class RecordingUITests: XCTestCase {

    private var app: XCUIApplication!
    private var recording: RecordingRobot!

    override func setUp() {
        super.setUp()
        continueAfterFailure = false

        app = XCUIApplication()
        recording = RecordingRobot(app: app)
        app.launch()
    }

    func testTapRecording() {
        recording
            .tapRecordButton()
            .longPressRecordButton()

        let detail = RecordDetailRobot(app: app)
        detail.tapCancelButton()
    }
}
```

---

## 10. Localization Rules

- Prefer identifiers over visible text.
- Only assert localized strings when behavior depends on exact wording.
- Never hardcode localized values in tests.

---

## 11. Stability Best Practices

- Always wait for UI state changes.
- Keep Robot methods small and single-purpose.
- Avoid cross-screen coupling inside Robots.
- One Robot = one screen or feature.

---

## 12. Validation Checklist

Before completing generation, verify:

- [ ] No raw `XCUIApplication` queries in tests
- [ ] All interactions use `waitForExistence`
- [ ] All selectors use identifier constants
- [ ] Robot methods return `Self`
- [ ] No `sleep()` usage
- [ ] Files are named in PascalCase and match main type
