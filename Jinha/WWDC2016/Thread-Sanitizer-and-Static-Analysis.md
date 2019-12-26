# Thread Sanitizer and Static Analysis

>  📅 2019.12.26 (THU)
>
> WWDC2016 | Session : 412 | Category : Debugging

🔗 [Thread Sanitizer and Static Analysis - WWDC 2016 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2016/412)

![](/Jinha/images/Thread-Sanitizer-and-Static-Analysis/Untitled.png)

## Runtime Sanitizers

### Sanitizers

- Find bugs at run time
- Similar to Valgrind
- Low overhead
- Work with Swift 3 and C/C++/Objective-C
- Integrated into Xcode IDE

### Address Sanitizer (ASan)

- Introduced last year
- Finds memory corruption issues
- Effective at finding critical bugs
- Now has full support for Swift

### 🆕 Thread Sanitizer (TSan)

- User of uninitialized mutexes
- Thread leaks (missing `pthread_join` )
- Unsafe calls in signal handlers(ex: `malloc` )
- Unlock from wrong thread
- Data races

![](/Jinha/images/Thread-Sanitizer-and-Static-Analysis/Untitled1.png)

Thread Sanitizer will tell us two race accesses. Read and Write.

Neither of those are a main thread, and the stack traces are the same which means that they are probably executing the same core from multiple threads without using synchronization.

![](/Jinha/images/Thread-Sanitizer-and-Static-Analysis/Untitled2.png)

→ proper fix here is to dispatch both toe counter increment and the UI update on to the main queue with GCD.

```Swift
    DispatchQueue.main.aync {
    	activityCount = activityCount + 1
    	self.updateNetworkActivityUI()
    }
```

This will both take care of the logical problem in our application and also take care of the race because wall the threads will access that count variable from the same thread.

### Thread Sanitizer (TSan) in Xcode

1. Edit Scheme - Diagnostics tab
2. "Enable Thread Sanitizer" checkbox
3. Build and Run
4. View all of the generated runtime issue
5. Can choose to break on every issue

![](/Jinha/images/Thread-Sanitizer-and-Static-Analysis/Untitled3.png)

### TSan Build Flow

In order to use Thread Sanitizer, Xcode passes a special flag to both Clang and Swift compilers that instruct them to produce an instrumented binary.

This binary links to a TSan runtime library that is used by the instrumentation to both monitor the execution of the program and detect those threading issues.

## Fixing Data Races

### Data Race

- Multiple threads access the same memory without synchronization
- At least one access is a write
- May end up with any value or even memory corruption!

![](/Jinha/images/Thread-Sanitizer-and-Static-Analysis/Untitled4.png)

![](/Jinha/images/Thread-Sanitizer-and-Static-Analysis/Untitled5.png)

### Choosing the Right Synchronization

1. Use GCD
Dispatch racy access to the same serial queue
2. Use pthread API, NSLock
`pthread_mutex_lock()` to synchronize accesses
3. New `os_unfair_lock` (use instead `OSSpinLock`)
4. Atomic operations

### There in No Such Thing as a "Benign" Race

- On some architectures (ex. x86) reads and writes are atomic
- But even a "begin" race is undefined behavior in C
- May cause issues with new compilers or architectures

## Finding Bugs with Static Analysis

### Find Bugs Without Running Code

- Does not require running code (unlike sanitizer)
- Great at catching hard to reproduce edge-case bugs
- Supported only for C, C++, and Objective-C

![](/Jinha/images/Thread-Sanitizer-and-Static-Analysis/Untitled6.png)

Check for missing localizability

Check for improper instance cleanup

Nullability

![](/Jinha/images/Thread-Sanitizer-and-Static-Analysis/Untitled7.png)

### Clang Static Analyzer in Xcode

1. Product > Analyze or Product > Analyze "SingleFile.m"
2. View in Issue Navigator
3. Explore Issue Path

## Nullability Viloations

### Why Annotate Nullability?

- New programming model communicate expectation to callers
- Violation can cause crashes or other unexpected behavior
- Swift enforces model in type system with optionals

### Finding Nullability Viloations

- Particularly useful in mixed Swift/Objective-C projects
- Logical problem in code
- Incorrect annotations

## Wrapping Up

### These Tools find Real Bugs!

- Address Sanitizer and Thread Sanitizer
- Clang Static Analyzer
- Use on your code!
