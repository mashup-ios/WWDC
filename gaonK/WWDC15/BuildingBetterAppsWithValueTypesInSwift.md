# [Building Better Apps with Value Types in Swift](https://developer.apple.com/videos/play/wwdc2015/414/)

@ WWDC 15

### Reference Semantics

#### A Temperature Class

```swift
class Temperature {
  var celsius: Double = 0
  var fahrenheit: Double {
    get { return celsius * 9 / 5 + 32 }
    set { celsius = (newValue - 32) * 5 / 9 }
  }
}
```

#### Using Our Temperature Class

```swift
let home = House()
let temp = Temperature()
temp.fahrenheit = 75
home.thermostat.temperature = temp

temp.fahrenheit = 425
home.oven.temperature = temp
home.oven.bake()
```

#### Unintended Sharing

home의 temperature와 oven의 temperature가 같은 인스턴스를 참조하고 있다.

sharing을 막기 위해서는 copy가 필요하다.

#### Manual Copying

```swift
let home = House()
let temp = Temperature()
temp.fahrenheit = 75
home.thermostat.temperature = temp.copy()

temp.fahrenheit = 425
home.oven.temperature = temp.copy()
home.oven.bake()
```

#### Defensive Copying

```swift
class Oven {
  var _temperature: Temperature = Temperature(celsius: 0)
  
  var temperature: Temperature {
    get { return _temperature }
    set { _temperature = newValue.copy() }
  }
}
```

#### Defensive Copying in Cocoa[Touch] and Objective-C

* Cocoa[Touch] requires copying throughout
  * `NSCopying` codifies copying an object
  * `NSString`, `NSArray`, `NSDictionary`, `NSURLRequest`, etc. all require copying
* Defensive copying pervades Cocoa[Touch] and Objective-C
  * `NSDictionary` calls `-copy` on its key
  * Property `copy` attribute provides defensive copying on assignment
  * 필요하기 때문에 `copy` 하지만 성능이 조금 떨어지게 된다.



### Immutability

#### Eliminating Mutation 

>  reference semantics의 문제가 아니라 mutation의 문제가 아닐까? 하는 관점

* Functional programming languages have reference semantics with immutability
* Eliminates many problems caused by reference semantics with mutation
  * No worries about unintended side effects

* Several notable disadvantages > Immutability의 단점도 있다
  * Can lead to awkward interfaces
  * Does not map efficiently to the machine model

#### An Immutable Temperature Class

```swift
class Temperature {
  let celsius: Double = 0
  var fahrenheit: Double { return celsius * 9 / 5 + 32 }
  
  init(celsius: Double) { self.celsius = celsius }
  init(fahrenheit: Double) { self.celsius = (fahrenheit - 32) * 5 / 9 }
}
```

#### Awkward Immutable Interfaces

```swift
// With mutability
home.oven.temperature.fahrenheit += 10.0

// Without mutability
let temp = home.oven.temperature
home.oven.temperature = Temperature(fahrenheit: temp.fahrenheit + 10.0)
```

Immutability 때문에 직관적이지 않은, 어색한 방식으로 코드가 작성될 수 있다. 사실 '='를 사용했기 때문에 또 다른 mutability라고 볼 수도 있다.

#### Sieve of Eratosthenes (에라토스테네스의 체 알고리즘)

```swift
func primes(n: Int) -> [Int] {
  var numbers = [Int](2..<n) // Create an array
  for i in 0..<n-2 {
    guard let prime = numbers[i] where prime > 0 else { continue }
    for multiple in stride(from: 2 * prime-2, to: n-2, by: prime) {
      numbers[multiple] = 0
    }
  }
  return numbers.filter { $0 > 0 } // Only prime numbers
}
```

very simple algorithm!

#### Functional Sieve of Eratosthenes

하스켈 버전! Immutable!

```haskell
primes = sieve [2..]
sieve [] = []
sieve (p : xs) = p : sieve [x | x <- xs, x 'mod' p > 0]
```

Swift 버전!

```swift
func sieve(numbers: [Int]) -> [Int] {
  if numbers.isEmpty { return [] }
  let p = numbers[0]
  return [p] + sieve(numbers[1..<numbers.count].filter { $0 % p > 0 })
}
```

> The Genuine Sieve of Eratosthenes

이 non-mutating version은 mutating version에 비해 효율적이지 못하다.

#### Immutability in Cocoa[Touch]

* Cocoa[Touch] has a number of immutable classes

  * `NSDate`, `NSURL`, `UIImage`, `NSNumber`, etc.
  * Improved safety (no need to use copy)

* Downsides to immutability

  ```objective-c
  NSURL *url = [[NSURL alloc] initWithString: NSHomeDirectory()];
  NSString *component;
  while (component == getNextSubdir()) {
    url = [url URLByAppendingPathComponent: component];
  }
  ```

  계속 object를 생성하는 이 방식은 비효율적이다.

  ```objective-c
  NSArray<NSString *> *array = [NSArray arrayWithObject: NSHomeDirectory()];
  NSString *component;
  while (component == getNextSubdir()) {
    array = [array arrayByAddingObject: component];
  }
  url = [NSURL fileURLWithPathComponents: array];
  ```

