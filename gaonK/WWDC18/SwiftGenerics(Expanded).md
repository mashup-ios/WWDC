# [Swift Generics (Expanded)](https://developer.apple.com/videos/play/wwdc2018/406/)

@ WWDC 18

소리도 중간에 안 나오고 ㅋㅋㅋ 중간에 (Expanded version)으로 영상도 나오는 특이한 영상 ! 🤓



### Parametric Polymorphism

```swift
struct Buffer<Element> {
  let count: Int
  
  subscript(at: Int) -> Element {
    // fetch from storage
  }
}
```



### Implied Generic Parameters

```swift
let words: Buffer<String> = ["generics", "ftw"]
let words: Buffer					= ["generics", "ftw"]
```



### Swift's In-Memory Representation

```swift
extension Buffer where Element == Int { 
  func sum() -> Element {
    var total = 0
    for i in 0..<self.count {
      total += self[i]
    }
  }
  return total
}
```

`where Element == Int` 부분이 없으면 += 연산을 지원하지 않는 타입의 경우 때문에 에러 삑



### Designing a Protocol

`Buffer`, `Array`, `Dictionary`, `Data`의 공통점?

```swift
protocol Collection {
  associatedtype Element
  associatedtype Index: Equatable
  
  subscript(at: Index) -> Element
  
  func index(after: Index) -> Index
  
  var startIndex: Index { get }
  
  var endIndex: Index { get }
  
  var count: Int { get }
}

struct Buffer<Element> { }
extension Buffer: Collection { }

struct Array<Element> { }
extension Array: Collection { }

struct Dictionary<Key, Value> { }
extension Dictionary: Collection {
  typealias Element = (Key, Value)
}
```

이런 식으로 사용



### Customization Points



### Protocol Inheritance

* Some collection algorithms need more than `Collection` provides:
  * `lastIndex(where:)` needs to walk backwards to be efficient
  * `shuffle()` needs to swap elments to work at all
* Some conforming types have these capabilities

프로토콜 상속 가능!



### Conditional Conformance

```swift
extension Slice: BidirectionalCollectioin where Base: BidirectionalCollection {
  func index(before idx: Index) -> Index { return base.index(before: idx) }
}
```

여기서 보이는 것처럼 `Base`가 `BidirectionalCollection`일 때만 `BidirectionalCollection`을 conform한다.



### Recursive Constraints

`String`의 `SubSequence`는 `SubString`.

그럼 `SubString`의 `SubSequence`는? `SubString`이다. 



Recursive하게 구현함으로써 위와 같은 문제 해결 가능

```swift
protocol Collection {
  associatedTyppe Element
  associatedType Index
  associatedType SubSequence: Collection
  	where SubSequence.Element == Element,
  				SubSequence.Index == Index,
  				SubSequence.SubSequence == SubSequence
}
```





### Generics and Classes

리스코프 치환 원칙을 잘 생각하자!

프로토콜을 채택도 서브 클래스에 상속된다.

required initializer는 반드시 모든 서브 클래스에서 구현되어야 한다.



### Summary

* Swift's generic provide code reuse while maintaining static type information
* Let the push-pull between generic algorithms and conforming types guide desing
  * Protocol inheritance captures specialized capabilites of some conforming types
  * Conditional conformance provides compposition for those capabilities
* Apply the Liskov Substitution Principle when working with classes