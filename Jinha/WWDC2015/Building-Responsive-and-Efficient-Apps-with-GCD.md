> 📅 2020.01.02 (THU)

WWDC2015 | Session : 718 | Category : Performance

🔗

[Building Responsive and Efficient Apps with GCD - WWDC 2015 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2015/718/)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/08592c04-4b9c-487e-aeae-0bb812aae4d1/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/08592c04-4b9c-487e-aeae-0bb812aae4d1/Untitled.png)

GCD is a way that you can help the system know which parts of your code you should run in order to be able to have a responsive application on a device this size.

## Handling Events

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ad5d17cc-80e3-45a5-b4d9-9dd0332926eb/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ad5d17cc-80e3-45a5-b4d9-9dd0332926eb/Untitled.png)

We will begin executing your code in main and you'll have your initial main thread that every app starts with.

You call UIApplicationMain or NSApplicationMain and that's going to bring up a run loop on the thread and the Framework code.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e12106b7-22dd-44f3-943f-cbac7823fe14/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e12106b7-22dd-44f3-943f-cbac7823fe14/Untitled.png)

And then that thread is going to sit there waiting for events.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/57717be9-217d-4dc6-be68-85e8f8ae2d49/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/57717be9-217d-4dc6-be68-85e8f8ae2d49/Untitled.png)

At some point something is going to happen.

Maybe you'll get a delegate method call out to your UIApplicationDelegate.

At this point, your code begins to run and needs to something. (Let's say it wants to read from a database)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7328a636-857f-47c4-bf9c-4b6783cce275/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7328a636-857f-47c4-bf9c-4b6783cce275/Untitled.png)

You go out and, you'll access that file on disk. That data will come back.

You'll update the user interface.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/29eea3d8-b9be-4227-b545-61ac789033ed/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/29eea3d8-b9be-4227-b545-61ac789033ed/Untitled.png)

And then finally return control back to the Frameworks and continue waiting for events on the thread.

This works great until that read from the database takes a little bit of time.

At that point on OS X you might see a spinning wait cursor. On iOS the app would hang and might event get terminated.

This is poor and unresponsive user experience. And this is where GCD can come in and help make things a little bit easier.

## Handle Events Asynchronously

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0e600f71-be31-4d5d-9b29-75de9b86218b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0e600f71-be31-4d5d-9b29-75de9b86218b/Untitled.png)

You'll get your delegate method of call out. But instead of doing the work immediately you can create a GCD queue, uses dispatch async to move the work on to the queue.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0f45fbbd-d6f6-441a-8095-443a96cd1b8e/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0f45fbbd-d6f6-441a-8095-443a96cd1b8e/Untitled.png)

Your code executes asynchronously with respect to the main thread. When you have the data available, you an dispatch async back to the main thread, update the UI.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/232bc38b-aed1-4772-9463-eaf08995cccf/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/232bc38b-aed1-4772-9463-eaf08995cccf/Untitled.png)

The advantage here is that while your work is happening on that GCD queue, the main thread can continue waiting on that GCD queue, the main thread can continue waiting for events.

It stays responsive. The user continues to get a great experience, and everyone is happy 😁

## Competing Threads

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2d602447-c69f-4ec5-85d4-1a9d6dbc353e/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2d602447-c69f-4ec5-85d4-1a9d6dbc353e/Untitled.png)

One thing you might have not thought about before, is we now have two threads that both want to execute code. Your main thread wants to handle new events and the GCD queue wants to execute the block you dispatched to it.

And maybe we're only on a single core device. In this case, which thread do we execute?

## Quality of Service Classes

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2c2c6a95-2483-4c0b-8ac7-891a7fcb9e01/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2c2c6a95-2483-4c0b-8ac7-891a7fcb9e01/Untitled.png)

- Single Abstract parameter
- Communicate developer intent
- Explicit classification of work
    - Move away from dictating specific configuration values

### Complex resource controls

- CPU scheduling priority
- I/O priority
- Timer coalescing
- CPU throughput vs efficiency
- More...

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c42eb6d7-d0e2-440b-b6f4-95c8cf40d16d/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c42eb6d7-d0e2-440b-b6f4-95c8cf40d16d/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6da42f73-d5fa-40cd-a81d-3bc60b4627db/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6da42f73-d5fa-40cd-a81d-3bc60b4627db/Untitled.png)

