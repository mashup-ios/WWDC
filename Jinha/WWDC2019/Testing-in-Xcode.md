# Testing in Xcode

WWDC 2019

[https://developer.apple.com/videos/play/wwdc2019/413/](https://developer.apple.com/videos/play/wwdc2019/413/)

![](Untitled-4f5c7dd7-e201-48a7-b14c-2a7ae0bf8774.png)

## Test Pyramid

### Unit

- 간결한 코드, 일반적으로 함수를 검증
- 함수에 변수를 input, 기대한 output 이 나오는지 체크
- 짧고 간결하고 빠르다

### Integration

- 코드의 더 큰 섹션을 검증
- 다른 컴포넌트들이 같이 올바르게 작동하는지
- Unit test 보다는 오래 걸리지만 한번에 더 많이 앱을 테스트 함

### User Interface

- 유저의 행동을 관찰
- 앱이 기대했던 것처럼 동작하는지
- 셋 중 가장 오래 걸림
- 앱의 컴

![](Untitled-300e7840-0465-4779-9e0d-1c4e4fee5651.png)

## Types of Tests in XCTest

### Unit Tests

- 모든 source code를 타겟팅함
- Unit, Integration Test 포함

### UI Tests

- App의 UI 단
- black box test : 앱에서 지원하는 함수나 클래스에 의존하지 않음

### Performance Tests

- 평균 시간, 메모리 사용, 다른 측정

## Test Plans (Xcode11)

- 다른 환경에서 한번 이상의 테스트가 돌 수 있게 해줌
- 모든 테스트 세팅을 한 곳에서 정의
- 다수의 스킴들끼리 공유 가능
- Test Configuration

## CI Workflows

## Result Bundle Format