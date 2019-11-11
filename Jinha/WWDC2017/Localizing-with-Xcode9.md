# Localizing with Xcode 9

> 📅 2019.11.11 (월)
>
> WWDC 2017 | Session : 401 | Category : Xcode

🔗 [Localizing with Xcode 9 - WWDC 2017 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2017/401/)

![](/Jinha/images/Localizing-with-Xcode9-1.png)

> 귀욥따

## Internationallization

언어가 길던 짧던, 오른쪽에서 왼쪽으로 쓰던, 동적으로 이 모든 시나리오에 적용해야 한다.

### String Management

런타임 시 `NSLocalizedString` 을 사용하면 알맞는 번역 된 언어를 display 하는 것을 쉽게 만들어 준다.

Storyboard 나 zip 파일에 있는 String은 default로 현지화 할 수 있다.

하지만 에러 메세지나 notification 같은 소스 코드에 정의 되어 있는 String도 있을 수 있다. 이 String들을 `NSLocalizedStringForm` 으로 감싸면 된다.

`NSLocalizedWithFormat` 으로 formatted string을 사용 할 수 있다.

```Swift
    // Set a label's text
    label.text = "Population"
    
    // Set a label's text to a localized string
    label.text = NSLocalizedString("Population", comment: "Label preceding the population value") 
    
    // Load localized string from a specific table 
    label.text = NSLocalizedString("Poluation", tableName: "Localizable", bundle: .main, value: nil, comment: "Label preceding the population value") 
    
    
    // Create a formatted string
    let format = NSLocalizedString("%d popular languages, comment: "Number of popular languages")
    label.text = String.localizedStrinWithFormat(format, popularLanguages.count)
```

comment → 번역 될 string에 대한 맥락 제공

tableName → framework나 shared component를 사용 시 출처를 위해 tableName 사용

`localizedStringWithFormat` format 되는 String 인지 하기 위해 문맥 파악 해야 함

fb.Iproj/Localizeable.string

```Swift
    /* Title label's text */
    "International Facts" = "Faits Internationaux";
    
    /* Label prompting a user to choose a territory */ 
    "Territory" = "Territoire";
```   

런타임 시, `NSLocalizedString` 은 유저의 선호 language를 결정 하고, localized string file을 찾을 것이다.

### Formatting

날짜, 시간, 숫자 등에 대한 다른 시각적인 표현에 대한 지역화

### Date and time

![](/Jinha/images/Localizing-with-Xcode9-2.png)

### User Interface

모든 언어에서 UI가 flexible해야 하고 nice해 보여야 한다.

![](/Jinha/images/Localizing-with-Xcode9-3.png)

1. Xcode가 Destructor을 위해 프로젝트를 수정 할 것이다.
2. 그리고 UI를  String으로 부터 분리 할 것이다. → User Interface와 관련 된 파일은(zip 파일이나  storyboard파일 같은) `Base.lproj` 에 저장 될 것이다
3. 스토리보드 파일이나 `NSLocalizedString`을 통해서 unwrap 된 String은 특정 언어 폴더에 저장 될 것이다. 
→ 앱의 새로운 언어를 만들 때 마다, 복사하는 대신에 User Interface에 하나의 특정 언어 폴더를 가지고 있도록
4. base internationalization은 Xcode 5부터 default로 가능 하다.

🆕 Localization warnings → localization을 고려 하지 않은 constraint에 warning

🆕 Pseudolocalization 언어 마다 특징을 적용해서 미리 볼 수 있음

### Summary

- Use standard APIs to handle the complexity of data formatting
- User base internationalization and Auto Layout
- Validate your strings and UI

## Localization Workflow

### Localization Process

![](/Jinha/images/Localizing-with-Xcode9-4.png)

1. Xcode는 프로젝트에 모든 localizable resource를 찾아서, 
2. 프로젝트가 localization 을 위해 export 될 때, Xcode는 localizable resource들로 부터 string을 추출해서 당신이 추가한 모든 언어에 대한 XLIFF 파일을 생성 할 것이다
3. XLIFF 파일은 localization 산업에서 표준 standard XML localization file이다.
4. localizer 중 하나에게 XLIFF 파일을 보내면, 이미 포맷과 사용법에 대해 친숙 할 것이다.
5. 일단 XLIFF 파일이 번역 되면 Xcode에 import 할 수 있고 자동으로 localized text에 통합 될 것이다

 → 이것이 Xcode 의 localization 하는 과정이다.  매우 간단하고 직관적이다.

![](/Jinha/images/Localizing-with-Xcode9-5.png)

🆕 **Stringdict**

Xcode9에서 String dic format의 String을 export, import에 대한 지원을 추가 했다. 

🆕 **Adaptive Strings**

![](/Jinha/images/Localizing-with-Xcode9-6.png)

### Summary

- Use strings dict for plurals and width variants
- Export XLIFF for localization
- Import localized XLIFF
- Localize non-string resources in Xcode

## Testing

**XCTest**

![](/Jinha/images/Localizing-with-Xcode9-7.png)

### UITest

![](/Jinha/images/Localizing-with-Xcode9-8.png)

위 코드는 localization이 안되고, 테스트를 실행하는 각각의 언어로 번역 될 것이다

`accessibilityIdentifier` 을 사용하면 화면의 모든  element에 고유하고, 해당 element에 어떤 string이 load 되는지 모른채 사용 할 수 있다.

**Set accessibilityIdentifier**

```Swift
    button.accessibilityIdentifier = "startButton"
```

![](/Jinha/images/Localizing-with-Xcode9-9.png)

- Use accessibility identifiers
- Use `XCTAttachment` to collect screenshots
- Get full coverage of UI for all localizations

### Summary

- Prepare your app for localization
- Validate your app's readiness
- Export localizable content to be translated
- Import localized content
- User `XCTest` to test your localized app
