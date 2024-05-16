---
layout: post
title: "Part 1: SwiftUI Under the Hood: Unpacking the @Observable and @AppStorage Conflict"
description: Explore the intricate relationship between @Observable and @AppStorage in SwiftUI in this detailed guide. Learn what triggers conflicts between these two features and gain insights into their operational mechanisms. Perfect for developers looking to deepen their understanding of SwiftUI's architecture and improve their coding practices
summary: In Part 1 of our SwiftUI series, "SwiftUI Under the Hood Unpacking the @Observable and @AppStorage Conflict," we delve into the complexities that arise when integrating @Observable with @AppStorage. This article provides a comprehensive analysis of why these conflicts occur, breaking down the technical aspects of both features within SwiftUI's modern framework. By understanding the foundational properties and behaviors of @Observable and @AppStorage, developers will be equipped with the knowledge to identify potential issues in their applications. This guide is an essential read for any SwiftUI developer aiming to leverage the full potential of iOS app development with advanced state management techniques.
comments: true
tags: [swiftui, swift, observation, property wrappers, observable, appstorage]
---

While working on the `Onboarding` feature of a SwiftUI application, my attempt to integrate the `@Observalbe` macro with the property wrapper `@AppStorage` resulted in the error: `Property wrapper cannot be applied to a computed property`


This article explores the reasons behind this and provides insights into Swift property wrappers and their interaction with `@Observable`.

#### Problem
Previously, the following `ObservableObject` pattern worked seamlessly:

```swift
class Defaults: ObservableObject {
	@AppStorage("name") public var name = "johnDoe"
	@AppStorage("age") public var age = 12
}

var defaults = Defaults()
Text(defaults.name)
TextField("janeDoe", text: defaults.$name)\
```

Migrating from `ObservableObject` to `Observable` with the following code: 

```swift
@Observable class DefaultsAsObservable {
		@AppStorage("name") public var name = "fatbobman"
		@AppStorage("age") public var age = 12
}
```

gives raise to the error message: 

```bash
	Property wrapper cannot be applied to a computed property
``` 

To understand why this happens, we need to dive deeper into how property wrappers interact with properties. 

#### Understanding Properties and Property Access
In Swift, the characteristics and behaviors of the types **class**, **struct** and **enumeration** are modeled by properties. 

1. **Stored Properties**
	Stored properties encapsulate date directly and represent the most basic form of properties in Swift. 

	For instance, the age, first name and last name of a person can be modeled by stored properties `age`, `firstName`, and `lastName` in a `Person` struct: 


	```swift
	struct Person {
      var age: Int = 0
      var firstName: String = ""
      var lastName: String = ""
	}
	```
	<br>

2. **Computed Properties**
	Computed properties do not store values directly. Instead, they compute their values on demand whenever they are accessed. Computed properties are particularly useful when a property value is derived from other data.

	For example, to get the _fullName_ of a person, we add computed propert for it in `Person` struct: 

	```swift
	struct Person {
      var age: Int = 0
      var firstName: String = ""
      var lastName: String = ""

      var fullName: String {
      	return "\(firstName) \(_lastName)"
      }
	}
	//Example Usage:
	let person = Person(firstName: "John" lastName: "Doe")
	print(person.fullName) // Outputs: "John Doe"

	```
	The computed property `fullName` combines `firstName` and `lastName` each time it is accessed, providing a real-time, updated full name.


3. **Getters and Setters**
	The ability to access properties are governed by getters and setters. Getters and setters  are mechanisms for mediating access to properties, used both in stored and computed properties where: 

	- Getters retrieve the value of a property
	- Setters modify the value of a property before it is saved

	To illustrate how getters and setters work with stored properties, let’s ensure that age is never negative:

	```swift 
	struct Person {
	  private var _age: Int = 0
	  var firstName: String = "" 
  	  var lastName: String = ""
			
	  var age: Int {
	  	get { return _age }

		set(newVal) { _age = max(0, newValue)} //ensure age is never negative
	  }

      var fullName: String {
      	return "\(firstName) \(_lastName)"
      }
	```
	> Note: the _underscore prefix_ (`_`)  differentiates the stored properties from their computed counterparts to avoid redeclaration errors. 
	
	Computed properties can also be declared with both a getter and a setter; thus allowing you to read (get) data from a property and write (set) date to a property: 

	```swift
	struct Person {
	  private var _age: Int = 0
	  var firstName: String = "" 
  	  var lastName: String = ""
			
	  var age: Int {
	  	get { return _age }

		set(newVal) { _age = max(0, newValue)} 
	  }	

      var fullName: String {
      	get { "\(firstName) \(_lastName)" }

      	set {
      		let nameComponents = newValue.split(separator: " ")
      		if nameComponents.count = 2 {
      			firstName = String(nameComponents[0])
      			lastName = String(nameComponents[1])
      		}
      	}
      }	  
	```
	In the example above, `fullName` is a computed property with both a getter and a setter. The getter combines `firstName` and `lastName`, while the setter splits the new value and updates `firstName` and `lastName` accordingly. 
	<br>

#### Encapsulating Property Behavior: Property Wrappers
Property wrappers simplify repetitive patterns by encapsulating behaviors around properties. Below is an example that encapsulates the age restriction logic above in a property wrapper: 

```swift
@propertyWrapper
struct ClampedAge {
	private var value: Int
	private var range: ClosedRange<Int> = 0...125

	var wrappedValue: Int {
		get { return value }
		set { value = min(max(range.lowerBound, newValue), range.upperBound)}
	}

	init(wrappedValue: Int) {
			self.value = min(max(range.lowerBound, wrappedValue), range.upperBound)
	}
}
```

<br>

