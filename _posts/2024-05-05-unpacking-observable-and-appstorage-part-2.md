---
layout: post
title: "@Observable vs @AppStorage Conflict: Part 2"
description: Explore the intricate relationship between @Observable and @AppStorage in SwiftUI in this detailed guide. Learn what triggers conflicts between these two features and gain insights into their operational mechanisms. Perfect for developers looking to deepen their understanding of SwiftUI's architecture and improve their coding practices
summary: In Part 2 of our SwiftUI series, "@Observable vs @AppStorage Conflict," we discuss how to resolve the conflict. This article provides a comprehensive analysis of why these conflicts occur, breaking down the technical aspects of both features within SwiftUI's modern framework. By understanding the foundational properties and behaviors of @Observable and @AppStorage, developers will be equipped with the knowledge to identify potential issues in their applications. This guide is an essential read for any SwiftUI developer aiming to leverage the full potential of iOS app development with advanced state management techniques.
comments: true
tags: [swiftui, swift, observation, property wrappers, observable, appstorage]
---

In a [previous article](https://msuzoagu.github.io/2024/05/02/unpacking-observable-and-appstorage-part-1), we explored the conflict that arises when combining @Observable and @AppStorage. This follow-up provides a solution grounded in an understanding of Observation, identity, reference and structural semantics.

## <u>Important Background: Reference and Structural Semantics</u>

* __Reference Semantics__: refers to objects where multiple variables can refer to the same instance in memory. Any changes made through one reference affect the underlying object and, consequently, all other references to it.

  > **Tip**: Think of reference types (e.g., classes, functions) when considering reference semantics - there are are allocated in shared memory spaces and accessed via pointers. 

* Structural Semantics: in contrast, this pertains to objects where variables hold independent copies of data. Modifications to one variable do not impact others.

  > Value types (e.g., structs, enums, tuples) are examples of structural semantics, emphasizing independent data handling 

## <u>Conflict Overview</u>

**@Observable** converts all stored properties in a class to computed properties to enable detailed observation, leveraging reference semantics for unique change tracking. Conversely, **@AppStorage** is a property wrapper for direct interaction with UserDefaults, relying on its own getter and setter methods and typically aligning with structural semantics.


## <u>The Solution</u>
The key to resolving the conflict lies in ensuring that **@AppStorage** adheres to reference semantics when used within an @Observable class: The [design details section of the original pitch](https://forums.swift.org/t/pitch-observation/62051#detailed-design-8) provides guidance: 

  > Observation of an entity implies that entity has identity. That means that things that are observable are inherently reference types. Structural types that are not rooted in a reference semantic storage do not have a well formed concept of external observation of changes. However, if the structure is a member variable of a reference type, descendant key paths to specific values passing through structural types with a root of a reference type do make sense as being an observable field.

To implement a solution alighed with these principles, first move properties annotated with **@AppStorage** into  a reference type:

```swift
class Storage {
    @AppStorage("name") public var name = "johnDoe"
    @AppStorage("age") public var age = 12
    
    init() {}
}
``` 

Next the reference type, `Storage`, is integrated into an observable class (`UserSettings`) and made a member variable of said class: 

```swift
import SwiftUI
@Observable class UserSettings {
    class Storage {
        @AppStorage("name") public var name = "johnDoe"
        @AppStorage("age") public var age = 12
        
        init() {}
    }
    
    private let storage = Storage()
}
```

Finally the property observer `didSet` is added for each variable we want to observe and update: 

```swift
import SwiftUI
@Observable class UserSettings {
    class Storage {
        @AppStorage("name") public var name = "johnDoe"
        @AppStorage("age") public var age = 12
        
        init() {}
    }
    
    private let storage = Storage()
    
    public var name: String {
      didSet {
        storage.name = name
      }
    }
    
    public var age: Int {
      didSet {
        storage.age = age
      }
    }
}
```

#### Code Breakdown:
Each property in `UserSettings` is backed by a corresponding property in the `Storage` class. 
 
* __Direct Access and Persistence__: when a property (e.g. _name_) in `UserSettings` is set, its `didSet` observe updates the corresponding value in the `Storage` instance, which immediately saves it in `UserDefaults` via the `AppStorage` wrapper. 

* __Initialization and Synchronization__: when `UserSettings` is initialized, it sets its properties from the corresponding values in `Stroage`, which reads these values from `UserDefaults`. This ensures that a user’s settings are populated with the previously saved values when the app starts. 
With `@AppStorage` properties encapsulated in a distinct reference type (`Storage`), `UserSettings` is able to indirectly access  `@AppStorage` properties through `Storage`, enabling observation via `@Observable` while avoiding direct application of `@AppStorage`. 

This approch effectively organizes data handling and observing changes in `UserSettings` by leveraging the strengths of both `@AppStorage` for persistent storage and `@Observable` for observing and responding to changes.

