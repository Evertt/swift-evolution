# Implicit and explicit casting of value types

* Proposal: [SE-NNNN](https://github.com/apple/swift-evolution/blob/master/proposals/NNNN-name.md)
* Author(s): [Evert van Brussel](https://github.com/Evertt)
* Status: **[Awaiting review](#rationale)**
* Review manager: TBD

## Introduction

Casting all kinds to types to each other can result in overly verbose code. I believe we should allow implicit casting when it's explicitly defined HOW to cast one type to anther.  

Swift-evolution thread: [link to the discussion thread for that proposal](https://lists.swift.org/pipermail/swift-evolution)

## Motivation

```swift
let a: Double = 2.5
let b: Int = -3
let c: Float = 1.1
let d: UInt = 5
let e: Double = (a + b) * (c + d) // This results in an error
let f: Double = (a + Double(b)) * (Double(c) + Double(d)) // This is the current correct way
```
    
The explicit conversion makes the code harder to read and it's just unnecessary. We all know that an integer converted to a double still represents the same value. And we all know that all integers can be converted to doubles.

## Proposed solution

My solution is to allow for implicit conversion when there's a global function defined for this conversion. I would call this function `cast(from:)`. For example:

```swift
func cast(from value: Int) -> Double {
    return Double(value)
}
```
    
Whenever there's a global function defined like this then the compiler will know that Double can be implicitly converted to Int. And so it will accept the `let e: Double = (a + b) * (c + d)` line of code above.

This should be possible for all value types, for example:

```swift
func cast(from square: Square) -> Polygon {
    return Polygon([
        square.topLeftPoint,
        square.topRightPoint,
        square.bottomRightPoint,
        square.bottomLeftPoint
    ])
}
```

## Detailed design

Now of course not all types can be converted without failing. All integers can become doubles, but not all doubles can become integers. And all squares can become polygons, but not all polygons can become squares.

```swift
func cast(from value: Double) -> Int? {
    if value > Double(Int.max) {
        return nil
    }
    
    if value != Double(Int(value)) {
        return nil
    }
    
    return Int(value)
}
```

This means a few things:

1. If I write: `let e = someInt * someFloat * someDouble` then the compiler should be able to infer that e should be a `Double`, since `Double` is the only type that all the other types can be casted to without failing.
2. If I do want the `e` constant to be an integer, I should declare it as an optional integer like so: `let e: Int? = someInt * someFloat * someDouble`.
3. Casting-functions that can fail / return optionals shouls be useable in the following syntax: `if let square = polygon as? Square`. Which I think shows your intention more clearly than when you use initializers, since you are trying to cast one type to another.

## Impact on existing code

This won't impact existing code. If someone coincidentally already defined a global function in this way then they can still use that function like normal.

## Alternatives considered

One alternative is to use protocols for this. Like `StringCastable` and `IntCastable` etc. The downside is that this casting will then be limited to the protocols and types that are implemented by the Swift team. I would like to keep this open for all possible types.
