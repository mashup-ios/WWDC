# Understanding Crashes and Crash Logs

>  📅 2019.12.23 (MON)
>
> WWDC2019 | Session : 414 | Category : Debugging

🔗

[Understanding Crashes and Crash Logs - WWDC 2018 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2018/414/)

## Fundamentals

What is a crash?

A crash is a sudden termination of your app when it attempts to do something that is not allowed.

What's now allowed?

### Why Do Crashes Occur?

- Impossible for the CPU to execute code
→ the CPU won't divide by zero.
- Operating system is enforcing a policy
→ the operating system will preserve the user experience by killing your app if it's taking too long to launch or it's using too much memory.
- Programming language is preventing failure
→ Swift array and NSArray will halt your process if you attempt to go outside of your array bounds.
- Developer is preventing failure
→ You may have an API where you assert is a parameter is not nill and that's perfectly all right.

### What Does a Crash Look Like?

**The debugger**

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled.png)

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled1.png)
operating system backtrace in plaintext and save it out to disk in a human readable crash log.

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled2.png)

Xcode take care of symbolication crash logs so that what you'll see are those pretty function names, file names and line numbers.

## Accessing crash logs

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled3.png)

### Crashes Organizer

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled4.png)

- TestFlight and App Store apps
- All platforms, app extensions
- Recent statistics
- Most affected devices
- Page through logs
- Open log in debugger
- Crash point highlighting

### Accessing Crash Logs

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled5.png)

When you have a device connected you can click this View Logs button and will show you all the logs that are saved on that device and these logs are symbolicated using the local symbol information on you Mac.

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled.6png)

When you run the XCTest with Xcode, Xcode Sever or Xcode Build the test results bundle will include any of the crash logs that come from your app that are written out during the execution of that test run

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled7.png)
Use Mac console app to view any crash logs from your Mac or Simulator.

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled8.png)

On the device, Settings, Privacy, Analytics, Analytics Data you can see all of the logs that are saved to disk and your users can share a log directly from this screen.

### Symbolication

- Upload app symbols for sever-side symbolication
→ This will ensure server-side symbolication works.
- Keep your app archive for local symbolication
→ Your archive contains a copy of your debug symbols, dYSM. Xcode uses Spotlight to find these dSYMs and to perform local symbolication when it's necessary automatically.
- Download debug symbols for bitcode apps
→ If you upload an app that contains bitcode you should use the Archives Organizer Download Debug Symbols button to download any dSYMs that come form a store-side bitcode compilation.

## Analyzing crash logs

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled9.png)

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled10.png)

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled11.png)

Binary images that were loaded in the process. This is the application executable and all the other libraries. Xcode uses this for symbolication in order to look up the symbols, the files and line number information for the stack traces.

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled12.png)

Start with crash reason.

In this case the crash is in EXC crash exception with the SIGKILL signal. That means the CPU was trying to execute an instruction that does not exist or is invalid for some reason and that's why the process died.

We can also look ate the thread that crashed.

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled13.png)

### Assertions and Preconditions

Deliberately halt the process when an error has occurred

Examples

- Forced unwrap of an `Optional` that stores nil
- Out-of-bounds `Swift.Array` access
- Swift arithmetic overflow
- Uncaught exception
- Custom assertion in your code

### Terminations by the Operating System

- Watchdog events(timeout)
- Device overhead
- Memory exhaustion
- Invalid code signature
- These crash logs are available in the Devices window

## Memory errors

cases like reference counting of an object being over-released or using an object after it has been free or a buffer overflow

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled14.png)

EXC bad access exception

This typically caused by a memory error.

1. Writing to memory that is read-only 
2. Reading from memory that does not exist at all.

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled15.png)

What does the objc release function do?

It reads the isa filed and then deferences the isa filed so it can get to the class object and perform method lookups. Ordinarily of course this works, this is how it's supposed to work.

What happens if our object has already been freed?

When the free function deletes an object it inserts it tinto a free list of other dead objects.

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled16.png)

It insert into free list of other dead objects. And it writes a free list pointer to the next object in the list where the isa field used to be. With one slight twist, it does write a pointer into that field it writes a rotated pointer into that field.

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled17.png)

It wants to make sure that the value written there is not a valid memory address precisely so that bad use of the object will crash.

So when objc release goes to read the isa field it instead gets a rotated free list pointer.

When it dereference the rotated free list pointer it crashes.

The memory allocator did that for us, it deliberately rotated that pointer to make sure we wold crash if we tried to use it again.

So that is the signature we see in this crash log.

We had the invalid address field looks like a pointer in the malloc region but rotated the same way that malloc rotates its free list pointers.

→ Whatever object we are trying to release at this point in the code has already been deallocated, that's the memory error that occurred.

Which object was being released by objc_release?

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled18.png)

Ordinarily, the function calling objc_release would give us a clue as to what that was. But the problem with the destroyer function is, it is a compiler generated function.

+42 is our clue because the +42 is an offset in the assembly code of the function.

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled19.png)

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled20.png)

This shows us the assembly code of our function.

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled21.png)

We used username property and that was successful. We have not yet gotten to the views property, it might be valid, it might be invalid we don't know. 

What we do know is we tried to release the database property and that object looked like it was already freed object based on the malloc free list pointer signature.

    class LoginViewController: UIViewController {
    	var userName: String
    	var database: DatabaseProxy // TODO: This property is being over-released.
    	var views: [UIView]
    	...
    }

### Crash Log Analysis Summary

- Understand the crash reason
- Examine the crashed thread's stack trac
- Look for more clues in bad address and disassembly

### Common Memory Error Symtoms

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled22.png)

### Crash Analysis Tips

- Look at code other than the line that crashed
- Look at thread stack traces other than the crashed thread
- Look at more than one crash log
- Use Address Sanitizer and Zombies to reproduce memory errors

## Multithreading Issues

### Symptoms of Multithreading Bugs in Crash Logs

- One of the hardest bug types to reproduce and diagnose
- Multithreading bugs often cause memory corruptions
- Multiple threads currently executing similar code
- One bug can appear as different crash points

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled23.png)

Edit Scheme → Dignostics → Thread Sanitizer

→ finding buffer overflows

debugger will break every time that Sanitizer detects a bug

> Related:
Thread Sanitizer and Static Analysis

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled24.png)

![](/Jinha/images/Understanding-Crashes-and-Crash-Logs/Untitled25.png)

### Takeaways

- Test your app on real devices
- Try to reproduce crashes
- Use bug-finding tools on hard-to-reproduce crashes
    - Address Sanitizer for memory corruption bugs
    - Thread Sanitizer for multithreading problems

### Summary

- User Organizer to access crash logs
- Analyze reproducible crahses
- Look for signs of memory corruption and threading issues
- Use bug-finding tools to help reproduce
