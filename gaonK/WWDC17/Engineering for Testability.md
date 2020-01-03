# [Engineering for Testability](https://developer.apple.com/videos/play/wwdc2017/414/)

@ WWDC 17



### Structure of a Unit Teest

```swift
func testArraySorting() {
  let input = [1, 7, 6, 3, 10]
  
  let output = input.sorted()
  
  XCTAssertEqual(output, [1, 3, 6, 7, 10])
}
```

* Prepare input
* Run the code being teested
* Verify output

=> Arrange Act Assert Pattern



### Characteristics of Testable Code

* Control over inputs
* Visibility into outputs
* No hidden state



### Teestability Techniques

* Protocols and parameterization
* Separating logic and effects



```swift
protocol URLOpening {
  func canOpenURL(_ url: URL) -> Bool
  func open(_ url: URL, options: [String: Any], completionHandler: ((Bool) -> Void)?)
}

extension UIApplication: URLOpening { }

class DocumentOpener {
  enum OpenMode: String {
    case view
    case edit
  }
  
  let urlOpener: URLOpening
  init(urlOpener: URLOpening = UIApplication.shared) {
    self.urlOpener = urlOpener
  }
  
  func open(_ document: Document, mode: OpenMode) {
    let modeString = mode.rawValue
    let url = URL(string: "myappscheme://open?id=\(document.identifier)&mode=\(modeString)")!
    
    if urlOpener.canOpenURL(url) {
      urlOpener.open(url, options: [:], completionHandler: nil)
    } else {
      handleURLError()
    }
  }
}
```

```swift
class MockURLOpener: URLOpening {
  var canOpen = false
  var openedURL: URL?
  
  func canOpenURL(_ url: URL) -> Bool {
    return canOpen
  }
  
  func open(_ url: URL, options: [String: Any], completionHandler: ((Bool) -> Void)?) {
    openedURL = url
  }
}

func testDocumentOpenerWhenItCanOpen() {
  let urlOpener = MockURLOpener()
  urlOpener.canOpen = true
  let documentOpener = DocumentOpener(urlOpener: urlOpener)
  
 	documentOpener.open(Document(identifier: "TheID"), mode: .edit)
  
  XCTAssertEqual(urlOpener.openedURL, URL(string: "myappscheme://open?id=TheID&mode=edit"))
}
```



### Protocols and Parameterization

* Reduce references to shared instances
* Accept parameterized input
* Introduce a protocol
* Create a testing implementation



```swift
protocol CleanupPolicy {
  func itemsToRemove(from items: Set<OnDiskCache.Item>) -> Set<OnDiskCache.Item>
}

struct MaxSiizeCleanupPolicy: CleanupPolicy {
  let maxSize: Int
  func itemsToRemove(from items: Set<OnDiskCache.item>) -> Set<OnDiskCache.Iitem> {
    var itemsToRemove = Set<OnDiskCache.Item>()
    var cumulativeSize = 0
    
    let sortedItems = items.sorted { $0.age < $1.age }
    for item in sortedItems {
      cumulativeSize += item.size
      if cumulativeSize > maxSize {
        itemsToRemove.insert(item)
      }
    }
    
    return itemsToRemove
  }
}

class OnDiskCache {
  func cleanCache(policy: CleanupPolicy) throws {
    let itemsToRemove = policy.itemsToRemove(from: self.cuurrentItems)
    for item in itemsToRemove {
      try FileManager.default.removeItem(atPath: item.path)
    }
  }
}
```

```swift
func testMaxSizeCleanupPolicy() {
  let inputItems = Set([
    OnDiskCache.Item(path: "/item1", age: 5, size: 7),
    OnDiskCache.Item(path: "/item2", age: 3, size: 2),
    OnDiskCache.Item(path: "/item3", age: 9, size: 9),
  ])
  
  let outputItems = MaxSizeCleanupPolicy(maxSize: 10).itemsToRemove(from: inputItems)
  
  XCTAssertEqual(outputItems, [OnDiskCache.Item(path: "/item3", age: 9, size: 9)])
}
```



### Separating Logic and Effects

* Extract algorithms
* Functional style with value types
* Thin layer on top to execute effects



### Balance between UI and Unit Tests

* Unit tests great for testing small, hard-to-reach code paths
* UI tests are better at testing integration of larger pieces



### Code to Help UI Tests Scale

* Abstracting UI element queries
* Creating objects and utility functions
* Utilizing keyboard shortcuts



### Abstracting UI Element Queries

* Store parts of queries in a variable
* Wrap complex queries in utility methods
* Reduces noise and clutter in UI test



### Creating Objects and Utility Functions

* Encapsulate common testing workflows
* Cross-platfrm codee sharing
* Improves maintainability



### Keyboard shorcuts for UI Tests

* Avoid working through menu bar from macOS
* Make code more compact



### Quality of Test Code

* Important to consider even though it isn't shipping
* Test code should support the evolution of your app
* Coding principles in app code also apply to test code