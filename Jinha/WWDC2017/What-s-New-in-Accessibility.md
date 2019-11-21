# What's New in Accessibility

>  📅 2019.11.21 (THU)
>
> WWDC 2017 | Session : 215 | Category : Accessibility


🔗 [What's New in Accessibility - WWDC 2017 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2017/215/)

![](/Jinha/images/What-s-New-in-Accessibility-1.png)

![](/Jinha/images/What-s-New-in-Accessibilit-2.png)

- Cognitive encompasses conditions like dyslexia  or autism
- Motor examines the way in which a user physically interacts with the system, whether they need special accommodations for a conditions like Parkinson's or cerebral palsy.
- Vision which encompasses a range of visionability from those with low vision conditions to those who are completely blind.
- Hearing which like vision encompasses a spectrum of hearing ability from those who are hard of hearing to those that are completely deaf.

### 🆕 New assistive features

![](/Jinha/images/What-s-New-in-Accessibilit-3.png)

> Basic text detection to try and figure out if there's text within that image and if it makes sense to speak it then we're going to speak to the user.

![](/Jinha/images/What-s-New-in-Accessibilit-4.png)

> Voice over says : "One face, one smiling, nightclub, blurry, bright

![](/Jinha/images/What-s-New-in-Accessibilit-5.png)

> Building Apps with Dynamic Type

![](/Jinha/images/What-s-New-in-Accessibilit-6.png)

> For users who have dexterity to use a trackpad, but can't necessarily use a physical keyboard. On screen keyboard that has things like system controls and a predictive text bar and fully customizable.

![](/Jinha/images/What-s-New-in-Accessibilit-7.png)

> Allows you to interact with Siri via text input like you would via speech. It's going to open up Siri to a class of users who are unable to use it before those who are nonverbal.

![](/Jinha/images/What-s-New-in-Accessibilit-8.png)

> Smart Invert Colors, which looks at specific kinds of content like graphics or images and doesn't invert them so that you can see the content as it was actually meant to be

### Accessibility API basics

![](/Jinha/images/What-s-New-in-Accessibilit-9.png)

**UIAccessibility Basics**

```Swift
    extension NSObject {
    	open var isAccessibilityElement: Bool
    	open var accessibilityLabel: NSString?
    	open var accessibilityTraits: UIAccessibilityTraits
    	open var accessibilityValue: NSString?
    	open var AccessibilityHint: NSString?
    }
```

### UIAccessibility

![](/Jinha/imagesWhat-s-New-in-Accessibilit-10.png)

```Swift
    open var isAccessibilityElement: Bool
    
    memoryView.isAccessibilityElement = true
    playButton.isAccessibilityElement = false
```

```Swift
    open var accessibilityLabel: NSString?
    
    memoryView.accessibilityLabel = "Memory, Faburary 10 2017"
```

```Swift
    open var accessibilityTraits: UIAccessibilityTraits
    
    memrotyView.accessibilityTraits |= UIAccessibilityTraitButton
```

![](/Jinha/images/What-s-New-in-Accessibilit-11.png)
```Swift
    open var accessibilityValue: NSString?
    
    videoScrubber.accessibilityValue = "\(elapsedTime) seconds"
```

```Swift
    open var accessibilityHint: NSString?
    
    videoScrubber.accessibilityHint = "Swipe up or down with one
                                      finger to adjust the value"
```    	


![](/Jinha/images/What-s-New-in-Accessibilit-12.png)

![](/Jinha/images/What-s-New-in-Accessibilit-13.png)

![](/Jinha/images/What-s-New-in-Accessibilit-14.png)

![](/Jinha/images/What-s-New-in-Accessibilit-15.png)
