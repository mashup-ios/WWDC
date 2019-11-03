# [Creating an Accessible Reading Experience](https://developer.apple.com/videos/play/wwdc2019/248/)

@ WWDC19


훌륭한 응용 프로그램의 특징 > 뛰어난 사용자 인터페이스
 * 많은 양의 텍스트를 읽는 것에 초점을 둔 앱의 경우 텍스트의 레이아웃 및 스타일링이 필요함
 * 여기에 `Accessability`를 더하면 더욱 좋음
`VoiceOver`에 대한 접근성이 뛰어난 읽기 환경을 만드는 데 필요한 API 및 기술에 대해 배웁니다.
  * VoiceOver 켜기
  
### Reading Content Protocol
* `UIAccessibilityLocationDescriptor` 프로토콜을 사용하여 텍스트 내용에 액세스 할 수 있습니다. 
```swift
public protocol UIAccessibilityReadingContent {

 // 포인트의 경우 지정된 터치 위치에 대한 행 번호를 반환하도록 요청합니다. 
 func accessibilityLineNumber(for point: CGPoint) -> Int

 // forLineNumber는 텍스트 내용을 요청하고 지정된 행을 각각 요청합니다. 
 func accessibilityContent(forLineNumber lineNumber: Int) -> String?

 // accessibility Frame이 어디인지 알려줍니다.
 func accessibilityFrame(forLineNumber lineNumber: Int) -> CGRect
 
 // 내용의 전체 페이지를 return
 func accessibilityPageContent() -> String?

}
```

* view 초기화시 `isAccessibilityElement`를 `true`로 설정켜주기
  * `accessibilityLineNumber(for point: CGPoint)` 에서 view를 설명할 수 있는 값을 넘겨준다.
  * `accessibilityPageContent` 문자열들을 모아서 넘겨준다.

### Automatic Page Turning
•  자동 페이지 넘김을 구현하려면 2 개의 접근성 API를 채택해야합니다. 
  * vie에 `causePageTurn` 접근성 특성을 포함시켜야합니다. 
  * `accessibilityScroll`과 `direction`를 설정해서 스크롤 방향을 구현해야합니다. 
```swift
UIAccessibilityTraits.causesPageTurn

func accessibilityScroll(_ direction: UIAccessibilityScrollDirection) -> Bool 
```
  
### Customizing Speech
* VoiceOver가 앱의 컨텐츠를 말하는 방식을 제어 할 수 있습니다.
```
public protocol UIAccessibilityReadingContent {

 func accessibilityAttributedContent(forLineNumber lineNumber: Int) -> NSAttributedString?

 func accessibilityAttributedPageContent() -> NSAttributedString?

}
```
#### 특정 구절을 읽을 언어 설정하기 
```swift
NSAttributedString(string: "Arc de Triomphe", attributes: [.accessibilitySpeechLanguage:
“fr-FR”])
```

#### accessibilitySpeechIPANotation 사용해서 IPA representation 추가하기
```swift
let label = NSMutableAttributedString(string: "Yosemite National Park")
let range = label.string.range(of: "Yosemite")!
label.addAttributes([.accessibilitySpeechIPANotation: "joʊˈsɛmɪti"], range:NSRange(range, in:
label.string))
```
