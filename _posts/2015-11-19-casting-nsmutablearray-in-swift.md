---
published: true
title: Casting NSMutableArray in Swift
---

Sometimes bridging `Objective-C` into `Swift` is not as straight forward. As example, consider casting `NSMutableArray` of objects into Swift's `[String]`.

So here are some examples how to do it.

## Not Optional variable

```swift
let array1 = NSMutableArray(array: ["a", "b", "c"])
if let items = array1 as NSArray as? [String] {
    print(items)
}
```

## Optional variable

```swift
var array2: NSMutableArray?

// Comment this to check
array2 = NSMutableArray(array: ["a", "b", "c"])

if let items = (array2 as NSArray?) as? [String] {
    print(items)
}
```

## Optional variable inside optional class

```swift
class MyData : NSObject {
    var items: NSMutableArray?
    
    func createItems() {
        items = NSMutableArray(array: ["a", "b", "c"])
    }
}

var myData: MyData?
myData = MyData()     // Initializing empty class
myData?.createItems() // Filling with items

if let items = (myData?.items as NSArray?) as? [String] {
    print(items)
}
```

Do you like the `if let items = (myData?.items as NSArray?) as? [String]` construction?
Any better suggestions?