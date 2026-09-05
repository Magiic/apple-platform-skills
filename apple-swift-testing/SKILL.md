---
name: apple-swift-testing
description: >
  Write and refactor Swift unit tests, UI tests, configurable dependency doubles, and
  deterministic fixtures. Use for behavioral testing, mock isolation, and test-support
  organization in modular Apple apps.
---

# Unit tests

## When to use
Use this skill when:
- writing new tests.
- refactoring existing tests.
- Writing or editing UI tests

## General principles
- Use Swift Testing for unit tests and XCTest for UI tests
- Prefer clarity over cleverness
- Small unit tests with single responsibility
- Use pattern Robot to make UI Tests. See `references/robot-pattern.md` for more details.
- Create UI Tests only if user ask to create.

## Naming
- Unit tests types should be suffixed with Tests for example: `UserTests`.
- UI tests types should be suffixed with UITests for example: `UserUITests`.
- Robot types should be suffixed with Robot for example: `UserRobot`.
- Methods in Robot use declarative pattern and begin with verb like `tap`, `longPress`, `record` etc.

## Project Structure

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

## Unit Test

Use Swift Testing framework for unit test.

- Always use `@Suite("Describe purposes of all tests in this suite")`
- Always provide description of test: `@Test("Given/when/then")`

```swift
import Testing
@testable import MarketingToolFeature

@Suite("DefaultMarketingToolClient")
struct DefaultMarketingToolClientTests {

    @Test("Fetches top chart apps and maps rank")
    func fetchTopChartApps() async throws {
        let client = makeClient()
        let request = ChartRequest(country: .fr, chartType: .topFree, limit: 2)

        let result = try await client.fetchTopChartApps(request: request)

        #expect(result.count == 2)
        #expect(result[0].appStoreId == 1)
        #expect(result[0].name == "Alpha")
        #expect(result[0].chartType == .topFree)
    }

    @Test("Looks up app details and maps lookup fields")
    func lookupApps() async throws {
        let client = makeClient()

        let details = try await client.lookupApps(appStoreIds: [123], country: .us)

        #expect(details.count == 1)
        #expect(details[0].appStoreId == 123)
    }
}
```
## UI Test

UI test have something similar to below:

```swift
import XCTest

@MainActor
final class RecordUITests: XCTestCase {
    
    private var robot: RecordRobot!

    func testTapRecording() throws {
        let app = XCUIApplication()
        robot = RecordRobot(app: app)
        app.launch()
        
        robot
            .tapRecordButton()
            .longPressRecordButton()
        
        let detail = RecordDetailRobot(app: robot.app)
        
        detail.tapCancelButton()
    }
}
```

## Test doubles and fixtures

- Inject doubles through the same contracts/configuration as production dependencies.
- Configure meaningful success/error outcomes and record only interactions the behavior requires.
- Use new mutable dependencies per test; avoid shared state, live services, and real preferences/keychain stores.
- Preserve the contract's actor isolation and cancellation semantics. Use deterministic clocks/data and explicit async control when needed.
- Keep fixtures separate from dependency behavior, and preview support independent of the test target.
- Do not add a protocol, ViewModel, or layer solely to mock a simple value or pure function.
- Read [test-doubles-and-fixtures.md](references/test-doubles-and-fixtures.md) when creating mocks, stubs, spies, fakes, fixtures, or shared test/preview support.

## Testing conventions
- Suffix test dependency doubles with `Mock` (`ArticleRepositoryMock`). Preview-only stubs may use `PreviewStub` in feature-local preview support.
- Place test doubles in a folder named `Mock` inside the owning test target.
- Tests mirror module structure
- AAA pattern (Arrange / Act / Assert) or Given/When/Then consistently

## References
- See `references/robot-pattern.md` for canonical examples, patterns and more details.