1. **Property Wrappers: Wrapped Value**
	Property wrappers must expose a `wrappedValue` property. This property is used to represent the “value” being wrapped. When getting or setting a variable annotated with a property wrapper the complier automatically uses `wrappedValue` to get or set the value: `@PropertyWrapperName var propertyName: propertyType = someInitialValue`

	In the example above, **`@ClampedAge`** is used to enforce age validation based on a specified range via the method signature `@ClampedAge var age: Int = 0` where: 

	1. the property being wrapped is `age`
	2. the initial value of the property being wrapped is 0
	3. and value can be changed through assignment:

	 	```swift
	 	var Person = Person()
	 	person.age = 10
	 	```


2. **Property Wrappers: Projected Value**
	In addition to **_wrappedValue_**, property wrappers can expose a **_projectedValue_**. This provides an additional way to interact with the wrapped property, offering more metadata or additional functionality. 
	
	In SwiftUI, property wrappers use projectedValue to offer a [Binding](https://developer.apple.com/documentation/swiftui/binding) to the wrapped property, enabling two-way data binding between views and the underlying data.


	**Example**: Extending the previous example to include projectedValue as a Binding:
	
		```swift
			import SwiftUI
		
			@propertyWrapper
			struct ClampedAge {
		    private var value: Int
		    private var range: ClosedRange<Int> = 0...125
			
		    var wrappedValue: Int {
		      get { return value }
		      set { value = min(max(range.lowerBound, newValue), range.upperBound) }
		    }
			
		    var projectedValue: Binding<Int> {
		      Binding(
		        get: { self.wrappedValue },
		        set: { self.wrappedValue = $0 }
		      )
		    }
			
		    init(wrappedValue: Int) {
		      self.value = min(max(range.lowerBound, wrappedValue), range.upperBound)
		    }
			}
		
		```
	
	Now, when a variable annotated with `@ClampedAge` is accessed with a dollar sign (**$**), the compiler will use projectedValue.

	**Example Usage**:

		```swift
			struct Person {
				@ClampedAge var age: Int = 0
			}
			
			struct ContentView: View {
	  		@State private var person = Person()
			
	  		var body: some View {
	    		VStack {
	      		Text("Age: \(person.age)")
	      		Slider(value: $person.$age, in: 0...125, step: 1)
	      			.padding()
	    		}
	  		}
			}
		```

	> Summary: _`wrappedValue`_ is the primary property being wrapped, used for both getting and setting; it must be implemented. While _`projectedValue`_ is an additional property provided by the wrapped accessed via _`$`_ prefix; its implementation is optional 


Applying **`@ClampedAge`** reduces the code used to implement struct `Person`:

```swift
struct Person {
    @ClampedAge var age: Int = 0 
    var firstName: String = ""
    var lastName: String = ""
	  
}
```

Now that we understand how _Property Wrappers_ work, lets turn our attention to their interaction with **`@observable`** 

#### Observation Framework: The Basics

In iOS 17, Apple introduced the [Observation framework](https://developer.apple.com/documentation/observation) to modernize the way SwiftUI and other frameworks handle state management. At the core of this framework is the [**`@Observable macro`**](https://developer.apple.com/documentation/observation/observable()), which simplifies the process of making classes observable.

**Key Concepts of the Observation Framework**

1. **Observable Framework**: includes a new Observable protocol, which:
		- Extends Swift's standard protocol for observation (Identifiable)
		- Provides the mechanism to notify observers of changes

2. **Macros**: are compile-time code transformations that can automatically generate boilerplate code. @Observable is a macro that annotates a class to automatically generate observation-related code

3. **Observation Tracking**: properties in classes marked with @Observable are automatically tracked. When any property is modified, the change is reported to observers using the synthesized observation mechanism.

**Role of the @Observable macro**: 
	- annotates a class to conform to the Observable protocol
	- automatically transforms all stored properties into computed properties with synthesized getters and setters
	- generates a mechanism for mutation tracking and change notification.


**Observation and Identity**:
The [initial pitch for Observation ](https://forums.swift.org/t/pitch-observation/62051) makes it clear that the **Observation framework** inherently implies that an observable entity must have identity. This is important because observation relies on tracking changes to a specific entity over time. The implication of Observation’s reliance on identity for **@Observable** is that when a class is annotated with the `@Observable` macro:

- all stored properties in the class are transformed into computed properties to enable observation
-  the macro synthesizes a mechanism to track and notify changes, ensuring that any mutation to a stored property notifies observers.

these behaviors inherently relies on the fact that [**class**](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/classesandstructures/#Classes-Are-Reference-Types) has reference semantics.

#### `@Observable`’s Conflict with `@AppStorage`

[**`@AppStorage`**](https://developer.apple.com/documentation/swiftui/appstorage) is a property wrapper designed to read/write values to UserDefaults with storage directly on the property. When a property is annotated with `@AppStorage`, the AppStorage property wrapper synthesizes its own getter and setter methods that handle that property’s interaction with UserDefaults. 

Since **`@Observable macro`** works at the parent (class) level, synthesizing getters and setters for all properties within the class, and **`@AppStorage property wrapper`** works directly at the (stored) property level of that same class, attempting to use `@aboservable` and `@AppStorage` together leads to a conflict:

- **`@Observable`** synthesizes computed properties for all annotated properties in the class
- **`@AppStorage`** attempts to override these properties with its own property wrapper mechanism

> Summary: Applying a property wrapper (like @AppStorage) to a computed property synthesized by @Observable creates a conflict because both try to control the getter/setter behavior.

#### Conclusion
The error `` arises because oGiven that `@Observable` automatically transforms all stored properties into computed properties to enable observation, property wrappers cannot be directly applied to those properties. This leads to the error `Property wrapper cannot be applied to a computed property`.

How to overcome this incompartibility will be disscussed in a follow up article.