### User Interactive

In iOS app, the user has finger on the screen and is dragging.

The main thread needs to be responsive in oreder to deliver the next frame of animation as the user is dragging. **The main user interactive code is specifically the code needed in order to keep that 60 frames per seconde animation running smoothly.**

This isn't loading the content potentially of that scroll view. It is just drawing the new animation.

### User Initiated

We talk about user initiated as being loading the results of an action done by the user. So this might be as I'm scrolling through scroll view loading the data for the next cells or if I'm in a photo or mail application and tap an email or photo, loaidng the full zie photo or e-mail, those kinds of actions are what we talk about as user initiated.

It's helpful not to any about it as user initiated but user blocking. If the user can't continue to make meaninful progress with your application, user initiated is the correct class.

### Utility

Utility is for those ghings that the user may have started,  or may have started automatically,  but are longer running tasks that don't prevent the user from continuing to use your app.

If you were a magazine app downloading a new isssue, is something that the user can continue to use the app while it's happening. They can read old issues or browse around. But you bight have a progress bar and the user is aware of the progress.

### Background

The user is not actively watching. Any kind of maintenance task, cleanup work, database vacuuming would all be background.

Background work is interesting because you want to think about when you're doing it.

---

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4f25aa1a-3f86-4193-ba3d-f5e2c9cb0161/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4f25aa1a-3f86-4193-ba3d-f5e2c9cb0161/Untitled.png)

We don't need a fan to dissipate heat in ost circumstanes, but running this machine at full speed is still going to generate some energy we need to dissipate. But our ability to dissipate heat is at a consatnt rate. We have other techniques to make user we can keep the enclosure of the maching at an appropriate temperature for the user.

If you have an app using and you have work happening at all 4 QoS classes. You are driving the machine hard, using a lot of energy and we need to help control that amount of energy in order to keep the machine at a reaonable temperature.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/49f16174-7a25-42e7-a900-0e2a9c88733d/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/49f16174-7a25-42e7-a900-0e2a9c88733d/Untitled.png)

We can squeeze the amount of work we are going to do at the less important QoS classes.

This allows us to manage he system's user of energy, keep the machine responsvie, and make sure that there's no responsiveness issues visible to the user.

This way it's very important that you have your work correctly classified when running on a maching like this.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1c87835d-de0a-456f-965c-a5c6fff21161/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1c87835d-de0a-456f-965c-a5c6fff21161/Untitled.png)

I've only got 2 CPUs. So if I have to use one to decode the video, what do i do with the next one?

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/468124d9-3c54-4afd-9d0e-0000043b5d55/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/468124d9-3c54-4afd-9d0e-0000043b5d55/Untitled.png)

This is another place for QoS classes can really help you out. Indicating to the operating system the QoS of each thread we can correctly decide  where to marshall those available resources.

## GCD Design Patterns with QoS

### Fundamentals

- QoS can be specified on Blocks and on queues
- dispatch_async() automatically propagats QoS
- Some priority inversions are resolved automatically

### Asynchronous Work

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4d65ca7a-941b-406d-83c7-cd598db402c1/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4d65ca7a-941b-406d-83c7-cd598db402c1/Untitled.png)

On the left side we have the main thread. This is where the UI rendering happens, and where the event handling happens, the appropriate caller service call here is user interactive.

The main thread of the application comes up at this QoS class.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4a02cd72-70fc-4815-b780-5bd35b587788/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4a02cd72-70fc-4815-b780-5bd35b587788/Untitled.png)

On the right side, that's the asynchronous work not happening on the main thread. Obviously, we shouldn't be running uesr interactive. We're not doing the UI rendering here. But, say the user tapped on the document icon and is waiting for his document to open. User initiated is the appropriate QoS class here.

Everything starts with this initial dispatch async that works off the asynchronous work from the main thread. Dispatch async does automatic propagation of QoS from the subitting thread to the queue where you submit the block to execute.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e9ea10d0-86e0-4814-b51b-a25d23c7df8c/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e9ea10d0-86e0-4814-b51b-a25d23c7df8c/Untitled.png)

