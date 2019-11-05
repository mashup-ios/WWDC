Swift Overlay for Accelerate.
 Linpack benchmark를 사용한 Measuring Accelerate’s performance

Accelerate 프레임워크란?
- CPU에서 실행되는 이미지 및 신호 처리, 벡터 산술, 선형 대수 및 기계 학습을 지원하는 수천 개의 저수준 수학적 예측치를 제공하는 것입니다.
- 이를 통해 뛰어난 성능을 내고 있는지 확인할 수 있습니다.
- 
새로운 Swift API를 만들었습니다.
- vDSP : 큰 벡터에 대한 산술, 푸리에 변환, 2 차 필터링 및 강력한 유형 변환을 포함한 디지털 신호 처리 루틴을 제공
- vForce. 삼각법과 로그 루틴을 포함하여 산술 및 초월 기능을 제공하는 vForce. 
- Quadrature,  함수의 수치 적분
- vImage는 다양한 이미지 처리 기능을 제공하며 핵심 그래픽 및 핵심 비디오와 쉽게 통합됩니다. 가속은 벡터화를 사용하여 성능 이점을 얻습니다.
- 

 Accelerate를 사용하여 배열의 요소를 처리하는 경우 단일 명령 다중 데이터 또는 simD 레지스터에서 계산이 수행됩니다. 이러한 레지스터는 여러 개의 항목을 단일 레지스터로 패킹하여 여러 데이터 항목에 대해 동일한 명령을 수행 할 수 있습니다. 예를 들어 단일 128 비트 레지스터는 실제로 4 개의 32 비트 부동 소수점 값을 저장할 수 있습니다. 따라서 벡터화 된 곱셈 연산은 한 번에 4 쌍의 요소를 동시에 곱할 수 있습니다. 즉, 작업이 더 빨라질뿐만 아니라 에너지 효율도 크게 향상됩니다.

vDSP
- 푸리에 변환, 
- 2 차 필터링, 
- Convolution 및 상관 관계 분석
- 벡터화 된 디지털 신호 처리 기능
- 요소 별 산술 및 형식 변환

 따라서 예를 들어 두 신호의 일관성을 계산할 필요가없는 경우에도 vDSP의 일반 계산 루틴은 앱 성능을 향상시키는 솔루션을 제공 할 수 있습니다.

예시 : 모든 배열에 대해 다음을 수행 + 평균
`result[I] = (a[I] + b[I]) * (c[I] - d[I])`

* for 구문으로 변환
* vDSP 클래식 API를 사용
```swift
Var result = [Float](repeating: 0, count: n)
vDSP_vasbm(a, 1, b, 1, c, 1, d, 1, &result, 1, vDSP_Length(result.count))
```
  * vDSP를 사용하면 4 개의 루프보다 약 3 배 빠릅니다.
* 새로운 vDSP 용 Swift API를 사용
```swift
Var result = [Float](repeating: 0, count: n)
vDSP_multiply(addition: (a, b), subtraction: (c, d), result: &result)
```
  * 카운트를 명시적으로 전달할 필요가 없으니 좀 더 명확하고 간결함.

### vDSP for Type Conversion
Double 배열을 16비트 인수값으로 반올림합니다.
#### 기존
```swift
Let result = source.map {
  Return UInt16($0.rounded(.towardZero))
}
```
#### vDSP : 이전보다 4배 빠름
```swift
Let result = vDSP.floatingPointToInteger (source, integerType: UInt16.self,
Rounding: .towardZero)
```


### vDSP for Discrete Fourier transform
- 리소스 해제를 걱정할 필요 없이 사용할 수 있음.
- 고유의 transform 함수를 제공함

### vDSP for Diquadratic Filtering
  - 오디오 작업을 하면 2차 필터링 작업을 할 수 있는데 이때 `biquard`기능을 써서 저주파수/고주파수를 제거할 수 있답니다.
#### 새로운 API
```swift
Var biquad = vDSP.Biquad(coefficients: [b0, b1, b2, a1, a2, b0, b1, b2, a1, a2], channelCount: channelCount,
sectionCount: sections:
ofType: Float.self
```
  - 계수를 인자로 전달하고 채널과 섹션 수를 지정해주면 자동 필터링 해쥼

## vForce

* Arithmetic functions: `floor, ceil, abs, remainder …`
* Exponential and logarithmic functions: `exp, log, …`
* Trigonometric functions: `sin, cos, tan, …`
* Hyperbolic functions: `sing, asking, …`

  - 제곱근(sqrt)을 계산하는 로직은 기존 map 사용 로직보다 최대 10배 빠른 벡터화된 함수를 제공한다.
```swift
Let result = vForce.sqrt(a)
```

## Quadrature (구적법)
유한 또는 무한 간격에 대한 명확한 적분 함수의 근사값을 제공하는 함수

* 새로운 API는 기존 코드에 비해 상당히 단순해지고 그 양도 줄었음
* 통합 함수를 C 함수 포인터가 아니라 클로저를 사용할 수 있음.

## vImage
Core Graphic, Core Video, Alpha blending, Format conversions, Histogram operation, Geometry, Morphology 를 위한 api 제공

 - Throwable initializer를 사용함.
 - `Any to any` : Core Video와 Core Graphic 간 변환할 수 있도록 API를 제공
- 비디오 픽셀 버퍼를 이미지로 손쉽게 전환할 수 있는 API 제공


## summary

Accelerate 프레임 워크가 동일한 플랫폼에서 Eigen보다 거의 2.5 배 빠릅니다.

Accelerate 프레임 워크가 플랫폼에 맞게 조정되어 플랫폼이 제공 할 수있는 기능을 완전히 활용할 수 있기 때문에, 앱에서 Accelerate를 사용하면 성능이 향상됩니다. 이 성능은 에너지 소비를 줄임으로써 배터리 수명이 향상되고 사용자에게 더 나은 환경을 제공합니다. 요약하자면 Accelerate는 빠르고 에너지 효율적인 대규모 수학 계산 및 이미지 계산을 수행하는 기능을 제공합니다. 그리고 이제 Accelerate의 라이브러리를 매우 쉽게 사용할 수있는 Swift 친화적 인 API를 추가하여 사용자가 해당 성능 및 에너지 효율성을 활용할 수 있도록합니다. 




