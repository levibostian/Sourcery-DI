# Sourcery-DI

Automatically generate a dependency injection graph for your source code. No frameworks needed. 

## How? 

There is this really neat tool called [Sourcery](https://github.com/krzysztofzablocki/Sourcery). It is really good at generating boilerplate Swift code for your project. Things that iOS developers usually do my hand (creating mocks of classes, creating a dependency injection graph) can be automated for you. 

Sourcery does this with the Sourcery CLI tool and [Stencil template files](https://stencil.fuller.li/en/latest/). This project houses a Stencil template file you can use in your project with Sourcery to automatically generate a dependency injection graph for your code. 

## Why use this project?

* **Zero dependencies.** You just need Sourcery (which is a compile time tool, not runtime) to use this project. 
* **Testing friendly.** Override dependencies in your graph with mocked/stubbed versions for your tests. That's why your using dependency injection, yeah? 
* **Fast.** The dependency graph generated is simply a collection of constructors and lazy loading properties. No need to initialize the graph. 
* **Flexible.** Add all your code's dependencies no matter if they are singletons or 3rd party library classes. 
* **Thread safe.** Get dependencies from any thread, safely. 

*Note: It is not a goal of this project to support circular dependencies. Adding a unit test against your graph should find circular dependencies for you so you can then fix them. If you have a circular dependency, it might be a sign to refactor your code, not use a different tool. If you need more help, follow the "set dependency by property" strategy outlined in the DI framework, [Swinject documentation](https://github.com/Swinject/Swinject/blob/966cc2f0db1637f535ed004d1a10ea041f6e68b5/Documentation/CircularDependencies.md)*

## Installation

* Install and configure [Sourcery](https://github.com/krzysztofzablocki/Sourcery) in your project. 
* Manually download the `AutoDependencyInjection.stencil` file found in this repository. Put the template file in with your other Sourcery templates. 
* This is optional, but **highly** recommended. Make a unit test in your project to test your dependency graph:
```swift
import Foundation
@testable import YourApp
import XCTest

class DiTests: XCTestCase {

    func testDependencyGraphComplete() {
        for dependency in Dependency.allCases {
            XCTAssertNotNil(DI.shared.inject(dependency), "Dependency: \(dependency) not able to resolve in dependency graph. Maybe you're using the Sourcery template incorrectly or there is a circular dependency in your graph?")
        }
    }
    
}
```
This test can help find circular dependencies. 

* Done! Keep reading on how to use this template. 

# How to add dependencies to graph

If you're used to using Sourcery, you understand the concept of using Sourcery annotations. This project replies on annotations to define how a dependency should be constructed. 

### Add a non singleton class

*Note: The instructions below are for using a Swift `class`. If your dependency is something other then a `class`, skip this section.*

Add a comment above your class:

```swift
// sourcery: InjectRegister = "OffRoadWheels"
class OffRoadWheels {
}
```

Run the Sourcery CLI tool and you will now be able to access your new dependency:
```swift
let wheels = DI.shared.offRoadWheels
```

If you plan to use mocking in your tests suite, it's recommended to use a `Protocol`:

```swift
protocol Wheels {
    ...
}

// sourcery: InjectRegister = "Wheels"
class OffRoadWheels: Wheels {
    ...
}
```

Run the Sourcery CLI tool and you will now be able to access your new dependency:
```swift
let wheels = DI.shared.wheels
// `wheels` is type checked to the Wheels Protocol. 
```

### Add a generic class

If you have a class that uses generics in the constructor:

```swift
class Car<EngineType: Engine> {}
```

You will not be able to add the class to the graph as is. Instead, you must add to the graph specific type definitions. Here is an example: 

```swift
// sourcery: InjectRegister = "ElectricCar"
// sourcery: InjectCustom
typealias ElectricCar = Car<ElectricEngine>

extension DI {
    var customElectricCar: ElectricCar {
        // If your constructor requires dependencies, you can use the graph to provide them:
        return ElectricCar(DI.shared.battery)
    }
}
```

> `typealias` is the recommended way of doing this. You can add this code to the same file your `Car<>` class is defined. 

Run the Sourcery CLI tool and you will now be able to access your new dependency:
```swift
let electricCar = DI.shared.electricCar
// `electricCar` is type checked to the `ElectricCar` typealias
```

### Add a singleton class

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
let wheels = DI.shared.offRoadWheels
let otherInstanceWheels = DI.shared.offRoadWheels
// `wheels` and `otherInstanceWheels` are the same instance
```

### Add a class from a 3rd party

*Note: If you need your 3rd party dependency to be a singleton, this is not supported by this project, sorry. View the generated DI graph .swift file to see examples of how this stencil creates singletons. You can then use `extension DI` to add functionality to the graph to add your singleton.*

In order to add classes from 3rd parties, we will extend our dependency injection graph. Let's add the iOS SDK `UserDefaults` to our graph. Here is the syntax to do that. 

```swift
// sourcery: InjectRegister = "UserDefaults"
// sourcery: InjectCustom
extension UserDefaults {}

extension DI {
    var customUserDefaults: UserDefaults {
        return UserDefaults.standard
    }
}
```

Run the Sourcery CLI tool and you will now be able to access your new dependency:
```swift
let userDefaults = DI.shared.userDefaults
// `electricCar` is type checked to the `ElectricCar` typealias
```

*Note: Yes, this syntax is not the most elegant code in the world but it's a limitation to Sourcery. This is how we can get it done.*

This technique above has some downfalls...Your graph .swift file might have compile-time errors because the file needs an `import` statement to find your 3rd party library. There are 2 ways to fix this:

1. The quick way. (not recommended but, quick)

Open up the `AutoDependencyInjection.stencil` file you copied into your project and add another `import` statement to the file:

```
import Foundation
import KeychainAccess
import Moya
```

Import whatever frameworks you need. 

Pros: 
* Quick and easy. 1 line of code. 
Cons:
* The next time there is an update to the stencil file in this project, you need to remember what frameworks you added. 

2. `typealias` (recommended but adds a little boilerplate code to your project)

Create `typealias` definitions to alias the 3rd party definitions. 

For example, if you wanted to use the [KeychainAccess](https://github.com/kishikawakatsumi/KeychainAccess) framework in your app, you could create this dependency on a `KeychainAccess` instance:

```swift
import KeychainAccess

// sourcery: InjectRegister = "KeychainAccess"
// sourcery: InjectCustom
extension KeychainAccess {}

extension DI {
    var customKeychainAccess: KeychainAccess {
        return Keychain(service: "com.example.github-token")
    }
}
```

If you don't want to need to add an `import` statement to the stencil file, you can create a typealias for `KeychainAccess`:

```swift
import KeychainAccess

typealias Keychain = KeychainAccess
// sourcery: InjectRegister = "Keychain"
// sourcery: InjectCustom
extension Keychain {}

extension DI {
    var customKeychain: Keychain {
        return Keychain(service: "com.example.github-token")
    }
}
```

Pros: 
* Separation between code and the Sourcery stencils. No need to maintain the list of import statements in the stencil file.
Cons:
* More code required to create a `typealias`. 
* If you decide in the future to no longer use the `KeychainAccess` framework, it would require you to edit lots of your code. 

3. Wrapper (requires most boilerplate but allows your 3rd party dependencies to be testable)

If you wanted to make your code even more maintainable long-term and making your code more testable, you could also create a wrapper:

```swift
// sourcery: InjectRegister = "Keychain"
protocol Keychain {
    func set(_ key: String, value: String)
    func get(_ key: String) -> String?
}

import KeychainAccess
class KeychainAccessKeychain: Keychain {
    // .. impl
}

// Use the protocol
class ViewModel {
    init(keychain: Keychain) {        
    }
}
```

Pros: 
* More testable code since it's uses the abstraction of a protocol. 
* More maintainable code long-term. 
* No need to mess with the Sourcery stencil. 
Cons:
* Requires the most boilerplate code. 

##### Add a `typealias`

Typealias are a great way to define dependencies that are *generic*. 

```swift
typealias GitHubMoyaProvider = MoyaProvider<GitHubService>
// sourcery: InjectRegister = "GitHubMoyaProvider"
// sourcery: InjectCustom
extension GitHubMoyaProvider {}

extension DI {
    var customerGitHubMoyaProvider: GitHubMoyaProvider {        
        return MoyaProvider<GitHubService>()
    }
}
```

*Note: Yes, you need to have the blank `extension` as defined above. It's a hack in order to [have Sourcery find your `typealias` annotations](https://github.com/krzysztofzablocki/Sourcery/issues/765). We may not need this in the future. Who knows.*

## Author

* Levi Bostian - [GitHub](https://github.com/levibostian), [Twitter](https://twitter.com/levibostian), [Website/blog](http://levibostian.com)

![Levi Bostian image](https://gravatar.com/avatar/22355580305146b21508c74ff6b44bc5?s=250)

## Contribute

Sourcery-DI is open for pull requests. Check out the [list of issues](https://github.com/levibostian/Sourcery-DI/issues) for tasks I am planning on working on. Check them out if you wish to contribute in that way.

**Want to add features to Sourcery-DI?** Before you decide to take a bunch of time and add functionality to the library, please, [create an issue](https://github.com/levibostian/Sourcery-DI/issues/new) stating what you wish to add. This might save you some time in case your purpose does not fit well in the use cases of Sourcery-DI.

