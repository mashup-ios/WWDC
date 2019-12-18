# Building Faster in Xcode

>  📅 2019.12.17 (TUE)
>  WWDC2018 | Session : 408 | Category : Performance


🔗 [Building Faster in Xcode - WWDC 2018 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2018/408/?time=1130)


## Parallelizing Your Build

> Increasing build efficiency

### Xcode's Target and Dependencies

Xcode configure your projects through the use of targets. And targets specify the output or the product that you would like to build

**Target specifies a product to build**

- iOS App
- Framework
- Unit Tests

**Target dependency requires another target**

- Explicit via Target Dependencies
- Implicit via Link Binary with Libraries

![](/Jinha/images/Building-Faster-in-Xcode/Untitled.png)

![](/Jinha/images/Building-Faster-in-Xcode/Untitled1.png)

Each of these targets are building in in order, sequentially. And they each have to wait until the previous target is done building.

→ Waste of potential hardware utilization. Waste of time as a developer.

**Parallelized Build Timeline**

![](/Jinha/images/Building-Faster-in-Xcode/Untitled2.png)

How do we get from the long, serialized build timeline to the better parallelized build time?

![](/Jinha/images/Building-Faster-in-Xcode/Untitled3.png)

Scheme Editor → Edit Scheme → Build Action → Build Options → Parallelize Build

![](/Jinha/images/Building-Faster-in-Xcode/Untitled4.png)

### Examine Tests Dependencies

"Do Everything'

![](/Jinha/images/Building-Faster-in-Xcode/Untitled5.png)

This testing way too many components. It's better to simply break up our tests so that it's testing each individual component.

![](/Jinha/images/Building-Faster-in-Xcode/Untitled6.png)

They can built as soon as their respective components are done,  

---

### Reduce Dependency Exposure

"Nosy Neighbors"

![](/Jinha/images/Building-Faster-in-Xcode/Untitled7.png)

![](/Jinha/images/Building-Faster-in-Xcode/Untitled8.png)

Shaders taraget produces a meta library, which is essentially just a bundle of GPU code that's going to run on our graphics card.

And Utitilies target just produces a normal frame, which is just CPU code

So there's already a little bit of suspect depedency here.

Utitilites target actually has a build phase in it that's genrating some inforation that's used by both targets.

![](/Jinha/images/Building-Faster-in-Xcode/Untitled9.png)

So it's best to break out that into its own target. And we're going to see that this small incremental change actually has a large and significant impact on our overall build timeline.

![](/Jinha/images/Building-Faster-in-Xcode/Untitled10.png)

So new green box that just moved in is our new code target.

So we are able to shrink our utilities target down because we moved that work into Code Gen.

And since Code Gen has no other dependencies, it can move to the very front of build process.

It can also be built in parallel with our Physics target, which is the red box on the bottom.

---

### Examine Unused Dependencies

"Forgotten Ones"

![](/Jinha/images/Building-Faster-in-Xcode/Untitled11.png)

Throughout the evolution or the lifecycle of our products and our code, we tend to move code around and delete things. And we get things like dead code.

We get the same thing that happens with our dependencies. Sometimes we simply forget to clean them up.

![](/Jinha/images/Building-Faster-in-Xcode/Untitled12.png)

So in these cases, it's actually safe to just remove that dependency

![](/Jinha/images/Building-Faster-in-Xcode/Untitled13.png)

And this last change tightens up or build graph even further by allowing the Utilities target to be built right after the CodeGen target instead of having to wait for all of the Physics target to be done.

### 🆕 Parallelized Target Build Process

![](/Jinha/images/Building-Faster-in-Xcode/Untitled14.png)

## Run Script Phases

> Reducing the work on rebuilds

Run script phases allow you to customize your build process to meet your needs.

→ Allows you to customize your build process for your exact needs.

Build Phases

![](/Jinha/images/Building-Faster-in-Xcode/Untitled15.png)

- Script Body

You can either put your entire script contents here or reference another script that's within your project. 

Throughout the entirely of your run script phase, there are a set of build settings that are available to you. → Source group.

This gives you a convenient way to not have to provide absolute paths or try to do some relative path hacks to get your stuff work

![](/Jinha/images/Building-Faster-in-Xcode/Untitled16.png)

- Input Files

As this is one of the key pieces of information that the Xcode Build System will use to determine if your run scripts should actually run or not.

So this should include any file that your run script phase, the script content, is actually going to read or look at during its process.

![](/Jinha/images/Building-Faster-in-Xcode/Untitled17.png)

- File Lists

Some of you may have a lot of inputs into your run script phase. So this task might seem a little bit daunting. Xcode 10 provide you the ability to maintain this list in an external file.

![](/Jinha/images/Building-Faster-in-Xcode/Untitled18.png)

![](/Jinha/images/Building-Faster-in-Xcode/Untitled19.png)

- Output Files

Output files are another key piece of information that's used throughout your build process. Xcode will use this information to determine if your run script phase actually needs to run.

![](/Jinha/images/Building-Faster-in-Xcode/Untitled20.png)

- Output File Lists

