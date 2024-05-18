---
layout: post
title: "@Observable vs @AppStorage Conflict: Part 1"
description: Explore the intricate relationship between @Observable and @AppStorage in SwiftUI in this detailed guide. Learn what triggers conflicts between these two features and gain insights into their operational mechanisms. Perfect for developers looking to deepen their understanding of SwiftUI's architecture and improve their coding practices
summary: In Part 1 of our SwiftUI series, "SwiftUI Under the Hood Unpacking the @Observable and @AppStorage Conflict," we delve into the complexities that arise when integrating @Observable with @AppStorage. This article provides a comprehensive analysis of why these conflicts occur, breaking down the technical aspects of both features within SwiftUI's modern framework. By understanding the foundational properties and behaviors of @Observable and @AppStorage, developers will be equipped with the knowledge to identify potential issues in their applications. This guide is an essential read for any SwiftUI developer aiming to leverage the full potential of iOS app development with advanced state management techniques.
comments: true
tags: [swiftui, swift, observation, property wrappers, observable, appstorage]
---

While working on the **Onboarding** feature of a SwiftUI application, my attempt to integrate the [@Observable macro](https://developer.apple.com/documentation/Observation/Observable()) with the property wrapper [@AppStorage](https://developer.apple.com/documentation/swiftui/appstorage) resulted in an error. This article explores the reasons behind the error and provides insights into Swift property wrappers and their interaction with the @Observable macro.

## <u>Problem</u>
Previously, the following [ObservableObject](https://developer.apple.com/documentation/combine/observableobject) pattern worked seamlessly:

```swift
class Defaults: ObservableObject {
	@AppStorage("name") public var name = "johnDoe"
	@AppStorage("age") public var age = 12
}

var defaults = Defaults()
Text(defaults.name)
TextField("janeDoe", text: defaults.$name)
```

Migrating from __ObservableObject__ to __Observable__ with the following code: 

```swift
@Observable class DefaultsAsObservable {
  @AppStorage("name") public var name = "fatbobman"
  @AppStorage("age") public var age = 12
}
```

gives rise to the error message: `Property wrapper cannot be applied to a computed property`. To understand why this happens, we need to review how property wrappers interact with properties. 

## <u>Understanding Properties and Property Access</u>
In Swift, the characteristics and behaviors of the types **class**, **struct** and **enumeration** are modeled by properties. 

* __Stored Properties__
	
	Stored properties encapsulate date directly and represent the most basic form of properties in Swift. For instance, the age, first name and last name of a person can be modeled by stored properties _age_, _firstName_, and _lastName_ in a **Person struct**: 

	```swift
	struct Person {
      var age: Int = 0
      var firstName: String = ""
      var lastName: String = ""
	}
	```

* __Computed Properties__
	
	Computed properties do not store values directly. Instead, they compute their values on demand whenever they are accessed. Computed properties are particularly useful when a property value is derived from other data.

	For example, to get the _fullName_ of a person, we add computed property:

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
	let person = Person(firstName: "John", lastName: "Doe")
	print(person.fullName) // Outputs: "John Doe"

	```
	The computed property **`fullName`** combines _firstName_ and _lastName_ each time it is accessed, providing a real-time, updated full name.

* __Getters and Setters__
	
	The ability to access properties are governed by __getters and setters__. Getters and setters are mechanisms for mediating access to properties, used both in stored and computed properties, where getters retrieve the value of a property while setters modify the value of a property before it is saved.

	To illustrate how getters and setters work with stored properties, let’s ensure that age is never negative:

	```swift 
	struct Person {
	  private var _age: Int = 0
	  var firstName: String = "" 
  	  var lastName: String = ""
			
	  var age: Int {
	  	get { return _age }

	  	set(newValue) { _age = max(0, newValue)} //ensure age is never negative
	  }

      var fullName: String {
      	return "\(firstName) \(_lastName)"
      }
	```
	<details>
	  <summary>Note</summary>
	  the underscore prefix (_) differentiates the stored properties from their computed counterparts to avoid redeclaration errors.
	</details>	
	
	Computed properties can also be declared with both a getter and a setter; thus allowing you to read (get) data from a property and write (set) date to a property: 

	```swift
	struct Person {
	  private var _age: Int = 0
	  var firstName: String = "" 
  	  var lastName: String = ""
			
	  var age: Int {
	  	get { return _age }
	  	
	  	set(newValue) { _age = max(0, newValue)} 
	  }	

      var fullName: String {
      	get { "\(firstName) \(_lastName)" }

      	set {
      		let nameComponents = newValue.split(separator: " ")
      		if nameComponents.count == 2 {
      			firstName = String(nameComponents[0])
      			lastName = String(nameComponents[1])
      		}
      	}
      }	
    }  
	```

## <u>Encapsulating Property Behavior: Property Wrappers</u>
Property wrappers simplify repetitive patterns by encapsulating behaviors around properties. Below is an example that encapsulates the age restriction logic above in a property wrapper: 

 ```swift
 import SwiftUI
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

* __Wrapped Value__

    Property wrappers must expose a [wrappedValue](https://developer.apple.com/documentation/swiftui/binding/wrappedvalue) property. This property is used to represent the __"value"__ being wrapped. When getting or setting a variable annotated with a property wrapper, the compiler automatically uses `wrappedValue` to get or set the value.

    In the example above, **@ClampedAge** is used to enforce age validation based on a specified range via the method signature **@ClampedAge var age: Int = 0** where:
    * the property being wrapped is `age`
    * the initial value of the property being wrapped is 0


		
* **Projected Value**
	
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
	
	Now, when a variable annotated with **@ClampedAge** is accessed with a dollar sign (**$**), the compiler will use projectedValue.

	**Example Usage**:

	```swift
	import SwiftUI

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
	```

	Applying **@ClampedAge** reduces the code used to implement struct `Person`:

	```swift
	import SwiftUI

	struct Person {
    	@ClampedAge var age: Int = 0 
    	var firstName: String = ""
    	var lastName: String = ""
	}
	```

Now that we understand how _Property Wrappers_ work, lets turn our attention to their interaction with the **@Observable macro** 

## <u>Observation Framework: The Basics</u>

In iOS 17, Apple introduced the [Observation framework](https://developer.apple.com/documentation/observation) to modernize the way SwiftUI and other frameworks handle state management. At the core of this framework is the [Observable macro](https://developer.apple.com/documentation/observation/observable()), which simplifies the process of making classes observable.

<u>Key Concepts of the Observation Framework</u>
* __Observation Framework__: includes a new Observable protocol, which:

    * extends Swift's standard protocol for observation (Identifiable)
    * provides the mechanism to notify observers of changes

* __Observable macro__: a macro that generate observation-related code that: 
    * automatically transforms all stored properties in classes annotated with **@Observable** into computed properties with synthesized getters and setters
    * generates a mechanism for mutation tracking and change notification; when a property is modified the change is reported to observers using the synthesized observation mechanism. 

## <u>@Observable’s Conflict with @AppStorage</u>

**@AppStorage** is a property wrapper designed to read/write values to UserDefaults with storage directly on the property. When a property is annotated with **@AppStorage**, the AppStorage property wrapper synthesizes its own getter and setter methods that handle that property’s interaction with UserDefaults.

Since **@Observable macro** works at class level, synthesizing getters and setters for all properties within the class, and **@AppStorage property wrapper** works directly on the stored property level, attempting to use both together leads to a conflict:
<ul class=nested>
	<li><b>@Observable</b>: synthesizes computed properties for all annotated properties in the class</li>
	<li><b>@AppStorage</b>: attempts to override these properties with its own property wrapper mechanism</li>
</ul>

The combination of **@Observable** and **@AppStorage** is not directly supported due to conflicting property access patterns. In a follow up article, I discuss how to overcome this conflict.
