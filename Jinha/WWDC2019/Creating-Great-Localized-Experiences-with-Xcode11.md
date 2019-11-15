# Creating Great Localized Experiences with Xcode 11

> 📅 2019.11.15 (금)
>
> WWDC 2019 | Session : 403 | Category : Localization

🔗 [Creating Great Localized Experiences with Xcode 11 - WWDC 2019 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2019/403/)

![](/Jinha/images/Creating-Great-Localized-Experiences-with-Xcode11-1.png)

## New localization feature

![](/Jinha/images/Creating-Great-Localized-Experiences-with-Xcode11-2.png)

### 🆕 Launch to Settings

Setting에서 앱 마다 언어 설정을 할 수 있다.

- 코드로 언어를 바꾸는건 지원하지 않는다.
- 앱에서 language option을 추가 하고 싶으면 Setting을 Launch

    UIApplication.shared.open(URL(string: UIApplication.openSettingURLString)!)

### ViewController State Restoration

- Setting을 통해 language를 바꾸면 앱은 relaunch된다.
- `AppDelegate` 에서 상태 값을 복구 할 수 있다.

    application(_:shouldSaveApplicationState:)
    application(_:shouldRestoreApplicationState:)

launguage가 끊김 없이 전환 될 수 있도록 controller에서 `restorationIdentifier` 을 세팅해라.

### 🆕 Scene State Restoration

iOS13에서 Scene Delegate 에서 상태값을 restoration 할 수 있는 API가 추가 되었다.

```Swift
    stateRestorationActivity(for scene:)
```

scene state를 encode 해주는 `NSUserActivity` 객체를 return한다.

### Bundle and Local APIs

- 앱 내의 모든 콘텐츠는 하나의 언어로 일치해야 한다.
- custom 이나 server-side resource 로딩을 위해  `Locale` 이나 `Bundle` API 를 사용해라.

    Local.preferredLanguages
    Bundle.main.preferredLocalizations
    Bundle.preferredLocalizations(from: availableLanguages)

**Locale APIs for Language Preference**

Setting의 유저의 언어와 지역 설정을 불러온다.

`Locale.preferredLanguages` 를 하면 user preference  언어 리스트를 가져온다.

(ex. ["pa-IN", "en-IN", "hi-IN", ...]

**Bundle APIs for Language Matching**

현재 실행 되고 있는 앱의 언어를 가져온다. 

`Bundle.main.preferredLocalizations.first`

외부 language 설정이 있다면, 그 중에서 세팅을 해준다.

```Swift
    let availableLanguages = Server.requestAvailableLanguages()
    Bundle.preferredLocalizations(from: availableLanguages).first
```

## Localization Enhancements in Xcode

### Performance

localization을 위한 Interface Builder export가 15배 빨라졌다.

### Device-Specific Strings

코드에서 하나의 `NSLocalizedString()` 를 호출 할 수 있다

Deivce-specific string은  `.stringdict`로 정의되어 있다.

여러개의 다양한 width rule로 조합 될 수 있다.

## Assets

Image sets, Watch complications, Apple TV image stacks, Sprite atlases, Game Center dashboards and leaderboards, Symbol sets 

→ 다 Localization 가능 해졌다!

![](/Jinha/images/Creating-Great-Localized-Experiences-with-Xcode11-3.png)

### Symbols

SF system symbols, Custom symbols  Localized 가능!

## Localized screenshots with XCTest

### Localization Testing

Clipping, Truncation, Layout overlapping, Right-to-Left languages

여러 개의 언어를 지원 한다면, 테스터가 언어 별로, 디바이스 별로 너무 많은 시나리오에 대한 테스트가 필요하다. 그래서 QA에서 수동으로 하는 작업을 위해 프로젝트에 테스트를 추가하는 것은 굉장히 중요하다. 

Xcode11에서는 이런 테스트를 XCTESTPLAN를 통해 쉽게 자동화 할 수 있다.

### Test Plan

Multiple tests, Multiple configurations

---

### **Set up Tests for Localization**

Accessibility Identifier 은 고유하고, 안정되고, 어떤 language에서도 테스트 할 수 있다고 보장 할 수 있다.

```Swift
    myCell.accessibilityIdentifier = "myCell"
    
    app.tabs.cells["myCell"].tap()
```

![](/Jinha/images/Creating-Great-Localized-Experiences-with-Xcode11-4.png)

### Generate Localized Screenshots

![](/Jinha/images/Creating-Great-Localized-Experiences-with-Xcode11-5.png)

Multiple Configuration 생성 할 수 있고, 각각의 configuration 마다 다른 language과 region을 설정 할 수 있다.

![](/Jinha/images/Creating-Great-Localized-Experiences-with-Xcode11-6.png)

UI Testing 섹션의 설정으로 스크린샷을 지속 할 수 있다.

![](/Jinha/images/Creating-Great-Localized-Experiences-with-Xcode11-7.png)

스킴 매니저를 통해 설정 할 수도 있다.

**Xcode Localization Catalog**

![](/Jinha/images/Creating-Great-Localized-Experiences-with-Xcode11-8.png)

Xcode Localization Catalog**는** String과 String이 아닌 것들 모두 한 곳에서 Localizing 처리 하는 일반적인 방식이다.

🆕Screenshot을 Xcode Localization Catalog에 자동으로 포함될 수 있게 하였다.

Localization export 시`Include screenshots` 체크 하면 됨
