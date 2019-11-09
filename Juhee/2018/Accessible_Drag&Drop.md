---
layout: post
title:  "Accessible Drag and Drop"
date:   2019-11-10 02:26:06 +0900
categories: iOS
tags: iOS WWDC Accessible Drag Drop
author: Juhee Kim
mathjax: true
comments: true
---

* content
{:toc}

## [Accessible Drag and Drop](https://developer.apple.com/videos/play/wwdc2018/241/)

### Agenda
1. Drag and Drop refresher
2. Accessible Drag and Drop concepts
3. Example—Exposing an ancestor’s drag
4. Example—Exposing multiple drops

### Drag and Drop refresher
Drag and Drop은 iOS 11에 도입 된 기술로, `Application`을 사용하는 사람이 `Application` 내에서 또는 `Application`간에 데이터를 전송할 수있는 강력한 방법을 제공합니다.

* Interactions are hosted by views
* Starting a drag: `UIDragInteraction`
* Accepting a drop: `UIDropInteraction`

### Accessible Drag and Drop Concepts
접근성 요소가 직접 Drag-Drop을 호스팅하지 않을 수 있습니다.
* Subviews might host interactions
* Element might descend from a view that hosts interactions
Solution
* Accessibility Drag and Drop API를 통해 접근성 기술을 사용할 수 있는 방식으로 논리적 드래그 앤 드롭을 지정할 수 있습니다.

#### API 살펴보기
`UIAccessibilityDragging`: drag sources / drop points 정의하는 `protocol`
* Drag sources : 어디서 drag가 시작됐는지?
* Drop points : 어디서 drop되었는지?
Users activate drags and drops like custom actions
```swift
extension NSObject {
 @available(iOS 11.0, *)
 open var accessibilityDragSourceDescriptors: [UIAccessibilityLocationDescriptor]?
 @available(iOS 11.0, *)
 open var accessibilityDropPointDescriptors: [UIAccessibilityLocationDescriptor]?
}
```

* Drags and drops 가끔 시스템에 의해 자동 노출 될 수 있습니다.
  * 이 경우 이름이 자동으로 지정됩니다.
  * Only interactions in an element’s subtree are exposed
* `UIAccessibilityDragging`
  * 이 프로토콜을 채택하면 원하는 뷰만 상호작용되도록 정확히 노출할 수 있습니다.
  * Allows specifying a specific name for each
  * Implement for the best experience

#### 예시 : Ancestor's Drag 노출
* Bar graph built with CALayer
* Drag and drop bar data by dragging the bar itself

레이어는 뷰가 아니기 때문에 상호작용을 호스팅 할 수 없습니다. > 막대 그래프 뷰에 넣어야 합니다.
```swift
func dragInteraction(_ interaction: UIDragInteraction,
 itemsForBeginning session: UIDragSession) -> [UIDragItem] {
 if let index = self.indexOfBar(point: session.location(in: self)) {
 let provider = NSItemProvider(object: "Bar: \(series[index])" as NSString)
 let dragItem = UIDragItem(itemProvider: provider)
 dragItem.localObject = index
 return [dragItem]
 }
 return []
}

func makeAccessibilityElements() {
 self.accessibilityElements = bars.enumerated().map { (index, barLayer) in
 let element = UIAccessibilityElement(accessibilityContainer: self)
 element.accessibilityFrameInContainerSpace = barLayer.frame
 element.accessibilityLabel = seriesLabels[index]
 element.accessibilityValue = "\(series[index])"
 return element
 }
}
```

`UIAccessibilityLocationDescriptor`
*  이 클래스는 뷰에서 특정 이름으로 점을 지정하여 상호 작용이 이루어지는 위치를 설명하는 데이터입니다.
  * `accessibilityDragSourceDescriptors` : 논리적으로 이 요소와 연관된 `drag source`를 노출
  * `accessibilityDropPointDescriptors` : 논리적으로 이 요소와 연관된 `drop point`를 노출
* 이 `Descriptor`는 실제 view를 참조하고 있습니다.

```swift
let descriptor = UIAccessibilityLocationDescriptor(name: “Drag from specified point",
 point: dragPoint, in: view)
element.accessibilityDragSourceDescriptors = [descriptor]
```
```swift
func makeAccessibilityElements() {
  self.accessibilityElements = bars.enumerated().map { (index, barLayer) in
    let element = UIAccessibilityElement(accessibilityContainer: self)
    element.accessibilityFrameInContainerSpace = barLayer.frame
    element.accessibilityLabel = seriesLabels[index]
    element.accessibilityValue = "\(series[index])"

    // 뷰의 dragPoint를 사용해서 descriptor를 생성하여 element에 넣어준다.
    let dragPoint = CGPoint(x: barLayer.frame.midX, y: barLayer.frame.midY)
    let descriptor = UIAccessibilityLocationDescriptor(name: "Drag bar data", point:dragPoint, in: self)
    element.accessibilityDragSourceDescriptors = [descriptor]

    return element
  }
}
```

### Exposing Multiple Drops
* Contact card
  * Card is one element
  * 카드 안에 드랍포인트가 여러개 있을 경우
* `accessibilityDropPointDescriptors`를 사용해서 노출시킬 드랍포인트를 지정해주고 `UIAccessibilityLocationDescriptor`로 드래그할 대상을 지정해줍시다!
```swift
override var accessibilityDropPointDescriptors: [UIAccessibilityLocationDescriptor]? {
  get {
    let photoWellMidpoint = CGPoint(x: self.contactPhotoWell.bounds.midX,
                y: self.contactPhotoWell.bounds.midY)
    let attachmentsWellMidpoint = CGPoint(x: self.attachmentsWell.bounds.midX,
                y: self.attachmentsWell.bounds.midY)
    return [
      UIAccessibilityLocationDescriptor(name: "Drop into portrait", point: photoWellMidpoint, in: self.contactPhotoWell),
      UIAccessibilityLocationDescriptor(name: "Drop into attachments", point: attachmentsWellMidpoint, in:self.attachmentsWell)]
  }
  set {}
}
```
### 참고하기
* [203-WWDC2017]()
