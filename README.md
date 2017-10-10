[![codebeat badge](https://codebeat.co/badges/b413ed75-c114-4137-9e60-8c92bbe1667a)](https://codebeat.co/projects/github-com-favret-magic-swift-bus-master)

# MagicSwiftBus
Use NotificationCenter with EventBus style

## What is MagicSwiftBus ?
- Implement publish-subscribe communication between object easily.
- Don't loose time with crash due to missing character, use enum instead.
- don't use NSNotification as parameters of your methods, define the real param directly.
- Define all your communication in a protocol.

[inspired By](https://medium.com/swift-programming/swift-nsnotificationcenter-protocol-c527e67d93a1#.5zinv4kr6)

### Before MagicSwiftBus

#### register / unregister

```swift
NSNotificationCenter.defaultCenter().addObserver(
    self,
    selector: #selector(testSuccess(_:)),
    name: "MyNotificationTest",
    object: nil)
```

#### post notification

```swift
NSNotificationCenter.defaultCenter().postNotificationName("MyNotificationTest", object: "bonjour" as AnyObject)
```

#### implement notification's method

```swift
func testSuccess(notification: Notification) {
  if let object = notification.object as String {
  }
}
```

### With MagicSwiftBus

#### register / unregister

```swift
Bus.register(self, event: .Test, with: "bonjour")
```

#### post notification

```swift
Bus.post(.Test)
```

#### implement notification's method

```swift
func testSuccess(str: String) {
  print(str)
}
```

## Installation

### CocoaPods

[CocoaPods](http://cocoapods.org) is a dependency manager for Cocoa projects. You can install it with the following command:

```bash
$ gem install cocoapods
```

To integrate MagicSwiftBus into your Xcode project using CocoaPods, it in your `Podfile`:

```ruby
platform :ios, '9.0'
use_frameworks!

target '<Your Target Name>' do
  pod 'MagicSwiftBus'
end
```

Then, run the following command:

```bash
$ pod install
```

### Carthage

[Carthage](https://github.com/Carthage/Carthage) is a decentralized dependency manager that builds your dependencies and provides you with binary frameworks.

You can install Carthage with [Homebrew](http://brew.sh/) using the following command:

```bash
$ brew update
$ brew install carthage
```

To integrate Magic-Swift-Bus into your Xcode project using Carthage, specify it in your `Cartfile`:

```ogdl
github "favret/Magic-Swift-Bus" => 3.0.0
```

Run `carthage update` to build the framework and drag the built `MagicSwiftBus.framework` into your Xcode project.

### Manually

If you prefer not to use either of the aforementioned dependency managers, you can integrate Magic-Swift-Bus into your project manually as an Embedded Framework.

## Usage

### Create your event protocol

```swift
@objc protocol MyEvent {
  func testSuccess(str:String)
  func makeCoffee()
}
```

Here, you can see that the parameter is a `String` but it can be something else.

### Implement your event protocol

```swift
extension ViewController: MyEvent {

  func makeCoffee() {
    print("☕️")
  }

  func testSuccess(str: String) {
    print(str)
  }
}

```

In this exemple, `MyReceiver` can be `UIViewController` or `NSObject` or wathever you want.

### Add your event protocol to MagicSwiftBus

```swift
class MyBus: Bus {
  enum EventBus: String, EventBusType {
    case test, makeCoffee

    var notification: Selector {
      switch self{
      case .test: return #selector(MyEvent.testSuccess(str:))
      case .makeCoffee: return #selector(MyEvent.makeCoffee)
      }
    }
  }
}
```
First, you have to create a class that inherit `Bus`.
Then, implement `EventBusType` protocol. in the above exemple, this protocol is implemented by an enum.

### Don't forget to register and unregister

```swift
MyBus.register(observer: self, events: .test, .makeCoffee)
...
MyBus.unregisterAll(observer: self)
```

### Fire simple notification

Finally, you can fire an event like that :

```swift
    MyBus.post(event: .test, with: "hello world")
```

### Thread

#### post event on main Thread

```swift
MyBus.postOnMainThread(event: .test, with: "hello world")
```

#### post event on background Thread (default)

```swift
MyBus.postOnBackgroundThread(event: .test, with: "hello world")
```

#### post event on specific Queue

```swift
MyBus.post(on: DispatchQueue.global(qos: .background), event: .test, with: "hello world")
```

## Exemple

```swift
import UIKit
import MagicSwiftBus

@objc protocol MyEvent {
  func testSuccess(str:String)
  func makeCoffee()
}

class MyBus: Bus {
  enum EventBus: String, EventBusType {
    case test, makeCoffee

    var notification: Selector {
      switch self{
      case .test: return #selector(MyEvent.testSuccess(str:))
      case .makeCoffee: return #selector(MyEvent.makeCoffee)
      }
    }
  }
}

class ViewController: UIViewController {

  override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view, typically from a nib.
  }

  override func didReceiveMemoryWarning() {
    super.didReceiveMemoryWarning()
    // Dispose of any resources that can be recreated.
  }

  // Register / Unregister
  override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    MyBus.register(observer: self, events: .test, .makeCoffee)
    MyBus.post(event: .test, with: "helloWorld")
    MyBus.post(event: .makeCoffee)
  }

  override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
    MyBus.unregisterAll(observer: self)
  }
}

//Implement method that you want to use
extension ViewController: MyEvent {

  func makeCoffee() {
    print("☕️")
  }

  func testSuccess(str: String) {
    print(str)
  }
}
```
