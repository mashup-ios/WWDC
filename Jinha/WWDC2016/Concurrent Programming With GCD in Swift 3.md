# Concurrent Programming With GCD in Swift 3

> 📅 2019.112.18 (WED)

WWDC 2016 | Session : 720 | Category : Performance

🔗

[Concurrent Programming With GCD in Swift 3 - WWDC 2016 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2016/720/)

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled.png)

## Concurrency

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%201.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%201.png)

Concurrency allows multiple parts of your application to run at the same time. On our system. you achieve concurrency by creating threads.

A CPU core can execute one of your threads at any given time, but the payoff for introducing concurrency, the penalty for introducing concurrency to your application is it's much more difficult to maintain your thread safety.

The other threads that you've introduced can observe the effects of you breaking your code invariants while you're performing operations on other threads.

→ GCD

multi-threaded code that works on everything from an Apple Watch trough all of our iOS devices.

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%202.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%202.png)

## Dispatch Queues and Run Loops

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%203.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%203.png)

A dispatch queue is a construct that allows you to submit items of work to that queue.
In Swift, this is closures. And dispatch will bring up a thread and service that work for you.

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%204.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%204.png)

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%205.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%205.png)

When dispatch is finished running all of the work on that thread, it can tear that worker thread down for you itself.

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%206.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%206.png)

You can create your own thread and on those threads, you might run Run loops.

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%207.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%207.png)

And then finally, on the first slide we saw you get your main thread, and the main thread is special. It gets both a main run loop and a main queue.

So dispatch queues have two main ways you can submit work for them, the first of which is asynchronous execution.

### Aysnchronous Execution

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%208.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%208.png)

This is where you can queue up multiple items of work to your dispatch queue and then dispatch will bring up a thread to execute that work.

Dispatch will one by one take items off that queue and execute them. And then when finished all items on the queue the system will reclaim the thread that it bought up for you.

### Synchronous Execution

This is where, if we have the same setup as we had before, the dispatch queue with some asynchrounous work.

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%209.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%209.png)

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2010.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2010.png)

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2011.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2011.png)

### Getting Work Off Your Main Thread

    let queue = DispatchQueue(label: "com.example.imagetransform")
    
    queue.async {
    	let smallImage = image.resize(to: rect)
    
    	DispatchQueue.main.async {
    		imageView.image = smallImage
    	}
    	
    }

- Create a Dispatch Queue to whick you submit work
- Dispatch Queues execute work items in FIFO order
- Use `.async` to execute your work on the queue
- Dispatch main queue executes all items on the main thread
- Simple to chain work between queues

### Controlling Concurrency

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2012.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2012.png)

You have to contorl a concurrency in you appliation.

The thread pool that dispath uses will limit the concurrency you achieve in order to use all of the callls in you device.

## Structuring You Application

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2013.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2013.png)

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2014.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2014.png)

### Grouping Work Together

If the user interface spawns off three separate work items here, you can create a dispatch group.

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2015.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2015.png)

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2016.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2016.png)

### Synchronizing Between Subsytems

Use synchroize execution to hepl you serialize state between subsystems.

Serial queues, dispatch queues are seral by natural.

- Can use subsystem serail queues for mutual exclusion
- Use `.sync` to safely access properties form usbsystems
- Be aware of "lock ordering: introduced between subtems

    var count: Int {
    	queue.sync { self.connections.count }
    }

## Dispatch Inside Subsystems

### Choosing a Qulity of Service

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2017.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2017.png)

### Using Quality of sevice Classes

- Use `.async` to subimt work with a specific QoS class
- Dipatch helps reslove priority inversions
- Create single-purpose queuses with a specific QoS class

    queue.async(qos: .background) {
    	print("Maintenance work")
    }
    
    queue.async(qos: .userInitiated) {
    	print("Button tapped")
    }

### DispatchWorkItem

