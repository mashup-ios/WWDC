# [Protocol-Oriented Programming in Swift](https://developer.apple.com/videos/play/wwdc2015/408/)

@ WWDC 15

### Class의 사용이 멋진 이유

* Encapsulation
* **Access Control**
* **Abstraction**
* **Namespace**
* Expressive Syntax
* Extensibility



### Class 사용의 단점

1. Implicit Sharing (여러 곳에서 한 레퍼런스를 참조하고 있다면 당연히 여러 문제점들이 발생)

   * Defensive Copying
   * Inefficiency
   * Race Conditions
   * Locks
   * More Inefficiency
   * Deadlock
   * Complexity
   * Bugs!

2. Inheritance All Up In Your Business

   * One superclass - 잘 골라야 함
   * Single Inheritance weight gain
   * No retroactive modeling
   * Superclass may have stored properties
     * You must accept them
     * Initialization burden
     * Don't break superclass invariants!
   * Know what/how to override (and when not to)

3. Lost Type Relationships

   ```swift
   class Ordered {
     func precedes(other: Ordered) -> Bool { fatalError("implement me!") }
   }
   
   class Number: Ordered {
     var value: Double = 0
     override func preces(other: Ordered) -> Bool {
       // Ordered 클래스가 value를 가지지 않기 때문에
       // 이런 처리를 해주고 있다...
       return value < (other as! Number).value 
     }
   }
   ```



### A Better Abstraction Mechanism

* Supports value types (and classes)
* Supports static type relationships (and dynamic dispatch)
* Non-monolithic
* Supports retroactive modeling
* Doesn't impose instance data on models
* Doesn't impose initialization burdens on models
* Makes clear what to implement



### Starting Over with Protocols

```swift
protocol Ordered {
  func precedes(other: Self) -> Bool // 자기 자신 타입과 동일한 타입을 이용하도록
}

struct Number: Ordered {
  var value: Double = 0
  func precedes(other: Number) -> Bool {
    // 클래스를 상속 받아 구현했을 때와는 달리 에러 발생 x
    return self.value < other.value 
  }
}
```



### Using Our Protocol

```swift
func binarySearch<T: Ordered>(sortedKeys: [T], forKey k: T) -> Int {
  var lo = 0
  var hi = sortedKeys.count
  while hi > lo {
    let mid = lo + (hi - lo) / 2
    if sortedKeys[mid].precedes(k) { lo = mid + 1 }
    else { hi = mid }
  }
  return lo
}
```



### Two Worlds of Protocols

| Without Self Requirement                | With Self Requirement                 |
| --------------------------------------- | ------------------------------------- |
| `func precedes(other: Ordered) -> Bool` | `func precedes(other: Self) -> Bool`  |
| Usable as a type                        | Only usable as a generic constraint   |
| `func sort(inout a: [Ordered])`         | `func sort<T: Ordered>(inout a: [T])` |
| Think 'heterogeneous'                   | Think 'homogeneous'                   |
| Every model must deal with all others   | Models are free from interaction      |
| Dynamic Dispatch                        | Static dispatch                       |
| Less optimizable                        | More optimizable                      |



### A Challenge for Crusty

```swift
struct Renderer {
  func moveTo(p: CGPoint) { print("moveTo(\(p.x), \(p.y))") }
  func lineTo(p: CGPoint) { print("lineTo(\(p.x), \(p.y))") }
  func arcAt(center: CGPoint, radius: CGPoint, startAngle: CGFloat, endAngle: CGFloat) {
    print("arcAt(\(center), radius: \(radius), startAngle: \(startAngle), endAngle: \(endAngle))")
  }
}

protocol Drawable {
  func draw(renderer: Renderer)
}

struct Polygon: Drawable {
  func draw(renderer: Renderer) {
    renderer.moveTo(corners.last!)
    for p in corners {
      renderer.lineTo(p)
    }
  }
  var corners: [CGPoint] = []
}

struct Circle: Drawable {
  func draw(renderer: Renderer) {
    renderer.arcAt(center, radius: radius, startAngle: 0.0, endAngle: twoPi)
  }
  var center: CGPoint
  var radius: CGFloat
}

struct Diagram: Drawable {
  func draw(renderer: Renderer) {
    for f in elements {
      f.draw(renderer)
    }
  }
  var elements: [Drawable] = []
}
```

#### Renderer를 protocol로 바꿔보자

```swift
protocol Renderer {
  func moveTo(p: CGPoint)
  func lineTo(p: CGPoint)
  func arcAt(center: CGPoint, radius: CGPoint, startAngle: CGFloat, endAngle: CGFloat)
}
```

#### Rendering with CoreGraphics

```swift
extension CGContext: Renderer {
  func moveTo(p: CGPoint) { 
  	CGContextMoveToPoint(self, position.x, position.y)
  }
  func lineTo(p: CGPoint) {
  	CGContextAddLineToPoint(self, position.x, position.y)
  }
  func arcAt(center: CGPoint, radius: CGFloat, startAngle: CGFloat, endAngle: CGFloat) { 
  	let arc = CGPathCreateMutable()
    CGPathAddArc(arc, nil, c.x, c.y, radius, startAngle, endAngle, true)
    CGContextAddPath(self, arc)
  }
}
```

extension을 이용해서 이런 식으로 사용이 가능하다

### More Protocol Extension Tricks

```swift
extension CollectionType where Generator.Element: Equatable {
	public func indexOf(element: Generator.Element) -> Index? {
    for i in self.indices {
      if self[i] == element {
        return i
      }
    }
    return nil
  } 
}
```

```swift
protocol Ordered {
  func precedes(other: Self) -> Bool
}
func binarySearch<T: Ordered>(sortedKeys: [T], forKey k: T) -> Int { ... }

// 1. Int type이 Ordered를 conform 하도록
extension Int: Ordered {
  func precedes(other: Int) -> Bool { return self < other }
}

// 2. Comparable type에 Ordered랑 매칭되는 메서드 추가
extension Comparable {
  func precedes(other: Self) -> Bool { return self < other }
}

// 3. (1은 무시하고 보쟈) Comparable type인 Int, String은 Ordered에서 요구하는 precedes가 구현되어 있음
extension Int: Ordered { }
extension String: Ordered { }

let position = binarySearch([2, 3, 5, 7], forKey: 5)
```

다른 방법은?

```swift
protocol Ordered {
  func precedes(other: Self) -> Bool
}
func binarySearch<T: Ordered>(sortedKeys: [T], forKey k: T) -> Int { ... }

// Ordered를 conform하는 Comparable type에 기본 구현
extension Ordered where Self: Comparable {
  func precedes(other: Self) -> Bool { return self < other }
}

extension Int: Ordered { }
extension String: Ordered { }
```

#### Bridge-Building

```swift
protocol Drawable {
  func isEqualTo(other: Drawable) -> Bool
  func draw()
}

extension Drawable where Self: Equatable {
  func isEqualTo(other: Drawable) -> Bool {
    if let o = other as? Self { return self == o }
    return false
  }
}
```



### 언제 class를 사용해야 할까?

* implicit sharing을 원할 때!
  * Copying or comparing instances doesn't make sense
  * Instance lifetime is tied to external effects
  * Instances are just "sinks" -- write-only conduits to external state
* 시스템에서 사용하라고 하는 것들! (Don't fight the system)
  * If a framework expects you to subclass, or to pass an object, do!
* 조심해서 잘 사용하자!



### 요약

* Protocol > Superclasses
* Protocol extensions = magic



Related Session -> Building Better Apps with Value Types in Swift