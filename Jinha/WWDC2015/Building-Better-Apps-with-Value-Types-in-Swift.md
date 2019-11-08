# Building Better Apps with Value Types in Swift

> 📅 2019.11.8 (금)
> WWDC 2015 | Session : 414 | Category : Swift

🔗 [Building Better Apps with Value Types in Swift - WWDC 2015 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2015/414/)

# Reference Semantic

### Unintented Sharing

![](/Jinha/images/Building-Better-Apps-with-Value-Types-in-Swift-1.png)

**Copying in Cocoa[Touch] and Objective-C**

Cocoa[Touch] and Objective-C 에서는 어디서 복사를 사용 할까

- `NSCopying` 프로토콜 개념을 사용하고 있다
- `NSString`, `NSArray`, `NSDictionary`, `NSURLRequest` 등 모두 안전하게 copy 하기 위해 `NSCopying`을 conform함

Defensive **Copying in Cocoa[Touch] and Objective-C**

copy가 필요한 시스템이고 엄청 좋은 이유로 방어적인 copy를 많이 볼 수 있다

- `NSDictionary`는 Dictionary의 key를 defensive copy 할 거다
왜냐면 `NSDictionary`에 추가할 key를 다룰 때, 그리고 변경 하려고 할 때,  hash value로 바꾸면 `NSDictionary`의 내부적인 분산이 다 깨질거고 bug라고 비난 하겠지
- 그래서 densive copy를 한다. 그런데 이런 추가적인 copy 때문에 성능을 잃는다
- 모든 assignment마다 defensive copying을 하는 Objective-C의 copy property를  language 레벨에서 다루지만 충분하지 않다

**Is Immutability the Answer?**

 reference sematic이 자체적으로 mutation하기 때문에 문제가 있다

→  reference semantic 의 immutable data structure를 바꿀 필요가 있다

## Elimination Mutation

unintended side effects에 대한 고민은 없어 졌지만, 불편한 인터페이스에 이르게 되고, machine model에 효율적으로 매핑 되지 못할 수 있다.

### An Immutable Temperature Class

```Swift
    class Temperature {
    	let celsius: Double = 0 // 
    	var fahrenheit: Double { return celsius * 9 / 5 + 32 }
    
    	init(celsius: Double) { self.celsius = celsius }
    	init(fahrenheit: Double) { self.celsius = (fahrenheit - 32) * 5 / 9 }
    }
```

저장 프로퍼티 `celsius`를 `var` 에서 `let`으로 바꾸고, 계산 프로퍼티인 `fahrenheit`는 setter를 잃게 된다. 

**Awkward Immutable Interfaces**

```Swift
    With mutability
    home.oven.temparture.fahrenheit += 10.0
    
    Without mutability
    let temp = home.oven.temperature
    home.oven.temperature = Temperature(fahrenheit: temp.fahrengiet + 10.0)
```

더 많은 코드가 필요하고, heap에 다른 객체를 allocationg 하는데 시간을 더 소모한다.

하지만 여전히 oven 스스로 mutate하는 assignment가 있다

## Sieve of Eratosthenes

mutation 기반의 알고리즘

![](/Jinha/images/Building-Better-Apps-with-Value-Types-in-Swift-2.png)

![](/Jinha/images/Building-Better-Apps-with-Value-Types-in-Swift-3.png)

mutation 사용하지 않는 알고리즘

![](/Jinha/images/Building-Better-Apps-with-Value-Types-in-Swift-4.png)

## Immutability in Cocoa[Touch]

- immutable class : `NSData`, `NSURL`, `UIImage`, `NSNumber` 등
- 안전성 향상( copy를 사용 할 필요가 없고 의도하지 않은 side effect를 생기게 하는 sharing에 대해서도 걱정 할 필요가 없다)

Downsides to immutability

```Swift
    NSURL *url = [[NSURL alloc] initWithString: NSHomeDirectory()];  NSString *component; 
    
    while ((component = getNextSubdir()) { 
    	url = [url URLByAppendingPathComponent: component];  
    }
```

reference semantic world에서 mutation이 없게 하기 위해서 `NSURL`을 생성하고 매 루프마다  `URLByAppendingPathComponent`를 통해 url을 만든다.

`NSURL` 을 매번 생성하고, 이전 객체는 사라지고, `NSURL`이 모든 루프마다 string data를 복사 하기 때문에 효율적이 지 않다.

```Swift
    NSArray<NSString *> *array = [NSArray arrayWithObject: NSHomeDirectory()];  NSString *component; 
    
    while ((component = getNextSubdir()) { 
    	array = [array arrayByAddingObject: component];  
    }
     
    url = [NSURL fileURLWithPathComponents: array];
```

array를 만들 때  특정 객체(home directory)와 함께  `NSArray` 를생성 할 거고, 매번 하나의 객체를 더해서 새로운 array를 생성할거야.

String data를 copy하지는 않지만 여전히 data를 copy 하고 있다

Thoughtful mutability

```Swift
    NSMutableArray<NSString *> *array = [NSMutableArray array];  [array addObject: NSHomeDirectory()]; 
    NSString *component; 
    
    while ((component = getNextSubdir()) { 
    	[array addObject: component];  
    } 
    url = [NSURL fileURLWithPathComponents: array];
```

모든 component들을 `NSMutableArray` 에 모으자. 그리고 `fileURLWithPathComponents` 를 사용해서 immutable `NSURL` 에 넣자

# Value Semantics

두개의 변수를 가지고 있고, 변수의 값들은 논리적으로 분리 되어있다. 
A를 B에 copy하고 A 값을 변경해도 B에 아무런 영향을 주지 않는다.

### Value Types Compose

- Swift의 fundamental type :`Int`, `Double`, `String`, `Character` 등
- Swift의 모든 collections : `Array`, `Set`, `Dictionary`
- value type을 포함한 tuple, struct, enum
- Value type은 Equatable를 구현해주어야 한다

![](/Jinha/images/Building-Better-Apps-with-Value-Types-in-Swift-5.png)

**Mutation When You Want It**

`let` 은 값이 절대 변하지 않는 다는 것 → 프로세스의 메모리를 손상시키지 않는다

`var` 가 여전히 mutation을 하지만 local에만 영향을 미친다

### Copies Are Cheat (Constant time)

value type은 매번 값 복사를 하지만 비용이 적다

- Copying a low-level, fundamental type is constant time: `Int` , `Double` , `Float` byte들만 copy하면 되니까
- Copying a struct,enum,or tuple of value types is constant time: `CGPoint` 는 `CGFloat` 로 이루어져 있고, struct, enum 얘네들은 고정된 숫자 필드를 가지고 있다
- Extensible data structures use copy-on-write
    - Copying involves a fixed number of reference-counting operations
    - `String` , `Array` , `Set` , `Dictionary`
    - copy-on-write value를 copy 하면 reference-counting operation도 고정 값

    ### Value Semantics Are Simple and Efficient

    - Different variables are logically distinct
    - Mutability when you want it
    - Copies are cheap
