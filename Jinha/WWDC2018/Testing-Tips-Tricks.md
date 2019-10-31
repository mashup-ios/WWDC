# Testing Tips & Tricks

Session 417

[Testing Tips & Tricks - WWDC 2018 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2018/417/)

## Working with Notifications

Notification 은 1대 다로 소통하는 메커니즘이다.
그래서 의도하지 않은 사이드 이팩트를 피하기 위해 분리된 방식으로 테스트 할 필요가 있다.

    class PointsOfInterestTableViewController {
        var observer: AnyObject?
        init() {
    			let name = CurrentLocationProvider.authChangedNotification
    			observer = NotificationCenter.default.addObserver(forName: name, object: nil,
    				self?.handleAuthChanged() 
    			}
    		}
        
    		var didHandleNotification = false
        func handleAuthChanged() {
            didHandleNotification = true
        }
    }

default NotificatonCenter를 통해 observer를 등록 해주고 있고 실제로 notification을 받았는지 flag값을 통해 확인 해주고 있다.

    class PointsOfInterestTableViewControllerTests: XCTestCase {
    	func testNotification() {
    		let observer = PointsOfInterestTableViewController() 
    		XCTAssertFalse(observer.didHandleNotification)
    	
    		let name = CurrentLocationProvider.authChangedNotification 
    		NotificationCenter.default.post(name: name, object: nil)
    		
    		XCTAssertTrue(observer.didHandleNotification) 
    	}
    }

ViewController가 사용하는 Notification Center에 post 해주고 있다.

`UIApplication`과 `appDidFinishLaunching` 같은 notification은 여러 레이어에서 observe 하고 있다.
그래서 사이드이팩트가 생기거나, 테스트가 느려 질 수가 있다.

→ 테스트를 분리해 주어야 한다.

NotificationCenter 은 여러 개의 인스턴스를 가질 수 있다. 
클래스 프로퍼티인 default 인스턴스를 갖고 있지만, 필요시에 추가 인스턴스를 생성 할 수 있다. 이것이 테스트를 분리 시키는 것의 핵심이다.

- `.default` 인스턴스 대신 사용할 `NotificationCeneter`를 생성 -> Dependency Injection

    class PointsOfInterestTableViewController {
    	let notificationCenter: NotificationCenter
    	var observer: AnyObject?
    
    	init(notificationCenter: NotificationCenter = .default) {
    		self.notificationCenter = notificationCenter
    		let name = CurrentLocationProvider.authChangedNotification
    		observer = notificationCenter.addObserver(forName: name, object: nil,
    		 
    			self?.handleAuthChanged() 
    	}
    }
    
    var didHandleNotification = false
    func handleAuthChanged() {
        didHandleNotification = true
    }
    }

init 시 unit tests시에 파라미터로 `NotificatioCenter`을 받을 수 있게 수정. 

    class PointsOfInterestTableViewControllerTests: XCTestCase {
        func testNotification() {
           let notificationCenter = NotificationCenter()
           let observer = PointsOfInterestTableViewController(notificationCenter:
                                                                      notificationCenter)
    				XCTAssertFalse(observer.didHandleNotification)
    		  
    			let name = CurrentLocationProvider.authChangedNotification notificationCenter.post(name: name, object: nil)
    			
    			XCTAssertTrue(observer.didHandleNotification) 
    		}
    }

테스트 코드도 `NotificationCenter` 을 따로 생성하도록 수정

## Mocking with Protocols

다른 클래스나 SDK 등과 상호작용 하는 클래스는 외부 클래스를 생성하는 것이 어렵거나 혹은 아예 불가능 하기 때문에, 테스트 작성하기가 어렵다. 특히 직접 생성 할 수 없게 디자인 된 API, Delegate 메소드를 사용하는 API 에서 어려움. 외부 클래스와의 상호작용을 mocking하고 프로토콜을 사용해 문제를 해결 할 수 있다. 테스트를 덜 의존적으로 만들면서.

## Test execution speed

    func application(_ application: UIApplication, didFinishLaunchingWithOptions opts: ...) -> Bool {
    	let isUnitTesting = ProcessInfo.processInfo.environment["IS_UNIT_TESTING"] == "1" 
    	if isUnitTesting == false {
    		// Do UI-related setup, which can be skipped when testing
    	}
    	return true
    }
