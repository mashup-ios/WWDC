# Optimizing Swift Performance

> 📅 2019.12.16 (MON)
>
> WWDC 2015 | Session : 409 | Category : Performance


🔗 [Optimizing Swift Performance - WWDC 2015 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2015/409/)

## Swift 2.0 performance update

How do we make Swift fast?

→ We made Swift fast by implementing compiler optimizations that target all of these high-level features. These compiler optimizations make sure that the overhead of high-level features is minimal.

![](/Jinha/images/Optimizing-Swift-Performance/Untitled.png)

### Bounds Checks Optimizations

**Array Bounds Checks Optimizations**

- Swift ensures that array access happen in bounds
- Swift can lift checks out of loops
- O(n) checks become O(1)

    for i in 0..<n {
    	A[i] ^= 13
    }

Reading and writing outside of the array is a serious bug and can also have security implications, and Swift if protecting you by adding a little bit of code that checks that you don't read or write outside of bounds of array

```Swift
    for i in 0..< n {
    	precondition (i < length)
    		A[i] ^= 13
    }
```

Now the problem is that this check slows your code down. And it blocks other optimizations. (we cannot vectorize this code with this check in place)

```Swift
    precondition (i < length)
    for i in 0..< n {	
    	A[i] ^= 13
    }
```

So we've implemented a compiler optimization for hoisting this check outside of the loop, making the cost of the check negligible, because instead of checking on each iteration of the loop that we are hitting inside the bounds of the array,

The way Xcode compiles files.

### Swift Compilation

![](/Jinha/images/Optimizing-Swift-Performance/Untitled1.png)

- Xcode compiles your files individually. Compile many files in parallel on multiple cores in your machine.
- Also recompile only file that need to be updated
- But optimizer is limited to the scope of one file.

### Whole Module Optimizations

![](/Jinha/images/Optimizing-Swift-Performance/Untitled2.png)

- With Whole Module Optimization, the compiler is able to optimize the entire module at once, which is great because it can analyze everything and make aggressive optimizations.
- Now, naturally, Whole Module Optimization builds take longer. But the generated binaries usually run faster.

![](/Jinha/images/Optimizing-Swift-Performance/Untitled3.png)

In Xcode 7, Whole Module Optimizations is one of the options that you can select.

## < Understanding Swift performance >

## Writing High Performance Swift Code

### Reference Counting

In general, the compiler can eliminate most reference counting overhead without any help. But sometimes you may still find slowdowns in your code due to reference counting overhead.

**How Reference Counting Works**

![](/Jinha/images/Optimizing-Swift-Performance/Untitled4.png)

Every time we made an assignment, we had to perform a reference counting operation to maintain the reference count of the class instance. This is important since we always have to maintain memory safety.

**Classes That Do Not Contain References**

![](/Jinha/images/Optimizing-Swift-Performance/Untitled5.png)

If I store one of these `Point` in an array, because it's class, I den't store it directly in the array. Instead, I store reference to the `Point` in the array.

So when iterate over the array, when initialize the loop variable `p` , I'm actually creating a new reference to the class instance, meaning that iI have to perform a reference count increment.

Structs That Do Not Contain References

![](/Jinha/images/Optimizing-Swift-Performance/Untitled6.png)

We can store each `Point` in the array directly, since Swift arrays can store structs directly. But more importantly, since a struct does not inherently require reference counting, we can immediately eliminate all the reference counting overhead from the loop.

**Structs Containing a Reference**

- A Struct requires reference counting if its properties require reference counting
→ Assigning a struct is equivalent to assigning each one of its properties independently of each other

![](/Jinha/images/Optimizing-Swift-Performance/Untitled7.png)

Of course, `UIColor` being a class, this is actually adding a reference to my struct.

Every time assign this struct, it's equivalent to assigning this UIColor independently of the struct, which means that I have to perform a reference counting modification

**Struct Containing Many References**

![](/Jinha/images/Optimizing-Swift-Performance/Untitled8.png)

