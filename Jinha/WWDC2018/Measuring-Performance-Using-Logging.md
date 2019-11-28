# Measuring Performance Using Logging

>  📅 2019.11.28 (THU)
>
> WWDC 2018 | Session : 405  | Category : Performance

🔗 [Measuring Performance Using Logging - WWDC 2018 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2018/405/)


## 🆕 Introducing Signposts


**Signposts**

- Part of the os_log family
- Performance-focused time markers

**Instruments**

- Aggregate and analyze signpost data
- Visualize activity over time

## Measuring Intervals with Signposts

![](/Jinha/images/Measuring-Performance-Using-Logging/Untitled.png)

Your trying to investigate the amount of time a particular part of the interface takes to refresh. To do that, to load some images and put them on the screen.

Single view of this app might be that you're doing the work to grab an asset. And after you've gotten them all, the interface if refreshed

1. Simple view of this app might be that you're doing the work to grab an asset.
2. the interface is refreshed.

![](/Jinha/images/Measuring-Performance-Using-Logging/Untitled1.png)

What a signpost allows us to do is to mark the beginning and the end of a piece of work and then associate those two points in  time, those two log events with each other. And they do it with an os signpost function call.

![](/Jinha/images/Measuring-Performance-Using-Logging/Untitled2.png)

relate those 2 points(b,e) to each other to give a sense of what the elapsed time is for that interval.

```Swift
    import os.signpost
    
    let refreshLog = OSLog(subsystem: "com.example.your-app", category: "RefreshOperation")"
    
    os_signpost(.begin, log: refreshLog, name: "Refresh Panel")
    for element in panel.elements {
    	os_signpost(.begin, log: refreshLog, name: "Fetch Asset")
    	fetchAsset(for: element)
    	os_signpost(.end, log: refreshLog, name: "Fetch Asset")
    }
    os_signpost(.end, log: refreshLog, name: "Refresh Panel")
```

Signpost name: A string literal that identifiers interval. Match up the begin point that we've annotated or that gets marked up with that os signpost begin called and end point.

![](/Jinha/images/Measuring-Performance-Using-Logging/Untitled3.png)

## Measuring Asynchronous Intervals

### Signpost IDs

Signpost IDs will tell system that these are the same kind of operation but each one is different from each other.

```Swift
    let spid = OSSignpostID(log: refreshLog)
    os_signpost(.begin, log: refreshLog, name: "Fetch Asset", signpostID: spid)
    os_signpost(.end, log: refreshLog, name: "Fetch Asset", signpostID: spid)
```

- Use signpost IDs to tell overlapping operations apart
- While running, use the same IDs for each pair of `.begin` and `.end`

![](/Jinha/images/Measuring-Performance-Using-Logging/Untitled4.png)

![](/Jinha/images/Measuring-Performance-Using-Logging/Untitled5.png)

Begin and end sites can be in separate functions. They associated with separate objects. They may even live in separate files.

## Adding Metadata to Signposts

### Custom Metadata in Signpost Arguments

```Swift
    os_signpost(.begin, log: log, name: "Compute Physics",
    	"%{public}s %.1f,	%.1f, %.2f, %.1f, %.1f", 
    	description, x1, y1, m, x2, y2)
```

- Add context to the `.begin` and `.end`
- Pass arguments with os_log format string literal
- Pass many arguments with different types
- Pass dynamic strings
- The format string is a fixed cost, so feel free to be descriptive!

## Adding Independent Events

### Signpost Events

Signpost that's not tethered to a particular time interval but rather just some fixed moment.

- Marking a single point in time

![](/Jinha/images/Measuring-Performance-Using-Logging/Untitled6.png)

```Swift
    os_signpost(.event, log: log, name: "Fetch Asset",
    		"Feted first chunk, size %u", size)
    
    os_signpost(.event, log: log, name: "Swipe",
    		"For action 0x%x", actionCode)
```

## Conditionally Enabling Signposts

### Signposts Are Lightweight

- Built to minimize observer effect
- Built for fine-grained measurement in a short time span

### Enabling and Disabling Signpost Categories
```Swift
    OSLog.disable
```
- Take advantages of special log handle
- Just change the handle-can leave calling sites alone

    let refreshLog: OSLog
    if ProcessInfo.processInfo.environment.keys.contains("SIGNPOSTS_FOR_REFRESH")
    	refreshLog = OSLog(subsytem: "com.exaple.your-app", category: "RefreshOperations")
    } else {
    	refreshLog = .disabled
    }

### Instrumentation-Specific Code

```Swift
    if refreshLog.signpostsEnabled {
    	let information = copyDescription()
    	os_signpost(..., information)
    }
```

For additional expensive code that is only useful for the signpost

## Instruments

### 🆕 os_signpost

![](/Jinha/images/Measuring-Performance-Using-Logging/Untitled7.png)

allows you to record,  visualized, and analyze all of the signpost activity in your application.

### 🆕 Points of Interest

![](/Jinha/images/Measuring-Performance-Using-Logging/Untitled8.png)

`SignpostLog.pointsOfInterest`

A way for you to pick and choose some of the most important points of interest in you app

### 🆕 Custom instruments

![](/Jinha/images/Measuring-Performance-Using-Logging/Untitled9.png)

Great way to take the signpost data and reshape it in a way that someone else can use and interpret without having details of how your code works
