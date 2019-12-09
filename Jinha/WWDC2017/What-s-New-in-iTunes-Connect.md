# What's New in iTunes Connect

>  📅 2019.12.09 (MON)
> WWDC2017 | Session : 302 |  Category : App Store Connect


🔗 [What's New in iTunes Connect - WWDC 2017 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2017/302/)


## Ratings, revies, and responses

![](/Jinha/images/What-s-New-in-iTunes-Connect.md/Untitled.png)

Why customers write reviews in the first place

→ Sometimes customers just love our apps and they want ot let the world know

→ Ohter times customers may be a little confused as to why maybe you've included or not included certain features or maybe around design decisions you may have made

→ Finally, customers often take to reviews to let us know about bugs or crashes in our apps.

![](/Jinha/images/What-s-New-in-iTunes-Connect.md/Untitled1.png)

Every review only has onew response. So customer can update the review as much as they like and you can update your reponse as often as you'd like, but this is not a threaded conversation. One paring per customer.

### Notifications

**Customer**

- Push and email notification
- Developer response submitted or updated

**Developer**

- Email notification
- Customer review updated

### Best practices

- Be responsive
- Stay on topic
- Leverage "What's New" text
- Be sensitive to customer privacy

> Related: What's New in StroreKit

## TestFlight Enhancements

### Features

![](/Jinha/images/What-s-New-in-iTunes-Connect.md/Untitled2.png)

![](/Jinha/images/What-s-New-in-iTunes-Connect.md/Untitled3.png)

### Groups for Development Cycle

![](/Jinha/images/What-s-New-in-iTunes-Connect.md/Untitled4.png)

Alpha - push frequent builds to "aplha" group

Beta - As build becomes stable, promote it to the larger "beta" group

Release Candidate - Once reached the level of confidence with the stability of build, go onto promote it to "release candidate" group. Even if you've fixed all bugs at this point, you could still use the testers in "release candidate" group to collect further feedback before releasing a final version of th App Store

### Build Expiration

- Build Expire 90 Days After Upload
- You can manually expire builds
- Manual Expiration Does Not Affect Builds Already Installed

### Tester Limets

- 2,000 → 10,000 External testers

## The new App Store

**App icon**

- Asset catalog
iOS11, Xcode9, no longer be managing app icon in iTunes Connect. Instead, include icon in the asset catalog of app binary
- Wide gamut
- Color management
- App thinning

**App subtitle**

- Brief summary
- Up to 30 characters
- Complements app name

**App previews**

- Up to three app prives
- Per language
- Autoplay

**Promotional text**

- Always editable
- Frequently changing information
- Locked description

## 🆕 Promoting in-app purchases

→ In app purchases discoverable on the App Store.

**Product page**

![](/Jinha/images/What-s-New-in-iTunes-Connect.md/Untitled5.png)

Promoted In-App purchases broken up in 2 distinct list. Subscriptions & In-App Purchases.

Can customize the order and visibility of promoted in-app purchases as they appear on product page. Can hide subscriptions the user already has purchased on the device or show the most relevant list.

![](/Jinha/images/What-s-New-in-iTunes-Connect.md/Untitled6.png)

Can search by the name or description of promoted in-app purchases and see some results.

![](/Jinha/images/What-s-New-in-iTunes-Connect.md/Untitled7.png)

Eligible to featured by the App Store editorial team. App could be featured on the "today" apps or games tab.

What developer need to do to implement and make an in-app purchase a promoted?

![](/Jinha/images/What-s-New-in-iTunes-Connect.md/Untitled8.png)

The first two of which are managed in iTunes Connect, but critically you must define some promotional data for your promoted in-app purchase. → image, name, description

So these are the 3 pieces of information that are surfaced and visible on the App Store, and they must be reviewed by the app review team.

Once you set up the promotional data in iTunes Connect, you need to configure your in-app purchases and you need to do things like activate it so we know to count it against your limit of 20 and you need to set a default order and visibility for your promoted in-app purchase as it would appear on the product page.

Need to implement some StoreKit APIs. Single delegate method on the `SKPaymentTransactionObserver` protocol.

> Related: What's New in StoreKit

![](/Jinha/images/What-s-New-in-iTunes-Connect.md/Untitled9.png)

![](/Jinha/images/What-s-New-in-iTunes-Connect.md/Untitled10.png)

→ **This must be reviewed, even though our Pro Subscription in-app purchases is already approved and ready for sale.**

### Availability

- All in-app purchase types
- iOS and universal apps
- iOS 11

## Phased release

Production is always a unique environment, no matter how much testing you've done. And that's why Phased release built.

### Features

- Immediately available on the App Store
- Users with automatic update enabled
- Seven-day delivery period

![](/Jinha/images/What-s-New-in-iTunes-Connect.md/Untitled11.png)

![](/Jinha/images/What-s-New-in-iTunes-Connect.md/Untitled12.png)

## App Review

### Best practices

- Make sure app functions throughout review
- Run on an actual device
- Sign-in information
- Use app Review notes
- Binary compatibility
- Relevant keyword
