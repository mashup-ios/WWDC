# Optimizing App Assets

>  📅 2019.11.30 (SAT)
> 
> WWDC2018 | Session : 227 | Category : Interface Builder

🔗 [Optimizing App Assets - WWDC 2018 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2018/227/)

### **Optimizing App Assets**

 Use new features of Asset Catalogs to optimize the
deployment of your application assets

## Compression

![](/Jinha/images/Optimizing-App-Assets/Untitled.png)

### Automatic image  packing

![](/Jinha/images/Optimizing-App-Assets/Untitled1.png)

Traditionally, before the inception of Asset Catalog, one way to deploy asset in your application is just to dump a bunch of image files in the application bundle of your project.

There are 2 sides of downsize that you have to be aware of.

The first comes from the additional disk storage that comes with doing so. Traditional image container formats uses extra space to store metadata, as well as other attributes associated with the underlying image. Now if your application has a huge number of assets, and if they have similar metadata, the same information gets duplicated over and over on disk for no real benefit. Additionally, if most of your assets are fairly small then you do not get the full benefit of most image compression. The other type of drawback comes mainly from the organization overhead that you have to pay for

- Loose images have hidden costs
- Difficult to manage large number of files
- Inconsistent and uncertain image formats

### Lossy compression

about trading minor losses in vision fidelity for the large savings that you gain from the underlying compression

- Quality versus size tradeoff
- Best suited for certain scenarios
→ for image artwork that have fairly short on-screen duration
(ex. splash screen or animations and effects)

**🆕High Efficiency Image File Format**

- Better compression ratio than JPEG
- Supports transparency
- Automatic conversion from other formats

### Lossless compression

default compression type and it's used for the majority of application assets.

Typically image artwork can be categorized into two groups base on the color spectrum profile, and they each benefit differently from any lossless compression.

![](/Jinha/images/Optimizing-App-Assets/Untitled2.png)

1. Commonly referred as simple artwork. 
they have fairly narrow color spectrum and a fairly small number of discrete color values and that is because of the simplistic designs.
2. Complex artwork

**🆕Apple Deep Pixel Image Compression**

- Adaptive to image color spectrum
- Selects optimal compression algorithm
- 15-20% size improvement

### Deployment target and app thinning

App thinning is a process that takes place in the App Store that generates all variants of your project targeting all the device models, as well as versions of your deployment target

App thinning is able to take care of generating all the variants of your project and deploy the most optimal one across all of your user base.

![](/Jinha/images/Optimizing-App-Assets/Untitled3.png)

### App Thinning Export

![](/Jinha/images/Optimizing-App-Assets/Untitled4.png)

## Design and Production

### Color management

- Apps run on broad range of displays
- Color matching maps colors to device
- Asset Catalogs perform color matching at build time
- Artwork ready on device
- Bonus: Color Profile eliminated

### Working space

- Use consistent color settings for all design files
- sRGB / 8 bits for broad applicability
- Display P3 / 15 bits for vibrant designs
- Flexible Wide Color options

### Strechable Images

- Adaptive UI uses stretching elements
- Design tools suport slicing
- Identify strechable portion of image

![](/Jinha/images/Optimizing-App-Assets/Untitled5.png)

**Asset Catalog Slicing**

- Keeps streching metadata close to artwork
- Better positioned for design updates

### Vector Assets

![](/Jinha/images/Optimizing-App-Assets/Untitled6.png)

Preserve Vector Data

when that image is put into an imageView that is larger to natural size of asset it'll go ahead and find the original PDF vector data, which we linked it out and cleaned of any extraneous metadata and profiles as well so it's nice and tight and it's slim as possible.

And we'll go ahead and re-rasterize that at runtime but only if you're going beyond the natural size, otherwise you use that optimized prerendered bitmap.

It means your app might be able to more flexibly respond to dynamic type and automatically your images will look more crisply when you resize your UIImageView

### Design for 2x

- Retina 2x is the most common display density
- Strokes landing between pixels still look fuzzy

![](/Jinha/images/Optimizing-App-Assets/Untitled7.png)

## Cataloging

### Bundles

- Large projects present challenges
- Solve it with multiple bundles
- Effective reuse strategy!

    UIImage(named: UIImage.Name, in: Bundle, compatibleWidth: UITraitCollection)
    Bundle.image(forResource: NSImage.Name) -> NSImage?

### Namespaces

![](/Jinha/images/Optimizing-App-Assets/Untitled8.png)

## Deployment

App Thinning

- You provide all the content variants
- App Thinking picks the right subset per device

Performance Classes

- Hadware capabilities vary
- Don't constrain to least capable device!
- Solve with adaptive resources
