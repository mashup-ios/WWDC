# [Embedding and Sharing Visually Rich Links](https://developer.apple.com/wwdc19/262)

@ WWDC 19

rich하게 링크를 표현할 수 있다! LinkPresentation 프레임워크를 이용해서!



### Retrieving metadata

```swift
// URL로부터 link presentation에 사용될 metadata 가져오기
let metadataProvider = LPMetadataProvider()
let url = URL(string: "https://www.apple.com/ipad")!

metadataProvider.startFetchingMetadata(for: url) { metadata, error in
    if error != nil {
        // The fetch failed: handle the error.
        return
    }
    
    // TODO: Make use of fetched metadata.
}
```

#### LPMetadataProvider and LPLinkMetadata

* Title, icon, image and video are all optional
* `LPLinkMetada` is serializable
* Can also be used for local fiel URLs



### Presenting links

```swift
let linkView = LPLinkView(metadata: metadata)
cell.contentView.addSubview(linkView)
linkView.sizeToFit()
```

`LPLinkView` 는 intrinsic size를 가지지만 주어진 제약에 맞게 size를 조절할 수 있다. (`sizeToFit()` 함수를 이용하는 것을 말하는 듯!)

### Accelerating the share sheet

원래는 async하게 동작하기 때문에 좀 늦게 뜨고 그럴 수 있다! prefetch를 통해서 좀 더 빨리 뜨게 해보자.

```swift
// UIActivityViewController에 UIActivityItemSource를 이용해서 prefetch된 metadata 전달
func activityViewControllerLinkMetadata(_: UIActivityViewController) -> LPLinkMetadata {
    return self.metadata
}

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

+) share sheet 사용해서 메시지로 보내면 rich하게 보내진다

### Summary

* Use `LPMetadataProvider` to fetch rich metadata for a URL
* Use `LPLinkView` to beautifully present links in your app
* Prefetch `LPLinkMetadata` to accelerate the share sheet preview in your app