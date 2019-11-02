# LLDB: Beyond "po"
LLDB는 매우 훌륭한 디버깅 툴로 콘솔에서 바로 디버깅을 할 수 있다.

```swift
struct Trip {
  var name: String
  var destination: [City]
  ...
}

let cruise = Trip(...)
```
```python
po cruise
```
하면 cruise 변수에 대한 설명이 나오는데, 이때의 `description`을 customize 할 수 있다.

### Customizing po description
`CustomDebugStringConvertible` `protocol`을 채택하면 debug description을 바꿔준다. 이건 이 프로토콜을 채택한 **클래스**에서만 적용된다. (끄니까 `destination`에는 적용되지 않는다.

만약 하위 struct에 대해서도 커스터마이징이 필요하다면 `CustomReflectable` `protocol` 문서를 확인하자.

po는 단순히 프린팅만을 하는 것이 아니라, 객체의 내부 프로퍼티에 접근하거나 함수를 호출하는데 사용할 수도 있다.

### Create expression with command alias
입력 가능한 표현식을 만들어 사용할 수 있다. po는 사실 오브젝트 설명을 인쇄하기 위한 expression이라는 명령의 별명일 뿐이다.

예를 들어 첫 번째 인수로 고유한 명령 이름을 지정한 다음 별칭을 지정할 커맨드를 입력하자.
별칭을 지정하는 방법은 command alias + 별칭 + expression + 고유명령이름

```python
expression —object-description — cruise
command alias my_po expression —object-description
my_po cruise
```

## Po Under the Hood
LLDB를 실제 입력된 표현식을 분석하거나 평가하지는 않는다. 하지만 사용자가 제공한 표현식을 컴파일 할 수 있는 소스코드를 생성한다.

내장된 Swift와 claim compiler를 사용해서 디버그된 프로그램의 컨텍스트에서 실행되는 코드를 컴파일한다.

실행이 완료되면 LLDB는 결과 값에 엑세스 할 수 있다..
이를 위해서는 LLDB가 이전 결과를 다른 소스 코드조각으로 감싸야한다. 이 ㅗ한 컴파일되고 디버그 프로세스 컨텍스트에서 수행된다. 수행 결과에 대한 문자열을 LLDB에서 보여준다.

`$R0` > 이후 LLDB에서 그대로 사용할 수 있다.
P $R0.destinations

P 또한 po와 마찬가지로 표현식을 컴파일 해서 수행한 결과에 접근 할 수 있다. 이후 LLDB는 Dynamic type resolution 라는 스텝을 수행한다. 좀 더 자세히 살펴보자.

Trip이 Activity라는 프로토콜을 채택하고 변수 cruise를 선언할 때 Static Type으로는 Activity를 선언하고 Trip 인스턴스를 할당해주자.
```swift
Let cruise: Activity = Trip(..)
```

그러면 컴파일 타임에서 cruise는 Activity Type이지만 Runtime에서 cruise는 Trip Type을 가진다.

Cruise 값을 인쇄하면 Trip Type으로 출력한다. 왜냐하면 LLDB가 결과 메타 데이터를 재전송하여 주어진 프로그램에서 이 변수에 대해 가장 정확한 유형을 표시하기 때문이다.

이를 동적 유형 확인(Dynamic type resolution) 이라고 한다.
P 명령을 사용하면 동적 결과 분석은 표현식의 결과에 대해서만 수행한다. 

그니까 cruise를 출력할 때는 Trip으로 보여주는데, cruise의 name을 출력하라고 하면 Activity에는 그런 필드가 없다고 오류를 보여줌

왜냐면 LLDB는 p가 실행될 때 소스코드에서 작성된 정적인 타입을 가지고 코드를 컴파일 하기 때문이다. 소스 코드에서 실제로 cruise.name을 작성하면 마찬가지로 컴파일 오류가 발생하는 것과 같다. 

오류없이 식을 평가하려면 먼저 객체를 Dynamic type으로 명시적인 캐스팅을 한 다음 결과에 액세스해야한다.

```swift
P (Cruise as! Trip).name
```

LLDB는 사용자가 읽을 수 있도록 Fomatter를 사용한다.
포메터를 사용하지 않았을 때는 
```
Expression —raw — cruise.name
```
이렇게 쓰면 나옴

￼
포메터를 쓰면 그냥 우리가 읽을 수 있는 문자열 형태로 나온다.
￼

포메터도 커스텀할 수 있다.

v 는 p랑 같이 포매터에 의존한다. 다른 두 명령과 마찬가지로 v 는 프레임 변수에 대한 명령을 위해 Xcode 10.2에서 도입 된 별칭이다.

다른 두 메커니즘과는 달리 v 명령은 코드를 컴파일하거나 실행하지 않기에 매우 빠르다. 하지만 코드를 컴파일링 하지 않기 때문에 디버깅에 사용하는 언어와 같을 필요는 없는 고유한 신텍스를 사용한다. 예를 들어서, .과 subscript 연산자를 필드에 엑세스 하는 방법으로 사용한다. 하지만 코드를 수행해야할 필요가 없기 때문에, resolution 을 overload 하지 않으며, 계산 속성을 수행할 수 없다. 

필요한 경우에는 p 와 po를 쓰자.

v 커멘드의 경우 프로그램 상태를 참조해서 메모리에서 변수를 찾는다. 그 다음 메모리에서 변수 값을 읽고, 동적 유형 확인을 ㅜ행한다. 사용자가 서브 필드에 엑세스 하도록 요청한 경우, 각 라운드에서 동적 유형 분석을 수행하는 각 서브 필드에 대한 단계를 반복한다.

완료되면 결과를 데이터 포맷터 서브 시스템으로 전달한다.

v는 동적 유형 확인을 여러번 수행하며, 포맷터는 동적 유형 확인을 한 번만 수행한다.

P cruise.name 은 실패하지만
v cruise.name 은 실패하지 않는다.

## Displaying Variables
￼

## Customizing Data Formatters
어떻게 디버거에 보여줄 것인지에 대해 알려준다.

기본 포멧은 매우 편리하지만 스스로 커스터마이징 해야할 일이 있을 것 이다.

이를 위해 LLDB는 다음 세가지를 지원한다.
Filters /. Tring summaries /synthetic children

${var.name} 방식으로 변수에 대한 정보를 가지고 description에 사용할 수 있다.

SBTarget, SBProcess, SBThread, SBFrame, SBValue

Xcope 11부터 Python 3 스크립트를 지원한다.
LLDB API

script를 쓰면 python shell로 이동함
lldb.frame.FindValues(“cruise”) > 변수 찾기

cruise.GetChildMemberWithName(“destination”) > 변수에서 하위 필드 찾기

GetNumChildren() GetChildAtIndex(0) 으로 배열 접근 가능

GetSummary() 로 하위 엘레멘트들에 대한 스크립트를 가져올 수 있다.


Custom Formatter를 python 파일로 작성해서 만들어보장.

그리고 그걸 lldb에서 `command script import Trip.py`로 임포트할 수 있다.

`type summary add Travel.Trip —python-function TripSummaryProvider` 로 formatter 추가하기

## summary
- LLDB에는 디버깅하는 동안 프로그램 상태를 확인할 수있는 다양한 기능이 있습니다.
- 값을 표시해야하는지, 코드를 실행해야하는지 또는 오브젝트 설명을 가져야하는지에 따라 v, p 또는 po를 사용하여 변수를 출력하라.
- 필터, 문자열 요약 및 합성 자식을 사용하여 고유 한 데이터 포맷터를 사용자 정의하거나 정의하십시오.
- 마지막으로 Python 2로 작성된 스크립트가 있으면 Python 3과 호환되도록 업데이트하십시오.

















