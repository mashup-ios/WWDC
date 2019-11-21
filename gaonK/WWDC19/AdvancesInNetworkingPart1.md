# [Advancd in Networking, Part 1](https://developer.apple.com/videos/play/wwdc2019/712/)

@ WWDC 19



### Low Data Mode

설정에서 low data mode를 설정할 수 있다. 이 옵션을 켰을 때 앱이 어떻게 작동할 지 개발하면서 설정해 줄 수 있다.

* User preference to minimize data usage
  * Explicit signal to reduce network data use
  * Peer Wi-Fi and Cellular network
* System plicy
  * Discretionary tasks deferred
  * Background App Refresh disabled



#### Application Adoption

내 앱에 적용할 때!, 이 옵션이 켜져 있다면

* Reduce image quality
* Reduce pre-fetching
* Synchronize less often
* Mark tasks discretionary
* Disble auto-play
* Do not block user-initiated work



#### Low Data Mode APIs

* URLSession
  * Try large/prefetch with `allowsConstrainedNetworkAccess = false`
  * On failure with `error.networkUnavailableReason == .constrained` try Low Data Mode alternative
* Network framework
  * Set `prohibitConstrainedPaths` on `NWParameteers`
  * Handle path updatees



#### Constrained and Expensive

* Constrained - Low Data Mode
* Expensive - Cellular and Personal Hotspot
* URLSession
  * `allowsExpensiveNetworkAccess`
* Network.framework
  * Set `prohibitExpensivePaths` on `NWParameters`
  * Check `isExpensive` on `NWPath`



### Combine in URLSession

13부터 나온 Combine을 어떻게 URLSession과 함께 쓰는지 데모와 함께 설명. Combine에 대해 좀 알고 봐야 이해가 가능하지만 대강 Rx 살짝 맛봤던 정도로 대강의 이해는 가능

* Streamline networking code with Combine
* Support retry
  * Use low retry count
  * Only idempotent request
* Best practices for Low Data Mode



```swift
// Generalized Publisher for Adaptive URL Loading
func adaptiveLoader(regularURL: URL, lowDataURL: URL) -> AnyPPublisher<Data, Error> {
  var request = URLRequest(url: regularURL)
  request.allowsConstrainedNetworkAccess = false
  return URLSession.shared.dataTaskPublisher(for: request)
  	.tryCatch { error -> URLSession.DataTaskPublisher in
			guard error.networkUnavailableReason == .constrained else {
        throw error
      }
			return URLSession.shared.dataTaskPublisher(for: lowDataURL)
    }
  	
  	.tryMap { data, response -> Data in
			guard let httpResponse = response as? HTTPURLResponse,
				httpResponse.statusCode == 200 else {
          throw MyNetworkingError.invalidServerResponse
        }
				return data
		}
  	.eraseToAnyPublisher()
}
```



### WebSocket

웹소켓 이용하면 양방향 통신이 가능하다.

* Two-way communication over TLS/TCP connection
* Works with Firewalls and CDNs
* Proxy support



이번에 URlSessioon과 Network.framework에서 웹 소켓을 이용 가능하게 만들었다!

#### URLSessionWebSocketTask

* Foundation APi for WebSocket
* Works with existing URLSession

```swift
// Create with URL
let task = URLSession.shared.webSocketTask(with: URL(string: "wss://websocket.example")!)
task.resume()

// Send a message
task.send(.string("Hello")) { error in /* Handle error */ }

// Receive a message
task.receive { result in /* Handle result */ }
```

#### Network.framework

* Both client and server support
* Receive partial or complete WebSocket messages

```swift
// Create parameters for WebSocket over TLS
let parameters = NWParameters.tls
let websocketOptions = NWProtocolWebSocket.Options()
parameters.defaultProtoocolStack.applicationProtocols.insert(websocketOptions, at: 0)

// Create a connection with those parameters
let websocketConnection = NWConnection(to: endppoint, using: parameters)

// Create a listener with those parameters
let websocketListener = try NWListener(using: parameters)
```



#### WebSocket APIs

* WebKit
  * JavaScript WebSocket
* URLSession
  * URLSessionWebSocketTask
* Network.framework
  * WebSocket Connection
  * WebSocket Listener



### Mobilty Improvements

네트워크를 이야기할 때, mobility 중요! Wi-Fi Assist, Multipath Transport가 강한 iOS 13 소개



#### Wi-Fi Assist in iOS 13

Cross-layer Mobility Detection, Improved Flow Recovery



* Use high-level APIs like URLSession or Network.framework
* Rethink `SCNetworkReachability` usage
* Control access with `allowsExpensiveNetworkAccess = false`



#### Multipath Transports

Responsiveness for Maps, Fewer streaming stalls in Music



* Multipath Transports for your App
  * `multipathServiceType` URLSessionConfiguration and Network.framework
* Server-side configuration
  * Linux Kernel at https://multipath-tcp.org



#### Mobility Improvements

* Mobility should not impair your Apps
* Use high-level APIs
* Rethink interface management
* Prepare your servers and use `multipathServiceType`



