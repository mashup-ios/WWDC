# [Making Apps More Accessible With Custom Actions](https://developer.apple.com/videos/play/wwdc2019/250/)

@ WWDC 19

### Reducing Clutter

* Removed repetitive control
  * 예를 들면 한 셀 안에 액션이 여러 개 있을 때, 컨텐츠가 아닌 이 액션에 대해서 계속 말해줄 필요는 없다!
* Reduced cognitive load
* Simplified the experience



### Convenience and Speed

* Removed cumbersome gestures
* Sped up interaction
* Improved prominence of interaction options

예를 들면 스위치 컨트롤을 이용해서 수행해야 하는 탭을 많이 줄일 수 있음



```swift
override var accessibilityCustomActions: [UIAccessibilityCustomAction]? {
  get {
    let actions = [UIAccessibilityCustomActions]()
    let myAction = UIAccessibilityCustomAction(name: "Custom Action") { (customAction: UIAccessibilityCustomAction) -> Bool in
			var success = false
			// action logic
			return success
		}
    actions.append(myAction)
    return actions
  }
  set { }
}

cellButton1.isAccessibilityElement = false
cellButton2.isAccessibilityElement = false
cellButton3.isAccessibilityElement = false
```



### Assistive Technology Support

* VoiceOver
* Switch Control
* Full Keyboard Access
* Voice Control