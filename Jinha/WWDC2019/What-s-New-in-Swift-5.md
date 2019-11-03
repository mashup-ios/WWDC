# What's New in Swift

Session 402

🔗 [What's New in Swift - WWDC 2019 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2019/402/)

## APIs

- Shared Swift Runtime for Apps
- Binary Frameworks
- Apple Swift-Only Frameworks
- Package Manager in Xcode
- More API Expressibility
- Library Evolution

## ABI and Module Stability

### What is ABI?

ABI = Application Binary Interface

컴파일 코드가 런타임에 어떻게 상호작용 할 것 인지에 대한 규칙

![](Untitled-d149d3ff-95a5-46e9-8b25-40f036b5723a.png)

exec : Swift로 짠 프로그램

framework : Swift로 짠 framework

이 모든게 같이 컴파일 되고 running process에서 작동한다

exec 는 framework의 API를 사용하고, 런타임에 서로 소통한다. 이 두개는 서로 독립적으로 컴파일 되지만 컴파일 코드는 같이 작동해야한다.
ABI stability 전에는,  같은 컴파일러로 빌드 되어야만 ABI가 호환 가능했다.

하지만 Swift5 에서는 이 두개가 같은 컴파일러에서 빌드 될 필요가 없다.

### What is Module Stability?

- 컴파일 타임 개념
- Module: Swift 프레임워크와 그 안의 모든 API를 사용할다면 Module이라고 부르는 공유되는 namespace가 있다.
- Module file: 컴파일러가 프레임워크를 빌드할 때, 프레임워크의 모든 API를 나열해 manifest를 만든다. Swift Module file이라고 불리는 manifest
- Module Interface file: Swift 5.1에서 보충적인 manifest를 만들었다. 프레임워크가 stable interface를 제공 하는 데에 사용할 수 있는

![](Untitled-a3100707-ab02-4450-afdb-465bda4e51ff.png)

> Module + ABI stability = Binary frameworks

## Performance

런치 타임, 코드 사이즈, Obj-C Swift 코드 bridging(ex. NSDictionary to Dictionay) 다 감소했다~

## Language and Standard Library

[Swift Evolution](https://apple.github.io/swift-evolution/)

![](Untitled-23d523ab-8ede-4e0a-90e7-bc7e5233e06e.png)

single expression 일때 return 생략 가능

![](Untitled-04f5c235-7e3e-4ce4-a695-8c4053603c72.png)

Struct 디폴트 값이 있다면 초기화 시에 명시 해주지 않아도 된다 컴파일러가 해준다

![](Untitled-10a874e5-0087-44f1-a89b-50276fd21528.png)

벡터 좌표에 대한 타입이 생겼다

### New Design for String Interpolation

### Reasons to Abstract a Return Type

### Limitations of Returning a Protocol Type

### Opaque Result Type

### Property Wrapper Types

    @propertyWrapperstruct 
    UserDefault<T> {
    	let key: String
    	let defaultValue: T
    
    	init(_ key: String, defaultValue: T) {
    		...
    		UserDefaults.standard.register(defaults: [key: defaultValue])  
    	}
    
    	var value: T {
    		get {
    			return UserDefaults.standard.object(forKey: key) as? T ?? defaultValue
    		}
    		set {
    			UserDefaults.standard.set(newValue, forKey: key)
    		}
    	}
    }

    // Using UserDefault property wrapper to declare and access properties
     @UserDefault("USES_TOUCH_ID", defaultValue: false)
     static var usesTouchID: Bool
    
     @UserDefault("LOGGED_IN", defaultValue: false)
     static var isLoggedIn: Bool

### DSLs(Domain Specific Languages)

SwiftUI파티
