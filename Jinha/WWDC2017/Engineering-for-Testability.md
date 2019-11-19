# Engineering for Testability

> 📅 2019.11.19 (화)
>
> WWDC 2017 | Session : 414 | Category : Testing

🔗

[Engineering for Testability - WWDC 2017 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2017/414/)

### Structure of a Unit Test

- Prepare input
- Run the code being tested
- Verify output

### Characteristics of Testable Code

- Control over inputs
- Visibility into outputs
- No hidden stage

### Testability Techiques

- Protocols and parameterization
- Seperarting logic and effects

## Protocols and Parameterization

![](/Jinha/images/Engineering-for-Testability-1.png)

```Swift
    @IBAction func openTapped(_ sender: Any) {
    	let mode: String
    	switch segmentedControl.selectedSegmentIndex {
    	case 0: mode = "view"
    	case 1: mode = "edit"
    	default: fatalError("Impossible case")
    	}
    	let url = URL(string: "myappscheme://open?id=\(document.identifier)&mode=\(mode)")!
    
     
    	if UIApplication.shared.canOpenURL(url) {
    		UIApplication.shared.open(url, options: [:], completionHandler: nil)
    	} else {
    		 handleURLError()
    	} 
    }
```

### UI Test

앱을 런칭해서 screen으로 이동해서, 탭 메뉴를 누르고, Open 버튼을 눌러서 다른 앱으로 변경 되었는지 확인

→ run 하는데 오래 걸린다. 특히 몇개의 다른 doument 까지 확인 하려면.
더 큰 문제는, UI Test는 앱을 바꾸기 위한 URL을 검사 할 수 없다

여기서는 URL을 테스트 하고 싶은 거기 때문에 Unit Test가 적절하다.

### Unit Test

```Swift
    func testOpensDocumentURLWhenButtonIsTapped() {
    	let controller = UIStoryboard(name: "Main", bundle: nil).instantiateViewController(withIdentifier: "Preview") as! PreviewViewControllercontroller
    
    	controller.loadViewIfNeeded()
    	controller.segmentedControl.selectedSegmentIndex = 1
    	controller.document = Document(identifier: “TheID")
    	
    	controller.openTapped(controller.button)
    
    	
    }
```

위의 코드 중 `if UIApplication.shared.canOpenURL(url)` 의 result 값은, 메소드의 다른 인풋에 영향을 준다

하지만 이것은 global system state에 의존 하기 때문에, result의 query를 프로그램적으로 테스트 할 방법이 없다

```Swift
    class DocumentOpener {
    	enum OpenMode: String {
    		case view
    		case edit
    	}
    	
    	func open(_ document: Document, mode: OpenMode) {
    		let modeString = mode.rawValuelet url = URL(string: "myappscheme://open?id=\(document.identifier)&mode=\(modeString)")!
    		
    		if UIApplication.shared.canOpenURL(url) {
    				UIApplication.shared.open(url, options: [:], completionHandler: nil)
    		} else {
    			handleURLError()
    		}
    	}
    }
```

- ViewController로 부터 빼고 시작해보자
- 로직과 행동을 encapsulate 하기 위한 Document opener class이다
- 하지만 여전히 `UIApplication.shared.canOpenURL(url)` 을 고칠 필요가 있다.

    class DocumentOpener {
    	let application: UIApplicationinit(application: UIApplication) {
    		self.application = application
    	}
    
```Swift
    	func open(_ document: Document, mode: OpenMode) {
    		let modeString = mode.rawValue
    		let url = URL(string: "myappscheme://open?id=\(document.dientifier)&mode=\(modeString")!
    
    		if application.canOpenURL(url) {
    			application.open(url, options: [:], completionGandler: nil)
    		} else {
    			handleURLError()
    		}
    	}
    }
```

```Swift
    class MockURLOpener: URLOpening {
    	 var canOpen = false
    	 var openedURL: URL?
     
    	func canOpenURL(_ url: URL) -> Bool {
    		return canOpen
    	}
    
    	func open(_ url: URL,options: [String: Any],completionHandler: ((Bool) -> Void)?) {
    		openedURL = url
    	} 
    }
``` 

## Separating Logic and Effects

### Balance between UI and Unit Tests

Unit tests great for testing small, hard-to-reach code paths

UI tests are better at testing integration of larger pieces

### Writing code to help UI tests scale

Abstracting UI element queries

- Store parts of queries in a variable
- Wrap complex queries in utility methods
- Reduces noise and clutter in UI test

Creating objects and utility functions

Utilizing keyboard shortcuts

###