Now in this case there's a special rule that applies which is that we automatically translate QoS class uer interactive to user initiated. We do this so we don't accidentaly over-propagate the QoS class that should be restricted to the main thread and UI rendering.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3fa08e44-1901-4692-b601-744b53c97495/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3fa08e44-1901-4692-b601-744b53c97495/Untitled.png)

Of course when you go back to the main thread to update the UI with the results this automatic propagation will alo try to take place.  Because the user main thread is QoS user interactive, that will take priority. It will not lower if you go to a thread that has some assigned QoS like the main thread.

Here we will ignore this propagated value and run the UI update block at the bottom at QoS user interactive.

### QoS Propagation

> Inferred Qos

- QoS captured at the time of Block submission
    - User Interactive translated to User Initiated
- Used if destination does not have QoS specified
    - Does not lower QoS

### Block Qos

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2cbd4b22-e9e1-4e7f-aefa-08b25fd66f80/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2cbd4b22-e9e1-4e7f-aefa-08b25fd66f80/Untitled.png)

### Maintenance Task

(다시 보고 정리하기)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e1220f13-8b06-4319-9e35-c67610cb0c3b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e1220f13-8b06-4319-9e35-c67610cb0c3b/Untitled.png)

How do we achieve running QoS background?

One thing we an do is to user the block API we saw earlier and have the inital async to this queue be the QoS backround.

Maybe this is a cases where there is multiple places in the app that generate this type of clean up work and there's multiple ways that you need to operate on the database in this mode.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4254d15e-d441-4a1c-8c5a-a6e94a826cd4/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4254d15e-d441-4a1c-8c5a-a6e94a826cd4/Untitled.png)

It might be appropriate to have a queue specifically dedicated to this task so you can also create queues with assigned QoS.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/687eb873-076b-4108-8ea9-5cd087dd6836/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/687eb873-076b-4108-8ea9-5cd087dd6836/Untitled.png)

### Queue QoS

- Queues that are single purposer
    - Detached Blocks may also be appropriate
- Ignore QoS in asynchronous Blocks
    - Exceptional cases can enforce Block QoS

### Queues as Locks

(다시 보고 정리하기 25:00)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7d2e3e99-f60e-4148-b043-7a2dab260661/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7d2e3e99-f60e-4148-b043-7a2dab260661/Untitled.png)

### Synchronous Priority Inversion

- High QoS thread waiting on lower QoS work
- QoS of waited work is raised for
    - `dispatch_sync()` and `dispatch_block_wait()` of Blocks on serail queues
    - `pthread_mutext_lock()`

## Queues, Threads, and Run Loops

### Run Loop Versus Queue

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1bda0a8d-a354-41c1-a90b-60a8b9bbc527/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1bda0a8d-a354-41c1-a90b-60a8b9bbc527/Untitled.png)

You dispatch async on to some queue.

A block, and we will bring up a thread for that block.

Start executing your code. We'll execute `performSelector withObject afterDelay`,

and that will put a timer source on the current thread's run loop.

These are thareds from our ephemeral thread pool. We may destroy the thread.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/166cdd16-9939-4000-8790-a6755aa6b307/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/166cdd16-9939-4000-8790-a6755aa6b307/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2e2bce5e-4434-46aa-b551-e982295b79a6/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2e2bce5e-4434-46aa-b551-e982295b79a6/Untitled.png)

### Thread Creation and Pooling

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e46a48cc-b644-46d3-a1b2-08aadb739f44/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e46a48cc-b644-46d3-a1b2-08aadb739f44/Untitled.png)

Imaging I'm executing and I do a whole bunch of dispatch asyncs.

I put those on the global queues. The system is going to take a thread from the thread pool and give it to the first block. Send it running on its way.

Take another therad, give it to the second block and send it running on its say.

### Waiting

A thread waits (blocks) when it needs to wait for a resource such as I/O or locks

When a thread waits, GCD may spin up a new thread to ensure one thread per core

