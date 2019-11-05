# What's New in Testing

Session 409

🔗 [What's New in Testing - WWDC 2017 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2017/409/)


## Async Testing

closure, delegate method, delayed completion 을 통해 콜백받아 즉시 실행 되지 않는 것을 테스트
Opening documents
Work on background threads
Communication with services and extensions
Network activity
Animations
UI test conditions

### XCTestCase APIs

```Swift
    let document = UIDocument(fileURL: documentURL)
    let documentExpectation = expectation(description: "Document opened")
    
    document.open() { success in
    	XCTAssert("Failed to open file")
    	documentExpectation.fulfill()
    }
    
    waitForExpectations(timeout: 10)
```

Limitations → Time out is a test failure, Waiting requires test object, Hard to factor out, No nested waiting

## 🆕 XCTWaiter

Extracted logic from XCTestCases

Explicit list of expectations

Calls back to `XCTWaiterDelegate`

Returns `XCTWaiter.Result`

```Swift
    // Test case waits implicitly
    waitForExpectations(timeout: 10)
    👇🏻
     // Test case waits explicitly 
    wait(for: [documentExpectation], timeout: 10)
    
    // Waiter instance delegates to test
     XCTWaiter(delegate: self).wait(for: [documentExpectation], timeout: 10)
    
    // Waiter class returns result 
    let result = XCTWaiter.wait(for: [documentExpectation], timeout: 10)
     if result == .timedOut { 
    	// handling the timeout...
    }
```

## 🆕 XCTestExpectation

Public initializer (Decoupled from XCTestCase)

Multiple fulfillments

Inverted behavior

### XCUIApplication

Launch, Terminate

Queries → User Interface elements를 찾아줌

Target Application 이 테스트 할 메인 타겟 (Project Setting에서 설정)

default initializer `let taregetApp = XCUIAPplication()`

### Multi-app Scenarios

appgroup 하나 이상의 앱을 테스트 할 수 있게 해줌. 데이터를 주고 받으며 상호작용한다.

### 🆕  Additions to XCUIApplication

**New Initializers**

```Swift
    init(bundleIdentifier: String)
    init(url: URL) // MacOS
```

**Activate method**
```Swift
    func activate()
```

이미 작동하고 있다면 앱을 background에서 foreground로 올려 주고

아니라면 새로운 인스턴스 생성.

테스트를 시작 하면 먼저 launch API는 이전에 작동 하고 있던 인스턴스를 종료시킬 것. Activate는 이전 상태를 날리고 싶지 않고, 그 전 특정 시점에서 다시 시작하고 싶을 때 좋다.

**State Property**
```Swift
    var state: XCUIApplication.State { get }
```

테스트 중인 앱의 변화를 감지할 수 있다.

### API capturing on demand
```Swift
    XCUIElement.screenshot
    XCUIScreen.screenshot
```
