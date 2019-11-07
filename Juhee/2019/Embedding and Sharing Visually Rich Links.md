---
layout: post
title:  "Embedding and Sharing Visually Rich Links"
date:   2019-11-06 21:26:06 +0900
categories: iOS
tags: iOS WWDC Link WebKit
author: Juhee Kim
mathjax: true
comments: true
---

## [Embedding and Sharing Visually Rich Links](https://developer.apple.com/wwdc19/262)

### Retrieving metadata

`URL`이 있다면 `LinkPresentation` Framework를 사용해서 `LP metadata provider` 클래스를 사용하여 메타데이너를 쉽게 가져올 수 있다. 그러면 `LPLinkMetada`가 반환되는데, 여기에는 기본적으로 title이 포함되어 있다.

```swift
// URL로부터 link presentation에 사용될 metadata 가져오기
let metadataProvider = LPMetadataProvider()
let url = URL(string: "https://www.apple.com/ipad")!

metadataProvider.startFetchingMetadata(for: url) { metadata, error in
    if error != nil {
        // 네트워크가 너무 느리거나, 없을 경우 오류가 나타난다.
        return
    }

    // TODO: Make use of fetched metadata.
}
```

#### LPMetadataProvider and LPLinkMetadata

* Title, icon, image and video are all optional
  * 결과 메타 데이터 객체는 사이트가 지정하지 않은 경우 제목, 아이콘, 이미지 또는 비디오를 전혀 포함하지 않을 수도 있다.
* `LPLinkMetada` is serializable
* Can also be used for `local file URLs`
  * 이 경우 `QuickLook` 썸네일 API를 사용해서 파일의 대표 썸네일을 가져올 수 있다.


### Presenting links

```swift
// URL로부터 link presentation에 사용될 metadata 가져오기
let metadataProvider = LPMetadataProvider()
let url = URL(string: "https://www.apple.com/ipad")!

metadataProvider.startFetchingMetadata(for: url) { metadata, error in
    if error != nil {
        // 네트워크가 너무 느리거나, 없을 경우 오류가 나타난다.
        return
    }

    // TODO: Make use of fetched metadata.
    let linkView = LPLinkView(metadata: metadata)
    cell.contentView.addSubview(linkView)
    linkView.sizeToFit()
}
```

`LPLinkView` 는 intrinsic size를 가지지만 주어진 제약에 맞게 size를 조절할 수 있다.

### Accelerating the share sheet
타이틀과 이미지가 뜨는 데 시간이 다를 수 있다. 서버를 찔러야 하기 때문이징!
하지만 한번 `LPLinkMetadata`를 가져오고 나면 빠르게 불러올 수 있다.

```swift
// UIActivityViewController에 UIActivityItemSource를 이용해서 prefetch된 metadata 전달
func activityViewControllerLinkMetadata(_: UIActivityViewController) -> LPLinkMetadata {
    return self.metadata
}
```

### 커스텀 metadata
만약 metadata를 이미 가지고 있다면 굳이 서버로 통신할 필요 없이 metadata 객체를 만들어서 반환하면 된다.

```swift
// 커스텀 metadata 전달
func activityViewControllerLinkMetadata(_: UIActivityViewController) -> LPLinkMetadata {
    let metadata = LPLinkMetadata()
    metadata.originalURL = URL(string: "https://www.example.com/apple-pie")
    metadata.url = metadata.originalURL
    metadata.title = "The Greatest Apple Pie In The World"
    metadata.imageProvider = NSItemProvider.init(contentsOf: Bundle.main.url(forResource: "apple-pie", withExtension: "jpg"))
    return metadata
}
```

### Summary

* Use `LPMetadataProvider` to fetch rich metadata for a URL
* Use `LPLinkView` to beautifully present links in your app
* Prefetch `LPLinkMetadata` to accelerate the share sheet preview in your app
