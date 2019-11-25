# What's New in Swift

>  📅 2019.11.25 (MON)
>
> WWDC2016 | Session : 402 | Category : Swift


🔗 [What's New in Swift - WWDC 2016 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2016/402/)


![](/Jinha/images/What-s-New-in-Swift-1.png)

## Goals for Swift3

Develop an open community

Portability to new platforms

Get the fundamentals right

Optimize for awesomeness

![](/Jinha/images/What-s-New-in-Swift-2.png)

The big thing is not just increased adoption at Apple, it's just we're using Swift in variety of different ways, whether it's in writing apps, we have internal frameworks that are now using Swift, Agents and Daemons, things that just kind of power the underlying experiences in the operation system.

In Sierra, iOS 10, you can see things like the New Music app using a significant amount of Swift.

The Console app in Sierra which is very much tied to the new logging initiatives, also has a lot of use of Swift.

Also with the Agents and Daemons, the new picture in picture feature in Sierra is 100 percent written in Swift

Xcode

The new documentation viewer in Xcode8 is 100 percent written in Swift.

And the new beautiful Swift Playgrounds for iOS, that's 100 percent written in Swift.

Dock

→ Large amount of the macOS Windows, management experience.

![](/Jinha/images/What-s-New-in-Swift-3.png)

 It's been a Swift adopter for two releases.

It started adopting Swift in El Capitan

### What Changed from EI Captian to Sierra?

Most of Mission Control completely rewritten in Swift

Accessibility engine completely rewritten in Swift

![](/Jinha/images/What-s-New-in-Swift-4.png)

### Swift Package Manager

### Foundation on Linux

- Early and actively in delveopment
- Croo-platform packages
- Designed for frictionless development

### Swift-Evolution

**Language Evolution Process**

- Socialize change on mailing list
- Proposal submitted as a pull request
- Pull request accepted to start review
- Formal review on mailing lists
- Core team arbitrates a decision

## Language and Experience

### Making the Core Experience Great

**Improve overall experience of writing Swift code**

- Swift language
- Standard library
- Cocoa in Swift
- Tools

### Zeroing in on Source Compatibility

→ Primary goal of Swift3

- Source compatibility is the most popular "feature" request
- Especially critical for cross-platform
- Source compatibility between Swift 3 and 4 is very strong goal

### Naming Guidelines

Carefully studied what is important in API design

- Strive for clarity-not terseness or verbosity
- Capture essential information
- Omit redundant information/boilerplate

![](/Jinha/images/What-s-New-in-Swift-5.png)

### Emaple API Changes

```Swift
    // Swift.Array
    array.appendContentsOf([2,3,4,])
    array.insert(1, atIndex: 0)
    
    
    // Foundation.NSURL
    if url.fileURL {}
    x = url.URLByAppendingPsthComponent("file.txt")
    
    
    ⬇️⬇️⬇️
    
    array.append(contentsOf: [1, 2, 3])
    array.insert(1, at: 0)
    
    if url.isFileURL {}
    x = url.appendingPathComoponent("file.txt")
```

## Importing Objective-C APIs

![](/Jinha/images/What-s-New-in-Swift-6.png)

![](/Jinha/images/What-s-New-in-Swift-7.png)

![](/Jinha/images/What-s-New-in-Swift-8.png)

![](/Jinha/images/What-s-New-in-Swift-9.png)

### Features Removed in Swift 3

![](/Jinha/images/What-s-New-in-Swift-10.png)

## Type Sytem

### Type System Purpose

**Type System and type checker work together**

- Validate correctness of code
- Infer types and overloads implicit in code

**Goal**

- Simpler, more consistent, and more predictable type system
- Remove "gotchas" and surprising behavior
- Improve type checker performance

### UnsafePointer Nullability

```Swift
    Swift 2
    let ptr: UnsafeMutablePointer<Int> = nil
    
    if ptr != nil {
    	ptr.memory = 42
    }
    
    
    Swift 3
    let ptr: UnsafeMutablePointer<Int>? = nil
    
    if let ptr = ptr {
    	ptr.memory = 42 
    }
```
    

### Implicitly Unwrapped Optional(IUO)

```Swift
    Swift 2
    func f(value: Int!) { 
    	let x = value + 1                  // x: Int - force unwrapped
    	let y = value                      // y: Int!
    	
    	let array = [value, 42]            // [Int], [Int!], [Int?], [Any]...
    
    	user(array)
    }
    
    
    Swift 3
    func f(value: Int!) {
    	let x = value + 1                  // x: Int - force unwrapped
    	let y = value                      // y: Int?
    
    	let array = [value, 42]            // [Int?]
    
    	use(array)
    }
```

## Standard Library

![](/Jinha/images/What-s-New-in-Swift-11.png)!![](/Jinha/images/What-s-New-in-Swift-12.png)

### Swift Tools

![](/Jinha/images/What-s-New-in-Swift-13.png)

### Whole Module Optimization

![](/Jinha/images/What-s-New-in-Swift-14.png)

Complier has a lot more information to write new, innovative optimizations and to make sure code run faster.

So we think this has so much promise from our internal benchmarking, that the year we are turning it by default for all new projects.

![](Untitled-7ccb790b-4e4f-4dc0-ba97-baa45662b773.png)

Really pararell  compliation flow. one file in, one file out.

⬇️⬇️⬇️

![](Untitled-546221d8-9163-4280-81a2-1b327a6e6304.png)

With whole module optimization, we expand the scope of compilation from one file to multiple files. This is really great because the compiler has a lot more information to write new, innovative optimizations and make your code run faster.
