# Continuous Integration and Code Coverage in Xcode

> 📅 2019.11.10 (일)
>
> WWDC 2015 | Session : 410 | Category : Testing

🔗 [Continuous Integration and Code Coverage in Xcode - WWDC 2015 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2015/410/)

## What Is Xcode Server

모든 코드를 주기적으로 pull 받아서 build와 test를 하고 build error 나 test failure를 신속하게 찾을 수 있게 해준다.

설정 하기가 쉽다.

Xcode와 통합이 잘 되어 있다.

### Terminology

![](/Jinha/images/Continuous-Integration-and-Code-Coverage-in-Xcode-1.png)

**Scheme**

- Xcode에서 프로젝트나 테스트를 run 할 때 마다, Scheme을 run 하는 것이다.
- 대부분 자동으로 생성 되지만, custom scheme을 만들 수 도 있고 기본적으로 프로젝트를 빌드 하는 방법의 형식이다.
- 어떤 타겟을 빌드 할지, 어떤 테스트 번들을 실행 할지, 등을 알고 있다.
- Xcode Server에서 bot을 세팅 할 때 중요하다.

**Bot**

- 기본적으로 특정 스킴을 가지고 정의해 놓은 스케쥴대로 build하고 run 한다. 정의 된 action을 수행하고 결과를 보고한다.

**Integration**

- 스케쥴이 끝나고 프로젝트를 실행 하는 것

### New in Xcode Server

- Better bot editing
- Choosing repositories and branches
- Source control security
- Updated reports
- Improved issue tracking

## What's new in Xcode 7

- UserInterface Testing
    - iOS 테스트나 Mac 테스트를 실행 할 때 Xcode Sever의 background에서 full window session을 생성한다.
    - 실제 디바이스에서도 UI test의 단계들을 볼 수 있다.
    - 유저가 바라보는 같은 방법으로 high level에서 앱을 테스트하기 좋다.
- On Demand Resources
    - 앱 bundle을 더 작게 만들어 준다. 번들에 가능한한 많은 resource를 저장하지 않기 때문에.
    - App Store가 대신 주체가 되어준다.
    - QA 시에 App Store에 올라가 있지 않기 때문에 테스트 할 수 가 없는데, QA빌드가 Xcode Sever에 올라가 있으면 알아서 해준다.
- Code Coverage
    - 테스트를 했을 때 어떤 코드가 실제로 실행 됐는지

## Code Coverage

![](/Jinha/images/Continuous-Integration-and-Code-Coverage-in-Xcode-2.png)

![](/Jinha/images/Continuous-Integration-and-Code-Coverage-in-Xcode-3.png)

![](/Jinha/images/Continuous-Integration-and-Code-Coverage-in-Xcode-4.png)

## Extending Xcode Server

### Triggers

- Custom actions : Email notification or scripts
- Use your language of choice
    - Include a #! in your script, otherwise Bash is assumed
- Before and after integrations run
- Gated on the result of the integration

### Environment variables in triggers

환경변수

![](/Jinha/images/Continuous-Integration-and-Code-Coverage-in-Xcode-5.png)

- Open stantdard
    - Secure communication over HTTPS
    - REST pattern of resources and action
    - Data exchanged using JSON
- Compatible with most scripting language
