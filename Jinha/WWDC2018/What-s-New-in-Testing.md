# What's New in Testing

Session 403

🔗 [What's New in Testing - WWDC 2018 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2018/403/)

## Code coverage

- Xcode 9.3
- Performance and accuray 향상
Xcode 9 보다 95% 빨라짐. File Size도 10배 감소.
- Target selection
third-party dependencies, 이미 테스트 된 framework가 있을 때 타겟 조절 가능

![](/Jinha/images/What-s-New-in-Testing1.png)

- xccov
Command line tool, Human-readable, View coverage data

    ![](/Jinha/images/What-s-New-in-Testing2.png)

    Coverage data
    - Derived data에 존재
    - `xcodebuild -resultBundlePath ...`  flag가 있다면 Result Bundle에도 포함

- Source editor

![](/Jinha/images/What-s-New-in-Testing3.png)

## Test selection and ordering

![](/Jinha/images/What-s-New-in-Testing4.png)

Xcode10 New

- Automatically include new test 테스트 새로 작성하면 자동으로 포함 시켜줌
- 원래 테스트 순서는 알파벳 순서가 디폴트인데 순서에 따라 결과가 달라질 수 있다 
→ Randomize execution order 체크 하면 랜덤으로 순서를 바꿔준다

## Parallel testing

### Parallel Destination Testing

    xcodebuild test
    -destination ‘platform=iOS,name=iPhone X’ 
    -destination ‘platform=iOS,name=iPad’

Xcode9에서 소개된, 모든 테스트를 동시에 여러 개의 destination에서 돌리는 기능
그 전에는 iPhone X에서 테스트를 하고 종료 하면 iPad를 하는 식.
Xcode9에서 테스트가 동시에 진행 될 수 있게 함. 실행 시간을 감소시킴.
하지만 제약이 있었다. 

- 여러개의 destination에서 테스트할 때만 유익함
- xcodebuild에서만 사용가능

### Parallel Distributed Testing

![](/Jinha/images/What-s-New-in-Testing5.png)

- 하나의 destination에 병렬로 테스트 실행
- xcodebuild, XCode에서 사용 가능

### Testing Architecture

Unit tests

- Unit test는 test bundle로 컴파일 된다
- 런타임에, Xcode는 테스트 runner로 취급되는 앱의 인스턴스를 실행시킨다
- runner는 test bundle을 load하고 모든 테스트를 실행한다

UI Test

- test bundle로 컴파일 된다
- 하지만 이 번들은 Xcode가 생성하는 custom app에 의해 load 된다
- 앱이 더이상 테스트를 실행 하는게 아니라,
- 테스트가 앱을 실행시키고 다른 UI 파트와 상호작용 하면서 앱을 자동화한다

Parallel Distributed Testing

- 그전처럼 Xcode가 테스트를 실행하기 위해  test runner를 런칭한다
- 하지만 여러개의 test runner들을 실행시킨다
- 각각 테스트 subset을 실행시킨다
- 기계를 최대한 활용하기 위해 Xcode는 test ruuner에게 동적으로 테스트를 분배한다
- 각각의 runnere들은 테스트 클래스를 분배 받는다
- 모든 테스트 클래스가 끝났을 때 테스트가 끝난다

### Classes Execute in Parallel

![](/Jinha/images/What-s-New-in-Testing6.png)

왜 테스트를 각각의 테스트 메소드 단위로 나누는게 아니라 테스트 클래스로 분배할까?

- 클래스의 테스트들 사이의 의존성을 숨기기 위해
- 각각의 테스트 클래스에는 계산 비용이 많이 드는 클래스 레벨의 `setUp`과 `tearDown`메소드를 갖고 있기 때문에. 하나의 runner에 테스트 클래스로 제한 함으로써 XE test는 이 메소드들을 한번만 호출 하면 된다.

### Parallel Testing on Simulator

![](/Jinha/images/What-s-New-in-Testing7.png)

시뮬레이터에서 Parellel Test를 실행하면, Xcode 는 내가 선택한 시뮬레이터를 시작하고

여러 개의 복사본을 만들거나 clone한다. 이 복사본들은 원본 시뮬레이터의 것과 동일하다.

그리고 Xcode는 필요하다면 자동으로 이 복사본을 생성하거나 삭제 할 것이다.

시뮬레이터에 몇차례 복사 한 후, Xcode는 각각의 복사본에서 test runner를 실행시키고, runner는 테스트 클래스를 실행하기 시작한다.

Implications

- 원본 시뮬레이터는 테스트 동안에 사용 되지 않는다
- 앱의 복사본이 여러 개가 있다. 각각의 복사본은 고유의 data container을 가지고 있다.
디스크에서 파일을 수정하는 테스트 클래스가 있을 때, 완전히 분리된 data container에 접근 하기 때문에, 다른 테스트 클래스에서 파일 수정을 볼 수 없다

### Tips and Tricks

- 오래 걸리는 클래스를 두 클래스로 나누는 것을 고려해라
- parallelization이 disabled된 상태에서, bundle에 성능 테스트를 추가하라
- 어떤 테스트들이 prallelization에 안전하지 않은지 이해하라
