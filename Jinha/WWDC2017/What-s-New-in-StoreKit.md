# What's New in StoreKit

>  📅 2019.12.10 (TUE)
> 
> WWDC2017 | Session : 303 | Category : App Store

🔗 [What's New in StoreKit - WWDC 2017 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2017/303/)


## 🆕 What's New in StoreKit

- Promoting in-pp purchases
- Sever-to-server subscription notifications
- Detailed subscription status information
- Responding to reviews
- Asking for ratings and reviews

## Review of In-App Purchases

### In-App Purchase Overview

- Digital content or service bought in-app
- Not for physical goods

### Types of In-APp Purchases

- Consumable products
→ that can be consumed and used up by a user.
ex. gold coins in a game that user buys and maybe spends.
It dosen't persist around subsequent restores or new devices.
- Non-consumable products
→ It does persist. 
ex. unlocking a pro feature in your application, downloading a level pack for a game
- Non-renewing subscriptions
→ It doesn't automatically charge user the end of the biling period. consumable product with an expiry date on it.
- Auto-renewing subscriptions

> Related: Advanced StoreKit

### In-App Purchases Process

![](/Jinha/images/What-s-New-in-StoreKit/Untitled.png)

### Load In-App Identifiers

**Options for storing the list of product identifiers**

After setting up product identifiers in iTunes Connect

Bakes into your app 

`let identifiers = ["com.myComany.myApp.product1,"com.myComany.myApp.product1]`

Or fetch from your sever

`let identifiers = remoteIdentifiers()`

### Fetch Product Info

```Swift
    let request = SKProductsRequest(productIdentifiers: identifierSet)
    request.delegate = self
    request.start()
    
    func productsRequest(_ request: SKProductsRequest, didReceive response: SKProductsResponse) {
    	for product in reponse.products {
    		// Localized title and description
    		product.localizedTitle
    		product.localizedDescription
    		// Price and Locale
    		product.price
    		product.priceLocale
    		// Content size and version (hosted)
    		product.downloadContentLengths
    		product.downloadContentVersion
    	}
    }
```

You shouldn't cache the SKProduct that comes back in this point. It's really important that you get up to date product information, by performing these requests regularly. Because things like currency can fluctuate.

### In-App Purchase UI

- Up to the application
- Can have a large effect on sales
- [https://developer.apple.com/in-app-purchase/](https://developer.apple.com/in-app-purchase/)

**Formatting the product price**
```Swift
    let formatter = NumberFormatter()
    formatter.numberStyle = .currency
    formatter.locale = product.priceLocale // 🚫device locale
    let formattedString = formatter.string(from: product.price)
```
### Request Payment
```Swift
    let payment = SKPayment(product: product)
    SKPaymentQueue.default().add(payment)
```

![](/Jinha/images/What-s-New-in-StoreKit/Untitled1.png)

### Detecting Irregular Activity

Suspicious activity during payment process

![](/Jinha/images/What-s-New-in-StoreKit/Untitled2.png)

**Provide an account identifier**

- For applications with their own account management
- Provide an opaque identifier for your user's account
    - Don't send us the user's Apple ID
    - Don't provide the actual account name
    - Don't provide the password
    - We suggest using a hash of the account name
    - 

```Swift
        let payment = SKPayment(product: product)
        payment.applicationUsername = hash(yourCustomerAccountName)
        SKPaymentQueue.defaults().add(payment)
```

### Process Transaction

![](/Jinha/images/What-s-New-in-StoreKit/Untitled3.png)

![](/Jinha/images/What-s-New-in-StoreKit/Untitled4.png)

### Unlock Content

- Unlock functionality in your app
- Download additional content

**Downloading Content**

- Apple-hosted content
    - On-demand resources
    - Hosted in-app purchase content
- Self-hosted content
    - Use background downloads with NSURLSession

## Finish the Transaction

- Finish all transactions once content is unlocked
    - If downloading hosted content, wail until after the download completes
- Includes all auto-renewable subscription transactions
- Otherwise, the payment will stay in the queue
- Subscription billing retry depends of up-to-date information about transaction
- `SKPaymentQueue.default().finishTransaction(transaction)`

## App Review

- You must have a **Restore button**
- Restore and Purchases must be separate buttons.
- Not just as  a "backup" tool
    - User with multiple devices

Restore Completed Transactions

- Only restores transactions for
    - Non-consumables
    - Auto-renewable subscriptions

- For consumables and-renewing subscriptions
    - You must persist the state!

    SKPaymentQueue.default().restoreCompltedTransactions()
    
    Observe the queue
    //Additional callbacks in SKPaymentTransactionObserver
    func paymentQueueRestoreCompletedTransactionsFinished(_ queue: SKPaymentQueue) {}
    func paymentQueue(_ queue: SKPaymentQueue, 
    	restoreCompletedTransactionsailedWithError error: NSError) {}
    
    Inspect the receipt and unlock content and features accordingly

## Promoting In-App Purchases

In-app purchases discoverable in the App Sotre.

### Discoverable

- App Page
- Editorial features
- Search results

Start purchase on the App Store

1. choose up the 20 purcahses to promote per app, and set them up in iTunes Connect with an accompanying image.

### Implemenentation Overview

Required

- Set up in iTunes Connect
- Handle info from App Store
Implement single delegate method in your app to handle to purchase info sent to you from the App Store. StoreKit will handle the rest of the transaction for you.

Optional

- Order and visibility

> Related: What’s New in iTunes Connect

![](/Jinha/images/What-s-New-in-StoreKit/Untitled5.png)

![](/Jinha/images/What-s-New-in-StoreKit/Untitled6.png)

StoreKit will automatically open your app and send information about the transaction to you via delegate method on the `SKPaymentTransactionObserver` protocol.

If your app isn't already installed, the App Store will download or prompt the user to buy it.

In this case, it can't be opened automatically, so instead, the user will receive a notification.

And when they tap on that, then your app will open and be sent the transaction info.
```Swift
    // Continuing a Transaction from the App Store
    // SKPayment TransationcObserver
    
    func paymentQueue(_ queue: SKPaymentQueue, shouldAddSotrePayment payment: SKPayment, 
    		forProduct product: SKProduct) -> Bool {
    	return true 
    	
    	// Hold on the payment
    	return false
    }
    
    SKPaymentQueue.default().add(savedPayment)
```

`return true` → the user will be shown that nice new in-app purchase payment sheet, so it can complete transaction

`return false` → If user is in the middle of onboarding, or if they're creating an accoung or if they've already unlocked the item they're trying to buy? You can holde onto the payment and return false

![](/Jinha/images/What-s-New-in-StoreKit/Untitled7.png)

### Order and Visibility

- Defaults in iTunes Connect
- Override on device
- Not synced

### Visibility

![](/Jinha/images/What-s-New-in-StoreKit/Untitled8.png)

Use the local visibility ovveride to hide the app purchases. And the, they'll only see items that are relevant to them on your app page


```Swift
    // Reading Visibility Override of a Promoted In-App Puschase
    // Fetch Product Info for Hidden Beaches pack
    
    let storePromotionController = SKProductStorePromotionController.default()
    storePromotionController.updaet(storePromotionVisibility: .hide, forProduect: proSubscription,
    		completionHandler: { (error: Error?) in
    		// Complete
    	})
```

### Order

```Swift
    // Updating Order Override of Promoted In-App Purchases
    // Fetch Product Info for Pro Subscription, Fishing Hot Spots, and Hidden Beaches
    
    let storePromotionController = SKProductStorePromotionController.default()
    let newProductsOrder = [hiddenBeaches proSubscription, fishignHotSpots]
    storePromotionController.updateStorePromotionOreder(newProductsOrder,
    		completionHandler: { (error: Error?) in
    		// Complete
    	})

    // Reading Order Override of Promoted In-App Purchases
    
    let storePromotionController = SKProductStorePromotionController.default()
    storePromotionController.fetchStorePromotionOreder(completionHandler: {
    	(products: [SKPrduct], error: Error?) in
    		// products == [hiddenBeaches proSubscription, fishignHotSpots]
    	})
```

## Promoting In-App Purchases

- Discoverable in App Store
- Set up in iTunes Connect
- Start purchase in App Store
- Handle in app via SKPaymentTransactionObserver
- Optional-order an visibility

## Ratings, Reviews, and Responses

### 🆕  What's new

- Reset your rating
- Respond to reviews
- Ask for ratings and reviews via SKStoreReviewController
- Deep link to write review in the App Store
- Helpfulness and Report a Concern on iOS
- [https://developer.apple.com/app-store/ratings-and-reviews/](https://developer.apple.com/app-store/ratings-and-reviews/)

![](/Jinha/images/What-s-New-in-StoreKit/Untitled9.png)

```Swift
    if shouldPropmtUser {
    	SKStoreReviewController.requestReview()
    }
    
    func shouldPromptUser() -> Bool {
    	// Local business rules
    }
```

![](/Jinha/images/What-s-New-in-StoreKit/Untitled10.png)
