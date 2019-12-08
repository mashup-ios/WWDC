# [Introducing Accelerate for Swift](https://developer.apple.com/videos/play/wwdc2019/718/)

@ WWDC 19



### Accelerate

##### 특징

* High-performance, energy-efficient vectorized computation
* Classic C interfaces are awikward when working with Swift
* New Swift API is clearer and more concise



##### APIs

* `vDSP`: Digital signal processing functions
* `vForce`: Arithmetic and transcendental functions
* `Quadrature`: Numerical integration functions
* `vImage`: Image-processing functions



##### Scalar calculations

: Scalar calculations execute serially

```swift
let a: [Float] = [10, 20, 30, 40]
let b: [Float] = [ 1,  2,  3,  4]
let c: [Float] = [ 0,  0,  0,  0]

for i in 0..< c.count {
  c[i] = a[i] * b[i]
}

// c = [10.0, 40.0, 90.0, 160.0]
```



##### Vectorized instructions

```swift
let a: [Float] = [10, 20, 30, 40]
let b: [Float] = [ 1,  2,  3,  4]
let c: [Float] = [ 0,  0,  0,  0]

vDSP.multiply(a, b, result: &c)

// c = [10.0, 40.0, 90.0, 160.0]
```

이 경우에는 곱셈이 하나의 instruction으로 취급된다. 그렇기 때문에 성능적으로 더 좋음. 위의 scalar calculation은 4번의 곱셈을 수행하지만 여기서는 단 한 번만! 이건 energy efficient하다는 말도 됨.



### `vDSP`

* Fourier transforms
* Biquadratic filtering
* Convolution and correlation
* Vector and matrix arithmetic
* Type conversion



### vDSP for Vector Arithmetic

```swift
var reuslt = [Float](repeating: 0, count: n)

for i in 0..< n {
	result[i] = (a[i] + b[i]) * (c[i] - d[i])
}
```

⬇️

```swift
// using existing API
var result = [Float](repeating: 0, count: n)

vDSP_vasbm(a, 1,
           b, 1,
           c, 1, 
           d, 1,
           &result, 1,
           vDSP_Length(result.count))
```

이 버전이 위의 버전보다 3배 더 빠름

```swift
// using new API
vDSP.multiply(addition: (a, b),
             	subtraction: (c, d),
             	result: &result)
```

```swift
let result = vDSP.multiply(addition: (a, b),
                           subtraction: (c, d))
```



### vDSP for Type Conversion

이 예제는 double-precision element를 `UInt16`으로 변환시킬 것임

```swift
// scalar version
let result = source.map {
  return UInt16($0.rounded(.towardZero))
}

// using existing API
let result = Array<UInt16>(_unsafeUninitializedCapacity: source.count) { (buffer, initializedCount) in
	 vDSP_vfixu16(source, 1,
                buffer.baseAddress!, 1,
                vDSP_Length(source.count))
   initializedCount = source.count
}

// using new API
let reuslt = vDSP.floatingPointToInteger(source,
                                         integerType: UInt16.self,
                                         rounding: .towardZero)
```



### vDSP for Discrete Fourier Transform

* Decomposes a signal into its component frequencies
* Recreate a signal from its components frequencies

```swift
// using existing API
let setup = vDSP_DFT_zop_CreateSetup(
	nil,
  vDSP_Length(n),
  .FORWARD)!

var outputReal = [Float](repeating: 0, count: n)
var outputImag = [Float](repeating: 0, count: n)

vDSP_DFT_Execute(setup,
                 inputReal, inputImag,
                 &outputReal, &outputImag)

vDSP_DFT_DestroySetup(setup)
```

```swift
// using new API
let fwdDFT = vDSP.DFT(
	count: n,
	direction: .forward,
	transformTypeA: .complexComplex,
	ofType: Float.self)!

var outputReal = [Float](repeating: 0, count: n)
var outputImag = [Float](repeating: 0, count: n)

fwdDFT.transform(inputReal: inputReal,
                 inputImaginary: inputImag,
                 outputReal: &outputReal,
                 outputImaginary: &outputImag)
// 리소스 해제에 대해 걱정할 필요가 없어짐!
```

```swift
// using self-allocating API
let returnedResult = fwdDFT.transform(
	inputReal: inputReal,
	inputImaginary: inputImag)
```



### vDSP for Biquadratic Filtering

이 값들로 연산 진행할 것임

```swift
let sections = vDSP_Length(1)

let b0 = 0.0001
let b1 = 0.001
let b2 = 0.0005
let a1 = -1.9795
let a2 = 0.98

let channelCount = vDSP_Length(2)
```

```swift
// setting up with existing API
var output = [Float](repeating: -1, count: n)

let setup = vDSP_biquadm_CreateSetup([b0, b1, b2, a1, a2,
                                      b0, b1, b2, a1, a2],
                                    	vDSP_Length(sections),
                                    	vDSP_Length(channelCount))!

// applying biquadratic filter using existing API
signal.withUnsafeBufferPointer { inputBuffer in
  output.withUnsafeMutableBufferPointer { outputBuffer in
    let length = vDSP_Length(n) / channelCount
    var inputs: [UnsafePointer<Float>] = (0 ..< channelCount).map { i in
      return inputBuffer.baseAddress!.advanced(by: Int(i * length))
    }
    var ouptuts: [UnsafeMutablePointer<Float>] = (0 ..< channelCount).map { i in
      return outputBuffer.baseAddress!.advanced(by: Int(i * length))
    }
    vDSP_biquadm(setup, &inputs, 1, &outputs, 1,
                 vDSP_Length(n) / channelCount)
  }
}
```