### When the Run Script Phase if Run

- No input files declared
Xcode build system will need to run to your run script in every single build
It's important to declare your inputs
- Input files changed
- Output files missing

## Source-Level Improvements

> Remove this build time workaround...
Reducing the work on rebuilds

![](/Jinha/images/Building-Faster-in-Xcode/Untitled21.png)
Previous version of Xcode, for some projects, turning on the Whole Module Compilation mode, even for debug builds, produced a faster overall build than when used in Default Incremental Modes.

And this improve build time because Swift's compiler was able to share work across files in a way that the Incremental Mode was not.

But is also meant you were giving up your incremental builds and wold rebuild the entire target's worth of Swift files every time.

In Xcode 10, the incremental build improved to have some of that same work sharing across files. So you should no longer need to use Whole Module mode to get good build times.

"When a build takes a long time, there is often a key piece of information yo u can add to make it better."

## Dealing with complex expressions

> Increasing build efficiency

### Provide Types in Complex Closoures

```Swift
    func sumNonOptional(i: Int?, j: Int?, k: Int?) -> Int? { 
    	return [i, j, k].reduce(0) {
    		soFar, next in
    
    
    	}
    }
```

If you have a closure with a single expression in its body, then the compiler will use that expression to help determine the type of the closure. 

Sometimes this is really convenient, other times it can lead to code like this.
```Swift
    func sumNonOptional(i: Int?, j: Int?, k: Int?) -> Int? { 
    	return [i, j, k].reduce(0) {
    		(soFar: Int?, next: Int) -> in
    		soFar != nil && next != nil ? soFar! + next! :
    			(soFar != nil ? soFar! : (next != nil ? next! : nil)) 
    		}
    }
```

this expression is getting so large, with so many independent pieces, the Swift Compiler will report that it's not able to compile this in reaonable amount of time.

`error: the compiler is unable to type-check this expression in reasonable time`

### Break Apart Complex Expressions
```Swift
    func sumNonOptional(i: Int?, j: Int?, k: Int?) -> Int? {
    	return [i, j, k].reduced(0) {
    		soFar, next in
    		if let soFar = soFar {
    			if let next = next { return soFar + next }
    			return soFar
    		} else {
    			return next
    		}	
    	}
    }
```

We already know that this callback for reduce is going to be operating just on optional integers, there's no need to put single expression in that closure. And it's perfectly okay to break it up into separate, more readable staements.

### Use `AnyObject` Methods and Properties Sparingly

![](/Jinha/images/Building-Faster-in-Xcode/Untitled22.png)

`AnyObject` is convenient type that describes any class instance.

If you try to call a method or access a property on a value of type `AnyObject`, Swift will allow you to do so, as long as that method is visible somewhere in you project. 

However this does come at a cost. Because compiler doen't know which method you're going to call, it has to go search out any possible implementations throughout your project and the frameworks you import and assume that they might be the none that's going to be used.

**Define a protocol instead**

```Swift
    weak var delegate: MyOperationDelegate?
    func reportSuccess() {
    	delegate?.myOperationDidSucceed(self{
    }
    
    protocol MyOperationDelegate: class {
    	func myOperationDidSucceed(_ operation: MyOperation)
    }
```

## UnderStanding Dependencies in Swift

> Reducing the work on rebuilds

![](/Jinha/images/Building-Faster-in-Xcode/Untitled23.png)

![](/Jinha/images/Building-Faster-in-Xcode/Untitled24.png)

### Swift Dependency Rules

- Complier must be conservative
- Changes in function bodies do not affect the file's interface
- Dependencies within a module are per-file
- Dependencies across targets are for the whole target

## Limiting Your Obejctive-C/Swift

> Reducing the work on rebuilds

![](/Jinha/images/Building-Faster-in-Xcode/Untitled25.png)

If we can shrink the content in these header, then we know that there's fewer chances for things to change, therefore less need to rebuild.

![](/Jinha/images/Building-Faster-in-Xcode/Untitled26.png)

If you don't need to expose these interacting with Interface Builder, markk these `private` and watch as property and method vanish from the generated header.

![](/Jinha/images/Building-Faster-in-Xcode/Untitled27.png)

When you migrate so Swift 4 to keep on a rule fro Swift 3 which eposes internal methods and properies to Objetive-C automatically on any subclass of NSObject/

![](/Jinha/images/Building-Faster-in-Xcode/Untitled28.png)

This will only be inferred for methods and properties that satisfy protocol requirements or those that override methods that com from Objective-C.

![](/Jinha/images/Building-Faster-in-Xcode/Untitled29.png)

![](/Jinha/images/Building-Faster-in-Xcode/Untitled30.png)

This means that if any of these headers change, the Swift cod in your target has to be recompile because it might depend on something that changed.

**→ Use categories to break up your interface**

![](/Jinha/images/Building-Faster-in-Xcode/Untitled31.png)

![](/Jinha/images/Building-Faster-in-Xcode/Untitled32.png)

## Summary

![](/Jinha/images/Building-Faster-in-Xcode/Untitled33.png)

> Related:
Building Your App with Xcode
