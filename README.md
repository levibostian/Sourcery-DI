# Sourcery-DI

Automated iOS dependency injection through [Sourcery](https://github.com/krzysztofzablocki/Sourcery). 

## What is Sourcery-DI

A pre-made stencil template you can use with the [Sourcery](https://github.com/krzysztofzablocki/Sourcery) tool to generate a dependency graph for your iOS app. 

## Why use Sourcery-DI?

* **Zero dependencies.** You just need Sourcery (which is a compile time tool, not runtime). 
* **Testing friendly.** Override dependencies in your graph with mocked/stubbed versions for your tests. 
* **Fast.** The dependency graph generated is simply a collection of constructors and lazy loading properties. 

## Goals of Sourcery-DI

This project strives to stay true to these promises:

1. Zero dependencies. Just use Sourcery.
2. Support singleton dependencies. Make the optional and not the default. 
3. Make it easy to unit test the dependency graph. Because we cannot prove at compile-time our dependency graph is complete, make it unit testable to become confident in it.
4. Add a dependency to our graph with as little work as possible. Focus on writing the dependencies, not maintaining a graph. 
5. Thread safe. Get dependencies from any thread, safely. 
6. Keep things flexible. If you need to manually write the code to construct your dependency instead of having Sourcery generate the construction automatically, we give you that option. 

*Note: It is not a goal of this project to support circular dependencies. The unit tests against your graph should find circular dependencies for you so you can then fix them. If you have a circular dependency, it might be a sign to refactor your code. If you need circular dependencies, follow the "set dependency by property" strategy outlined in the DI framework, [Swinject](https://github.com/Swinject/Swinject/blob/966cc2f0db1637f535ed004d1a10ea041f6e68b5/Documentation/CircularDependencies.md)*

## Installation

* Install and configure Sourcery in your iOS project. 
* Manually download the `AutoDependencyInjection.stencil` file found in this repository. Put the template file in with your other Sourcery templates. 
* This is optional, but **highly** recommended. Make a unit test for your dependency graph:
```swift
import Foundation
@testable import YourApp
import XCTest

class DiTests: XCTestCase {

    func testDependencyGraphComplete() {
        for dependency in Dependency.allCases {
            XCTAssertNotNil(DI.shared.inject(dependency), "Dependency: \(dependency) not able to resolve in dependency graph")
        }
    }
    
}
```
* Done! Keep reading on how to use this template. 

# Getting started

If you're used to using Sourcery, you understand the concept of using Sourcery annotations. This project replies on annotations to define how a dependency should be constructed. 

### Define a class as a non-singleton dependency

*Note: The instructions below are for using a Swift `class`. If your dependency is something other then a `class`, skip this section.*

```swift
// sourcery: InjectRegister = "OffRoadWheels"
class OffRoadWheels {
}
```

This will allow you to then get a reference to `OffRoadWheels` from the dependency graph like this:

```swift
class ViewController: UIViewController {
    let wheels = Di.shared.inject(.offRoadWheels)
    // `wheels` is of type `OffRoadWheels`
}
```

If your iOS app relies heavily on protocol based programming to make your code easily testable, define your dependency like this:

```swift
protocol Wheels {
}

// sourcery: InjectRegister = "Wheels"
class OffRoadWheels {
}
```

This will define the dependency as an abstracted protocol that you can use in your code:

```swift
class ViewController: UIViewController {
    let wheels = Di.shared.inject(.offRoadWheels)
    // `wheels` is of type `Wheels`
}
```

### Define a class as a singleton dependency

*Note: The instructions below are for using a Swift `class`. If your dependency is something other then a `class`, skip this section.*

If you have a dependency that you want to share 1 instance across your entire app, define your dependency as a singleton like this:

```swift
// sourcery: InjectRegister = "OffRoadWheels"
// sourcery: InjectSingleton
class OffRoadWheels {
}
```

Now, when your code references `OffRoadWheels`, it will get the same shared instance. 

```swift
let wheels = Di.shared.inject(.offRoadWheels)
let otherInstanceWheels = Di.shared.inject(.offRoadWheels)
// `wheels` and `otherInstanceWheels` are the same instance
```

### Define a custom dependency 

If you need to add a dependency to your graph that is not a class, or, you need to inject some 3rd party code that you didn't write such as `UserDefaults`, here is how you will do so:

```swift
// sourcery: InjectRegister = "UserDefaults"
// sourcery: InjectCustom
extension DI {
    var userDefaults: UserDefaults {
        return UserDefaults.standard
    }
}
```

*Note: You must define 1 dependency for every `extension DI` definition. This is the way that Sourcery works.*
*Note: If you need your custom dependency to be a singleton, that is up to you to perform.*

## Author

* Levi Bostian - [GitHub](https://github.com/levibostian), [Twitter](https://twitter.com/levibostian), [Website/blog](http://levibostian.com)

![Levi Bostian image](https://gravatar.com/avatar/22355580305146b21508c74ff6b44bc5?s=250)

## Contribute

Sourcery-DI is open for pull requests. Check out the [list of issues](https://github.com/levibostian/Sourcery-DI/issues) for tasks I am planning on working on. Check them out if you wish to contribute in that way.

**Want to add features to Sourcery-DI?** Before you decide to take a bunch of time and add functionality to the library, please, [create an issue](https://github.com/levibostian/Sourcery-DI/issues/new) stating what you wish to add. This might save you some time in case your purpose does not fit well in the use cases of Sourcery-DI.

