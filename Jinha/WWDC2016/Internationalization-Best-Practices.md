# Internationalization Best Practices

> 📅 2019.11.17 (일)
>
> WWDC 2016 | Session : 201 | Category : Localization


🔗 [Internationalization Best Practices - WWDC 2016 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2016/201/)

## What's New

### 🆕 Multilingual Keyboards

![](/Jinha/images/Internationalization-Best-Practices-1.png)

키보드의 지구본 모양으로 언어를 바꾸지 않더라도, 자동 완성이 autocorrection과 prediction을 해 준다.

### 🆕 Latin American Spanish

### 🆕 Measurement Formatter

    온도, 에너지, 압력 등등 의 단위 제공

### 🆕 Keyboard Number Pads

### 🆕 macOS Sierra right-to-left support

## Fundamentals

![](/Jinha/images/Internationalization-Best-Practices-2.png)

![](/Jinha/images/Internationalization-Best-Practices-3.png)

![](/Jinha/images/Internationalization-Best-Practices-4.png)

→ Localized String, Formatted Strings, Layout 만 세팅 해 놓으면 언어 설정에 따라 자동으로 바뀐다.

![](/Jinha/images/Internationalization-Best-Practices-5.png)

language code는 hierarchy의 일부분이다. 앱이 Spanish(United States)을 지원하지 않는 다면 그 다음으로 가장 유저가 선호할 만한 language가 무엇 일지 hierarchy를 보고 알아서 해준대

## Quick Fixes

### Localize Strings

Load strings for views created in code using `NSLocalizedString`, which uses `Bundle`’s language matching logic

```Swift
    label.text = NSLocalizedString("World Clock", comment: "Title text")
```

When fetching strings from a server, use the language your app is launched in by calling `preferredLocalizations`, which uses `Bundle`’s language matching logic

```Swift
    let currentLocalization = Bundle.main().preferredLocalizations.first
```

Get the best matched localization from a list of available localizations by calling `preferredLocalizations`, which uses `Bundle`’s language matching logic

```Swift
    let matchedLocalization = Bundle.preferredLocalizations(from: available).first
```

![](/Jinha/images/Internationalization-Best-Practices-6.png)

실제로 localized 가 된지 더블 체크를 하고 싶다면, Build Settings의 `Static Analyzer` 사용하면 이슈를 확인 할 수 있다.

### Formatters

**Date & Time**

```Swift
    var formatter = DateFormatter()
    
    🚫 formatter.dateFormat = "hh:mm a"
    ✅ formatter.timeSytle = .shortStyle
    
    let str = formatter.string(from: date)
    
    // 9:41 AM for English (U.S.)
    // 9:41  for Simplified Chinese
```

**Names**

```Swift
    var components = PersonNameComponents()
    components.givenName = "John"
    components.middleName = "Adam"
    components.familyName = "Appleseed"
    
    let formatter = PersonNameComponentsFormatter()
    formatter.style = .long
    let str = . (from: components) // John Adam Appleseed
```

### Layout

- Part 1: Horizontal Flexibility
    - Use Auto Layout
    - Use UIStackView & NSStackView
- Part 2: Vertical Flexibility
    - Vertical Flexibility – Tall Scripts
    `label.clipsToBounds = true` 하면 안돼!
    - Vertical Flexibility – Multi-Line Labels
    Dynamic Type will ensure script-appropriate line spacing
    Use predefined text styles to get Dynamic Type for free
    `label.font = UIFont.preferredFont(forTextStyle: UIFontTextStyeBody)`
    default로 되어 있지만  custom font 쓸 때 세팅 해 주어야 함
    - Vertical Flexibility – Table Views
    언어의 높이가 높다면 table row height이 커진다
    User built-in table view styles
    `UITableViewCell` is highly customizable

## Getting World-Ready

### Iconography

**Avoid words and numbers in icons**

![](/Jinha/images/Internationalization-Best-Practices-7.png)

앱 아이콘은 앱 구매에 중요한 요소이기 때문에 특정 언어에 묶이는 word, number 없고, left-to-right, right-to-left 같은 방향성도 없어야 한다.

**Flip directional art for right-to-left languages**

### App Name

사용자는 그들의 언어로 작업된 앱을 구입하고 싶어 한다. 앱 이름 또한 적용된다.
앱의 이름과 설명 만으로 앱을 쉽게 이해 할 수 있다면 앱에 대해 더 알려고 할 것이다.

### Multilinual

English UI , Hindi Content 가 올 수 있기 때문에 특정 언어에만 제한을 두면 안된다.
