# Improving Existing Apps with Modern Best Practices

> 📅 2019.11.27 (WED)
>
> WWDC 2016 | Session :213 | Category : Xcode

🔗 [Improving Existing Apps with Modern Best Practices - WWDC 2016 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2016/213/)


## Reduce Technical Debt

### Cycle of Development

![Improving%20Existing%20Apps%20with%20Modern%20Best%20Practices/Untitled.png](Improving%20Existing%20Apps%20with%20Modern%20Best%20Practices/Untitled.png)

🆕Treat Warnings as Errors

![Improving%20Existing%20Apps%20with%20Modern%20Best%20Practices/Untitled%201.png](Improving%20Existing%20Apps%20with%20Modern%20Best%20Practices/Untitled%201.png)

Treat warning as errors forces you and your team to address the issue.

“Each bug report is as unique as a snowflake.”

## Asset Catalogs

![Improving%20Existing%20Apps%20with%20Modern%20Best%20Practices/Untitled%202.png](Improving%20Existing%20Apps%20with%20Modern%20Best%20Practices/Untitled%202.png)

For scaled images, we're asking you to provide 3 different renderings of the artwork from 1X devices, non-retina, 2X and 3X.

![Improving%20Existing%20Apps%20with%20Modern%20Best%20Practices/Untitled%203.png](Improving%20Existing%20Apps%20with%20Modern%20Best%20Practices/Untitled%203.png)

If we don't find, because you didn't include it, the 2X and 3X, we'll take the 1X image and scale it up so that it becomes as image that's used on the higher end devices or the higher density devices.

If you just provided 3X artwork, we'd scale it down at runtime. 

![Improving%20Existing%20Apps%20with%20Modern%20Best%20Practices/Untitled%204.png](Improving%20Existing%20Apps%20with%20Modern%20Best%20Practices/Untitled%204.png)

Scale up 1X image, it's going to look jaggy. It's going to have a visual artifact called aliasing.

Scale down a 3X image, we have to open a 3X image and it's really big. And then we take an extraction of the pixels and create a scaled-down version for it

![Improving%20Existing%20Apps%20with%20Modern%20Best%20Practices/Untitled%205.png](Improving%20Existing%20Apps%20with%20Modern%20Best%20Practices/Untitled%205.png)

Your app is terminated because you didn't provide the artwork.

### Vector Assets

→ Scalable to any size

→ Scale and rasterized at build time

For Vector Assets, Vector Assets are amazing because the file contains a set of instructions on how to draw the image as opposed to having it pre-rasterized.

🆕Image Compression

![Improving%20Existing%20Apps%20with%20Modern%20Best%20Practices/Untitled%206.png](Improving%20Existing%20Apps%20with%20Modern%20Best%20Practices/Untitled%206.png)

![Improving%20Existing%20Apps%20with%20Modern%20Best%20Practices/Untitled%207.png](Improving%20Existing%20Apps%20with%20Modern%20Best%20Practices/Untitled%207.png)

## Dependency Injection

🚫

Each ViewController reaches back to some common object, maybe that you're storing in your app delegate.

![Improving%20Existing%20Apps%20with%20Modern%20Best%20Practices/Untitled%208.png](Improving%20Existing%20Apps%20with%20Modern%20Best%20Practices/Untitled%208.png)

✅

Using Dependency Injection, you take the model object that one ViewController has an you pass it forward to the next ViewController at the time that ViewController is presented.

![Improving%20Existing%20Apps%20with%20Modern%20Best%20Practices/Untitled%209.png](Improving%20Existing%20Apps%20with%20Modern%20Best%20Practices/Untitled%209.png)

### Coming back from Dependency Injection

- Crate and implement a Protocol
- Pass a Closure to the Destination View Controller
- Pass Model Objects by Reference
- Unwind/Exist Segues, with `prepareForSegue:`
