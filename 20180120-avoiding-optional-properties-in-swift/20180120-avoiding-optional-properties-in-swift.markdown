# How to avoid optional properties in your classes

## First solution: Relay responsibility to the caller

I bet you all have seen code like this. The owner of this instance is expected to set `thing` at some point of time, but when you look at `ThingController` you have no idea how important `thing` is to this class, how often it is expected to change, if it is expected to go away and so on. The complete lifecycle is a blackbox.

```swift
class ThingController {

	var thing: Thing?

	init() { }

	func doSomething() {
		guard if let thing = thing else { return }
		thing.doIt()
	}
}

```

I cannot overstate how often this unspectacular pattern declutters code and makes it easier to understand: Just make the property a required argument of your init function. Additional being a `let` the lifecycle of `thing` is now more clear: It is set once, it is always available and it won't change over the lifecylce of the instance.

```swift
class ThingController {

	let thing: Thing

	init(thing: Thing) {
		self.thing = thing
	}

	func doSomething() {
		thing.doIt()
	}
}

```

Sometimes a property needs to be `private` and initiating it may fail. Even in this case you may consider making it the responsibility of the caller to handle the case that the type (in this case `ThingController`) that was created cannot work properly.

```swift
class ThingController {

	private let thing: Thing

	init?(value: Value) {
		guard let thing = Thing(value: value) else { return nil }
		self.thing = thing
	}
}

```

A fail-able initializer is not your only option. Make your instances throw an error if something goes wrong ...

```swift
class ThingController {

	enum Error: Swift.Error {
		case invalidValue
	}

	private let thing: Thing

	init(value: Value) throws {
		guard let thing = Thing(value: value) else { throws Error.invalidValue }
		self.thing = thing
	}
}

```

... or make the initializer of the type you rely on throw an error and relay it to your caller.

```swift
class ThingController {

	private let thing: Thing

	init(value: Value) throws {
		self.thing = try Thing(value: value)
	}
}

```


## Second solution: Be lazy

Often you use Optionals (T?) or Implicitly Unwrapped Optionals (T!) to work around the fact that you need `self` to create the instance, but you cannot access `self`, because it's not fully initialized itself yet.

In this case your code may look like that:

```swift
class APIManager: NSObject, URLSessionDelegate {
    
    var session: URLSession?
    
    override init() {
        super.init()
        let configuration = URLSessionConfiguration()
        self.session = URLSession(configuration: configuration, delegate: self, delegateQueue: nil)
    }
}
```

By being lazy `session` will be instantiated "later" when you are first accessing it. At this point `self` will be there for you:


```swift
class APIManager: NSObject, URLSessionDelegate {
    
    lazy var session: URLSession = {
        let configuration = URLSessionConfiguration()
        let session = URLSession(configuration: configuration, delegate: self, delegateQueue: nil)
        return session
    }()
    
    override init() {
        super.init()
    }
}
```


## Third solution: Remove `self` out of the equation

In some cases you may need to call a function you wrote to instantiate your object. 

```swift
class ThingController {
    
    var thing: Thing?
    
    init() {
        let value = self.computeValue()
        self.thing = Thing(value: value)
    }
    
    func computeValue() -> Value {
        return Value()
    }
}
```

You could use solution 2, but it maybe worth it taking a hard look at the function you are calling. Maybe you can refactor it into a `static` or `class` function, because it can work independently from the instance you are creating. If not, maybe it is relying on a property you may already have in your init function? Pass it as an argument :)


```swift
class ThingController {
    
    let thing: Thing
    
    init() {
        let value = ThingController.computeValue()
        self.thing = Thing(value: value)
    }
    
    static func computeValue() -> Value {
        return Value()
    }
}
```

## This is it

With that being said, there are many cases where optional properties are useful and having them is not inherently bad. Like always it depends and the goal of the article is solely to give you other options to consider. Happy coding!
