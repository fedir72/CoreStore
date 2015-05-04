# HardcoreData
[![Version](https://img.shields.io/cocoapods/v/HardcoreData.svg?style=flat)](http://cocoadocs.org/docsets/HardcoreData)
[![Platform](https://img.shields.io/cocoapods/p/HardcoreData.svg?style=flat)](http://cocoadocs.org/docsets/HardcoreData)
[![License](https://img.shields.io/cocoapods/l/HardcoreData.svg?style=flat)](http://cocoadocs.org/docsets/HardcoreData)

Simple, elegant, and smart Core Data programming with Swift

## Features
- Supports multiple persistent stores per *data stack*, just the way .xcdatamodeld files are supposed to. HardcoreData will also manage one *data stack* by default, but you can create and manage as many as you need. (see "Setting up")
- Ability to plug-in your own logging framework (or your favorite 3rd-party logger). (see "Logging and error handling")
- Makes it hard to fall into common concurrency mistakes. All Core Data tasks are encapsulated into safer, higher-level abstractions without sacrificing flexibility and customizability. (see "Saving and processing transactions")
- Provides convenient API for common use cases. (see "Fetching and querying")
- Pleasant API designed around Swift’s code elegance and type safety. (see "TL;DR sample codes")

#### TL;DR sample codes

Quick-setup:
```swift
HardcoreData.defaultStack.addSQLiteStore("MyStore.sqlite")
```

Simple transactions:
```swift
HardcoreData.beginAsynchronous { (transaction) -> Void in
    let object = transaction.create(MyEntity)
    object.entityID = 1
    object.name = "test entity"

    transaction.commit { (result) -> Void in
        switch result {
            case .Success(let hasChanges): println("success!")
            case .Failure(let error): println(error)
        }
    }
}
```

Easy fetching:
```swift
let objects = HardcoreData.fetchAll(MyEntity)
```
```swift
let objects = HardcoreData.fetchAll(
    MyEntity.self,
    Where("entityID", isEqualTo: 1),
    SortedBy(.Ascending("entityID"), .Descending("name")),
    CustomizeFetch { (fetchRequest) -> Void in
        fetchRequest.includesPendingChanges = true
    }
)
```

Simple queries:
```swift
let count = HardcoreData.queryValue(
    MyEntity.self,
    Select<Int>(.Count("entityID"))
)
```



## Architecture
For maximum safety and performance, HardcoreData will enforce coding patterns and practices it was designed for. (Don't worry, it's not as scary as it sounds.) But it is advisable to understand the "magic" of HardcoreData before you use it in your apps.

If you are already familiar with the inner workings of CoreData, here is a mapping of `HardcoreData` abstractions:

| *Core Data* | *HardcoreData* |
| --- | --- |
| `NSManagedObjectModel` / `NSPersistentStoreCoordinator`<br />(.xcdatamodeld file) | `DataStack` |
| `NSPersistentStore`<br />("Configuration"s in the .xcdatamodeld file) | `DataStack` configuration<br />(multiple sqlite / in-memory stores per stack) |
| `NSManagedObjectContext` | `BaseDataTransaction` subclasses<br />(`SynchronousDataTransaction`, `AsynchronousDataTransaction`, `DetachedDataTransaction`) |

RestKit and MagicalRecord set up their `NSManagedObjectContext`s this way:
<img src="https://cloud.githubusercontent.com/assets/3029684/6734049/40579660-ce99-11e4-9d38-829877386afb.png" alt="nested contexts" height=271 />

This ensures maximum data integrity between contexts without blocking the main queue. But as <a href="http://floriankugler.com/2013/04/29/concurrent-core-data-stack-performance-shootout/">Florian Kugler's investigation</a> found out, merging contexts is still by far faster than saving nested contexts. HardcoreData's `DataStack` takes the best of both worlds by treating the main `NSManagedObjectContext` as a read-only context, and only allows changes to be made within *transactions*:
<img src="https://cloud.githubusercontent.com/assets/3029684/6734050/4078b642-ce99-11e4-95ea-c0c1d24fbe80.png" alt="nested contexts and merge hybrid" height=212 />

This allows for a butter-smooth main thread, while still taking advantage of safe nested contexts.



## Setting up


## Saving and processing transactions


## Fetching and querying


## Logging and error handling


## Observing changes and notifications (currently in the works)