* Thoughtful mutability

  ```objective-c
  NSMutableArray<NSString *> *array = [NSMutableArray array];
  NSString *component;
  while (component == getNextSubdir()) {
    array = [array addObject: component];
  }
  url = [NSURL fileURLWithPathComponents: array];
  ```

  

### Value Semantics

mutability의 장점도 있다. 걱정되는 것은 sharing! 그래서 value semantics를 사용한다.

#### Variables Are Logically Distinct

* Mutating one variable of some value type will never affect a different variable

  ```swift
  var a: Int = 17
  var b = a
  assert(a == b)
  b += 25
  print("a = \(a), b = \(b)") // a = 17, b = 42
  ```

#### Value Types Compose

* 스위프트의 모든 기본(fundamental) 타입은 value 타입이다.
  * Int, Double, String, ...
* 스위프트의 모든 collection은 value 타입이다.
  * Array, Set, Dictionary, ...
* 스위프트의 tuple, struct, enum 같은 value 타입을 포함하는 것들도 value 타입이다.

#### Value Types Are Distingushed by Value

* Value 타입은 값에 의해 구분된다.
  * Identity 구분할 수 없다.
  * 어떻게 값을 가지는 지는 중요한 포인트가 아니다.

#### Equatable

> Value 타입은 Equatable을 구현해야 한다.

```swift
protocol Equatable {
  /// Reflexive - `x == x` is `true`
  /// Symmetric - `x == y` then `y == x`
  /// Transitive - `x == y` and `y == z` then `x == z`
  func ==(lhs: Self, rhs: Self) -> Bool
}
```

#### Using Value Semantics Everywhere

```swift
var home = House()
var temp = Temperature()
temp.fahrenheit = 75
home.thermostat.temperature = temp

temp.fahrenheit = 425
home.oven.temperature = temp
home.oven.bake()
```

#### Mutation When You Want It

> But not when you don't

`let`은 값이 절대 바뀌지 않음을 의미

```swift
let numbers = [1, 2, 3, 4, 5]
```

`var`은 다른 값들에는 영향을 주지 않으면서 값을 업데이트할 수 있음을 의미

```swift
var strings = [String]()
for x in numbers {
  strings.append(String(x))
}
```

#### Freedom from Race Conditions

```swift
var numbers = [1, 2, 3, 4, 5]
schedule.processNumbersAsynchronously(numbers)
for i in 0..<numbers.count { numbers[i] = numbers[i] * i }
schedule.processNumbersAsynchronously(numbers)
```

Reference semantics를 가졌었더라면 numbers는 Race condition을 겪게 되었을 것이다.

#### Copies Are Cheap

> Constant time

* Copying a low-level, fundamental type is constant time
  * Int, Double, ...
* Copying a struct, enum, or tuple of value types is constant time
  * CGPoint, ...
* Extensible data structures use copy-on-write
  * Copying involves a fixed number of reference-counting operations
  * String, Array, Set, Dictionary, ...



### Mixing Value Types and Reference Types

#### Reference Types Often Contain Value Types

* Value types generally used for "primitive" data of objects

  ```swift
  class Button: Control {
    var label: String // primitive data
    var enabled: Bool // primitive data
  }
  ```

#### A Value Type Can Contain a Reference

* Value 타입을 복사하면 reference를 공유한다.

  ```swift
  struct ButtonWrapper {
    var button: Button // ButtonWrapper를 복사한 객체는 이 button을 공유한다.
  }
  ```

* Value semantics를 유지하려면 특별한 고려가 있어야 한다.

  * Referened object의 mutation을 어떻게 관리할 지
  * Reference의 identity가 equality에 어떤 영향을 미칠 지

#### Copy-on-Write

* Referenced object에 대한 제한되지 않은 mutation은 value semantics를 망가뜨린다
* Mutating 시키는 작업을 non-mutating 작업들로 분리한다
  * Non-mutating 작업은 항상 안전하다.
  * Mutating operations must first copy

```swift
struct BezierPath: Drawable {
  private var _path = UIBezierPath()
  
  var pathForReading: UIBezierPath {
    return _path
  }
  
  var pathForWriting: UIBezierPath {
    mutating get {
      _path = _path.copy() as! UIBezierPath
      return _path
    }
  }
}

extension BezierPath {
  var isEmpty: Bool {
    return pathForReading.empty
  }
  
 	mutating func addLineToPoint(point: CGPoint) {
    pathForWriting.addLineToPoint(point)
  }
}
```

#### Uniquely Referenced Swift Objects

```swift
struct MyWrapper {
  var _object: SomeSwiftObject
  var objectForWriting: SomeSwiftObject {
    mutating get {
      if !isUniquelyReferencedNonObjC(&_object) {
        _object = _object.copy()
      }
      return _object
    }
  }
}
```



### Summary

* Reference semantics and unexpected mutation
* Value semantics solve these problems
* Expressiveness of mutability, safety of immutability