- By default `.async` captures execution context at time of submission
- Create `DispatchWorkItem` from closures to control execution properties
- Use `.assignCurrentContext`  to capture current QoS at time of creation

    let item = DipatchworkItem(flage: .assignCurrentContext) {
    	pritn("Hello WWDC 2016!")
    }
    
    queue.async(execute: item)

### Waiting for Work Items

- Use `.wait` on work items to signal that this item needs to excute
- Dispatch elevates priority of queued work ahead
- Waiting with a `DispatchWorkItem` gives ownership information
- Semaphores and Groups do not admit a concept of ownership

15

### Use GCD for Synchronization

Use `DispatchQueue.sync(execute:)`

- harder to misuse than traiditional locks, more robust
- better instrumentaion (Xcode assertions...)

    // Use Explicit Synchronization
    
    class MyObject {
    	private let internalState: Int
    	private let internalQueue: DispatchQueue
    	var state: Int {
    		get {
    			return internalQueue.sync { internalState }
    		}
    		set (newState) {
    			internalQueue.sync { internalState = newState }
    		}
    	}
    }

### 🆕 Preconditions

-Avoid data corruption

GCD lets you express several precoditions

- Code is running on a given queue
- Code is not running on a given queue
- 

    dispatchPrecondition(.onQueue(expectedQueue)))
    dispatchPrecondition(.not

## Object Lifecycle in a Concurrent World

All these subsystems have a reference in these objects, and getting rid of them actually can be a challenge.

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2018.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2018.png)

1. Set up is about creating the object and giving it the property you need it to have for its purpose.
2. You actually make this object be known to other subsystems. You start using it in a more concurrent world in performance duties.
3. Invalidation is about making sure that all the parts, all your subsystems know that this object is going away.
4. Deallocation

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2019.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2019.png)

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2020.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2020.png)

- Setup

picking the properties that you want for your code, and the animation, and all that.

    class BusyController: SubsystemObserving {
    	init(...) { ... }

- Activation

start receiving these state notifications, state changes notifications, so we will register with that subsystem, and ask for notifications to be received on the main queue.

    class BusyController: SubsystemObserving {
    	init(...) { ... }
    
    	func activate() {
    		DataTransform.sharedInsance.register(observer: self, queue: DispatchQueue.main)
    	}
    }

- Delloation

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2021.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2021.png)

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2022.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2022.png)

BusyController is reference from the main queue and your user interface.

However, when we register it with the ata transform subsystems, it was very likely that reference was taken from the data structure onto this object, which means that deinit doesn't run, which means hat it get unregisterd, it doesn't get collected, and you end up with abandoned mememory.

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2023.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2023.png)

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2024.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2024.png)

BusyController should be managed from the main thread, and you want to make sure that users of your API do that properly. So you will want to have a precondition that this only happens ont the main thread, or the main queue even.

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2025.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2025.png)

You want to track invalidation, for example, as a Boolean in your object, and you remeber when you did. At the same time, let's throw in more preconditions, and make sure, enforce, that before your object is deallocated, it has been properly invalidated. It will help you find bugs.

⬇️

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2026.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2026.png)

In your code that handles the notification for the state transitions, we an observe that the object was invalidated and actually drop the notification on the floor and update the UI in a way that wold be open.

## GCD Object Lifecycle

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2027.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2027.png)

Setup for dispatch objects is all the things you can do when you build the object and all the attricutes you can pass. 

handler → that will run when the resource that your monitoring fires, and that is events pending.

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2028.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2028.png)

It used to be that for dispatch sources, the initial resume had exactly that meaning. We've actually now made supension and activiation be two seperate concepts. Also in resume activate can be called several times, and it only acts once. The contract however is that once you've called activate, you won't mutate the properties of your objects anymore.

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2029.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2029.png)

Cancellation in sources does the thing that you'd expect, which is that you stop getting events for the thing you're monitoring.

## Summary

Organize your application around data flows into independent subsystems

Synchronize state with Dispatch Queues

Use the activate/invalidate pattern

![Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2030.png](Concurrent%20Programming%20With%20GCD%20in%20Swift%203/Untitled%2030.png)
