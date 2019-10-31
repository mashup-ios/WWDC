---
layout: post
title:  "Implementing Dark Mode on iOS"
date:   2019-11-01 01:26:06 +0900
categories: iOS
tags: iOS DarkMode WWDC
author: Juhee Kim
mathjax: true
comments: true
---

## Dark Mode의 3가지 요소

### Color
기존에는 UIColor를 사용할 때 RGB value 를 썼지만 다크모드가 나오면서 이에 대응할 수 있는 다양한 기본 Color들이 제공된다.
Table systempGroupedBackground, systemBlue 등등~
기본 UIComponent들은 기본적으로 다크모드를 대응되니까 필요한 부분만 customizing 해서 사용하자.
배경이나 테이블 뷰 배경 같은 이런 기본적인 애들은 걍 있는 컬러 가져다 쓰면 너도 좋고 나도 좋고 코드도 좋다~~

### Material
- Blur effect도 새로운 것들이 추가되었다.
  - UIBlurEffect - UIVisualEffectView
  - Thick / regular / thin / ultrathin * (Dark/Light)
- Vibrancy content : blur 위에 컨텐츠를 추가할 수 있는데 그러면 좀 예쁘게 올라간다.
  - UIVibrancyEffect - UIVisualEffectView

#### Asset에서 Appearance 추가하면 color든 image 든 대응할 수 있음.

### LEVEL
* Background level
* elevated level

Modal 창을 보여주는 방식이 변화하면서 + 다크모드가 등장하면서 다른 화면 위에 뜨는 화면이 늘어남
그래서 그 경우 elevated level이라고 하고 system background 컬러로 지정하면 level에 따라 layer가 추가되어 다른 색상이 나온다.

## Trait collection
Dark Mode인지 Light Mode인지는 `TraitCollection` 안에 user `userInterfaceStyle` 로 들어가 있다.
그리고 각 `view` 와 `view controller`는 `TraitCollection` 을 가진다.

- userInterfaceIdiom : iPhone / iPad / CarPlay
- userInterfaceStyle : Light/ Dark
- userInterfaceLevel : base / elevated

Custom dynamic color를 사용해야 한다면 traitCollection을 가지고 color를 결정할 것. traitCollection은 current value를 가지고 있기 때문에 **유용하지만 위험할 수 있다.**

### Where to set dynamic resources?
traitCollection은 다음 메서드들에서 사용된다.
* UIView
  * dray(), layoutSubviews()
  * traitCollectionDidChange()
  * tintColorDidChange()
* UIViewController
  * viewWillLayoutSubViews()
  * viewDidLayoutSubviews()
  * traitCollectionDidChange()

이 외에서는 `TraitCollection`이 올바른 값을 가지고 있을지, 기존 값을 가지고 있을지 **보증하지 않는다.**

Core 단, CoreAnimation Framework는 이 Dynamic Color/Trait의 개념을 가지고 있지 않기 때문에, CALayer를 사용하는 경우는 조금 다르게 다루어야 한다. (cgcolor는 dynamic 하지 않음)

#### Color를 올바르게 가져오는 법 3가지
```swift
let newTraitCollection = traitCollection

UIColor.resolveColor(with: newTraitCollection)

newTraitCollection.performAsCurrent { }

view.traitCollection = newTraitCollection
```

Size class가 변경되었을 때도 traitCollectionDidChanged가 불리니까 traitCollection color appearance did changed 메서드를 활용해서 컬러가 변경되었을 때에만 적절한 조취를 취하자.

#### Image를 가져오는 법.
UIImageView는 TraitCollection과 상관 없다. image만 잘 가져오면 된다. image asset 에서 가져오면 된다.

```swift
UIImage(named: “imageName”)?.imageAsset?.(with: newTraitCollection)
```

### TraitCollection with View Hierarchy

`TraitCollection`은 앱 전체에서 하나만 존재하는 것이 아니라, `ViewHierarch`의 최상단인 `UIScreen`부터 가지고 있다. UIWindowScene, UIWindow, UIPresentationController, UIViewController, UIView ...

그렇기 때문에 현재 보이는 화면의 traitcollection을 사용하는 것이 적절하다~

#### 새로운 View를 추가할 때 발생되는 동작
뷰를 추가해본다고 생각해보자. 가장 가까운 목적지(보이는 화면?)의 traitCollection을 가져와서 view에 미리 할당해준다.
그리고 실제로 addSubView가 되면 그때 부모 에게서 traitcollection을 다시 가져온다.

### debug with trait collection
특성이 변경될 때에만 traitcollection이 변경되었다는 메소드가 불린다.

디버그 로깅에 trait collection 로깅+뭐가 바꼈는지 알 수 있도록 추가할 수 있지~

### override trait collection

trait은 레이아웃이 일어나기 전에 항상 변경된다.

만약 trait collection을 일부 화면에서 혹은 뷰에서 고정하고 싶다면 `overrideUserInterfaceStyle`을 override하자

앱 전체에서 설정할거면 plist에서 UserInterfaceStyle을 설정해주자.

traitCollection 자체를 override할 수도 있는데, trait collection에는 style만 있는게 아니니까 super를 꼭 불러 준 다음에 필요한 항목을 지정해주자.

## 변경된 사항.
### Status Bar
Status Bar도 UserInterface Style에 따라 바뀐다.
* 이전 : Default, lightContent
* 지금 : Default, lightContent, darkContent

### ActivityIndicatorView
* 이전 : gray, white, whiteLarge
* 지금 : medium, large *2 set own color default gray

### Text Containers
UILAbel, UITextField, UITextView 는 기본적으로 label color를 사용한다.

NSAttributedString.Key: Any 에서는 foregroundColor에 그냥 Dynamic color 쓰면 먹힘

### Web Contents
webcontent에서는 color-scheme prefers-color-scheme 등의 태그를 사용해서 다크모드에 대응할 수 있다.

필요하다면 다른 세션 있으니까 거기서 찾아보자.


### 요약
* [동영상](https://developer.apple.com/videos/play/wwdc2019/214/)
* 코드를 작성하기 전에 디자인을 익히고 기본적으로 제공되는 기능을 활용하는 방법을 알아냅시다. 그런 다음 사용자 정의 할 항목을 찾으세요.
