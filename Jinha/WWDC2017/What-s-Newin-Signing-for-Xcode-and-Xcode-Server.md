# What's New in Signing for Xcode and Xcode Server

>  📅 2019.12.05 (THU)
>
> WWDC2017 | Session : 403 | Category : Signing


🔗 [What's New in Signing for Xcode and Xcode Server - WWDC 2017 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2017/403/)


[](/Jinha/images/What-s-Newin-Signing-for-Xcode-and-Xcode-Server/Untitled.png)

Making it possible to use automatic signing for distribution, Xcode Build, and Xcode Server.

## ✓ Why do we code sign?

- Protexts user privacy and security
- Authenticatews the app creator
- Authoizes adccess to app services

[](/Jinha/images/What-s-Newin-Signing-for-Xcode-and-Xcode-Server/Untitled1.png)

Every app has code signiture, and when the app is run, the system checks for a couple things.

1. It first checks that, that signiture is valid. That it hasn't been tampered with since it came from the original developer.
2. That app is allowed to use special system services, like App Groups.
3. Also if it's been provisioned for this device. If it's for development, it's been provisioned for a couple of devices. And if it's for the App Store, that code signature says it can run on any device.

[](/Jinha/images/What-s-Newin-Signing-for-Xcode-and-Xcode-Server/Untitled2.png)

Composing the signiture, Xcode uses 3 essential ingredients.

- Signing Certificate
- Provisioning profile
- entitlements

> related : What's New in Xcode App Signing (2016)

### Automatic Signing

[](/Jinha/images/What-s-Newin-Signing-for-Xcode-and-Xcode-Server/Untitled3.png)

→ When enable automatically managed signing, Xcode will automatically 

- create app IDs
- create certificates
- update provisioning profiles
- update entilements

🆕 in Xcode9, all of this great functionality is available in Xcode Server!
(+ manual signing support)

[](/Jinha/images/What-s-Newin-Signing-for-Xcode-and-Xcode-Server/Untitled4.png)

### Xcode Server

- Continuous integration powered by Xcode
- Built into Xcode
- Runs your tests on simulators and devices

[](/Jinha/images/What-s-Newin-Signing-for-Xcode-and-Xcode-Server/Untitled5.png)

In Xcode, you work on a project and upload updates to that project to a remote repository. Then, set up Xcode Server with a bot to pull from that respository. build your app, and run test on the device.

[](/Jinha/images/What-s-Newin-Signing-for-Xcode-and-Xcode-Server/Untitled6.png)

Running on the device needs a code signature

Xcode compose the signature using those essential ingredients, the provisioning profile and signing certificate

How are we going to get those onto Xcode Server?

If you have targets enabled with automatic signing, this can just happen for you.

Xcode Server will automatically create certificates, it will update provisioning profiles, it will register devices. Also that signing can be performed.

And if you manual target for your targets, Xcode Server now has UI for you to send any certificates from your administrative Mac over to the server that will be performing integrations and code signing.

### 🆕 Xcode Server

Development signing

- Automatic or manual signing
- Xcode Server joins your team for devolopment signing
- Supports two-factor authentication

### 🆕xcodebuld

Development signing

- Command line support for automatic signing repairs
- `xcodebuild -allowProvisioningUpdates`
- `xcodebuild -allowProvisioningDeviceRegistration`

## Setting up Xcode Server


### Archive

- Development signed (reccommended)
- Machine code and bitcode
- Debugging a symbols

[](/Jinha/images/What-s-Newin-Signing-for-Xcode-and-Xcode-Server/Untitled7.png)

