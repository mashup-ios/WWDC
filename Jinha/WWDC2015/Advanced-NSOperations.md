# Advanced NSOperations

>  📅 2020.01.05 (SUN)
> WWDC2015 | Session : 226 | Category : Performance

🔗 [Advanced NSOperations - WWDC 2015 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2015/226/)

## Exploring the WWDC App

## Core Concepts

### NSOperationQueue

- High-level `dispatch_queue_t` 
by providing a wrapper around an NSOperationQueue, we can gain some additional functionality.
- Cancellation
NSOperationQueue makes it very easy to cancel operations that have not yet begun executing.
While you can perform cancellation of dispatch blocks, it is somewhat tricky to do so, but NSOperationQueue makes this quite easy.
- `maxConcurrentOperationCount`

### Serial Queues

![](/Jinha/images/Advanced-NSOperations/Untitled.png)

With the max concurrent operation count of 1, **the queue will pull off these operations one by one and execute them in order.** The next operation will not begin executing until the previous one has finished.

![](/Jinha/images/Advanced-NSOperations/Untitled1.png)

By default, the value of this property is a default value,  which means as many as the system allows. So this means that our operation queue can perform multiple operations simultaneously as system resources allow. So in this case, our operation queue might be performing two operation at once.

The ability to change the behavior of an operation queue like this can be very powerful.

### NSOperation

- High-level `dispatch_block_t`
- Long-running task
- Subclassable

### Lifecycle of an NSOperation

![](/Jinha/images/Advanced-NSOperations/Untitled2.png)

1. When you create an NSOperation, it always start off in state  that we call the pending state.
So this is the operation when it's initialized and as it's being put onto its operation queue.
2. Now, at some point,  the operating is going to become ready to execute, and it enters the ready state.
3. After it becomes ready, the operation queue will pull it off of the queue and begin executing it.
This execution can be anywhere from a couple of milliseconds to several minutes to even longer.
4. After execution finishes, the operation enters the finish state, its final state.
5. At any point, it can enter a canceled state.

### Cancellation

- `var cancelled: Bool { get }`
- Only changes state of the property
- Subclasses dictate meaning
Be sure to observe this value changing and react appropriately if there's any reaction you need to do.
- Susceptible to race conditions
- Call the cancel method `operationA.cancel()`

### Readiness

- `var ready: Bool { get }`
- Execute when ready

### Serial Operation  Queues

![](/Jinha/images/Advanced-NSOperations/Untitled3.png)

We are going to load up a bunch of operations, and they are all in the initial blue pending state.

Now, the first operation to enter the ready state is the first operation that will be executed.

![](/Jinha/images/Advanced-NSOperations/Untitled4.png)

Even for example, in this case, if it's the fourth operation that was put onto the queue. So once the operation is ready, it begins executing.

![](/Jinha/images/Advanced-NSOperations/Untitled5.png)

Then, as other operations become ready, they are pulled off the queue and executed. 

![](/Jinha/images/Advanced-NSOperations/Untitled6.png)

![](/Jinha/images/Advanced-NSOperations/Untitled7.png)

And in this case, since we have a serial queue, only executing one at a time, if two operations become ready at the same time, then the first one that has a higher priority will be pulled off first, and then the second one will be executed.

![](/Jinha/images/Advanced-NSOperations/Untitled8.png)

And then, as the other operations become ready, they are also pulled off the queue and executed.

### Dependencies

- "First this, then that"
- Strict ordering
- Implemented via readiness
- Not limited by queues
- Add dependency method `operationB.addDependency(operationA)`

**Operation deadlock**

![](/Jinha/images/Advanced-NSOperations/Untitled9.png)

If I have an operation A and another operation B that is dependent upon the execution of A, this if fine.

However, if I inadvertently make A also dependent on B,  then these two operations will never execute because they will both be waiting on each other to finish, and since they are both waiting, they will both never start.  

![](/Jinha/images/Advanced-NSOperations/Untitled10.png)

## Beyond the Basics

### UI Operations

> Not just the background

- Authentication
- Videos
- Alerts
- "Modal" UI

### Block Operations

> It's blocks all the way down

- NSOperation to execute a block
- Dependencies for blocks
- Leaving feedback

### Generating Dependencies

> When two operations love each other.

- "Never execute this until after executing that"
- Saving a favorite
- Downloading pass
- Anything requiring login

### Extending Readiness

> Adding requirements

- "Only execute this if..."
- Examples
    - Network is reachable
    - Access to user location
    - User is "logged in"

### Composing Operations

> I heard you like operations...

- Downloading and parsing
- Fetching favorites

## Summary

- Operations abstract login
- Dependencies clarify intent
- Describe complex behaviors
- Enables powerful patterns
