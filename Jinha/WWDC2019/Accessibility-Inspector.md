# Accessibility Inspector

> 📅 2019.11.14 (목)
>
>WWDC 2019 | Session : 257 | Category : Accessibility


🔗 [Accessibility Inspector - WWDC 2019 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2019/257/)

## Accessibility Inspector

Accessibility Inspector는 앱의 accessibility issue를 쉽게 찾고, 진단하고, 고칠 수 있는 방법을 제공

![](/Jinha/images/Accessibility-Inspector-1.png)

![](/Jinha/images/Accessibility-Inspector-2.png)

### **audit 탭**

→ 앱의 accessibility issue를 가장 쉽게 찾는 방법을 제공

- VoiceOver 같은 Assisitive technologies 는 유저 인터렉션을 돕고 view의 UI에 도달 하기 위해 Label같은 accessibility information을 사용한다.
- `help` 버튼을 누르면 suggestion이 나온다.

![](/Jinha/images/Accessibility-Inspector-3.png)

### Inspector 탭

- custom버튼을 inspect하기 위해서는 `point inspection` 버튼을 누르고 시뮬레이터의 accessibily element위에 움직여보면 된다.
- basic 섹션에 이미지 파일 이름이 세팅 된 Label 프로퍼티를 볼 수 있다. 이것은 유저에게 안좋은 경험을 만들어 준다. → 버튼을 넘기면서, VoiceOver가 description을 말해준다.

UIKit이나 app key control을 사용한다면 accessibility 는 자동으로 적용 되겠지만, 이 경우에 우리는 brand name을 `CTextLayer` 을 사용하고 있다. 그렇기 때문에 직접 accessibility를 다루어 주어야 한다.

```Swift
    brandTitleBackgroundView.isAccessibilityElement = true
    
    if let brandName = textLayer.string as? String {
    	brandTitleBackgroundView.accessibilityLabel = brandName
    }
```

### Color Contrast Calcuator

→ 3.0 비율이 권장 됨

사용성 관련 된 Accessibility 이야기 였다...
