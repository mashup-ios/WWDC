# Seamless Linking to Your App

>  📅 2019.11.20 (WED)
>
> WWDC2015 | Session : 509 | Category : Safari and Web

🔗 [Seamless Linking to Your App - WWDC 2015 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2015/509/)

# Building Seamless Integration

![](/Jinha/images/Seamless-Linking-to-Your-App-1.png)

## Linking to Your App

Often web content mirrors app content

An app exists as part of an ecosystem

If your app, or website, needs to access content provided by another app, or website, it needs to communicate with it somehow

### Handling URLs in Your App

**Custom URL schemes**

- Let apps communicate with each other
- Don't always map to your app
Two apps can claim the same URL scheme
- Don't work without your app installed
- Don't protect users' privacy

⬇️⬇️⬇️

- Let apps communicate with each other just as easily
There should be a strong two-way association between your app and your URLs.
- Securely map to apps you choose
- Work Universally, fall back to Safari
Safari, and web browsers in general, are a great way to display rich content, and are present on nearly every platform, so we'll make sure that these links can open in a web browser.
- Protect users' privacy
There is no need for a third party to be involved.

### Breakdown of a Universal Link

[`https://developer.apple.com/videos/wwdc/2014/?include=101#101`](https://developer.apple.com/videos/wwdc/2014/?include=101#101)

- Scheme : "https" or "http"
- Domain(or Host name) :  developer.apple.com
→ The domain is securely associated with your app, by using an SSL certificate to sign a file that is stored on your secure web server.
- Path or path prefix : /videos/wwdc/2014/ 
→ The path can either match exactly, or it can match with a prefix.

![](/Jinha/images/Seamless-Linking-to-Your-App-2.png)

**Universal links mean having your app, not Safari, handle links to your website**, but in order to get in order to open your app, when a user taps a link to your website, iOS needs to know that your app should open that link.

## Getting Your Server Ready

- Create your "apple-app-site-association" file
- Generate an SSL certificate
- Sign your file
- Upload to your server

### UIApplicationDelegate support

```Swift
    func application(application: UIApplication, continueUserActivity 
    userActivity, NSUserActivity, restorationHandler: ([AnyObject]!) -> Void) ->
    Bool 
    
    var webpageURL: NSURL?
    var activityType: String
    let NSUserActivityTypeBroswingWeb: String
```

![](/Jinha/images/Seamless-Linking-to-Your-App-3.png)

Project Setting → Capabilities tab → select associated domains 

### Best practices

- Validate input
- Fail gracefully
- Send data over HTTPS

🔗 Networking with NSURLSession [https://developer.apple.com/videos/play/wwdc2015/711](https://developer.apple.com/videos/play/wwdc2015/711/)

## Linking to Your App

### Some advantages of universal links

- No flashes or bouncing through Safari
- No code required on your website
- Graceful fallback to Safari
- Platform-neutral
- No extra sever round-trips
- Full user privacy protection

## Smart App Banners

### Promote your app from your website

![](/Jinha/images/Seamless-Linking-to-Your-App-4.png)

### Deploying smart app banners

Add a `meta` tag to your website's `head` :

```Swift
    <head>
    	<meta name="apple-itunes-app" content="app-id=640199958, app- argument=https://developer.apple.com/wwdc/schedule, 
       affiliate- data=optionalAffiliateData">
    </head>
```

`app-argument` is passed to `application(_:openURL:sourceApplication:annotation:)`, and is also used to index your app.

![](/Jinha/images/Seamless-Linking-to-Your-App-5.png)

## Shared Web Credentials

### Help your users log into your app

### How Safari helps users and you

- Securely save credentials for users
- Sync to all devices with iCloud Keychain
- Suggest secure passwords for users
- AutoFill saved credentials

### Adopting shared web credentials

- Associate your app and your website
- Teach your app to ask for Shared Web
- Credentials

![](/Jinha/images/Seamless-Linking-to-Your-App-6.png)
