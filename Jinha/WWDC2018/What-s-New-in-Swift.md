# What's New in Swift

>  📅 2019.11.26 (TUE)
>
> WWDC 2018 | Session : 401 | Category : Swift


🔗 [What's New in Swift - WWDC 2018 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2018/401/)

## Community-Hosted Continuous Integration

![](/Jinha/images/What-s-New-in-Swift-1.png)

[Swift Forums](http://forums.swift.org)

[](http://docs.swift.org)

## Apple Engagement in the Broader Swift Community

![](/Jinha/images/What-s-New-in-Swift-2.png)

## What is Swift 4.2?

1. A huge focus on developer productivity.

- Faster builds
- Language features to improve efficiency and remove boilerplate
- SDK improvements for Swift

2. A massive effort on under the hood improvements and changes to the runtime towards this binary compatibility goal which culminates in Swift 5.

- Converging towards binary compatibility

### Binary compatibility

→ you can build your Swift code with Swift 5 compiler and later. And at the binary level it will be able to interoperate with other code built with that complier or any bother compiler layer.

→ This will enable Apple to shift the Swift runtime in the operation system, which means apps can directly use it, meaning that they **no longer need it include in the application bundle.** So this is a code size win, **impacts things like startup time, memory usage.**

## Source Compatibility

![](/Jinha/images/What-s-New-in-Swift-3.png)

Just like in Xcode 9, the compiler also support multiple language compatibility modes.

### Swift 4.2 SDK Changes

- Updates to important framework APIs
- Some API changes will land in later Xcode 10 betas

### Trajectory with Source Compatibility

- Source-impacting SDK changes converging
- Last release to support "Swift 3" compatibility mode

## Faster Swift Debug Builds

![](/Jinha/images/What-s-New-in-Swift-4.png)

How much faster the Swift code built, it actually builds 3 times faster than it did before.

![](/Jinha/images/What-s-New-in-Swift-5.png)

So that's why the project has that more modest 1.6x speed up.

overall build improvement will depend on the nature of your project, how much Swift code it's using, the number of cores on your machine.

### Speedup for Debug Builds

- Exact speed increase varies by  project size and number of cores
- 2x speed up of full builds for many projects
- Compilation pipeline optimized to reduce redundant work across files

![](/Jinha/images/What-s-New-in-Swift-6.png)

Compilation Mode is how your project builds.

- So for release builds the default is whole module compilation that means all the files within your target are always built together. This is to enable maximum opportunities for optimization.
- And for Debug builds the default is incremental. This means not all the files are all built, re-built always all together. So this is a tradeoff in performance for build times.

Optimization level for Debug builds continues to be no optimization by default. This is for faster builds and better Debug information and the release builds are optimized for speed.

![](/Jinha/images/What-s-New-in-Swift-7.png)

The problem with this combination is, it impedes incremental builds. So anytime you touch a file within a target the whole target gets rebuilt.

## Runtime Optimizations

Swift uses automatic memory management and it uses reference counting just like Objective-C for managing object instances.

![](/Jinha/images/What-s-New-in-Swift-8.png)

## Reduced Code Size

![](/Jinha/images/What-s-New-in-Swift-9.png)

This can be useful for applications that care very much about app size limits such as from cellular over the air download limits.

### Performance Tradeoffs of -Osize

- Code size reduced by 10% to 30%
- Runtime performance usually 5% slower

## New Language Features

### Collection of Enum Cases

```Swift
    // SE-0194 Derived Collection of Enum Cases
    
    🚫
    enum Gait {
    	case walk
      case trot
      case canter
      case gallop 
    
    	static var allCases: [Gait] = [.walk, .trot, .canter, .gallop] } 
    }
    
    for gait in Gait.allCases {
    	print(gait)
    }
    
    🆕 CaseIterable protocol
    
    enum Gait: CaseIterable {
    	case walk
    	case trot
    	case canter 
    	case gallop
    	case jog
    }
    
    
    for gait in Gait.allCases {
    	print(gait)
    }
```

`CaseIterable` Compiler will synthesize an all cases property

### Conditional Conformance

**Inconsistent Behavior in Swift 4.0**

```Swift
    extension Sequence where Element: Equatable {  
    	func contains(_ element: Element) -> Bool 
    }
    
    let animals = ["cat", "dog", "weasel"]
    animals.contains("dog") // OK
    
    let coins = [[1, 2], [3, 6], [4, 12]]
    coins.contains([3, 6]) // 🚫Complie error
```

🆕 **What If Element Is Equatable?**

```Swift
    extension Array: Equatable where Element: Equatable {
    	static func ==(lhs: Array<Element>, rhs: Array<Element>) -> Bool {
    		let count = lhs.count
    		if count != rhs.count { return false }
    		for x in 0..<count {
    			if lhs[x] != rhs[x] { return false }
    		}
    		return false
    	}
    }
```

**🆕Conditional Conformance**

```Swift
    extension Optional: Equatable where Wrapped: Equatable { ... } 
    extension Array: Equatable where Element: Equatable { ... } 
    extension Dictionary: Equatable where Value: Equatable { ... }
    
    + Equtable, Hashable, Encodable, Decodable
```

**🆕Conditional Conformance Allows Composition**

```Swift
    let s: Set<[Int?]> = [[1, nil, 2], [3, 4], [5, nil, nil]]
    // Int is Hashable
    // => Int? is Hashable
    // =>? [Int?] is Hashable
```

## Synthesized Equatable and Hashable

```Swift
    // SE-0185 Synthesizing Equatable and Hashable Conformance
    struct Restaurant {
    	let name: String
      let hasTableService: Bool
      let kidFriendly: Bool 
    }
    
    extension Restaurant: Equatable {
    	static func ==(a: Restaurant, b: Restaurant) -> Bool {
    		 return a.name == b.name && 
    					 a.hasTableService == b.hasTableService &&
    					  a.kidFriendly == b.kidFriendly 
    	} 
    }
```
    
```Swift    
    🆕
    struct Restaurant: Equatable {
    	let name: String
      let hasTableService: Bool
      let kidFriendly: Bool 
    }
    
    // Synthesizing Conditional Equatable and Hashable
    enum Either<Left, Right> {
    	case left(Left) 
      case right(Right)
    }
    
    extension Either: Equatable where Left: Equatable, Right: Equatable { }
    extension Either: Equatable where Left: Equatable, Right: Hasable { }
```

## Hashable Enhancements

```Swift
    protocol Hashable {
    	func hash(into hasher: inout Hasher)
    }
    
    extension City: Hasable {
    	func hash(into hasher: inout Hasher) {
    		name.hash(into: &hasher)
    		state.hash(into: &hasher)
```

### New Hashing Algorithm

- Balances hash quality with performance
- Random per-process seed
- Fix any code that relies on:
    - Specific has values
    - `Set` or `Dictionary` iteration order

![](/Jinha/images/What-s-New-in-Swift-10.png)

## Random Number Generation

```Swift
    🆕
    let randomIntFrom0To10 = Int.random(in: 0..<10)
    let randomFloat = Float.random(in: 0..< 1)
    
    let greetings = ["hey", "hi", "hello", "hola"] 
    print(greetings.randomElement()!)
       
    let randomlyOrderedGreetings = greetings.shuffled()
    print(randomlyOrderedGreetings)
```

## Checking Platform Conditions

```Swift
    🚫 #if os(iOS) || os(watchOS) || os(tvOS)
    
    🆕 #if canImport(UIKit)
    	 #if canImport(AppKit)
    
    🚫 #if os(iOS) || os(watchOS) || os(tvOS) && (cpu(i386) || cpu(x86_64))
    
    🆕 #if hasTargetEvironment(simulator)
    	 #warning("We need to test this better")
```

## Implicitly Unwrapped Optionals

### Mental Model

- IUO is an attribute of a declaration, not a type of an expression
- First, try type checking value as ?? - otherwise, force unwrap to get `T`

### Value Type Checks as Type T?

- Optional `Int` can be stored inside `Any`
- Force unwrapping is not performed
```Swift
    func computeDangerously(_ b: Bool) -> Int { return b ? 3 : nil }
    func takesAnAny(_ x: Any) { print(x) }
    
    takesAnAny(computeDangerously(.random))
```

- Must force unwrap the result of the call
```Swift
    func computeDangerously(_ b: Bool) -> Int { return b ? 3 : nil }
    func takesAnAny(_ x: Int) { print(x) }
    
    takesAnAny(computeDangerously(.random)!)
```

## Enforcing Exclusive Access to Memory

![](/Jinha/images/What-s-New-in-Swift-11.png)

![](/Jinha/images/What-s-New-in-Swift-12.png)
