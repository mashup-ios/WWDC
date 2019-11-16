# UI Testing in Xcode

> 📅 2019.11.16 (토)
>
> WWDC 2015 |  Session : 406 | Category : Testing


🔗 [UI Testing in Xcode - WWDC 2015 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2015/406/)

UI testing

Find and interact with UI elements
Validate UI properties and state

UI recording
Test reports

> Core Technologies = XCTest + Accessibility

## XCTest

- Test case subclasses
- Test methods
- Assertions
- Integrate with Xcode
- CI via Xcode Server and xcodebuild
- Swift and Objective-C

## Accessibility

장애가 있는 사람들도 앱에서 모든 사람이 받는 똑같은 경험을 할 수 있게 해주는 기술

- Rich semantic data about UI
- UIKit and AppKit integration
- APIs for fine tuning
- UI test interact with the app the way a user does

### Requirements

UI testing depends on new OS feature

- iOS 9, OS X 10.11

Privacy protection

- iOS devices
    - Enabled for development
    - Connected to a trusted host running Xcode
- OS X must grant permission to Xcode Helper
    - Prompted on first run

### UI Testing Xcode Targets

UI tests have special requirement

- **Execute in a separate process**
- Permission to use Accessibility

New Xcode target templates

- Cocoa Touch UI Testing Bundle(iOS)
- Cocoa UI Testing Bundle(OS X)

"Target to be Tested" setting

### APIs

Three new classes

- `XCUIApplication`
- `XCUIElement`
- `XCUIElementQuery`

### UIRecording

Interact with your app

Recording generates the code

- Create new tests
- Expand existing tests

### XCUIApplication

Proxy for tested application

- Tests run in a separate process (앱의 라이프 사이클과 독립적)

Launch 
Launch 시 항상 새로운 프로세스를 불러온다

- Always spawns a new process
- Implicitly terminates any preexisting instance

Starting point for finding elements

### XUIElement

Proxy for element in application 
XCUIApplication처럼 앱의 proxy 역할을 하지만 element는 테스트 앱의 user interface element 역할이다.

Types

- Button, Cell, Window, etc.

Identifiers

- Accessibility identifier, label, title, etc.

Most elements are found by combining types and identifier

![](/Jinha/images/UI-Testing-in-Xcode-1.png)

### Element Uniqueness

Every XCUIElement is backed by a query

Query must resolve to exactly one match

- No matches or multiple matches cause test failure
- Failure raised when element resolves query

Exception

- `exists` property

### Event Synthesis

- Simulate user interaction on elements
- APIs are platform-specific

```Swift
    button.click()
    button.tap()
    textField.typeText("Hello, World!")
```

### XCUIElementQuery

API for specifying elements

Queries resolve collections of accessible elements
Accessibility 에게 보이는 element만 찾을 수 있다

- Number of matches: `count`

- Specify by identifier: subscripting
- Specify by index: `elementAtIndex()`

## Accessibility and UI Testing

Accessibility data의 질이 좋을 수록 테스트를 작성 하는 것이 쉽고 더 신뢰 할 수 있다.

![](/Jinha/images/UI-Testing-in-Xcode-2.png)
