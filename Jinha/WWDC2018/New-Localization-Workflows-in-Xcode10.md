# New Localization Workflows in Xcode 10

> 📅 2019.11.13 (수)
>
> WWDC 2018 | Session : 404 | Category : Xcode

🔗 [New Localization Workflows in Xcode 10 - WWDC 2018 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2018/404/)

## Localization process overview

### Overview

![](/Jinha/images/New-Localization-Workflows-in-Xcode10-1.png)

Xcode10 전에는, localization을 지원하는 프로젝트에서 우리는 String base localizable 을 본다. 이 string은 소스 코드, 스토리보드, string 파일이나 , string dictionary에 정의 될 수 있다.

이 파일들에서 XLIFF  파일로 export 한다.

![](/Jinha/images/New-Localization-Workflows-in-Xcode10-2.png)

XLIFF 파일을 import 하면, localizeable resource들을 Xcode project에 추가한다.

### XLIFF Benefits

- localization을 코드로 부터 분리 할 수 있다.
- XLIFF 파일에 제공할 언어들을 가지고 있으면 된다.
- 여러개의 파일들을 지원 할 필요 없이, XLIFF 문서 하나만 있으면 된다.

### XLIFF Limitations

- 시각적이고 functional 한 context를 제공 하지 않는다.
- 프로젝트의 asset 같은 resource를 지원하지 않는다.
- XLIFF에 대한 custom metadata를 지원하지 않는다.
- XLIFF  에 대한 사이즈와 길이 제한이 없다.

## 🆕 Xcode localization catalog

**Purpose**

- Xcode10에서 localization을 export와 import 할 수 있는 `.xcloc` extension을 갖는 새로운 포맷
- 모든 localizable asset을 지원
- 추가적인 문맥상의 정보를 제공

![](/Jinha/images/New-Localization-Workflows-in-Xcode10-3.png)

localizable resources 들은 XLIFF 포맷으로 추출 되고 프로젝트에서 localizable이라고 마크 해놓은 모든 것은 Localization Catalog에 의해 지원 되고 `.xcloc` 으로 export된다

![](/Jinha/images/New-Localization-Workflows-in-Xcode10-4.png)

**1. contents.json**

`.xcloc` 내부에는, Localization Catalog 메타데이터를 포함한 `contents.json` 파일이 있다

- Development region
- Target language
- Tool info (version number, build number of Xcode)
- Version of exported Localization Catalog

**2. Localized Contents**

다음 생성된 `.xcloc`  내부에는 Localized Contents directory를 볼 수 있따.

그리고 Localized Contents 는 프로젝트의 모든 localizable resource들을 포함 하고 있다. 이것이 localizer가 작업하는 주요 main directory이다

- XLIFF 1.2 document containing the project localized strings
- Non-string localizable assets (ex. image)
- Organized into same file system hierarchy as Xcode project
- Override resources (ex. interface builder file)

**3. Source Contents**

- Assets used to produce the Localized Contents (ex. storyboard)
- Organized into same file system hierarchy as Xcode project
- Provided for content

**4. Notes**

- Provide additional information to the localizers
- Screenshots
- Readme file
