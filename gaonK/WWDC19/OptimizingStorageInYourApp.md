# [Optimizing Storage in Your App](https://developer.apple.com/videos/play/wwdc2019/419/)

@ WWDC 19

> Better performance and efficiency

### 스토리지 최적화를 통해 얻을 수 있는 이점은?

* 긴 배터리 수명
* 더 좋은 성능
* 디스크 footprint 감소
* 더 좋은 디바이스 상태



### 오늘 말할 순서는 이렇다!

* Efficient image assets
* File system metadata
* Syncing to disk
* Serialized data files
* Core Data
* SQLite



### Efficent image assets 효율적인 이미지 에셋

이미지 에셋은 화면 사이즈에 맞게 크기가 조정된다.

HEIC과 Asset Catalog를 이용해서 footprint를 줄여라!

#### HEIC

* JPEG를 대체할 수 있는 더 효율적인 포맷
* 동일 퀄리티 대비 ~50% 더 작은 파일 크기
* 더 작은 디스크 footprint
* 더 빠른 네트워크 다운로드, 업로드
* 더 빠른 디스크 로드, 세이브
* auxiliary image를 저장한다. (depth, disparity 등의 정보 저장)
* alpha를 지원함
* 무손실 압축
* 하나의 컨테이너에 여러 개의 이미지
* iOS 11, macOS High Sierra부터 지원되는 포맷

#### Asset Catalog

* 간단하게 앱의 리소스를 관리
* 앱 아이콘
* 다양한 디바이스, 사이즈에 적용되는 이미지
* On demand resources 실제 사용하고자 할 때 사용됨
* Storage efficiency
  * On-disk foot pprint
  * App slicing (iOS) - 앱스토어에서 다운 받을 때 다운 받는 기기에 필요한 에셋만 다운로드 받기 때문에 더 효율적임
* Performance
  * 이미지를 더 빨리 로딩함
  * 앱 런치 시점에 더 퍼포먼스가 좋다 (up to 10% faster!)
  * GPU 베이스라 기본적으로 무손실 압축을 지원한다. 
  * 손실되게 압축도 가능
    * 압축풀 때 더 빠름
    * 낮은 메모리 footprint



### File System Metadata

* 파일 시스템의 메타데이터를 업데이트하는 것은 비용이 든다.

* APFS를 체크해보자 (Apple File System)
* Efficient non-persistent file
  * 파일 생성
  * 오픈된 상태로 두고 연결은 하지 않는다 (Keep it open and unlinked)
  * `fsync`를 호출하지 마라



### Syncing to Disk

User Space - App

Kernel - OS Cache (Logical I/O)

Hardware - Disk Cache, Permanent Storage (Physical I/O)



* `fsync()`
  * Flushes cached data to disk
    * OS cache에서 disk cache로 데이터를 옮긴다.
    * 데이터가 permanent storage로 바로 write 되지 않을 수 있다.
    * 순서를 보장하지 않는다.
    * 비용이 많이 든다.
    * OS에 의해 주기적으로 실행된다.
* `F_FULLFSYNC`
  * cache를 disk로 옮긴다.
    * disk cache에 있는 모든 데이터를 옮긴다.
    * 비용이 많이 든다.
    * OS에 의해 주기적으로 실행된다.
* `F_BARRIERFSYNC`
  * I/O ordering을 만든다.
    * `fsync()` with a barrier
  * `F_FULLFSYNC`보다 훨씬 비용이 적게 든다.



순서를 보장하기 위해서 `F_BARRIERFSYNC`를 사용해라! 



### Seralized Data Files

예를 들면, Plists, XML, JSON

* 장점
  * 편리하다
  * 자주 수정되는 데이터가 아닐 때 훌륭하다
  * 분석하기 쉽다.
* 단점
  * 모든 변경점마다 파일 전체가 새로 작성된다.
  * 확장성이 좋지 않다.
  * 잘못 사용하기 쉽다.
  * 메타데이터의 집약체이다.
* 데이터베이스로의 의미가 없다.



### Core Data

* Core Data management

  * SQLite 기반
  * Object graph와 relationship을 관리한다.
  * 변화를 감지(tracking)하고 알린다(notify). 
  * 자동적으로 버전 트래킹을 하고 여러 작성자가 있음으로써 생기는 충돌을 해결한다.
    * Automatic connection pooling
  * CloudKit과의 통합 (iOS 13부터)
  * 실시간 쿼리
  * Automatic memory management
  * Statement aggregation in transactions
  * Schema migrations
  * Denomalization (iOS 13부터)
  * 그 외 기타 등등

  

### SQLite

> Journaling은 트랜잭션과 관련이 있다.

* Connection을 열고 닫는 것은 비용이 많이 드는 작업을 유발할 수 있다!
  * Consistency checking
  * Journal recovery
  * Journal checkpointing
* Recommended usage model
  * Connection 상태를 open인 채로 가능한 오래 유지해라
  * 필요할 때만 connection을 close해라
  * Pool connections on multi-threaded processes
* WAL Mode Journaling Reduces Writes (대부분의 경우 WAL mode 사용이 효율적임)
  * Write ahead logging
  * Combines multiple writes to the same page
  * Uses fewer barriers
  * Supports multiple readers at the same time as a writer
  * Suports snapshots

#### Transactions

* Transactions Help Reduce Writes

  * Multiple `INSERT`, `UPDATE`, `DELETE` statement에 쓰임

  * Multiple statement에 의한 페이지 변경은 한 번만 write 된다.
  * Useful for aggregating changes over time!

#### File Size and Privacy

```
PRAGMA schema.secure_delete=FAST;
```

* How do we securely delete sensitive data?
  * Automatically zeros deleted data
  * No cost for data within the same page as the header
  * Default behavior starting with iOS 13
* Don't use `VACUUM`
  * `VACUUM` is a slow I/O intensive operation
  * 저널 파일에 썼닫가 다시 데이터베이스에 쓰는 과정은 cost가 비싸다.
  * All valid data gets written at least twice!

```
PRAGMA schema.auto_vacuum=INCREMENTAL;
```

```
PRAGMA schema.incremental_vacuum(N);
```

* Incremental auto vacuum과 fast secure delete를 이용해서 file size와 privacy를 관리해라.

#### Partial Indexes

* Index
  * Faster ORDER BY, GROUP BY and WHERE clauses
  * Each index adds additional I/O when writing to the database
* Partial Indexes
  * Indexes with a WHERE clause
  * Partial indexes only store data which matches the WHERE clause
  * Queries that do not match the WHERE clause cannot use the index



### File Activity Instrument

* Supports all Apple devices
* Support for tracing single process and all processes
* Obtain both logical and physical I/O information
* Offers automated reasoning

#### Automated Reasoning

* Excessive physical writes
* Failed I/O related calls
* Suboptimal caching



### Summary

* Apply these lessons
* Profile with File Activity Instrument
* Continue optimizing storage!