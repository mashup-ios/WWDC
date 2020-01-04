# Swift API Design Guidelines

> 📅 2019.01.04 (SAT)
> 
> WWDC2016 | Session : 403 |  Category : Swift

🔗 [Swift API Design Guidelines - WWDC 2016 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2016/403/)


### Language Have Character

Every language has its own feel

Feel of everyday APIs
```Swift
    DispatchQueue.main.sync {
    	self.listDocumentsViewController?.present(signedOutController, animated: true)
    }
```

Swift rendering an opinion about certain things here.

- It uses trailing closures. So the control flow works with your libraries and your APIs nicely.
- It has optional. So you have to consider the possibility of nil everywhere.

## Swift API Design Guidelines

### Principles

- Clarity at the point of use
- Clarity is more important than brevity
- Concise code is a consequence of using contextual cues

### Clarity at the Point of Use

- Design APIs to make uses clear and concise
- Use of APIs always have surrounding context
- Don't optimize for bad code
```Swift
    ❌❌❌
    if let c = a.index(of: b) {
    	a.remove(at: c)
    }
    
    ✅✅✅
    if let completedPosition = tasks.index(of: completed) {
    	tasks.remove(at: completedPosition)
    }
  ```  

### Strive for Clear Usage

    friends.removeItem(ted)
    friends.removeObject(ted)
    friends.removeElement(ted)
    organicCompounds.removeElement(caffeine)
    
    // Omit needless words.
    friend.remove(ted)

### Omit Needless Words

    friend.remove(ted)

### Clarity Is More Important Than Brevity

- Brevity itself is not a worthwhile goal
- Concise code is consequence of using contextual cues
```Swift
    let end: [Ingredient].Index = list.index(list.startIndex, offsetBy: 3, limitedBy: list.endIndex)!
    let shortList: ArraySlice<Ingredient> = list[0..<end]
```

That verbosity is all of these explicit type annotations. These aren't adding to readability. You can get a sense of what the types are just by reading the APIs. And indeed, in Swift you probably wouldn't write the code this way.
```Swift
    let end = list.index(list.startIndex, offsetBy: 3, limitedBy: list.endIndex)!
    let shortList = list[0..<end]
```
You would probably align that type information and let the static type inference do it for you leading to more concise code that you can still read through just as well.

### Which Words Are Needed?

- Write out a use case
```Swift
    mainView.addChild(sideBar, atPoint: origin)
    
    ⬇️⬇️⬇️
    
    mainView.addChild(sideBar, at: origin)
```
Does each word contribute to understanding?

- Clarify parameter role
- Don't restate type information

### Make Uses of Your APIs Read Grammatically
```Swift
    friends.remove(ted)
    ❌ friends.remove(positionOfFormerFriend)
    ✅ friends.remove(at: positionOfFormerFriend)
```

Notice how we've clarified the behavior of the API by putting in this first argument label to describe the relationship of the argument to the method.

### Compound Names

Different APIs can be distinguished by argument label alone
```Swift
    friends.remove(ted)                // remove(_:)
    friends.remove(at: positionOfTed)  // remove(at:)
```
Two APIs should share a compound name if they have the same semantics
```Swift
    text.append(aCharcter)            // append(_:)
    text.append(aString)
```
### First Argument Should Read Grammar
```Swift
    // If the first argument is part of a prepositional phrase, give it a label
    truck.removeBoxes(withLabel: "WWDC 2016")
    
    // If the first argumetn is not part of a grammatical phrase, give it a label
    ❌ viewController.dismiss(true)
    ✅ viewController.dismiss(animated: true)
    
    // Otherwise, don't use a first argument label
    frineds.insert(michael, at: friends.startIndex)
```
### Name Methods Based on Their Side Effects

Use a verb to describe the side effect
```Swift
    friends.reverse()
    viewController.present(animated: true)
    oragnicCompounds.append(caffeine)
```
Use a noun to describe the result
```Swift
    button.backgroundTitle(for: .disabled)
    friends.suffix(3)
```
### Mutating/Non-Mutating Pairs

> The "ed/ing" rule

"ed" rule
```Swift
    x.reverse()            // mutating
    let y = x.reversed()   // non-mutating
```
"ing"rule
```Swift
    documentDirectory.appendPathComponent(".list")                         // mutating
    let documentFile = documentDirectory.appendingPathCompnent(".list")    // non-mutating
```

[Swift.org](https://swift.org/documentation/api-design-guidelines/)

## The Grand Renaming

### One API, Two Names

### 🆕 Use #selector for Objective-C Selectors
```Swift
    ❌ control.addTarget(self, action: Selector("handleDragWithSender:for"),
    									for: [.touchDragInside, .touchDragOutside])
    ✅ control.addTarget(self, action: #selector(handleDrag(sender:for:)),
    									for: [.touchDragInside, .touchDragOutside])
```

### 🆕 Key Path via #keyPath
```Swift
    ❌ album.addObserver(self, forKeyPath: "artist.name" options: .new, context: &artistNameContext)
    ✅ album.addObserver(self, forKeyPath: #keyPath(Album.artist.name), options: .new, context: &artistNameContext)
```

## Mapping Objective-C API into Swift

### 🆕 Automatic Translation of Objective-C APIs

Identify first argument labels
```Swift
    func saveToURL(_ url: NSURL)
    func revertToContentsOfURL(_ url: NSURL)
    
    // ⬇️⬇️⬇️ Identify first argument labels
    
    func save(toURL url: NSURL)
    func revert(toContentsOfURL url: NSURL)
    
    // ⬇️⬇️⬇️ Remove words that restate type information
    
    func save(to url: NSURL)
    func revert(toContentsOf url: NSURL)
    
    // ⬇️⬇️⬇️ Introduced default arguments
    func save(to url: NSURL, completionHandler: ((Bool) -> Void)? = nil)
    
    
    // ⬇️⬇️⬇️ Use bridged value types
    
    func save(to url: URL)
    func revert(toContentsOf url: URL)
```

### Types

Names don't go far enough
```Swift
    // Objective-C
    typedef NSString *NSCalendarIdentifier;
    NSCalenarIdeintifier NSCalendarIdentifierGregorian;
    
    // Generated Swfit Interface
    typealias NSCalendarIdentifier = NSString
    let NSCalendarIdentifierGregorian: NSCalendarIdentifier
```

### Better Types = Clearer Use Site
```Swift
    let cal = NSCalendar(calendarIdentifier: NSCalendarIdeintifierGregorian)
    let cal = NSCalendar(identifier: .gregorian)
    let cal = Calendar(identifier: .gregorian)
```

### 🆕 Import as Member

Properties
```Swift
    // Generated Swift Interface
    extension CGColor { static let white: CFString }
    
    
    // Swift use
    let color = CGColor.white
```

Initializers
```Swift
    // Generated Swift Interface
    extension CGAffineTransform { init(translationX: CGFloat, y: CGFloat) }
    
    
    // Swift use
    let translate = CGAffineTransform(translationX: 1.0, y: 0.5)
```

Method
```Swift
    // Generated Swift Interface
    extension CGContext { func fillPath() }
    
    
    // Swift use
    context.fillPath()
```

Computed Properties
```Swift
    // Generated Swift Interface
    extension Artist { var name: CFString { get set} }
```