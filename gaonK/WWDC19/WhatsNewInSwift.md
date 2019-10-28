# [What's New in Swift](https://developer.apple.com/videos/play/wwdc2019/402/)

@ WWDC19

### Performance

* ABI Stability, Module Stability
  * ABI: Application Binary Interface
  * ABI stability 이전에는 executable과 framework가 동일한 컴파일러에서 빌드되어야 했지만 Swift 5 or later 버전 컴파일러를 이용하면 다른 컴파일러에서 빌드가 가능하다.
  * Swift 5.1에서 module interface 파일 도입
  * Module + ABI stability = Binary frameworks

* Apps and the Shared Swift Runtime in the OS
  * Swift 5 or later, Apps use the runtime from OS when it is available
* SourceKit

### Language and Standard Library

* Implicit return from single expressions
* Synthesized default values for the memberwise initializer
* SIMD vectors API
* New design for String interpolation
* Opaque result types
* Property Wrapper types
* DSLs