### Thread Creation and Waiting

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c0512764-14f8-49c4-8222-c8d0f1e65b0f/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c0512764-14f8-49c4-8222-c8d0f1e65b0f/Untitled.png)

We're executing the first two on two different threads

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6228a4f0-b12a-406d-bf72-70aa6a9447e7/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6228a4f0-b12a-406d-bf72-70aa6a9447e7/Untitled.png)

And the firsy guy goes hey, I need to do some I/O. We go great, we'll issue the I/O to the disk.

But we are going to have you wait for that I/O to come back.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b2b1cb66-db2a-42de-8bca-2aba7d7e9a75/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b2b1cb66-db2a-42de-8bca-2aba7d7e9a75/Untitled.png)

So then we are going to bring up another thread, execute the next block and so on as threads wait we'll bring  up another thread to execute the next block on that queue while there's still work to be done.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fdf20bc9-6c2b-404f-8b92-b4433184e54f/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fdf20bc9-6c2b-404f-8b92-b4433184e54f/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e3865f95-673a-4bef-b282-db74fa9a4cb4/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e3865f95-673a-4bef-b282-db74fa9a4cb4/Untitled.png)

The problem here, if I only have four blocks, that works great.  If I have lots of blocks and they all want to wait, we can get what we call thread explosion.

There are lots of threads all using resources. If they all stop waiting at the same time,  we have lots of contention. So this isn't great for performance. But it's also a liitle dangerous in that there is a limit to the number of thread we can bring up.

When you exhaoust the limit we have a proble of what do we do with new work that comes in?

And this can produce deadlock.

### Threads Explosion Causing Deadlock

(다시 보고 정리하기 34:00)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ae490d59-afa1-4a27-8121-bbdf4ed1c11f/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ae490d59-afa1-4a27-8121-bbdf4ed1c11f/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/867bd89f-ef84-4f09-b5b5-132137144215/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/867bd89f-ef84-4f09-b5b5-132137144215/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/48a41c3b-3904-447b-a286-fdb8cdf3d99d/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/48a41c3b-3904-447b-a286-fdb8cdf3d99d/Untitled.png)

### Avoding Thread Explosion

- Always good advice: use asynchronous APIs, especially for I/O
- User serial queues
- User NSOperationQueues with concureency limits
- Don't generate unlimited work...



## GCD and Crash Reports

### Manager thread

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/060fe2be-38aa-4099-9e67-2d0385e24327/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/060fe2be-38aa-4099-9e67-2d0385e24327/Untitled.png)

The manager thread is something that you are going to see in alomost all GCD using applications. It's there to help process dispatch sources. You notice dispatch manager thread is the root frame. You can generally ignore it.

### Idle GCD thread

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c5ea9664-a1db-4931-8b06-31218f534dba/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c5ea9664-a1db-4931-8b06-31218f534dba/Untitled.png)

These are idle threads in the thread pool. You can see start work queue thread at the bottom of the stack. There's an indication it's a GCD thread. Work queue current return indicates it's sitting there idle.

### Active GCD thread

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b2333c87-32df-4f7a-91f6-f0ec8641616b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b2333c87-32df-4f7a-91f6-f0ec8641616b/Untitled.png)

Start work queue thread but you'll see someting like dispatch client call out and dispatch call block and release, followed by your code. You'll also see the dispatch queue name that you passed  when you created the queue. It's important to give descriptive queue names.

### Idle main thread

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/64551334-6df1-44c6-8ab5-64a8f873752f/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/64551334-6df1-44c6-8ab5-64a8f873752f/Untitled.png)

The main thread when idle you'll see sitting in mock message trap, CF run loop port and CF run loop run  and you can see com.apple.main.thread.

### Main queue

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0c174357-b946-4891-85fa-51f0acae1021/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0c174357-b946-4891-85fa-51f0acae1021/Untitled.png)

If your main queue is active, you might see CF run loop is servicing the main dispatch qeueu if it was active because of the mai nqueue GCD queue. And in this case which have NSBlock operation that we're calling out.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a5ea0b7c-e4a0-4365-831d-897dd11ab40f/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a5ea0b7c-e4a0-4365-831d-897dd11ab40f/Untitled.png)
