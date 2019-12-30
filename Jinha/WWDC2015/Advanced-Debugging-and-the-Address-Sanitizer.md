# Advanced Debugging and the Address Sanitizer

>  📅 2019.12.30 (MON)
> 
> WWDC2015 | Session : 413 | Category : Debugging

🔗 [Advanced Debugging and the Address Sanitizer - WWDC 2015 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2015/413/)


## View Debugger

### View Debugger

- Focus on troublesome views
- Visualize your constraints

### Advanced Breakpoint Actions

- Catch exceptions at throw, print message
- Print expressions without adding clutter

## Address Sanitizer

### What Is Address Sanitizer?

![](/Jinha/images/Advanced-Debugging-and-the-Address-Sanitizer/Untitled.png)

### Analyze Memory Corruption

- Use after free
- Heap buffer overflow
- Stack buffer overflow
- Global variable overlfow
- Overflows in C++ containers
- Use after return

### Memory Corruption

![](/Jinha/images/Advanced-Debugging-and-the-Address-Sanitizer/Untitled1.png)

![](/Jinha/images/Advanced-Debugging-and-the-Address-Sanitizer/Untitled2.png)

Edit Scheme → Run → Diagnostic  → Enable Address Sanitizer

Unlike other memory management tools, Address Sanitizer requires re-compilation.

![](/Jinha/images/Advanced-Debugging-and-the-Address-Sanitizer/Untitled3.png)

It tells us that the faulty address is one byte after 2240 byte heap region, and it also tells us where that heap region has been allocated. Even though this is not a live thread, but historical snapshot of the process execution when that allocation event occurred, we can look at streams as it was a live thread, and here it takes us right to the point where the memory has been allocated.

![](/Jinha/images/Advanced-Debugging-and-the-Address-Sanitizer/Untitled4.png)

## Under the Hood

> How Address Sanitizer works

Traditionally, Xcode compiles your source code using the clang complier, which produces an executable binary.

![](/Jinha/images/Advanced-Debugging-and-the-Address-Sanitizer/Untitled5.png)

In order to use Address Sanitizer, Xcode passes a special flag to clang which produces an instrumented binary that contains more memory checks.

And at runtime, this binary links with asan runtime dylib, that contains even more checks, and that dylib is required by the instrumentation.

How does these memory checks work?

→ Address Sanitizer checks all allocations in your process. 

### Shadow Mapping

![](/Jinha/images/Advanced-Debugging-and-the-Address-Sanitizer/Untitled6.png)

Address Sanitizer maintains so called shadow memory, that tracks each byte in your real memory, and it has information of whether that byte is address-accessible or not.

Byte on invalid memory are called red zones or as we say, memory there is poisoned. 

![](/Jinha/images/Advanced-Debugging-and-the-Address-Sanitizer/Untitled7.png)

When you compile your program with Address Sanitizer, it instruments every memory access and prefixes it with a check. If the memory is poisoned,  the Address Sanitizer will track the program and generate a diagnostics report. Otherwise, it will allow you to continue.

![](/Jinha/images/Advanced-Debugging-and-the-Address-Sanitizer/Untitled8.png)

Assume p is a pointer, then IsPoisoned function checks the relevant byte in the shadow memory.

In this case, the memory is valid, so the program is allowed to write that memory location. 

![](/Jinha/images/Advanced-Debugging-and-the-Address-Sanitizer/Untitled9.png)

However, if it does not point to valid memory, the condition will be true,  and the program will trap right there, where the invalid memory access was about to happen.

![](/Jinha/images/Advanced-Debugging-and-the-Address-Sanitizer/Untitled10.png)

We  maintain a lookup table where every 8 bytes of your memory are tracked by one byte in the shadow. This is a very large lookup table.

So we don't actually allocate it, instead, we reserve it when the process launches, and use it as needed.

With that, we can look up the address by simply taking the value of the original pointer, dividing it by 8, adding a constant offset, which is the location of the shadow in the memory.

Even if the byte of the computed address is nonzero, we know that the memory is poisoned. 

### Small Performance Overhead

CPU slowdown usually between 2x-5x

Memory overhead 2x-3x
This overhead is much smaller than what you would get from other tools that find similar issues. And compiling the compiler instrumentation with the runtime techniques is the key  that makes Address Sanitizer so effective and scalable.
