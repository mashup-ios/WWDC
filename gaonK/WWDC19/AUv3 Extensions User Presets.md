# [AUv3 Extensions User Presets](https://developer.apple.com/videos/play/wwdc2019/509/)

@ WWDC 19



### Presets

* Provide a fine-tuned set of parameteer values
* Capture a snapshot of the state of the Audiio Unit's parameters
* Loading a preset restores the Audio Unit to the same state



### User Presets for Audio Unit Extensions

* We already support factory presets

  ```swift
  open var factoryPresets: [AudioUnitPreset]? { get }
  ```

* Provided by the Audio Unit Developer

* Immutable, built into the Audio Unit



### User Presets for Audio Unit Extensions

```swift
open var userPresets: [AUAudioUnitPreset]? { get }
```

* Created and managed by the user

* Mutable



### New API

```swift
open var supportsUserPresets: Bool { get }
```

* Set by the Audio Unit to opt-in
* Checked by the host to verify support

```swift
open func saveUserPreset(_ userPreset: AUAudioUnitPreset) throws
open func deleteUserPreset(_ userPreset: AUAudioUnitPreset) throws
```

* Have default implementations in `AUAudioUnit`
* Can be overriden to implement custom logic

```swift
open func presetState(foor userPreset: AUAudioUnitPreset) throws -> [Striing : Any]
```

* Implemented in the superclass, but can be overriden
* Returns the contents of a user preset
* Can be assigned to `fullStateForDocument`

```swift
open var isLoadedInProcess: Bool { get }
```

* Returns true if the audio unit is loaded in-process
* Loading in process is only available on macOS



### Summary

* We now support user presets for Audio Units in addition to factoryPresets
* The Audio Unit has to opt in
* The methods have default implementations in the super class, but can be overriden