---
layout: post
title:  "Architecting Your App for Multiple Windows"
date:   2019-11-06 21:26:06 +0900
categories: iOS
tags: iOS Windows WWDC
author: Juhee Kim
mathjax: true
comments: true
---

* content
{:toc}

### 들어가기에 앞서서

iOS 13에서부터 멀티 윈도우를 지원합니다. 기존 앱을 더욱 유용하게 만들고 사용자의 생산성을 크게 향상시키는 환상적인 방법입니다. 이 세션은 다음 세 가지를 다룹니다.
* Changes to app lifecycle
* Using them scene delegate : `UISceneDelegate`를 알아보고 어떤 작업을 해야하는지
* Architecture: 더 나은 멀티태스킹을 위한 `Architecture Kit`의 모범사례

### Changes to app lifecycle
#### 기존 App Delegate의 역할
* Application의 프로세스 레벨 수준 이벤트를 알려주는 일
  * 앱 시작/대기/종료
  * 그 외 각종 노티 수렴
* Application의 UI 상태를 알려줌
  * 포그라운드/백그라운드 이동
  * ResignActive/BecomeActive

iOS 12 및 이전 버전에서는 하나의 Application 프로세스와 하나의 UI 인스턴스만 존재했기 때문에 관계없지만, 그 이후에는 여러 화면이 존재할 수 있으므로 `App Delegate`의 역할과 책임이 바뀌어야 합니다.

### Using the Scene Delegate
* **App Delegate** : Application의 프로세스 레벨 수준 이벤트를 알려주는 일
  * 앱 시작/대기/종료
  * 그 외 각종 노티 수렴
  * **화면(scene)이 생성/삭제되었을 때 알림**
* **Scene Delegate** :  Application의 UI 상태를 알려줌
  * 포그라운드/백그라운드 이동
  * ResignActive/BecomeActive

> 하지만 iOS 12 및 이전 버전을 유지하기 위해서는 AppDelegate 로직을 유지해야 합니다..!

#### Scene lifecycle
* `App Delegate` 
  * `didFinishLaunchingWithOptions(:)`
  * create `scene session`
  * configure `UIScene`
    * specify `UISceneDelegate`, `Storyboard`, `Subclass View`
    * 혹은 info.plist에서 정의
* `Scene Delegate` 
  * `willConeectToSession(:)`
  * `willResignActive`, `didEnterBackground`
  * `didDisconnected`
    * 화면이 백그라운드에 들어가면 시스템에 의해서 disconnected 될 수 있다. 
    * 그래서 나중에 연결될 때를 대비해서 데이터를 저장해야할 필요성이 있다.
*`App Delegate`
  * 사용자가 화면을 없애면 `didDiscardSceneSession`이 호출된다.

#### Cleanning up Discarded Session
프로세스가 실행중이 아닌 경우 시스템은 discard된 세션을 추적하고 애플리케이션이 실행된 직후 이를 다시 호출합니다.

### Architecture
#### State Restoration
앱의 화면이 여러 개가 사용될 수 있기 때문에 화면에 대한 복구가 필요합니다. 만일 상태 복원을 구현하지 않으면 멀티 윈도우 상태에서 disconnect 된 후 다시 연결되었을 때 완전히 새로운 화면이 뜰 것이기 때문에 사용자 경험에 좋지 않을 것입니다.

하지만 iOS 13에서는 이를 지원하기 위해 State Restoration API를 제공합니다. 상태를 기반으로 화면을 인코딩할 수 있으며 이는 `NSUserActivity`를 기반으로 합니다. 
  * 이는 `spotlightsearch`나 `handoff`에도 적용할 수 있습니다.
  * session이 stateRestorationActivity를 가지고 있다면 이를 기반으로 화면을 구성하고, 그렇지 않다면 새로운 화면을 구상하면 됩니다.

#### Application Sync 
만약 채팅 화면을 여러 곳에서 띄운다면? 같은 채팅방을 여러 창에서 띄울 때 동기화 문제를 어떻게 해야할까요?
  * `ViewController`가 이벤트를 수신하면 `Model Controller`에 다시 이벤트를 전달
  * `Model`을 구독하고 있는 구독자 혹은 `View Controller`에게 새로운 데이터가 업데이트 되었음을 알림

실제 구현 방식으로는 `Delegate`와 `Notification`, `Swift Combine`을 사용할 수 있습니다.
  * View Controller가 스스로 View를 업데이트하지 않고, Model이 업데이트 됨에 따라 View가 자동으로 업데이트 되도록 해야합니다.


## Summary
우리는 `AppDelegate`와 `SceneDelegate`의 차이점과 책임의 차이에 대해 살펴 보았습니다. 우리는 또한 몇 가지 주요 `SceneDelegate``method`를 알아보고 여기에서 어떤 종류의 작업을 해야하는지 살펴 보았습니다. 또한 iOS 13에서 상태 복원이 중요한 이유와 새로운 장면 기반 API를 활용하는 방법에 대해서도 설명했습니다. 마지막으로, 단방향 데이터 흐름을 생성하기 위한 일부 고수준 패턴에 대해 이야기하며, 모든 장면이 동일한 데이터를 공유하는 동안 동기화 상태를 유지할 수 있도록 하였습니다. 

* [Architecting Your App for Multiple Windows](https://developer.apple.com/videos/play/wwdc2019/258/)