```swift
// setting up with new API
var biquad = vDSP.Biquad(coefficients: [b0, b1, b2, a1, a2,
                                        b0, b1, b2, a1, a2],
                         channelCount: channelCount,
                         sectionCount: sections,
                         ofType: Float.self)!

let output = biquad.apply(input: signal)
```



### `vForce`

* Arithmetic functions: `floor, ceil, abs, remainder, ...`
* Exponential and logarithmic functions: `exp, log, ...`
* Trigonometric functions: `sin, cos, tan, ...`
* Hyperbolic functions: `sinh, asinh, ...`



##### Calculating square root

```swift
let a: [Float] = ...

let result = a.map {
  sqrt($0)
}
```

vForce를 이용하면 10배 더 빠르게 square root를 계산할 수 있다.

```swift
// using existing API
var result = [Float](repeating: 0, count: count)
var n = Int32(result.count)

vvsqrtf(&result, a, &n)
```

```swift
// using new API
var result = [Float](repeating: 0, count: count)
vForce.sqrt(a, result: &result)

// self-allocating API
let result = vForce.sqrt(a)
```



### Quadrature

: Integrate a function over an interval

```swift
// defining the integrate function using existing API
var integrateFunction: quadrature_integrate_function = {
  return quadrature_integrate_function(
  	fun: { (arg: UnsafeMutableRawPointer?, n: Int,
            x: UnsafePointer<Double>, y: UnsafeMutablePointer<Double>) in
      guard let radius = arg?.load(as: Double.self) else { return }
      
      (0 ..< n).forEach { i in
        y[i] = sqrt(radius * radius - x[i] * x[i])
      }
    },
    fun_arg: &radius)
}()

// defining the integrate options using existing API
var options = quadrature_integrate_options(integrator: QUADRATURE_INTEGRATE_QNG, 
                                          abs_tolerance: 1.0e-8,
                                          rel_tolerance: 1.0e-2,
                                          qag_points_per_interval: 0,
                                          max_intervals: 0)

// performing the integration using exsiting API
var status = QUADRATURE_SUCCESS
var estimatedAbsoluteError: Double = 0

let result = quadrature_integrate(&integrateFunction,
                                  -radius,
                                  radius,
                                  &options,
                                  &status,
                                  &estimatedAbsoluteError,
                                  0,
                                  nil)
```

```swift
// using new API
let quadrature = Quadrature(integrator: .nonAdaptive,
                            absoluteTolerance: 1.0e-8,
                            relativeTolerance: 1.0e-2)

let result = quadrature.integrate(over: -radius ... radius) { x in
  return sqrt(radius * radius - x * x)
}
```

```swift
// using alternative integrator with exsiting API
let quadrature = Quadrature(integrator: .adaptive(pointsPerInterval: .fifteen, maxIntervals: 7),
                            absoluteTolerance: 1.0e-8,
                            relativeTolerance: 1.0e-2)

let result = quadrature.integrate(over: -radius ... radius) { x in
  return sqrt(radius * radius - x * x)
}
```



### `vImage`

사용 분야

* Core Graphics interoperability
* Core Video interoperability
* Alpha blending
* Format conversions
* Histogram operations
* Convolution
* Geometry
* Morphology

특징

* Flags are not Swift `OptionSet`
* Throws proper Swift errors
* Enumerations for pixel formats and buffer types
* Hides requirements for unmanaged types and mutable buffers
* Moves free functions to properties on buffers and formats



##### Create a buffer from a Core Graphics image

* Create format description
* Instantiate the buffer
* Initialize the buffer from `CGImage`

```swift
// creating a buffer from an image using new API
let sourceBuffer = try? vImage_Buffer(cgImage: image)

// or
let format = vImage_CGImageFormat(cgImage: image)!
let sourceBuffer = try? vImage_Buffer(cgImage: image, 
                                      format: format)

// creating image from a buffer using new API
let cgImage = try? sourceBuffer.createCGImage(format: format)
```



### vImage Converting Any-to-Any

* Core Graphics to Core Graphics
* Core Graphics to Core Video
* Core Video to Core Graphics

```swift
// using new API
let converter = try? vImageConverter.make(sourceFormat: cmykSourceImageFormat,
                                          destinationFormat: rgbDestinationImageFormat)

try? converter?.convert(source: cmykSourceBuffer,
                        destination: &rgbDestinationBuffer)
```



### vImage Working with CV Image Formats

* Create format description from `CVPixelBuffer`
* Calculate channel count

```swift
// using new API
let cvImageFormat = vImageCVImageFormat.make(buffer: pixelBuffer)

let channelCount = cvImageFormat?.channelCount
```



이런 게 있다. 이만큼 성능이 좋아졌고, new api를 사용하면 이만큼 간단하게 작성이 가능하다!를 보여주는 wwdc 였당.