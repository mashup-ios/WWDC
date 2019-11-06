# [Advances in Foundation](https://developer.apple.com/videos/play/wwdc2019/723/)

@ WWDC 19



### Ordered Collection Diffing

[B, E, A, R] 에서 [B, I, R, D] 를 만드려면

1. 공통되는 요소를 찾는다. [B, R]
2. [B, E, A, R]에서 공통되지 않는 요소 제거 -> [B, R]
3. [B, I, R, D]로 만들기 위해 필요한 요소 삽입 -> [B, R] -> [B, I, R, D]



```swift
let diff = bird.difference(from: bear)
let newBird = bear.applying(diff)
```



### Data

#### Contiguity

`Data`는 연속적임이 보장된다.

```swift
public protocol ContiguousBytes {
  func withUnsafeBytes<R>(_ body: (UnsafeRawBufferPointer) throws -> R) rethrows -
  > R
}
```

`ContiguousBytes` 프로토콜을 채택함으로써 차후 flattening을 걱정할 필요가 없어진다.

```swift
public protocol DataProtocol: RandomAccessCollecton where Element == UInt8, ... { }

public protocol MutableDataProtocol: DataProtocol, MutableCollection, RangeReplaceableCollection { }
```

연속성을 보장하지 않는 다른 버퍼 타입에 대해서는 채택할 수 있는 두 개의 프로토콜을 소개한다.

#### Compression

```swift
let compressed = try data.compressed(using .lzfse)

public enum CompressionAlgorithm: Int {
  case lzfse
  case lz4
  case lzma
  case zlib
}
```

데이터 압축은 위와 같은 api를 이용해서 쉽게 처리 가능하다. 



### Units

* `UnitDuration`
  * Added `milliseconds`, `microseconds`, `nanoseconds`, and `picoseconds`
* `UnitFrequency`
  * Added `framePerSecond`

* `UnitInformationStorage`
  * `bits`, `bytes`, `nibbles` for common usage
  * SI- and binary-prefixed units (`kilo`, `kibi`, ..., `yotta`, `yobi`)
  * Format with `MeasurementFormatter` and `ByteCountFormatter`



### Relative Date Formatter

```swift
let formatter = RelativeDateTimeFormatter()
let dateString = formatter.localizedString(for: aDate, relativeTo: now)
```



### List Formatter

```swift
let string = ListFormatter.localizedString(byJoining: ["🐶", "🐷", "🦄"])
```

```swift
let listFormatter = ListFormatter()
let dateFormatter = DateFormatter()

listFormatter.itemFormatter = dateFormatter
let string = listFormatter.string(from: dates)

// en_US: "8/15/19, 9/13/19, and 2/1/20"
// en_ES: "15/8/19, 13/9/19 y 1/2/20"
```

```swift
let listFormatter = ListFormatter()
let dateFormatter = DateFormatter()
dateFormatter.dateStyle = .medium

listFormatter.itemFormatter = dateFormatter
let string = listFormatter.string(from: dates)
```



### Operation Queue

모든 작업이 끝나고 난 다음 저장을 하고 싶다면

```swift
// 잘못된 예시
if (queue.operationCount == 0) {
  save()
}
// 잘 작동하는 예시
queue.addBarrierBlock {
  save()
}
```

이 경우 미리 스케쥴 된 작업들이 완료되고 난 후 save()가 실행됨을 보장한다.



#### Progress reporting

```swift
let queue = OperationQueue()
queue.progress.totalUnitCount = 3

queue.addOperation {
  task1()									// Finished task: 1 / 3
}

queue.addOperation {
  task2()									// Finished task: 2 / 3
}

queue.addOperation {
  task3()									// Finished task: 3 / 3
}
```



### Be Prepared for USB and SMB on iOS

* Multiple volumes
  * Use `FileManager.SearchPathDirectory.itemReplacementDirectory`
* Disappearing volumes
  * Use `Data.ReadingOptions.mappedIfSafe`
* Slower file system operations
  * Defer access to non-main thread
* Varying capabilities
  * Test capabilites with `URLResourceKey`, e.g. `volumeSupportsFileCloningKey`
  * Handle errors



### Swift Update

#### Scanner

```swift
// Swift 4
var nameNSString: NSString?
if scanner.scanUpToCharacters(from: .newlines, into: &nameNSString) {
  let name = nameNSString! as String
}

// Swift 5.1
let nameString = scanner.scanUpToCharacters(from: .newlines)
let matchedString = scanner.scanString(string: "hi, 👩‍💻") // emoji 가능
```

#### FileHandle

* Error-based API

  ```swift
  let fileHandle = FileHandle()
  let data = try fileHandle.readToEnd()
  ```

* Works with `DataProtocol`

  ```swift
  extension FileHandle {
    public func write<T: DataProtocol>(contentsOf data: T) throws
  }
  ```



### Try It

* Use `DataProtocol` instead of `[UInt8]`
* Format dates and lists with `Formatter`
* Use `OperationQueue`'s barrier and progress reporting

