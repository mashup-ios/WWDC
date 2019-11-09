# Implementing UI Designs in Interface Builder

> 📅 2019.11.09 (토)
>
>WWDC 2015 | Session : 407 | Category : Interface Builder

🔗 [Implementing UI Designs in Interface Builder - WWDC 2015 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2015/407/)

## Design Time

* IBDesignable : custom drawing code를 Interface builder 에서 볼 수 있다

* IBInspectable :Interface builder에 inspector를 생성 해준다

![](/Jinha/images/Implementing-UI-Designs-in-Interface-Builder-1.png)

![](/Jinha/images/Implementing-UI-Designs-in-Interface-Builder-2.png)

## Build Time

![](/Jinha/images/Implementing-UI-Designs-in-Interface-Builder-3.png)

design 타임에는 XML document로 작업을 한다

build 타임에는 이 doument들을 실행하기 위해 프로세스가 IB 툴을 호출 하고 nib 파일로 컴파일 한다.

nib 파일은 작고, 최적화된 binary 파일이다. 

그리고 이것들을 생성하는데에 keyed archiving이라고 불리는 프로세스를 사용한다.

### Compiling Storyboards

인터페이스 빌더는 스토리보드를 컴파일 할 때,

1. 앱의 성능을 최대화 하려고 한다.
2. nib file 개수를 최소화 하려고 한다.

### Loading Storyboards At Run Time

![](/Jinha/images/Implementing-UI-Designs-in-Interface-Builder-4.png)

UIStoryboard를 통해 storyboard를 할당 할 때(`init(name: bundle:)`), UIStoryboard 인스턴스 스스로 모든 할당 메모리들을 초기화 한다. ViewController, View 모두 아직 없다

InitialViewController를 instantiate 할 때(`instantiateInitialViewController()`), InitialViewController의 nib을 로드 할 것이고, 아직 View hierarchy는 없다. 누군가 요청 하기 전까지는.

유사하게, NavigationController와 TableViewController도 `instantiateViewControllerWithIdentifier(_:)` 을 통해 ViewController을 가져올 수 있지만, 역시 view들은 요청 전까지는 아직 메모리에 로드되지 않는다.

Interface Builder은 자동으로 TableViewCell, nib을 가져오고 TableView와 함께 TableViewCell에 세팅 해 놓은 reuse identifer로 등록한다. 이것은 cell들이 누군가가 실제로 cell을 deque하기 전까지는 로드 되지 않는 다는 것을 의미 한다. 

일단 nib 런타임이 로드 되면(nib파일이 메모리에), 그것은 빠르게 instantiate된다는 것을 의미한다.

### TakeAways

- 성능 
→ nib 파일은 요구가 있을 때 만 로드 된다. nib 파일 자체는 매우 작고 최적화 되어 있다.
- 재사용
→ Interface Builder은 다른 nib 파일들을 재사용 한다. 예를 들어, 우리가 보는 TableViewCell은 일단 런타임이 nib 파일을 가지고 있다면 새로운 cell이 필요로 하는 만큼 아주 엄청 빠르게 재인스턴스화 할수 있다.
- 라이프사이클
→ 빌드 타임과 런 타임 사이의 라이프사이클을 확인 할 수 있다. ViewController와 View hierarchy 중 어떤 객체를 사용 할지 알 수 있다.

## Runtime

![](/Jinha/images/Implementing-UI-Designs-in-Interface-Builder-5.png)

런타임 시 가능 한 것

- 스토리보드와 소스 코드를 IBAction과 IBOutlet을 사용해 연결 할 수 있다.
- **Connections** : 스토리보드와 소스 코드를 `IBAction` 과 `IBOutlet` 을 통해 연결 할 수 있다
- **API** : Segue의 행동을 커스텀화 할 수 있다 혹은 동적으로 인스턴스화 하고 storyboard API를 사용해 ViewController들을 추가 할 수 있다
- **Adaptability** : AutoLayout이나 size 클래스를 사용해 컨테이너가 바뀔 때 사이즈에 맞게  UI를 적용 시킬 수 있다

### Connection

```Swift
    class AccountViewController : UIViewController {
        @IBOutlet var usernameLabel: UILabel!
    
    		override func viewDidLoad() { 
    			usernameLabel.text = username
    		}
        
    		var username: String? {
            didSet {
    					usernameLabel?.text = username 
    				}
    		}
    	 }
    
    		@IBAction func toggledAutoLoginSwitch(sender: UISwitch) { 
    			UserSettings.autoLogin = sender.on
    		}
        
    		@IBAction func tappedLoginButton() {
            if attemptLogin() {
                performSegueWithIdentifier("unwindAfterLogin", sender: nil)
            } else {
                performSegueWithIdentifier("presentLoginError", sender: nil)
            }
    		}
    }
```

`IBOutlet`  은 default로 암묵적인 언래핑(implicitly unwrapped optionals)을 하고 있다. ViewController와 hierarchy에 있는 뷰 사이에 outlet이 있다면, `viewDidLoad()` 이후에 안전하게 언래핑 할 수 있다. 가끔 ViewController에서 화면에 영향을 주는 추가적인 프로퍼티를 저장한다면 `didSet` 옵저버로 옵셔널 체이닝을 통해 언래핑 하면 된다.

`IBAction` 은 gesture reconizer과 control을 통해 전달 된 이벤트에 응답할 수 있게 해준다. 또한 이벤트 후에 동적으로 어떤 segue를 perform할지 정할 수 있다.

### API

```Swift
    UIStoryboard:
    	init(name:bundle:)
    	func instantiateInitialViewController()
    	func instantiateViewControllerWithIdentifier(_:)
    
    UIViewController:
        var storyboard: UIStoryboard? { get }
```

`UIStoryboard`나 맥의 `NSStoryboard` 클래스는 스토리보드 파일로 reference를 가져다주고 ViewController를 인스턴스화 해준다. 이것은 반복적으로 인스턴스화 해서 할 재사용 할  UI가 있을 때 유용하다.

```Swift
    UIViewController:
    	func prepareForSegue(_:sender:)
    	func performSegueWithIdentifier(_:sender:)
    	func shouldPerformSegueWithIdentifier(_:sender:) -> Bool func unwindForSegue(_:towardsViewController:)
    
    UIStoryboardSegue:
       func perform()
```

### Adaptability

**IBOutlet Strong Weak?**
보통 strong 사용. 특히 subView나 contraint에(view hierarchy에 의해 항상 retain되지 않는) outlet을 연결 할 때.

weak으로 연결 해야 할 때는, view hierarchy 뒤에 있는 view와 reference가 있는 custom view와 연결 할 때이지만 추천되지 않는다!
