# Unwrapping nested Optionals

## Starting position

Consider you want to unwrap a type with a signature of `Bool??`. You end up with this kind a nested optional, when you, for example, had a function like `doSomething() throws -> Bool?` and you assign the result to variable `x` by using `try?`.


```swift
func doSomething() throws -> Bool? {
	// Code that returns Bool, nil or throws
	// ...
}

let x = try? doSomething()
// x is now of type Bool??
```

But how you got there is not what these notes are about. Its about how you unwrap nested optionals, without writing too much unnecessary code. So, the obvious approach might be to unwrap `x` twice:

```swift
if let optionalX = x, let unwrappedX = optionalX {
	// unwrappedX is now of type `Bool`
}

```

But you might find this super verbose and you're thinking, that there has to be a better way.

## The solution

But there is a more direct and less verbose way using pattern matching. First, you have to know that pattern matching is not something that only works with `switch`. Using `if case` or `guard case` you can use it with `if` and `guard`, too. In the case of `if case` unwrapping nested optionals will work like this:


```swift
if case let unwrappedX?? = x {
	// unwrappedX is now of type `Bool`
}

```

## Why does this work?

Now you might ask how this works? Well, Optionals are nothing but Enums with two cases: `.some(T)` and `.none`. So pattern matching `anOptional`, to match the `.some(T)` case would look like this:

```swift
if case .some(let wrapped) = anOptional {
	// You can now use `wrapped`
}

```

Since you can nest patterns, this is how unwrapping in our example works, if we would use the more verbose syntax.

```swift
if case .some(.some(let unwrappedX)) = x {
	// unwrappedX is now of type `Bool`
}

```

To make the code more readable, the creators of Swift provided a shorter syntax for this. Instead of writing `if case .some(let wrapped) = ...` you can use `if case let wrapped? = ...`. And because even this syntax can be nested, we end up with the double question-marks:

```swift
if case let unwrappedX?? = x {
	// unwrappedX is now of type `Bool`
}

```