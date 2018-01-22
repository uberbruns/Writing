# Unwrapping nested Optionals

## Starting position

Consider you want to unwrap a type with a signature of `Bool??`. For example, you cam end up with a nested optional, when you had a function like `doSomething() throws -> Bool?` and you assign the result to variable `x` by using `try?`.


```swift
func doSomething() throws -> Bool? {
	// Code that returns Bool, nil or throws
	// ...
}

let x = try? doSomething()
// x is now of type Bool??
```

But how you got there is not what these notes are about. They are about how you unwrap these nested optionals, without writing too much unnecessary code. So, the obvious approach might be to unwrap `x` twice:

```swift
if let optionalX = x, let unwrappedX = optionalX {
	// unwrappedX is now of type `Bool`
}

```

But this is very verbose and you end up defining a variable you don't need. Now you propably think, that there has to be a better way and you're right.

## The solution

But there is a more direct and less verbose way by using pattern matching. First, you have to know that pattern matching is not something that only works with `switch` clauses. Using `if case` or `guard case` you can match patterns in `if` or `guard` statements, too. In the case of `if case` unwrapping a nested Optional works like this:


```swift
if case let unwrappedX?? = x {
	// unwrappedX is now of type `Bool`
}

```

## Why does this work?

Now you might ask how this works? Well, Under the hood Optionals are Enums with two cases: `.some(T)` and `.none`. So to setup pattern matching of `anOptional`, to match the `.some(T)` case and read the value of that case, you write code that looks something like this:

```swift
if case .some(let wrapped) = anOptional {
	// You can now use `wrapped`
}

```

Since you can nest patterns, this is how unwrapping would work, if we would use the more verbose syntax for our example o top.

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