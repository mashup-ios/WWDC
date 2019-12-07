# [What's New in Core Data](https://developer.apple.com/videos/play/wwdc2016/242)

@ WWDC 16



### Faults in Core Data

core data는 faults를 아주! 많이 사용한다. faults가 뭔데?! - placeholder objects(faults)를 persistent store에 저장해서 메모리 사용량을 줄인다. 아래의 object들이 fault가 될 수 있다.

* Managed objects
* Relationships
* Batch fetching result array



### Why Faullts

* Avoid I/O
* Avoid CPU usage
* Avoid memory consumption



### Handling Deletions

* `managedObjectContext.shouldDeleteInaccessibleFaults`
* Prefetch everything
  * Extra I/O, CPU, memory
* Write lots of code



### Stepping Back

* User viewing UI doesn't care
  * Stability over immediacy
  * Deferred/batched UI updates
* User saving data doesn't care
  * This is why we have merge policies



### Query Generations

* All reads from a context see a single generation of data
* Advance at predictable times
* Never see "could not fulfill a fault"
* Efficiently



### Benefts

* Transactionality at context level
* Isolation of work in peer contexts
* Miniimize speculative/preventative prefetching



### Query Generations - The basiics

* Context chooses behavior
  * Unpinned (current, default)
  * PPiin when data first loaded
  * Piin to specific generatioin
* Nested contexts always unpinned
  * Inherit data from parent
* When explicitly updated
* On save
  * Linear stream of changes
* During `managedObjectContext.mergeChanges()`
* As a result of `managedObjectContext.reset()`



### Requirements

* SQL store
* WAL mode
* Fails gracefuly
  * Non-SQL stores have a single "TOT" generation



### Query Generations - API

* Opaque token to track generation
  * New class `NSQueryGeneratioinToken`
  * `NSQueryGenerationToken.current()`
* Methods on `NSManagedObjectContext` that use it
  * `managedObjectContext.queryGeneratioinToken`
  * `managedObjectContext.setQueryGenerationFrom()`
* Generation will not include stores added after token creation
  * No results from stores not in generation
* Does not prevent you from removing stores from coordinator
  * Error if no stores from generation remain



### Concurrency in Core Data

* Actor model for `NSManagedObjectContext`
  * Use `perform()`, `performAndWait()`
  * `confinmentConcurrencyType` is deprecated
* Actor mode for `NSPersistentStoreCoordinator`
  * Use `perform()`, `performAndWait()`
* Coordinator serializes coontexts' requests



### And Now an Important Message

* `-performBlockAndWait`: now has an autorelease pool
  * Watch your `NSError`
* Affects MRR
* Link time check



### New Stuff

* SQL Store can now handle multiple concurrent requests
  * Multiple readers
  * Single writers
* Connectin pool



### What does this mean?

* More responsive UI
  * Fault/fetch while background work occurs
* Simpler application architecture
  * Less need for multiple stacks
  * Fewer complicated handoffs
  * Bonus: sared row cache means lower memory footprint



### Connection Pool

* On by default
  * SQL stores only
  * All coordinated stores must be SQL stores
* New store option: `NSPersistentStoreConnectionPooMaxSizeKey`
  * Specify max pool size
  * Set to 1 if you want serialized request handling
* We reserve the right to adjust pool size



### Considerations

* Shoud be transparent
* Better performance overal under contention
* Code simplication



### NSPersistentStoreDescription

* Easy customization of common options
  * Read-only
  * Timeout
  * Automatic lightweight migration
  * Automatic mapping model inference
  * Add store asynchronously



### Representing a Core Data Stack

* Encapsulates model configuration
* Name
* Store description
* Loads stores



### Where Did It All Go?

* Model looked up by container name
* Store filename determined by container name
* Store directory provided by the container class



### NSPersistentContainer

* Main queue context property
* Private queue context factory method
* Method for enqueuing background work



### NSManagedObjectContext

* New property `automatiicallyMergesChangesFromParent`
* Works with generation tokens



### Generics

* New protocol `NSFetchRequestResult`
  * `NSManagedObjejct` (and subclasses)
  * `NSManagedObjectID`
  * `NSDictionary`
  * `NSNumber`
* `NSFetchRequest<ResultType>`
* `NSManagedObjectContext.fetch(NSFetchRequest<ResultType>) throws -> [ResultType]`
* `NSFetchedResultsController<ResultType>`



### Automatic Subclass Generation

* Configured per entity
* DrivedData
* Automatically regenerated
* Class or extension



## What's New in SQLite

### Multithreading Assertions

* Database connectionis are not thread-safe
* Multithreading bugs manifest as crashes deep inside SQLite
* `SQLITE_ENABLED_THREAD_ASSERTIONS=1`



### Built-in Logging

* SQLite allows user-defined logging functions via `SQLite_CONFIG_LOG`
* `sqlite3_config` must be called before library initialized
* `SQLITE_ENABE_LOGGING=1`



### File Operatioins and SQLite

* Databases are represented by multiple files
* File operations cannot be atomic across multiple files
* All file operations are unsafe



### Database File Tracking

* SQLite now uses dispatch sources to track illegal file operations
* API calls after illegal operations will return `SQLITE_IOERR_VNODE`
* `SQLITE_ENABLE_FILE_ASSERTIONS=1`



### How to Avoid Database Corruption

* Clear database ownership
* Guarantee excusive file access
* Use Core Data!