Even though all of these properties are value types,  internally, they contain a class which is used to manage the lifetime of their internal data.

So this means that every time I assign one of these structs, every time I pass it off to a function, I actually have to perform five reference counting modifications.

**Use a Wrapper Class**

![](/Jinha/images/Optimizing-Swift-Performance/Untitled9.png)

Instead of standing on its own, it's contained within a wrapper class.

We've changed from using something with value semantics to something with reference semantics. This may cause unexpected data sharing that may lead to weird results or things that you may not expect.

### Generics

![](/Jinha/images/Optimizing-Swift-Performance/Untitled10.png)

Compiler is using indirection to compare both x and y.

Compiler must be correct in all cases and be able to handle any of them.

Additionally, because the compiler can't know if T requires reference counting modifications or not, it must insert additional indirection so that min T function can handle both types T that require reference counting and those types T that do not.

**Generic Specialization**

![](/Jinha/images/Optimizing-Swift-Performance/Untitled11.png)

![](/Jinha/images/Optimizing-Swift-Performance/Untitled12.png))

![](/Jinha/images/Optimizing-Swift-Performance/Untitled13.png))

Complier can not see the definition of the generic min-T function when it's compiling file 1, and so we must call the generic min-T function.

![](/Jinha/images/Optimizing-Swift-Performance/Untitled14.png)

If we have Whole Module Optimization enabled, both file1.Swift and file2.Swift are compiled together. This means that definitions from file 1 and file 2 are both visible when you are compiling file 1 or file 2 together.

### Dynamic Dispatch

![](/Jinha/images/Optimizing-Swift-Performance/Untitled15.png)

The compiler must insert this indirection because it cannot know given the current class hierarchy whether or not the property name or the method noise are meant to be overridden by subclasses.

The compiler can only emit direct calls if it can prove that there are no possible overrides by any subclasses of name or noise.

### Communicate API constraints

**Inheritance**

API with `final` keyword means this declaration will never be overridden by a subclass.

![](/Jinha/images/Optimizing-Swift-Performance/Untitled16.png)

By default, the compiler must use indirection to call the getter for name.

By attaching the `final` keyword to name, compiler can look at name and realize that this will never be overridden by a subclass, and the dynamic dispatch, the indirection, can be eliminated

**Access Control**

![](/Jinha/images/Optimizing-Swift-Performance/Untitled17.png)

By attaching `private` keyword, `noiseimpl` is no longer visible outside of pet.Swift

This means that the compiler can immediately know that there cannot be any overrides of `noiseimpl` in cat or dog.

The compiler can emit a direct call to `noiseimpl` in this case

**Whole Module Optimization**

![](/Jinha/images/Optimizing-Swift-Performance/Untitled18.png)

When we have Whole Module Optimization enabled, the compiler has module-wide visibility. It can see all the files in the module together. 

![](/Jinha/images/Optimizing-Swift-Performance/Untitled19.png)

And so the compiler is able to see, there are no subclasses of dog, so the compiler can call noise directly on instances of class Dog.

### Swift vs. Objective-C

Why is Swift so much faster than Objective-C on these object-oriented benchmarks?

![](/Jinha/images/Optimizing-Swift-Performance/Untitled20.png)

The reason why is that in Objective-C, the compiler cannot eliminate the dynamic dispatch through Objective-C message send.

The compiler must assume that there could be anything on the other side of an Obj-C message send.

But in Swift, the compiler has more information. It's able to see all the certain things on the other side. It's able to eliminate this dynamic dispatch in many cases. Resulting significantly faster code.

### Communicate you API Intent

- Use the `final` keyword and access control
    - Help the compiler understand your class hierarchy
    - Be aware of breaking existing clients
- Enable Whole Module Optimization

## Summary

- Swift is flexible, safe programming language with ARC
- Write your APIs and code with performance in mind
- Profile your application with Instruments

## Using Instruments to analyze the performance of Swift programs
