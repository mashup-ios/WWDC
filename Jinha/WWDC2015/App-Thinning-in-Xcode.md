# App Thinning in Xcode

> 📅 2019.12.03 (TUE)
>
> WWDC2015 | Session : 404 | Category : Xcode


🔗 [App Thinning in Xcode - WWDC 2015 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2015/404/)


![](/Jinha/images/App-Thinning-in-Xcode/Untitled.png)

![](/Jinha/images/App-Thinning-in-Xcode/Untitled1.png)

What's in an App

- Executable Code
armv7, armv7s, arm64
- Resources
1x, 2x, 3x

![](/Jinha/images/App-Thinning-in-Xcode/Untitled2.png)

![](/Jinha/images/App-Thinning-in-Xcode/Untitled3.png)

You still will be uploading one universal app with all of the differenct variants of art work and other resources as you have today, and it is in the App Store that the slicing happens.

App Store look at all the different device traits of the different devices that your app can support. And it builds premade, separate IPAs that get downloaded.

![](/Jinha/images/App-Thinning-in-Xcode/Untitled4.png)

This is not stored in your app's bundle on the device, also not on the container that gets backed up to iCloud. This is stored in memory managed by the system that can cache the on-demand resources from the different appications on the device.

Divide into those that are shared or always needed that we want to keep in the app itself.

By splitting this up, then you can actually shrink the base footprint of app and still have access to content.

Asset packs that are built by Xcode based on asset tags get moved off. They are stored separately from your IPA in the App Store.

### On Demand Resources

- Asset packs are built by tagging assets in Xcode
- Can contain any non-executable assets
- Hosted by the App Store
- Reclaimed as appropriate
- Device-thinned just like the other content

related : Introducing On Demand Resources

### Advantages

- Support devices with constrained storage
- Shorter download times
- Easier to stay within over-the-air size limits
- Support more types of devices with less compromise
- Add features that couldn't previously fit

## Asset Slicing

### How Does It Work?

- Seamlessly intergrated into build, export, and pubhils workflows
- Post-processing of Asset Catalog and executable files

![](/Jinha/images/App-Thinning-in-Xcode/Untitled5.png)

## Build Workflow

- Xcode Build and Run automatically thins resources for the active run destination.
- Every time you hit build, it's actually only going to consider and produce and automatically create the proper runtime asset catalog for the target device that you are using.
- Speeds up interativce development
- Test impact of cataloging changes on thinned outputs

![](/Jinha/images/App-Thinning-in-Xcode/Untitled6.png)

## Distributing Thinned Applications

### Over-the-Air Installation

![](/Jinha/images/App-Thinning-in-Xcode/Untitled7.png)

When Xcode is generating that exported set of IPAs, it'a also going to generate a manifest list containing URLs for each of the app variants that it produces.

![](/Jinha/images/App-Thinning-in-Xcode/Untitled8.png)

## What You Should Do

- Create tailored version of assets
- Use Asset Catalogs to organize your assets
- Test your thinned app variants using Xcode
- Use Xcode Server to automate builds
- Take advantage of On Demand Resources
