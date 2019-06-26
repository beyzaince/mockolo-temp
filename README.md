

# Welcome to Mockolo

`Mockolo` is a lightweight commandline tool which uses the `MockoloFramework` framework for creating mocks in Swift.  It uses `SourceKittenFramework` for parsing, and a custom template renderer for generating a mock output.  

## System Requirements 

* Swift 4.2 or later
* Xcode 10.1 or later
* Support is included for the Swift Package Manager


## Build / Install

First, clone the project. 

```
$ git clone https://github.com/uber/mockolo.git
$ cd mockolo
```

Optionally, see a list of released versions of `Mockolo`, and check one out by running the following. 

```
$ git tag -l
$ git checkout [tag]
```

Run the following to make a release build. 

```
$ swift build --static-swift-stdlib -c release
```

This will create a binary called `mockolo` in the `.build/release` directory.

To install, just copy this executable into a directory that is part of your `PATH` environment variable.


To use Xcode, run the following. 

```
$ swift package generate-xcodeproj 
```


## Add MockoloFramework to your project 

```swift

dependencies: [
    .package(url: "https://github.com/uber/mockolo.git", from: "1.1.0"),
],
targets: [
    .target(name: "MyTarget", dependencies: ["MockoloFramework"]),
]

```


## Run

`Mockolo` is a commandline executable. To run it, pass in a list of the source directories or source file paths of a build target, and the ouptut filepath for the mock output. To see other arguments to the commandline, run `mockolo --help`.

```
.build/release/mockolo -s srcsFoo srcsBar -d ./MockResults.swift -x Images Strings
```

This parses all the source files in `srcsFoo` and `srcsBar`, excluding any files ending with `Images` or `Strings` in the file name (e.g. MyImages.swift), and generates mocks to a file at `./MockResults.swift`. 


## Sample Code 

For example, Foo.swift contains: 

```swift 
/// @mockable
public protocol Foo { 
    var num: Int { get set }
    func bar(arg: Float) -> String
}
```

Running the commandline ```.build/release/mockolo -srcs Foo.swift -d ./MockResults.swift ``` will produce: 

```swift 
public class FooMock: Foo { 
    init() {}
    init(num: Int = 0) {
        self.num = num
    }
    
    var numSetCallCount = 0
    var underlyingNum: Int = 0
    var num: Int {
        get {
            return underlyingNum
        }
        set {
            underlyingNum = newValue
            numSetCallCount += 1
        }
    }
    
    var barCallCount = 0
    var barHandler: ((Float) -> (String))?
    func bar(arg: Float) -> String {
        barCallCount += 1
        if let barHandler = barHandler {
            return barHandler(arg)
        }
        return ""
    }
}
```

Now the mock can be used in a test as follows: 

```swift 
func testMock() {
    let mock = FooMock(num: 5) 
    XCTAssertEqual(mock.numSetCallCount, 1) 
    mock.barHandler = { arg in 
        return String(arg)
    }
    XCTAssertEqual(mock.barCallCount, 1) 
}
```


## Limitations
It currently supports protocol mocking.  Class mocking will be added in the future. 


## Report any issues

If you run into any problems, please file a git issue. Please include:

* The OS version (e.g. macOS 10.14.0)
* The Swift version installed on your machine (from `swift --version`)
* The Xcode version 
* The specific release version of this source code (you can use `git tag` to get a list of all the release versions or `git log` to get a specific commit sha)
* Any local changes on your machine 
