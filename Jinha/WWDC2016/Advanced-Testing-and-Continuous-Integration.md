# Advanced Testing and Continuous Integration

Session 409

🔗 [Advanced Testing and Continuous Integration - WWDC 2016 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2016/409/)

## Testing Concepts

### Test Hosting

![](/Jinha/images/Advanced-Testing-and-Continuous-Integration-1.png)

Unit Tests 의 경우, test bundle 이 application에 직접적으로 load된다 (application 을 Host Application이라고 부름)

UI Tests 의 경우, test bundle이 앱과 분리된 UI test runner로 test bunlde이 load 된다 (application 을 Target Application)

**Unit Tests** 

- 앱의 자료 구조, API 를 직접적으로 접근
- host application의 같은 실행 아래서 작동
- 그래서 test invocation 사이에서 clean up 해주어야함

**UI Tests** 

- Accessibility를 통해 앱에 접근, 앱에 event를 보내고, 외부에서 유저가 사용하는 것 처럼 봐야함
- 테스트가 terminate되고 relaunch 될 수 있다

### Test Observation

![](/Jinha/images/Advanced-Testing-and-Continuous-Integration-2.png)

**XCTestObservation Protocol**

protocol을 따르고 shared observation center에 등록 하면 call back을 받는다 

```Swift
    override init() {
    	super.init()
    	
    	let sharedCenter = XCTestObservationCenter.shared()
    	sharedCenter.addTestObserver(self)
    }
```

```Swift
    // 테스트 시작 전에
    optional public func testBundleWillStart(_ testBundle: AnyObject!)
    
    // suite 시작 전에
    optional public func testSuiteWillStart(_ testSuite: AnyObject!)
    
    //test case를 run 시
    optional public func testCaseWillStart(_ testCase: AnyObject!)
    
    // test case 종료 시
    optional public func testCaseDidFinish(_ testCase: AnyObject!)
    
    // 뭔가 잘못 됐을 때
    optional public func testCase(_ testCase: AnyObject!, didFailWithDescription description: AnyObject!, inFile filePath: AnyObject!, atLine lineNumber: AnyObject!)
    
    // 테스트 마쳤을 떄
    optional public func testSuiteDidFinish(_ testSuite: AnyObject!)
    
    // 작업을 더 처리 할 수 있는 마지막 콜백
    optional public func testBundleDidFinish(_ testBundle: AnyObject!)
```

## Crash Log Gathering

### Crashes during testing

- Crash는 테스트 failure 되는 빈번한 원인이다.
- test host application, target application 모두 crash 가 날 수 있다
- Xcode는 보통 test suite을 완성시키기 위해서 host application을 relaunch 할 거다

### 🆕 Crash Log Gathering

Xcode가 test report에 crash log들을 모아서 줄것이야

UI, Unit tests 둘 다 . local , Xcode Server 둘다.

## Xcode Server

![](/Jinha/images/Advanced-Testing-and-Continuous-Integration-3.png)

## Issue Tracking and Blame

![](/Jinha/images/Advanced-Testing-and-Continuous-Integration-4.png)

사람은 완벽하지 않아~ 완벽한 코드도 없어~ 그래서 Unit test를 하는거지. 

그리고 가끔 빌드 안되는 코드를 커밋 하지~ 그래서 이런 경우에 Xcode는 이메일을 보내 줄거야 너가 코드를 broke했다고 알려주려고

이거 뿐만 아니라 너가 완벽한 코드를 짰다고 해도, 너의 주변의 것들이 변할 수가 있지. 예를 들면 새 Xcode를 받았을 때, 새로운 SDK를 받고 새로운 deprecation들이 있을 수 있어. 언어의 improvement로 이슈가 생길 수 도 있지. 그런데 모든 Xcode는 그 전 꺼 보다 smart해. 그래서 그전에 없던 이슈를 발견 할 수 도 있어 Xcode가 추적 해주기 때문이지

![](/Jinha/images/Advanced-Testing-and-Continuous-Integration-5.png)
