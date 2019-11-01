# Protocol Oriented Programming

Session 408

🔗 [Protocol-Oriented Programming in Swift - WWDC 2015 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2015/408/)

OOP를 그켬하는 Crusty를 설득시키다 본인이 설득 당하는 내용

![](/Jinha/images/Protocol-Oriented-Programming1.png)

OOP가 짱이야 클래스가 짱이야

- 관련 데이터들과 연산자들을 그룹시킬 수 있다 → 캡슐화
- 외부로부터 코드 내부에 각각의 벽을 세울 수 있다 → 접근 제어
- 윈도우, 커뮤니케이션 채널 같은 관련 아이디어를 대표 할 수 있다 → 추상화
- 충돌을 방지해주는 네임스페이스를 제공 해 준다
- 어메이징한 표현 문법들이 있다
- 확장성이 있다

![](/Jinha/images/Protocol-Oriented-Programming2.png)

아니야 Type이 짱이야 이거 다 Struct랑 enum으로 할 수 있어

마자 Swift 에서 이름을 붙일 수 있는 어떤 타입이든 1급 객체야(first class citizen) 그리고 이 모든 장점들을 다 가지고 있어

꼭 클래스만 할 수 있는 핵심 기능을 생각해 봤어

- 슈퍼클래스는 복잡한 로직의 실질적인 메소드를 정의 할 수 있고 서브클래스에서는 슈퍼클래스에 의해 모든 작업이 완료된다. 상속만 했을 뿐인데~
- 진짜 마법은 서브클래스가 override 할때 생기지. 이런 커스터마이징 한 것이 상속한 구현들과 함께 놓인다. 이것은 복잡한 로직들을 재사용 할 수 있다. 개방된 유연성, 다양성을 가능하게 한다

Crusty: 나 Struct로 커스터마이징 맨날해. 그래 클래스 강력하지만 비용에 대해 얘기해보자

![](/Jinha/images/Protocol-Oriented-Programming3.png)

1. Implicit Sharing

    너무 많은 복사가 생긴다. 코드를 느리게 한다. 동기화 문제가 있고. 스레드가 변하기 쉬운 상태들을 공유 하고 있어서 lock을 해줘야 된다. 이 lock이 코드를 더 느리게 한다. 데드락이 생길 수 있다. 복잡하다

    이 모든 영향들이 결국 버그 가 된다.

    → 하지만! Swift Collection들은 모두 Value 타입이기 때문에 적용이 되지 않는다!

    ![](/Jinha/images/Protocol-Oriented-Programming4.png)

2.  Inheritance All Up In Your Business
상속은 하나만 받을 수 있어. 클래스를 정의 할 때 결정해야 되잖아 나중에 extension 할 수 없잖아. 슈퍼클래스에 저장 프로퍼티들이 있다면 그냥 무조건 받아 드려야한다. 그리고 초기화 해줘야 한다. 메소드들이 어떤식으로 override되고 체이닝 될지 알 수 없다

→ 그래서 Delegate 패턴을 쓴다

3.  Lost Type Relationships
클래스를 추상화를 하기 때문에 생기는 문제들

![](/Jinha/images/Protocol-Oriented-Programming5.png)

그래서 더 나은 추상화 메커니즘이 필요해 → Swift Protocol

**Swift Is a Protocol-Oriented Programming Language**

## Start with a Protocol

```Swift
    protocol Ordered {
    	func precedes(other: Self) -> Bool
    }
    
    struct Number : Ordered {
    	var value: Double = 0
    	
    	func precedes(other: Number) -> Bool {
    		return self.value < other.value 
    	}
    }
```

```Swift
    func binarySearch<T : Ordered>(sortedKeys: [T], forKey k: T) -> Int { var lo = 0
    	var hi = sortedKeys.count
    
    	while hi > lo {
    		let mid = lo + (hi - lo) / 2
    		if sortedKeys[mid].precedes(k) { lo = mid + 1 } else { hi = mid }
    	}
    	
    	return lo 
    }
```
### Protocol Extensions

```Swift
    protocol Renderer {
      func moveTo(p: CGPoint)
      func lineTo(p: CGPoint)
      func circleAt(center: CGPoint, radius: CGFloat)
      func arcAt(
        center: CGPoint, radius: CGFloat, startAngle: CGFloat, endAngle: CGFloat)
    }
    
    extension Renderer {
    	func circleAt(center: CGPoint, radius: CGFloat) { ... } 
    	func rectangleAt(edges: CGRect) { ... }
    }
```

## When to Use Classes

- 암시적 공유(Implicit Sharing)이 필요할 때
- 시스템(Cocoa, Cocoa Touch)는 객체를 다루고, 슈퍼클래스를 제공하고 서브클래싱을 해야되거나 객체를 전달해야돼.

## Summary

- Protocols > Superclasses
- Protocol extensions = magic(almost)
- Be like Crusty
