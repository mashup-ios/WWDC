# Swift in Practice

> 📅 2019.11.07 (목)

WWDC 2015
Session :  411
Category : Swift

🔗

[Swift in Practice - WWDC 2015 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2015/411/)

## Asset Catalog Identifiers

```Swift
    let isabellaImage = UIImage(named: "Isabella")!
    
    let williamImage = UIImage(named: "William")!
    
    let oliviaImage = UIImage(named: "Olivia")!
```

앱에서 Asset에 있는 이미지들을 이런 식으로 쓰려면 unwrap 해주어야 한다. 이미지는 3개가 있지만 코드 내부에서 얼마나 많이 쓰이고 있을지 모름 

→ **String Constants**

```Swift
    let IsabellaUnicornImageName = "Isabella"
    
    let imsabellaImage = UIImage(named: IsabellaUnicornImageName)!
```

하지만 이것 또한 unwrap 해야 한다. compiler나 framework는 유효한 이름 인지 아닌지 모름! 

### String as Distinct Types

Strongly typed solution이 필요

요구사항

- String을 새로운 타입으로 Mapping
- 초기화 시 실패 하지 않는 UIImage

해결방법

**application specific enumeration**

```Swift
    let isabellaImage = UIImage(assetIdentifier: .Isabella)
    
    let williamImage = UIImage(assetIdentifier: .William)
    
    let oliviaImage = UIImage(assetIdentifier: .Olivia)
```

이런 식으로 UIImage를 생성 할 때마다 enum을 전달하고, unwrap 필요 없게 하고 싶어

```Swift
    extension UIImage {
    	enum AssetIdentifier: String {
    		case Isabella = "Isabella"
    		case William = "William"
    		case Olivia = "William"		
    	}
    }
```

이렇게 value가 중복 될 경우 컴파일러가 에러를 알려준다.		`error: raw value for enum case is not unique`

```Swift
    let isabellaImage = UIImage(assetIdentifier: .Isabella)
    
    let williamImage = UIImage(assetIdentifier: .William)
    
    let oliviaImage = UIImage(assetIdentifier: .Oliia)
```

asset정보를 structure로 코드에 포함 시켜주었기 때문에, 이제는 오타가 생겼을 때 컴파일러가 에러 알려줌

`error: 'UIImage.AssetIdentifier.Type' does not have a member name 'Oliia"`

### AssetIdentifier Enum Benefits

- Centrally located constants : 이미지 asset이 추가 되었을 때, 어디에 constant를 추가 해주어야 하는지 명확함
- Doesn't pollute global namespace : global namespace를 더럽히지 않는다
- Must use one of the enum cases : enum case 안에서만 사용 할 수 있다
- Image initializers are not failbale : 언제나 non-optional image를 return 해준다

## Segue Identifiers

```Swift
    override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
    	switch segue.identifier {
    		case "ShowImportUnicorn"?: 
    		case "ShowCreateNewUnicorn"?: 
    		default: fatalError("Invalid segue identifier \(segue.identifier).")
    	}
    }
```

스토리보드에 segue들을 정의해 놓았어도 컴파일러는 몰라

그래서 default 안 넣어 주면 `error: switch mush be exhaustive, consider adding a default case` 

그리고 만약에 segue를 추가 해주었을 때, 어디에 코드를 추가 해야 되는지 하나하나 다 찾아야 함

→ enum으로 해결

```Swift
    override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
    	switch segue.identifier {
    		case "ShowImportUnicorn"?: 
    		case "ShowCreateNewUnicorn"?: 
    		default: fatalError("Invalid segue identifier \(segue.identifier).")
    	}
    }
```

```Swift
    override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
    	guard let identifier = segue.identifier, segueIdentifier = SegueIdentifier(rawValue: identifier)
    	else { fatalError("Invalid segue identifier \(segue.identifier).") }
      
      switch segueIdentifier {
    		case .ShowImportUnicorn: 
    		case .ShowCreateNewUnicorn: 
    	} 
    }
```

segue 추가 시에, 코드 추가 안해 주면 `error: switch must be exhaustive, consider adding a default case` 가 뜨기 때문에 어디에 로직을 업데이트 해야하는지 바로 알 수 있다.

```Swift
    class UnicornBrowserViewController: UIViewController {
        func handleAction(sender: AnyObject?) {
    			performSegueWithIdentifier(.ShowImportUnicorn, sender: sender) 
    		}
    }
```

![](/Jinha/images/Swift-in-Practice-1.png)

protocol을 사용해 하나의 implementation를 여러 다른  ViewController에서  계층에 상관 없이 사용 할 수 있다.

ViewController는 아래 protocol을 conform해야함

```Swift
    protocol SegueHandlerType {
        typealias SegueIdentifier: RawRepresentable
    }

    extension SegueHandlerType where Self: UIViewController, SegueIdentifier.RawValue == String {
    	func performSegueWithIdentifier(segueIdentifier: SegueIdentifier, sender: AnyObject?) { 
    		performSegueWithIdentifier(segueIdentifier.rawValue, sender: sender)
    	}
    }
```

위의 프로토콜만 ViewController에 추가해주면 된다!

```Swift
    class UnicornBrowserViewController: UIViewController, SegueHandlerType {
        enum SegueIdentifier: String {
    		...
    	} 
    
    	func handleAction(sender: AnyObject?) {
    		performSegueWithIdentifier(.ShowImportUnicorn, sender: sender)
    	}
    }
```

```Swift
    func segueIdentifierForSegue(segue: UIStoryboardSegue) -> SegueIdentifier {
    	guard let identifier = segue.identifier,
    	segueIdentifier = SegueIdentifier(rawValue: identifier)
    	else { 
    		fatalError("Invalid segue identifier \(segue.identifier).") }
        
    		return segueIdentifier
    }
```

```Swift
    override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
        switch segueIdentifierForSegue(segue) {
    			case .ShowImportUnicorn: 
    			case .ShowCreateNewUnicorn: 
    		} 
    }
```

**SegueHandlerType Protocol Benefits**

- 새로운 segue가 추가 될 때 마다 아직 해당 segue가 처리 되지 않았으면, 컴파일러가 에러를 알려준다
- protocol을 통한 재사용
- 편리한 문법
