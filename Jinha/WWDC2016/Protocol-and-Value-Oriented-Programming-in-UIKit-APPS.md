# Protocol and Value Oriented Programming in UIKit Apps

> 📅 2019.12.15 (SUN)

WWDC2016
Session : Swift
Category :

🔗

[Protocol and Value Oriented Programming in UIKit Apps - WWDC 2016 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2016/419/)

## Local Reasoning

→ When you look at the code right in front of you, you don't have to think about how the rest of your code interacts with that one function.

### Model View Controller

![](/Jinha/images/Protocol-and-Value-Oriented-Programming-in-UIKit-APPS/Untitled.png)

- Model stores data
- View presents that data
- Controller coordinates between the two

> Think Different
Protocol-Oriented Programming in Swift(2015)
Building Better Apps with Value Types in Swift(2015)

# Value types and protocols

[https://developer.apple.com/library/archive/LucidDreams/Introduction/Intro.html](https://developer.apple.com/library/archive/LucidDreams/Introduction/Intro.html)

## Recap - Model

    // Reference Semantics
    
    class Dream {
    	var descriptions: String
    	var creature: Creature
    	var effects: Set<Effect>
    
    	...
    }
    
    var dream1 = Dream(...)
    var dream2 = dream1
    dream2.description = "Unicorn all over"

Classes have reference semantics, meaning that references to the same instance share their storage and that sharing is implicit. 

![](/Jinha/images/Protocol-and-Value-Oriented-Programming-in-UIKit-APPS/Untitled1.png)

![](/Jinha/images/Protocol-and-Value-Oriented-Programming-in-UIKit-APPS/Untitled2.png)

if someone tries to modify dream2's description, If we only care about dream1, we may be surprised the variable's value changed from underneath our control.

And this really hurts local reasoning.

![](/Jinha/images/Protocol-and-Value-Oriented-Programming-in-UIKit-APPS/Untitled3.png)

→ Just test and try the dream type on its own! By making dream type a struct which has value semantics.

```Swift
    // Value Semantics
    
    struct Dream {
    	var description: String
    	var creature: Creature
    	var effects: Set<Effect>
    
    	...
    }
    
    var dream1 = Dream(...)
    var dream2 = dream1
    dream2.description = "Unicorns all over"
```
    

This means each variable has independent storage. So changing the value in one doesn't change the value in the other.

![](/Jinha/images/Protocol-and-Value-Oriented-Programming-in-UIKit-APPS/Untitled4.png)


![](/Jinha/images/Protocol-and-Value-Oriented-Programming-in-UIKit-APPS/Untitled5.png)

This guarantees us that dreams aren't involved in the complicated relationships that we saw earlier. And so this really improves our ability to reason locally because no code can change the value we're using from underneath our control.

## Focus - View and Controller

## View

![](/Jinha/images/Protocol-and-Value-Oriented-Programming-in-UIKit-APPS/Untitled6.png)

DecorationLayoutCell

We couldn't reuse our layout cell outside of tableView.

![](/Jinha/images/Protocol-and-Value-Oriented-Programming-in-UIKit-APPS/Untitled7.png)
```Swift
    DecoratinLayoutCell
    
    // View Layout
    
    struct DecoratingLayout {
    	var content: UIView
    	var descoration: UIView
    
    	mutating func layout(in rect: CGRect) {
    		// Perform layout...
    	}
    }
    

    
    class DreamCell: UITableViewCell {
    	...
    
    	override func layoutSubviews() {
    		var descoratingLayout = DecoratingLayout(content: content, decoration: decoration)
    		decoratingLayout.layout(in: bounds)
    	}
    }
```
    
    ```Swift
    class DreamDetailView: UIView {
    	...
    
    	override func layoutSubviews() {
    		var decoratingLayout = DecoratingLayout(content: content, decoration: decoration) {
    		decoratingLayout.layou(in: bounds)
    	}
    }
    ```

Layout logic is decoupled from tableViewCells, we can use it in any UIView.

Layout can be used in isolation, it's really easy for unit test.
```Swift
    func testLayout() {
    	let child1 = UIView()
    	let child2 = UIView()
    
    	var layout = DecorationLayout(content: child1, decoration: child2)
    	layout.layout(in: CGRect(x: 0, y: 0, width: 120, height: 40))
    
    	XCTAssertEqual(child1.frame, CGRect(x: 0, y: 5, width: 35, height: 30))
    	XCTAssertEqual(child2.frame, CGRect(x: 0, y: 5, width: 70, height: 30))
    }
```

### Easier to understand, easier to test
```Swift
    // Layout
    
    struct DecorationLayout<Child: Layout> {
    	var conetent: Child
    	var decoration: Child
    	mutating func layou(in rect: CGRect) {
    		content.frame = ...
    		decoration.frame = ...
    	}
    }
    
    protocol Layout { 
    	var frame: CGRect { get set }
    }
    
    
    extension UIView: Layout {}
    extension SKNode: Layout {}
```

We can use retroactive modeling to make `UIView` and `SKNode` conform to our new protocol. 

add this functionality to unrelated types to use both of them.

```Swift
    // SpriteKit Layout
    
    struct NodeDecoratingLayout {
    	var content: SKNode
    	var decoration: SKNode
    	mutating func layou(in rect: CGRect) {
    		content.frame = ...
    		decoration.frame = ...
    	}
    }
```

### Generic Types

- More control over types
- Can be optimized more at compile time

> Related:
Understanding Swift Performance(2016)

How can we share code instead?

**inheritance**

![](/Jinha/images/Protocol-and-Value-Oriented-Programming-in-UIKit-APPS/Untitled8.png)

→ But with inheritance, you have both your code, also have to consider what your superclass might be doing and what your subclasses might want to change or override.

You mind has to pull together a large amount of code that's spread across your app. A lot of the time you also inherit from a framework class, like `UIView` or `ViewController` and there's orders of magnitude more code there.

**Composition**

Share code without reducing local reasoning

When composing, you can understand those independent pieces in isolation.

Also enforce encapsulation without worrying about subclasses or superclasses poking holes in your abstractions.

### Composition of ~~Views~~ Values!

- Classes instances are expensive!
When you make another object, you have an extra heap allocation and this is even worse with view.
- Structs are cheap
- Composition is better with value semantics

### Techinques

- Local reasoning with value types
- Generic type for fast, safe polymorhpism
- Composition of values

## Controller

![](/Jinha/images/Protocol-and-Value-Oriented-Programming-in-UIKit-APPS/Untitled9.png)

![](/Jinha/images/Protocol-and-Value-Oriented-Programming-in-UIKit-APPS/Untitled10.png)

```Swift
    // DreamListViewController - Isolating the Model
    
    class DreamListViewController: UITableViewController {
    	var model: Model
    
    	...
    }
    
    struct Model: Equtable {
    	var dreams:[Dreams]
    	var favoriteCreature: Creature
    }
```

![](/Jinha/images/Protocol-and-Value-Oriented-Programming-in-UIKit-APPS/Untitled11.png)

In the first undo step, we're going to remove the dream that the user just added and then we're going to delete the row in that tableView.

And we can continue with the next undo step.

This approach of mutating individual model properties and updating our view independently is really easy to get wrong and that's because you need to match the change in the model the change in the view, precisely.

How we going to update UI?

And so in our ViewController whenever a model changes, we're going to call `modelDidChange` method.

```Swift
    // DreamListViewController: Isoalting the Model
    
    class DreamListViewController: UITableViewController {
    	...
    	func modelDidChange(old: Model, new: Model) {
    		if olf.favoriteCreature != new.favoriteCreature {
    			// Reload table view section for favorite creature.
    			tableView.realoadSections(...)
    		}
    
    		...
    
    		undoManager?.registerUndo(withTargets: self, handler: { target in
    			target.model = old
    		})
    	}
    }
```

### Benefits

- Single code path
    - Better local reasoning
- Values compose well with other values

![](/Jinha/images/Protocol-and-Value-Oriented-Programming-in-UIKit-APPS/Untitled12.png)

Model → Making the Dream type a struct. And this makes it easier for us to locally reason about our code because there's no implicit sharing of our dream variables.

View → Components took advantage of protocol in generics to make sure that the generic components were reusable with views, SpriteKit nodes, and image rendering. And all of this led to better local reasoning since each of these types were small, testable, and isolated value types.

### Techniques and Tools

- Customization through coposition
- Protocols for generic, reusable code
- Taking advantage of value semantics
- Local reasoning
