# What's New in Universal Links

>  📅 2019.11.18 (월)
>
> WWDC2019 | Session : 717 | Category : Networking

🔗 [What's New in Universal Links - WWDC 2019 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2019/717/)

## Overview

### **Universal Link**

- 웹과 앱의 resource 를 가리키는 Apple OS가 인식하는 http 또는 https 로 된 URL
사용자가 앱을 설치했던, 다운 받지 않았던, 하나의 URL은 콘텐츠를 대표한다.
- 앱과 웹사이트 간 안전하게 연관 되어 있다.
대표 할 수 있는 도메인을 json 형식으로 서버에서 가지고 있고, 앱에서 도메인을 적용하면 된다.
- custom URL scheme 을 universal link로 바꾸는 것을 권장한다.
cutom URL은 본질적으로 위험하고 악의적인 개발자들에 의해 악용 될 수 있다.

## Configuring Your Web Sever

- 웹 서버에서는 유효한 HTTPS 인증서가 있어야 한다.
- apple-app-site-association 파일을 추가한다.
애플 디바이스에 앱을 설치하면 운영체제가 서버가 어떤 서비스를 앱에서 사용하게 할 지 결정하기 위해 이 파일을 다운로드 한다.
`https://example.com/.well-known/apple-app-site-association` 에 저장되어야 한다.

**apple-app-site-association file**

```Swift
    {
    	"applinks": {
    		"apps": [],
    			 "details": [
    			 {
    				 "appID": "ABCDE12345.com.example.app",
    				 "path": ["/path/*/filename"]
    				 "components": [
    						{
    							"/": "/path/*/filename",
    							"#": "*fragment",
    							"?": { "widget": "?*", "grommet": "please" }						
    						},
    						{
    							{"/": "/taco/*", "exclude": true}
    						}
    					]
    			}
    		]
    	},
    }
```

- iOS 13, tvOS13, macOS 10.15 를 타겟팅 한다면 apps는 제거 가능하다.
- appID : App Identifier → Apple에서 제공된 10자리 숫자.bundleID, 배열로 여러 앱에서 같은 universal link 사용 가능
- path → components 로 대체됨
- URL fragment component 를 # 키로, query ? 로 대응 할 수 있다.
query item을 dictionary로 나타낼 수 있다.
- `"exclude": true`시 제외

### Internationalization

- URL 과 pattern-matching은 ASCII
- 지원하는 모든 국가에 대한 country-specific pattern 이 필요하다.
`{  "en", "fr", "mx", ...  } → "??"
{"en_US", "fr_CA", "de_CH", ...} → "??_??"`
- .com, .net, .org,  ccTLD 우선순위

## Configuring Your App

**Project → add Associated Domains**

```Swift
    <array>
    	<string>applinks:www.exapmle.com</string>
    	<string>applinks:*.exapmle.com</string>
    	<string>applinks:xn--fhqz97e.exapmle.xn--fiqs8s</string>
    </array>
```

More specific subdomains have higher priority

Internationalized domains must be encoded as Punycode

```Swift
    func application(_ application: UIApplication, continue userActivity: NSUserActivity,
    restorationHandler: @escaping ([UIUserActivityRestoring]?) -> Void) -> Bool {
    	guard userActivity.activityType == NSUserActivityTypeBrowsingWeb, 
    	let url = userActivity.webpageURL,
    	let components = URLComponents(url: url, resolvingAgainstBaseURL: true) else {
    		return false
    	}
    
    	for queryItem in components.queryItems ?? [] {
    		 ...
    	}
    	
    	return true
    }
```

### macOS Differences

![](/Jinha/images/What-s-New-in-Universal-Links-1.png)

### Opening Universal Links

**UIApplication**

```Swift
    UIApplication.shared.open(url, options: [.universalLinksOnly: true]) {
    	...
    }
```

**NSWorkspace**
```Swift
    let configuration = NSWorkspace.OpenConfiguration()
    configuration.requiresUniversalLinks = true
    NSWorkspace.shared.open(url, configuration: configuration) {
    	...
    }
```

## Best Practice

- Fail gracefully
기간이 만료 되었거나, 유효하지 않거나, 존재하지 않는 콘텐츠 일 때
 universal link가 앱에서 열 수 없다면, Safari View Controller를 통해 열어 주자.
최소한 어떤 이슈 인지 표시 해주자.
- Use the Smart App Banner
사용자가 웹을 방문했다면, smart banner를 사용해서 앱스토어나 앱의 콘텐츠로 링크를 제공하자.
