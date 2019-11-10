

---
layout: post
title:  "Delivering Intuitive Media Playback with AVKit"
date:   2019-11-10 21:26:06 +0900
categories: iOS
tags: iOS WWDC Link WebKit
author: Juhee Kim
mathjax: true
comments: true
---

## [Delivering Intuitive Media Playback with AVKit](https://developer.apple.com/videos/play/wwdc2019/503/)

### AVKit in the Stack
`AVKit`은 `CoreMedia`의 `AVFoundation`을 기반으로 구축 된 크로스 플랫폼 미디어 재생 UI framework입니다. Apple의 자체 앱에서 사용하는 것과 동일한 사용자 인터페이스를 사용하여 `AVPlayer` 기반 미디어 컨텐츠를 쉽게 표시하고 재생할 수 있도록하는 것이 우리의 임무입니다.

* UIKit 앱의 경우 `AVPlayerViewController`를 제공합니다.
* `APKit` 앱의 경우 `AVPlayerView`를 제공합니다.
* 또한 UIKit 및 APKit 모두 재생 선택화면을 추가하는 `AVRoutePickerView`를 제공합니다.

```swift
// Media Playback with AVPlayerViewController on iOS and tvOS
import AVKit

 // 1) AVPlayer 만들기
 let player = AVPlayer(url: “https://my.example/video.m3u8”)

// 2) AVPlayerViewController 만들고 AVPlayer 객체 할당해주기
let playerViewController = AVPlayerViewController()
playerViewController.player = player

 // 3) present AVPlayerViewController
 present(playerViewController, animated: true)
```

### Media playback with AVKit
이렇게 하면 Apple이 제공하는 가장 기본적인(그리고 기본 앱과 동일한) UI의 미디어플레이어를 생성할 수 있습니다. 또한 `CoreMedia` 플레이어 UI안에서 `AVFoundation`의 모든 기능 - `AVPlayer`, `AVPlayerItem`, `AVAsset` - 들을 사용할 수 있습니다.

그러면서도 AVKit은 UIKit 및 AppKit 위에 있기 때문에 각 Apple platform에 대응되는 고유 UI를 가지기 때문에 더 나은 경험을 제공할 수 있습니다.

### What's new for iOS 13
#### AVPlayerViewController
* Full screen callbacks
* AVPlayerViewController in iPad apps on the Mac
* External metadata
* Improved support for custom controls

#### Full screen callbacks
`AVPlayerViewControllerDelegate` : 전체 화면 재생이 시작되거나 끝날 때 알림을 받을 수 있음, iOS 12 부터 가능  

```swift
@available(iOS 12.0, *)
func playerViewController(_ playerViewController: AVPlayerViewController, willBeginFullScreenPresentationWithAnimationCoordinator coordinator: UIViewControllerTransitionCoordinator)

func playerViewController(_ playerViewController: AVPlayerViewController, willEndFullScreenPresentationWithAnimationCoordinator coordinator: UIViewControllerTransitionCoordinator) {
  coordinator.animate(alongsideTransition: { (context) in
    // Add coordinated animations
  }) { (context) in
    // 여기서 전환이 성공햇는지, 취소됐는지 알 수 있습니다.
    if context.isCancelled {
      // Still embedded inline
    } else {
      // Presented full screen
      // Take strong reference to playerViewController if needed
    }
  }
}
```
만약 스크롤뷰에 플레이어 뷰 컨트롤러가 있다고 했을 때 전체화면 상태에서 기기가 회전하면서 스크롤뷰 오프셋이 변경되는 경우가 있는데, 테이블 뷰나 컬렉션 뷰에서 이럴 경유 뷰 자체가 할당해제될 수 있어서 문제가 될 수 있습니다. 그래서 context가 정상적으로 전체화면으로 변경되었는지 확인한 다음 필요하다면 `강한 참조`를 사용해야 합니다.

#### AVPlayerViewController in iPad Apps on the Mac
* Exact same API as on iOS Picture-in-picture support
* Includes AVPictureInPictureController (also for AppKit) And AVPlayerView, for AppKit-based apps
* Touch Bar, keyboard, and Now Playing support Audio and AirPlay routing
* Available in macOS 10.15

