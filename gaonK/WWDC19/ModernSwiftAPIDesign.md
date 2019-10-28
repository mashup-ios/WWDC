# [Modern Swift API Design](https://developer.apple.com/videos/play/wwdc2019/415/)

@ WWDC19

https://swift.org/documentation/api-design-guidelines 에서 이미 API Design에 대해 언급하고 있다. 이 문서에서 말하고자 하는 것은 **Clarity at the point of use** 이다.



* No Prefixes in Swift-only Frameworks

  * C나 Objective-C에서는 prefix를 사용해야 했지만 Swift module system에서는 prefix를 사용할 필요가 없음
  * 하지만 general한 이름은 혼동을 줄 수 있으므로 명확성을 주는 네이밍을 하기 위해 노력하자

* Values and References

  * class, struct, enum

  * 정답은 없지만 일반적인 가이드는 다음과 같다.

    * class 보다는 struct를 사용해라. reference semantics가 중요할 때만 class를 사용해라.
    * reference counting과 deinitialization이 필요하다면 class를 사용해라. (resource 관리가 필요할 때)
    * 값이 공유되는 경우에는 class를 사용해라.
    * identity가 equality와 구분되어야 할 때는 class를 사용해라.

  * Value types make copies of references

    * ```swift
      struct Material {
          public var roughness: Float
          public var color: Color
          
          private var _texture: Texture // Reference type
          
          public var isSparkly: Bool {
              get { _texture.isSparkly }
              set {
                  if !isKnownUniquelyReferenced(&_texture) { _texture = Texture(copying; _texture) }
                  _texture.isSparkly = newValue
              }
          }
      }
      ```

* Protocols and Generics

  * Don't literally start with a Protocol
    * Start with concrete use case
    * Discover a need for a generic code
    * Try to compose solutions from existing protocols first
    * Consider a generic type instead of a protocol

* Key Path Member Lookup (SE-0252)

* Property Wrappers

  * ```swift
    public struct MyType {
        var imageStorage: UIImage? = nil
        
        public var image: UIImage {
            mutating get {
                if imageStorage == nil {
                    imageStorage = loadDefaultImage()
                }
                return imageStorage!
            }
            set {
                imageStorage = newValue
            }
        }
    }
    ```

    위와 같은 코드는 lazy를 이용해서 간단하게 만들 수 있다.

    ```swift
    public struct MyType {
        public lazy var image: UIImage = loadDefaultImage()
    }
    ```

    ```swift
    public struct MyType {
        var textStorage: String? = nil
        
        public var text: String {
            get {
                guard let value = textStorage else {
                    fatalError("text has not yet been set!")
                }
                return value
            }
            set {
                textStorage = newValue
            }
        }
    }
    ```

    이런 경우에는 getter와 setter의 동작이 다르기 때문에 `lazy`를 이용해서 해결할 수 없다.

    boilerplate code를 줄이기 위해 propertyWrapper를 사용할 수 있다.

    ```swift
    @propertyWrapper
    public struct LateInitialized<Value> {
        private var storage: Value?
        
        public init() {
            storage = nil
        }
        
        public var wrappedValue: Value {
            get {
                guard let value = storage else {
                    fatalError("value has not yet been set!")
                }
                return value
            }
            set {
                storage = newValue
            }
        }
    }
    
    public struct MyType {
        @LateInitialized public var text: String
    }
    ```

    defensive copying은 아래와 같이 구현이 가능하다.

    ```swift
    @propertyWrapper
    public struct DefensiveCopying<Value: NSCopying> {
        private var storage: Value
        
        public init(wrappedValue value: Value) {
            storage = value.copy() as! Value
        }
        
        public var wrappedValue: Value {
            get { storage }
            set {
                storage = newValue.copy() as! Value
            }
        }
    }
    
    extension DefensiveCopying {
        public init(withoutCopying value: Value) {
            storage = value
        }
    }
    
    public struct MyType {
        @DefensiveCopying(withoutCopying: UIBezierPath())
        public var path: UIBezierPath
    }
    ```

    