#### External Metadata
Title, artwork, 혹은 그 외의 메타데이터를 AVKit이 자동으로 처리합니다.

혹은 원하는 Metadata를 보완하도록 할 수 있습니다.
```swift
@available(iOS 12.0, *)
extension AVPlayerItem {
  open var externalMetadata: [AVMetadataItem] }
}
```

#### Custom Playback Controls
* `showsPlaybackControls = false`
* present `AVPlayerViewController` modally
* Add controls to `contentOverlayView`
* For the best user experience, you should:
  * Override UIViewController methods for status bar, home indicator
  * Pass unhandled touches through your view
  * Let AVKit handle double-tap for video zoom

### AVPlayerViewController on iOS : Best practices
* Showing full screen
  * Covers `UIWindowScene` coordinate space
    * Splash screen
    * Full screen playback
* Embedding inline
* Picture-in-picture

#### Showing Full Screen Usecase1: Splash screen
```swift
// Add as child:
parent.addChild(playerViewController)
parent.view.addSubview(playerViewController.view) // Or other UIView insertion API

// Enable auto layout and set up constraints
playerViewController.didMove(toParent: parent)

// Remove from parent:
playerViewController.willMove(toParent: nil)
playerViewController.view.removeFromSuperview()
playerViewController.removeFromParent()

// 재생 컨트롤을 표시하지 않음:
playerViewController.showsPlaybackControls = false

// 전체 화면을 채우도록 함:
playerViewController.videoGravity = .resizeAspectFill

// 필요하다면 백그라운드 컬러 변경:
playerViewController.view.backgroundColor = .clear // Or any other UIColor
```

* `AVPlayer.allowsExternalPlayback` 비활성화
* `AVAudioSession`을 2차 media playback으로 설정
  * Use `.ambient` category
* Observe `AVAudioSession.silenceSecondaryAudioHintNotification`
  * Honor `AVAudioSession.secondaryAudioShouldBeSilencedHint`

#### Showing Full Screen Usercase 2: Full screen playback
```swift
// Media Playback with AVPlayerViewController on iOS and tvOS
import AVKit
// 1a) Create an AVPlayer
let player = AVPlayer(url: “https://my.example/video.m3u8”)
// 1b) Add external metadata if needed
player.currentItem?.externalMetadata = // Array of [AVMetadataItem]
// 2) Create an AVPlayerViewController
let playerViewController = AVPlayerViewController()
playerViewController.player = player
// 3) Show it
present(playerViewController, animated: true)
```

* Present modally : 상태 표시 줄 처리가 용이
* Use the default modalPresentationStyle
  * 기본값을 사용하는 이유는 화면 뷰가 UIKit에 의해서 제거될 수 있기 때문
  * 화면이 회전될 때 보이지 않는 다면 레이아웃을 할 이유가 없음
* Do not set AVPlayerViewController.videoGravity
* 전체 화면 상태를 확인하기 위해서는 `AVPlayerViewControllerDelegate`를 사용합시다.
  * Do not rely on viewWillAppear(_:) and friends
* `AVPlayerItem`를 미리 설정해두기
  * Set AVPlayer.rate for setting item
* `AVPlayer.status` 와 `AVPlayerItem.status` 관찰해서 제어하기
  * Don’t begin playback until status is `.readyToPlay`
  * Check `error` property when status is `.failed`
    * Rebuild AVFoundation objects if `.mediaServicesWereReset`
* Enable `AVPlayer.usesExternalPlaybackWhileExternalScreenIsActive`
* Configure `AVAudioSession` for `.playback`

#### Embedding AVPlayerViewController Inline
* Be prepared to begin and end full screen playback
  * `AVPlayerViewControllerDelegate` 사용해서 트래킹
  * Keep strong reference to embedded `AVPlayerViewController` when full screen
  * Retarget dismissal animations when ending full screen presentation
* Leave `modalPresentationStyle` as `.fullScreen`
* Can style embedded content without impacting full screen:
  * `videoGravity`
  * `view.layer.cornerRadius, cornerCurve, and maskedCorners`
  * `view.backgroundColor`
* For automatically entering and exiting full screen:
  * `entersFullScreenWhenPlaybackBegins`
  * `exitsFullScreenWhenPlaybackEnds`
* Adopt UIViewController containment API
