# Develop in Swift Data Collections

## Unit 1 Tables and Persistence

- In this unit, you'll learn three important sets of techniques in app development, which—taken together—will enable you to build more much complex apps.
  - First, you'll learn how to use tables to display lists of information, and to use the master-detail design pattern to allow users to interact with the information.
  - Second, you'll learn how to organize the files, structures, and classes in your apps, making it easier for you (or other developers) to work with them in the future.
  - Third, you'll learn about an approach for saving data to the device, so that any information the user enters or changes in your app will still be there the next time they open the app.
  - By the end of this unit, you'll be comfortable building many useful apps that display all kinds of information and that allow users to enter, edit, and save in-app information.

### Lesson 1.1 Protocols

- A protocol is a set of rules or procedures that define how things are done. Computers communicate with each other using protocols like HTTP (Hyper Text Transfer Protocol) and TCP/IP (Transmission Control Protocol/Internet Protocol).
  - HTTP is a standard that defines how website data is communicated between two computers.
  - TCP/IP is a communications standard that defines how computers find and send data to each other.
- In programming, a protocol defines the properties or methods that an object must have to complete a task. For example, the `Equatable` protocol says that a type must define a `==` method to check whether two instances are equal to each other.
- In this lesson, you'll learn what protocols are, when to use them, and how to write your own. You'll also learn about delegation, a pattern for enabling objects to communicate with each other.
- A protocol defines a blueprint of methods, properties, and other requirements that suit a particular task or piece of functionality. The protocol can then be adopted by another type, which provides the actual implementation for those requirements. Any type that satisfies the requirements of a protocol conforms to that protocol.
- The Swift standard library defines many protocols that you'll use when building apps. In the following sections, you'll look at four of them:
  - `CustomStringConvertible`, which allows you to control how your custom objects are printed to the console;
  - `Equatable`, which allows you to define how instances of the same type are equal to each other;
  - `Comparable`, which allows you to define how instances of the same type are sorted; and
  - `Codable`, which allows you to encode your type's properties as key/value pairs that can then be saved between app launches.
- When you adopt a protocol in Swift, you're promising to implement all the methods required by that protocol. The compiler will check that everything's in order and won't build your program if anything is missing.
- Why might you use protocols when building an app?
  - To share attributes and functionality across different types
  - To make custom types work well with system or debugging functionality
  - To enforce similarity and provide type functionality
  - To define events but delegate implementation to an instance of another type

#### Printing Information with `CustomStringConvertible`

- You can begin to understand the role of a protocol by considering how objects are printed to the console using the `print()` function. As you've learned, running the print() function on a variable will write the textual representation of the object to the console. You can print String values, Int values, and Bool values with predictable results.
- But what if you define a Shoe class?

  - ```swift
      class Shoe {
        let color: String
        let size: Int
        let hasLaces: Bool
       
        init(color: String, size: Int, hasLaces: Bool) {
          self.color = color
          self.size = size
          self.hasLaces = hasLaces
        }
      }
       
      let myShoe = Shoe(color: "Black", size: 12, hasLaces: true)
      let yourShoe = Shoe(color: "Red", size: 8, hasLaces: false)
      print(myShoe)
      print(yourShoe)
      Console Output:
      Shoe
      Shoe
    ```

- (Your print statement is likely prefixed by something like \_lldb_expr_1, which you can ignore.)
- As you can see, this print statement isn't at all helpful. Swift defines a protocol called CustomStringConvertible to address this kind of situation.
- CustomStringConvertible has one required computed property, description, which returns a String representation of the instance. If you adopt the protocol, you control exactly how your custom types are represented as printable String values.

  - First, adopt the protocol by adding a colon and the name of the protocol you want to adopt:

    - ```swift
        class Shoe: CustomStringConvertible {
          let color: String
          let size: Int
          let hasLaces: Bool
        }
      ```

  - Second, conform to the protocol by adding the required functions or variables. In this example, you'll add the description computed property. If you don't complete this step, the compiler will display an error that says you're not conforming to the protocol.
  - In the Strings lesson, you learned how to use string interpolation to interweave constants or variables into a String value. The description property in the following code uses string interpolation to create a String that includes each property of the Shoe instance.

    - ```swift
        class Shoe: CustomStringConvertible {
            let color: String
            let size: Int
            let hasLaces: Bool
         
            init(color: String, size: Int, hasLaces: Bool) {
                self.color = color
                self.size = size
                self.hasLaces = hasLaces
            }
         
            var description: String {
                return "Shoe(color: \(color), size: \(size), hasLaces: \(hasLaces))"
            }
        }
        let myShoe = Shoe(color: "Black", size: 12, hasLaces: true)
        let yourShoe = Shoe(color: "Red", size: 8, hasLaces: false)
        print(myShoe)
        print(yourShoe)

        Console Output:
        Shoe(color: Black, size: 12, hasLaces: true)
        Shoe(color: Red, size: 8, hasLaces: false)
      ```

  - Note that this code prints out what looks like an initializer for each object. That's a common practice, because it enables you to see each property on the object in a familiar, concise format. However, you can print anything you want:

    - ```swift
        var description: String {
            let doesOrDoesNot = hasLaces ? "does" : "does not"
            return "This shoe is \(color), size \(size), and \(doesOrDoesNot) have laces."
        }
         
        let myShoe = Shoe(color: "Black", size: 12, hasLaces: true)
        let yourShoe = Shoe(color: "Red", size: 8, hasLaces: false)
        print(myShoe)
        print(yourShoe)

        Console Output:
        This shoe is Black, size 12, and does have laces.
        This shoe is Red, size 8, and does not have laces.
      ```

#### Comparing Information with `Equatable`

- Imagine your team is building an employee directory app for your company. You're tasked with building the data model to represent all the employees, their job titles, and their phone numbers.

  - ```swift
      struct Employee {
        var firstName: String
        var lastName: String
        var jobTitle: String
        var phoneNumber: String
      }
       
      struct Company {
        var name: String
        var employees: [Employee]
      }
    ```

- While building the app, you're asked to build a feature that allows any employee to edit their own information and prevents them from editing anyone else's information. You'll accomplish this by displaying an Edit button when the user navigates to their own detail screen.
- But how can you make sure it's the right employee? Each time the current employee navigates to an employee detail screen, you'll need to compare that employee to the employee whose information is displayed.
- Imagine you have access to the current employee with a `Session.currentEmployee` variable, which returns an Employee instance matching the current user. The view controller in charge of displaying the employee detail screen has an employee property. When the app loads a new employee detail screen, you'll need to check whether Session.currentEmployee and employee are equal. If so, you'll enable the Edit button. If not, you'll hide the button.
- You might consider writing something like the following:

  - ```swift
      let currentEmployee = Session.currentEmployee
      // Employee(firstName: "Daren", lastName: "Estrada", jobTitle: "Product Manager", phoneNumber: "415-555-0692")

      let selectedEmployee = Employee(firstName: "James", lastName: "Kittel", jobTitle: "Marketing Director", phoneNumber: "415-555-9293")
       
      if currentEmployee == selectedEmployee {
        // Enable "Edit" button
      }
    ```

- The code above is a good thought. The == operator checks whether two values are equal. But because Employee is a custom type, you must tell Swift exactly how to compare two instances for equality. You'll do this by adopting the Equatable protocol.
- The Equatable protocol requires you to provide an implementation for the == operator on your custom type with a static == function that takes lhs (left-hand side) and rhs (right-hand side) parameters and returns a Bool that says whether the two values are equal:

  - ```swift
      struct Employee: Equatable {
        var firstName: String
        var lastName: String
        var jobTitle: String
        var phoneNumber: String
       
        static func ==(lhs: Employee, rhs: Employee) -> Bool {
          // Logic that determines whether the value on the left-hand side and right-hand side are equal
        }
       
      }
    ```

- Consider the example above with two employees. How might you determine whether the two employees are equal? The following code returns true if both the firstName and lastName values are the same for both employees:

  - ```swift
      struct Employee: Equatable {
        var firstName: String
        var lastName: String
        var jobTitle: String
        var phoneNumber: String
       
        static func ==(lhs: Employee, rhs: Employee) -> Bool {
          return lhs.firstName == rhs.firstName && lhs.lastName == rhs.lastName
        }
      }
    ```

- That wasn't a bad start at implementing an equality check. But it breaks down quickly. Consider two different employees with the same name:

  - ```swift
      let currentEmployee = Employee(firstName: "James", lastName: "Kittel", jobTitle: "Industrial Designer", phoneNumber: "415-555-7766")
      let selectedEmployee = Employee(firstName: "James", lastName: "Kittel", jobTitle: "Marketing Director", phoneNumber: "415-555-9293")
       
      if currentEmployee == selectedEmployee {
        // Enable "Edit" button
      }
    ```

- This code returns true, so the employees would be able to edit each other's information in the directory.
- How can you design a better check to see whether two Employee instances are equal? You can update the == method to check for each property:

  - ```swift
      struct Employee: Equatable {
        var firstName: String
        var lastName: String
        var jobTitle: String
        var phoneNumber: String
       
        static func ==(lhs: Employee, rhs: Employee) -> Bool {
            return lhs.firstName == rhs.firstName && lhs.lastName == rhs.lastName && lhs.jobTitle == rhs.jobTitle && lhs.phoneNumber == rhs.phoneNumber
        }
      }
    ```

- If you compare the first names, last names, job titles, and phone numbers, you'll get a more reliable result. As you grow as a programmer, you'll start to recognize these types of edge cases, and you'll learn how to account for them in your code.
- Sometimes, the requirements of a protocol can be autogenerated for you. For example, if you specify that one of your types adopts Equatable and don't write your own == method, the Swift compiler will autogenerate an implementation that compares all the instance properties, like the one you just saw. So if all your == method is going to do is compare all the instance properties, you don't have to write it. The compiler will take care of it for you.
- To use autogeneration, your type must be a `struct` or `enum` — classes do not support autogeneration. Also, each property's type must also conform to Equatable.

  - ```swift
      struct Employee: Equatable {
        var firstName: String
        var lastName: String
        var jobTitle: String
        var phoneNumber: String
      }
    ```

#### Sorting Information with `Comparable`

- Now imagine you're tasked with displaying a list of all the employees, sorted alphabetically by last name. Your manager gives you a sample list of five employees so you can experiment with writing code that will sort the list in the correct order.
- You start out with the following:

  - ```swift
      let employee1 = Employee(firstName: "Ben", lastName: "Stott", jobTitle: "Front Desk", phoneNumber: "415-555-7767")
      let employee2 = Employee(firstName: "Vera", lastName: "Carr", jobTitle: "CEO", phoneNumber: "415-555-7768")
      let employee3 = Employee(firstName: "Glenn", lastName: "Parker", jobTitle: "Senior Manager", phoneNumber: "415-555-7770")
      let employee4 = Employee(firstName: "Stella", lastName: "Lee", jobTitle: "Accountant", phoneNumber: "415-555-7771")
      let employee5 = Employee(firstName: "Daren", lastName: "Estrada", jobTitle: "Sales Lead", phoneNumber: "415-555-7772")
       
      let employees = [employee1, employee2, employee3, employee4, employee5]
    ```

- Note that the list above isn't in any particular order.
- Swift provides a protocol named `Comparable`, that allows you to define how to sort objects using the <, <=, >, or >= operators.
- `Comparable` has two requirements: It requires that the type also adopt the Equatable protocol, and it requires the type to implement the < operator, which returns a Bool for whether the left-hand value is less than the right-hand value.
- In this case, you want to sort the employees alphabetically by last name. The String type is itself Comparable and uses the < operator to sort strings alphabetically, so you can implement the < function on Employee to return true if the last name of the left-hand value comes before the last name of the right-hand value alphabetically:

  - ```swift
      struct Employee: Equatable, Comparable {
        var firstName: String
        var lastName: String
        var jobTitle: String
        var phoneNumber: String
       
        static func < (lhs: Employee, rhs: Employee) -> Bool {
            return lhs.lastName < rhs.lastName
        }
      }
    ```

- You can then use the `sorted(by:)` function to return the array of employees sorted by last name:

  - ```swift
      let employees = [employee1, employee2, employee3, employee4, employee5]
       
      let sortedEmployees = employees.sorted(by: <)
       
      for employee in sortedEmployees {
        print(employee)
      }

      Console Output:
      Employee(firstName: "Vera", lastName: "Carr", jobTitle: "CEO", phoneNumber: "415-555-7768")
      Employee(firstName: "Daren", lastName: "Estrada", jobTitle: "Sales Lead", phoneNumber: "415-555-7772")
      Employee(firstName: "Stella", lastName: "Lee", jobTitle: "Accountant", phoneNumber: "415-555-7771")
      Employee(firstName: "Glenn", lastName: "Parker", jobTitle: "Senior Manager", phoneNumber: "415-555-7770")
      Employee(firstName: "Ben", lastName: "Stott", jobTitle: "Front Desk", phoneNumber: "415-555-7767")


      let sortedEmployees2 = employees.sorted(by:>)
       
      for employee in sortedEmployees2 {
        print(employee)
      }
      Console Output:
      Employee(firstName: "Ben", lastName: "Stott", jobTitle: "Front Desk", phoneNumber: "415-555-7767")
      Employee(firstName: "Glenn", lastName: "Parker", jobTitle: "Senior Manager", phoneNumber: "415-555-7770")
      Employee(firstName: "Stella", lastName: "Lee", jobTitle: "Accountant", phoneNumber: "415-555-7771")
      Employee(firstName: "Daren", lastName: "Estrada", jobTitle: "Sales Lead", phoneNumber: "415-555-7772")
      Employee(firstName: "Vera", lastName: "Carr", jobTitle: "CEO", phoneNumber: "415-555-7768")
    ```

- Swift is smart. It can use both the == operator and the < operator that you defined for the Equatable and Comparable protocols to provide functionality for the !=, <=, >, and >= operators as well.

#### Encoding and Decoding Objects with `Codable`

- Many apps save user input so that the data still exists the next time the user opens the app. To save user data, the values that live in memory must to be encoded to a form of data that can be written to a file. The Codable protocol makes this simple by creating key/value pairs from your object's property names and values that can then be used by an Encoder or Decoder object.
- Most Swift types that you use from the standard library already conform to Codable. If all of your custom type's properties conform to Codable, then all you have to do is add Codable to the type declaration, and the Swift compiler will autogenerate the necessary implementation.

  - ```swift
      struct Employee: Equatable, Comparable, Codable {
          var firstName: String
          var lastName: String
          var jobTitle: String
          var phoneNumber: String
       
          static func < (lhs: Employee, rhs: Employee) -> Bool {
              return lhs.lastName < rhs.lastName
          }
      }
    ```

- This lets an Encoder or Decoder object know that your type has all of the information it needs to be able to encode your object to or decode it from a certain data format.
- Take `JSONEncoder` as an example. The JSON data format, commonly used when working with web services, is essentially a list of key/value pairs that represent information. A JSONEncoder can convert an object conforming to Codable to JSON, which can then easily be encoded as Data or displayed as a String showing the list of key/value pairs.
- The `encode(_:)` method on JSONEncoder is a throwing function — a special type of Swift function that can return specific types of errors. You'll see the `try?` syntax, which allows the function to return an optional value, in the following sample code. If there's no error, the optional will hold the expected value; if there is an error, the optional will be nil. You'll learn more about try? in another lesson. If you get an error saying "Use of unresolved identifier JSONEncoder," make sure you import `Foundation`, the framework in which JSONEncoder is defined.

  - ```swift
      import Foundation
       
      let ben = Employee(firstName: "Ben", lastName: "Stott", jobTitle: "Front Desk", phoneNumber: "415-555-7767")
       
      let jsonEncoder = JSONEncoder()
      if let jsonData = try? jsonEncoder.encode(ben),
          let jsonString = String(data: jsonData, encoding: .utf8) {
          print(jsonString)
      }
       
      {"firstName":"Ben","lastName":"Stott","jobTitle":"Front Desk","phoneNumber":"415-555-7767"}
    ```

- The string printed to the console gives you a glimpse of how your object was converted into a different format but still represents the same information. By using Codable, you can easily convert your app's information to and from a variety of formats. The Codable protocol will come in handy in future lessons when you are saving user data or working with web services.

#### Creating a Protocol

- You've learned about four protocols defined in the Swift standard library, but you can also write your own protocols.
- Remember how to define structures, classes, and enumerations? You define a protocol in a very similar way. Use the protocol keyword followed by the name you want to use, and then define the requirements in a set of curly braces.
- When requiring a property, you must define whether the property is read-only or read/write. Read-only means you can get the variable, but you can't set it. Read/write means you can both get and set the value. If a property is read-only, you can implement it using a computed property. If it's read/write, it should be a regular property.
- When requiring a method, you need to specify the name, parameters, and return type of the method.
- The following code defines a FullyNamed protocol that requires a fullName property and a sayFullName() method. You can see that it looks similar to a type definition:

  - ```swift
      protocol FullyNamed {
        var fullName: String { get }
        // settable property
        // var fullName: String { get set }

        func sayFullName()
      }
    ```

- Just like with the Swift protocols earlier in this lesson, your types can then adopt the protocol by adding a colon and appending the name of the protocol.

  - ```swift
      struct Person: FullyNamed {
        var firstName: String
        var lastName: String
      }
    ```

- The compiler recognizes that the Person struct has adopted the protocol, but also recognizes that the struct doesn't yet meet the protocol's requirements. If you adopt a protocol but don't meet its requirements, you won't be able to build or run your code until you address the error.
- Notice how simple it is to conform to the protocol in the following code. The Person struct is updated with a fullyNamed computed property and a sayFullName() function that prints the full name to the console.

  - ```swift
      struct Person: FullyNamed {
        var firstName: String
        var lastName: String
       
        var fullName: String {
          return "\(firstName) \(lastName)"
        }
       
        func sayFullName() {
          print(fullName)
        }
      }
    ```

- You may have noticed that the syntax for adopting a protocol is similar to the syntax for declaring a subclass. That's not coincidental. Protocols and class inheritance are two ways to adopt a shared set of properties and functionality. In fact, you can use a protocol to provide a default implementation, just as a class can inherit an implementation from a superclass. You'll learn more about protocols and default implementations as you learn more about Swift.

#### Delegation

- Delegation is a design pattern, or common practice, that enables a class or structure to hand off, or delegate, some of its responsibilities to an instance of another type.
- It can help to think about delegation in the real world. If you delegate a task, you assign it to another person and expect them to handle it. For example, a manager may delegate a presentation to one of her employees, who would then be expected to prepare and deliver the presentation. Or a parent may delegate vacuuming the floor to one of their children, who would then handle the chore.
- The important thing to understand is that there are two sides: the one who's delegating the task, and the delegate whose job it is to actually do the task.
- This relationship is the same in programming. You might have an object that needs a task done, but it might not be the object that actually implements the code to make it happen. Types that delegate implementation to other types typically do so by defining a protocol. The protocol defines the responsibilities that can be delegated, and the delegate adopts the protocol to execute the actual task.
- The delegate pattern is especially important for working with frameworks which have objects that define built-in functionality but require you to provide behavior specific to your app. UIKit and other iOS frameworks use delegation extensively. They provide objects such as `UIApplication`, `UITextField`, and `UITableView`, which use delegation to customize their behavior. For example, a `UIApplicationDelegate` can define how an app responds to remote notifications, a `UITextFieldDelegate` can validate text input, and a `UITableViewDelegate` can perform actions when a row is tapped.
- You'll work more with the delegate pattern in future lessons.
- Protocols are one of the more advanced Swift topics you'll use in this course. Don't be intimidated. You'll learn to master protocols as you use them while working your way through the course.

#### Lab

- App Exercise - Heart Rate Delegate
- These exercises reinforce Swift concepts in the context of a fitness tracking app.
- Your fitness tracking app will likely have a class dedicated to receiving information from the fitness tracking hardware. The HeartRateReceiver class below represents a very simplified example of what this may look like with monitoring heart rate. The function startHeartRateMonitoringExample will generate random heart rates and assign them to currentHR, simulating how an instance of HeartRateReceiver may pick up on new heart rate readings at specific intervals.
- HeartRateViewController below is a view controller that will present the heart rate information to the user. Throughout the exercises below you'll use the delegate pattern to pass information from an instance of HeartRateReceiver to the view controller so that anytime new information is obtained it is presented to the user.

  - ```swift
      class HeartRateReceiver {
          var currentHR: Int? {
              didSet {
                  if let currentHR = currentHR {
                      print("The most recent heart rate reading is \(currentHR).")
                  } else {
                      print("Looks like we can't pick up a heart rate.")
                  }
              }
          }

          func startHeartRateMonitoringExample() {
              for _ in 1...10 {
                  let randomHR = 60 + Int(arc4random_uniform(UInt32(15)))
                  currentHR = randomHR
                  Thread.sleep(forTimeInterval: 2)
              }
          }
      }

      class HeartRateViewController: UIViewController {
          var heartRateLabel: UILabel = UILabel()
      }
    ```

- First, create an instance of HeartRateReceiver and call startHeartRateMonitoringExample. Notice that every two seconds currentHR get set and prints the new heart rate reading to the console.

  - ```swift
      let receiver1 = HeartRateReceiver()
      receiver1.startHeartRateMonitoringExample()

      console:
      The most recent heart rate reading is 66.
      The most recent heart rate reading is 72.
      The most recent heart rate reading is 68.
      The most recent heart rate reading is 70.
      The most recent heart rate reading is 70.
      The most recent heart rate reading is 71.
      The most recent heart rate reading is 71.
      The most recent heart rate reading is 60.
      The most recent heart rate reading is 70.
      The most recent heart rate reading is 62.
    ```

- In a real app, printing to the console does not show information to the user. You need a way of passing information from the `HeartRateReceiver` to the `HeartRateViewController`. To do this, create a protocol called `HeartRateReceiverDelegate` that requires a method `heartRateUpdated(to bpm:)` where `bpm` is of type `Int` and represents the new rate as _beats per minute_. Since playgrounds read from top to bottom and the two previously declared classes will need to use this protocol, you'll need to declare this protocol above the declaration of `HeartRateReceiver`.

  - ```swift
      protocol HeartRateReceiverDelegate {
          func heartRateUpdated(to bpm: Int)
      }
    ```

- Now make `HeartRateViewController` adopt the protocol you've just created. Inside the body of the required method you should set the text of `heartRateLabel` and print "The user has been shown a heart rate of <INSERT HEART RATE HERE>."

  - ```swift
      protocol HeartRateReceiverDelegate {
          func heartRateUpdated(to bpm: Int)
      }
      class HeartRateViewController: UIViewController, HeartRateReceiverDelegate {
          var heartRateLabel: UILabel = UILabel()

          func heartRateUpdated(to bpm: Int) {
              heartRateLabel.text = "The user has been shown a heart rate of \(bpm)"
              print("The user has been shown a heart rate of \(bpm)")
          }
      }
    ```

- Now add a property called `delegate` to `HeartRateReceiver` that is of type `HeartRateReceiverDelegate?`. In the `didSet` of `currentHR` where `currentHR` is successfully unwrapped, call `heartRateUpdated(to bpm:)` on the `delegate` property.

  - ```swift
      class HeartRateReceiver {
          var currentHR: Int? {
              didSet {
                  if let currentHR = currentHR {
                      print("The most recent heart rate reading is \(currentHR).")
                      delegate?.heartRateUpdated(to: currentHR)
                  } else {
                      print("Looks like we can't pick up a heart rate.")
                  }
              }
          }
          var delegate: HeartRateReceiverDelegate?

          ...
      }
    ```

- Finally, return to the line of code just after you initialized an instance of `HeartRateReceiver`. Initialize an instance of `HeartRateViewController`. Then, set the `delegate` property of your instance of `HeartRateReceiver` to be the instance of `HeartRateViewController` that you just created. Wait for your code to compile and observe what is printed to the console. Every time that `currentHR` gets set, you should see both a printout of the most recent heart rate, and the print statement stating that the heart rate was shown to the user.

  - ```swift
      let receiver2 = HeartRateReceiver()
      let hrController = HeartRateViewController()
      receiver2.delegate = hrController
      receiver2.startHeartRateMonitoringExample()

      Console:
      The most recent heart rate reading is 61.
      The user has been shown a heart rate of 61
      The most recent heart rate reading is 61.
      The user has been shown a heart rate of 61
      The most recent heart rate reading is 68.
      The user has been shown a heart rate of 68
      The most recent heart rate reading is 64.
      The user has been shown a heart rate of 64
      The most recent heart rate reading is 64.
      The user has been shown a heart rate of 64
      The most recent heart rate reading is 63.
      The user has been shown a heart rate of 63
      The most recent heart rate reading is 64.
      The user has been shown a heart rate of 64
      The most recent heart rate reading is 67.
      The user has been shown a heart rate of 67
      The most recent heart rate reading is 62.
      The user has been shown a heart rate of 62
      The most recent heart rate reading is 70.
      The user has been shown a heart rate of 70
    ```

- The final

  - ```swift
      class HeartRateReceiver {
          var currentHR: Int? {
              didSet {
                  if let currentHR = currentHR {
                      print("The most recent heart rate reading is \(currentHR).")
                      delegate?.heartRateUpdated(to: currentHR)
                  } else {
                      print("Looks like we can't pick up a heart rate.")
                  }
              }
          }

          var delegate: HeartRateReceiverDelegate?

          func startHeartRateMonitoringExample() {
              for _ in 1...10 {
                  let randomHR = 60 + Int(arc4random_uniform(UInt32(15)))
                  currentHR = randomHR
                  Thread.sleep(forTimeInterval: 2)
              }
          }
      }

      class HeartRateViewController: UIViewController, HeartRateReceiverDelegate {
          var heartRateLabel: UILabel = UILabel()

          func heartRateUpdated(to bpm: Int) {
              heartRateLabel.text = "The user has been shown a heart rate of \(bpm)"
              print("The user has been shown a heart rate of \(bpm)")
          }
      }
      //let receiver1 = HeartRateReceiver()
      //receiver1.startHeartRateMonitoringExample()

      protocol HeartRateReceiverDelegate {
          func heartRateUpdated(to bpm: Int)
      }

      let receiver2 = HeartRateReceiver()
      let hrController = HeartRateViewController()
      receiver2.delegate = hrController
      receiver2.startHeartRateMonitoringExample()
    ```

### Lesson 1.2 App Anatomy and Life Cycle

- In this lesson, you'll learn more about the different life cycle states and the delegate hooks for executing logic as the app moves through each state.
- Most iOS apps run when the user opens them, and they stop running when the user returns to the Home screen, uses the app switcher, or otherwise switches to a different app.
- As a programmer, you have delegate methods, or hooks where you can execute code, at any of these events in your app's life cycle. For example, when the app launches, you may want to trigger a network call to fetch new data. When the app closes, you may want to save your user's progress. App and scene delegates also ensure your app works properly with iOS, multiple instances of your app, and with other iOS apps.
- In this lesson, you'll examine the AppDelegate.swift and SceneDelegate.swift files that Xcode creates with every new project. This will help you learn the most common delegate methods that are called as the app transitions through its life cycle, such as when it moves from the foreground to the background.
- But before digging into the code, you should understand the different stages of the app life cycle.
- Before digging into the code, you should understand the different stages of the app life cycle.

  | App State | Description |
  | --------- | ----------- |
  | Not running | The app has not been launched or has been terminated, either by the user or by the system. |
  | Inactive | The app is running in the foreground but isn't receiving touch events. (It may be executing other code, though.) An app usually stays inactive only briefly as it transitions to a different state. |
  | Active | The app is running in the foreground and receiving events. In the active state, an app has no special restrictions placed on it and should be responsive to the user. |
  | Background | The app is executing code but is not visible onscreen. When the user quits an app, the system moves the app to the background state briefly before suspending it. At other times, the system may launch the app into the background (or wake up a suspended app) and give it time to handle specific tasks. For example, the systemm may wake an app so that it can process background downloads, certain types of location events, remote notifications, and other events. An app in the background state should do as little work as possible. |
  | Suspended | The app is in memory but isn't executing code. The system will suspend apps that are in the background may purge suspended apps at any time, without waking them up, to make room for other apps. |

  - <img src="./resources/app_delegate.png" alt="App Delegate" width="500" />

#### Break Down the App Delegate

- Create a new project using the iOS App template and name it "AppLifeCycle." When creating the project, make sure the interface option is set to "Storyboard."
- In the Project navigator, open AppDelegate, which defines the AppDelegate class. The AppDelegate class conforms to the UIApplicationDelegate protocol, which defines methods that serve as hooks into the different events in the app's life cycle.
- Take a look at the methods included in the app delegate. Read through the comments to learn about what the methods do. Prior to iOS 13, the app delegate had a much larger role in the app life cycle. iOS 13 introduced the ability for apps to have multiple instances running on iPadOS, each of which is managed by a UISceneDelegate.
- While it's possible to forgo using UISceneDelegate, falling back to using UIApplicationDelegate as it was prior to iOS 13, it's best to conform to any new design patterns that have been introduced. By doing so you'll be in a better position to handle any future features, devices, or deprecations — or simply to extend your app to support multiple scenes on iPad when you're ready. In this course, you'll adhere to scene - based life cycle semantics, but all your apps will only have a single scene — you won't do any work to explicitly support multiple scenes.

- **Did Finish Launching**

  - When your app has finished launching, the first method will be called, providing your first opportunity to run your own code in the app. This capability isn't duplicated in a scene delegate, so you'll need to use your app delegate to perform actions when your app is first launched.

    - ```swift
        func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
            // Override point for customization after application launch.

            return true
        }
      ```

  - The app delegate also manages scenes as they're connected and disconnected.

- **Configuration For Connecting a Scene Session**

  - The next function is called when a new scene session is being created. By default Xcode generates your project with an Info.plist that has the information necessary to create a default scene configuration, named “Default Configuration”, as evident in this implementation. This all you need to get started – supporting various scene configurations is an advanced topic that won't be covered here.

    - ```swift
        func application(_ application: UIApplication, configurationForConnecting connectingSceneSession: UISceneSession, options: UIScene.ConnectionOptions) ->
          UISceneConfiguration {
            // Called when a new scene session is being created.
            // Use this method to select a configuration to create the new scene with.

            return UISceneConfiguration(name: "Default Configuration", sessionRole: connectingSceneSession.role)
        }
      ```

- **Did Discard Scene Sessions**

  - When a user discards an instance of your app on iPad, this method is called with information about that scene. It's possible you'll receive multiple scene sessions if the user discards multiple instances at the same time or if the action was taken while your app was suspended. It's up to you to decide what should happen in your app when this event occurs – keeping in mind what your users might reasonably expect to happen.

    - ```swift
        func application(_ application: UIApplication, didDiscardSceneSessions sceneSessions: Set<UISceneSession>) {
            // Called when the user discards a scene session.
            // If any sessions were discarded while the application was not running, this will be called shortly after didFinishLaunchingWithOptions.
            // Use this method to release any resources that were specific to the discarded scenes, as they will not return.
        }
      ```

  - There are other capabilities unique to the app delegate, but most basic lifecycle events can be handled by a scene delegate.

#### Break Down the Scene Delegate

- In the Project navigator, open SceneDelegate. Your scene delegate is responsible for handling UI-level components during your app's life cycle. These methods will be called for all scenes that your app has created. You'll notice that each method receives a UIScene instance — this is the scene that the event occurred for.

  - <img src="./resources/scene_delegate.png" alt="Scene Delegate" width="500" />

- Read through the provided methods and their comments to learn about what the methods do. As always, you can Option-click a method's signature to see more information.

- **Will Connect To**

  - This method is called with a scene instance that is being connected to your app. This provides you with an opportunity to perform any necessary steps to prepare for the additional scene such as fetching data from your local persistence store or opening network connections.

    - ```swift
        func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
            // Use this method to optionally configure and attach the UIWindow `window` to the provided UIWindowScene `scene`.
            // If using a storyboard, the `window` property will automatically be initialized and attached to the scene.
            // This delegate does not imply the connecting scene or session are new (see `application:configurationForConnectingSceneSession` instead).

            guard let _ = (scene as? UIWindowScene) else { return }
        }
      ```

- **Scene Did Disconnect**

  - This method, called when the scene has been removed from your app, is the best place to clean up and release any resources or to save files that were necessary for the scene. UIKit disconnects scenes when a user explicitly closes them in the app switcher and can disconnect scenes in the background state on behalf of the system to free up resources for other active applications. It's not as commonly overridden as sceneDidEnterBackground(_:) below. A scene is not guaranteed to be disconnected when the user switches to another app.

    - ```swift
        func sceneDidDisconnect(_ scene: UIScene) {
            // Called as the scene is being released by the system.
            // This occurs shortly after the scene enters the background, or when its session is discarded.
            // Release any resources associated with this scene that can be re-created the next time the scene connects.
            // The scene may re-connect later, as its session was not necessarily discarded (see `application:didDiscardSceneSessions` instead).
        }
      ```

- **Scene Did Become Active**

  - This method is called to let your scene know that it moved from the foreground inactive state to the foreground active state, which can occur when a user switches to your scene. Scenes can also return to the foreground active state if the user chooses to ignore an interruption, such as an incoming phone call or system alert, that temporarily sent the app into the inactive state.

    - ```swift
        func sceneDidBecomeActive(_ scene: UIScene) {
            // Called when the scene has moved from an inactive state to an active state.
            // Use this method to restart any tasks that were paused (or not yet started) when the scene was inactive.
        }
      ```

- **Scene Will Resign Active**

  - The next method is called when the scene is about to leave the foreground active state. This event can happen when the user has closed the scene or quit the app, but it's also called when the app is interrupted temporarily by a phone call or system alert.

    - ```swift
        func sceneWillResignActive(_ scene: UIScene) {
            // Called when the scene will move from an active state to an inactive state.
            // This may occur due to temporary interruptions (ex. an incoming phone call).
        }
      ```

- **Scene Will Enter Foreground**

  - This method is called as part of the transition from the background state to the foreground active state — immediately before sceneDidBecomeActive(_:). You can use this method to undo many of the changes made to your scene when it entered the background.

    - ```swift
        func sceneWillEnterForeground(_ scene: UIScene) {
            // Called as the scene transitions from the background to the foreground.
            // Use this method to undo the changes made on entering the background.
        }
      ```

- **Scene Did Enter Background**

  - The next method is called immediately after the sceneWillResignActive(_:) method, once the scene has actually entered the background state. This is the right time to spin down demanding processes and save the user's work or progress.

    - ```swift
        func sceneDidEnterBackground(_ scene: UIScene) {
            // Called as the scene transitions from the foreground to the background.
            // Use this method to save data, release shared resources, and store enough scene - specific state information to restore the scene back to its current state.
        }
      ```

#### Try It Out

- Add a print statement to each of the delegate methods in both AppDelegate and SceneDelegate that describes what happens to trigger the method. For example, print("Application did finish launching.") in the application`(_:didFinishLaunchingWithOptions:)` method, and print("Scene will resign active.") in `sceneWillResignActive(_:)`.
- Run the app in Simulator. When you open the app, you'll see four messages print to the console:
  - Application did finish launching.
  - Scene will connect to session.
  - Scene will enter foreground.
  - Scene did become active.
- Dismiss the app and return to the Home screen. You'll see two more messages print to the console:
  - Scene will resign active.
  - Scene did enter background.
- Open the app switcher and reopen the app. You'll see another two messages print to the console:
  - Scene will enter foreground.
  - Scene did become active.
- Open the app switcher again, then return to the app. You'll see two more messages print to the console:
  - Scene will resign active.
  - Scene did become active.
- Take note of the various delegate methods and how you might use them when building your apps. As you can see from this exercise, they provide useful hooks to execute code at each app transition—even though you may not need to use all of them in every app you build.

#### Which Method Should I Use?

- You've learned about the many options for responding to different transitions in your app. For now, focus on three methods that will run when launching, reopening, or closing your app:
  - `application(_:didFinishLaunchingWithOptions:)`
  - `sceneDidBecomeActive(_:)`
  - `sceneWillResignActive(_:)`
- As you become more experienced and build more complex apps, you'll encounter situations where you'll want to take advantage of the other delegate methods.

#### Lab - App Event Count

- **Objective**

  - The objective of this lab is to create an app that provides a visual representation of the app life cycle. Your app will update labels on the view when different delegate methods are called.
  - Create a new project called "AppEventCount" using the iOS App template. When creating the project, make sure the interface option is set to "Storyboard."

- **Step 1 Add Event Counters to AppDelegate**

  - Open AppDelegate and create two variables at the top to count the number of times the app has launched and the number of times it has created a configuration for connecting to a scene.

    - ```swift
        var launchCount = 0
        var configurationForConnectingCount = 0​
      ```

  - Increment both of these counts by 1 within the corresponding methods.

- **Step 2 Set Up Your View and View Controller**

  - As with all apps created using the iOS App template, your app starts out with one view controller. Drag seven labels to it, one for each of the following seven AppDelegate and SceneDelegate life cycle methods. Set up constraints as necessary. Hint: Consider using a stack view.
    - application(_:didFinishLaunchingWithOptions:)
    - application(_:configurationForConnecting:options:)
    - scene(_:willConnectTo:options:)
    - sceneDidBecomeActive(_:)
    - sceneWillResignActive(_:)
    - sceneWillEnterForeground(_:)
    - sceneDidEnterBackground(_:)
  - You'll use the labels to display the number of times that each event has occurred. Create an outlet for each label, using descriptive names like `didFinishLaunchingLabel` and `didBecomeActiveLabel`.
  - Also, for each label corresponding to the scene events (not AppDelegate events), create a variable to store a count of how many times its delegate method has been called. Set the initial value of each variable to 0, as in the following example:
`var willConnectCount = 0`

- **Step 3 Access Count vars From AppDelegate**

  - To update your UI for the application(_:didFinishLaunchingWithOptions:) and application(_:configurationForConnecting:options:) calls, you'll need access to the two variables you created in AppDelegate. In ViewController, add the following variable: `var appDelegate = (UIApplication.shared.delegate as! AppDelegate)`
  - This will allow you to access AppDelegate and your count variables within ViewController. Your AppDelegate instance is known as a singleton and can be accessed in this manner. Since you know what the type is, you can safely downcast it using `as! AppDelegate`.
  - ​​While it may be convenient to access AppDelegate anywhere, making it tempting to fill it with information, it's a best practice to keep it focused on its responsibilities. In this case, you're accessing it for information that it is responsible for.
  - Create an updateView() method that updates each label with its count. For the AppDelegate events, you'll access your variables via your new appDelegate variable, and for scene events, you'll use the instance variables that you created within ViewController. Update the labels in updateView(), as in the following example: `launchLabel.text = "The App has launched \(appDelegate.launchCount) time(s)"`

- **Step 4 Give SceneDelegate Access To ViewController**

  - In the SceneDelegate class, just below the line var window: UIWindow?, create a variable property called `viewController` that is of type `ViewController?`. Be sure to make it optional.
  - At the top of the scene(_:willConnectTo:options:) method, set the property `viewController` equal to the `rootViewController`. This will give the SceneDelegate access to the instance of ViewController, enabling you to write code that increments the corresponding count properties on ViewController each time an app life cycle method has been called, as in the following example: `viewController = window?.rootViewController as? ViewController`
 
- **Step 5 Increment the Count Properties**

  - In scene(_:willConnectTo:options:), increment the count of the ViewController property corresponding to the scene connecting: `viewController?.willConnectCount += 1`
  - Repeat this step for the other SceneDelegate methods:
    - sceneDidBecomeActive(_:)
    - sceneWillResignActive(_:)
    - sceneWillEnterForeground(_:)
    - sceneDidEnterBackground(_:)

- **Step 6 Regularly Update the View**

  - Your view will need to update regularly to display the new counts for each app event. You might think to call updateView() in the viewDidLoad() method of your ViewController, but there's a problem: viewDidLoad() won't be called at all the app events. 
  - There's a better place to update the view. One of the scene event methods, `sceneDidBecomeActive(_:)`, will be called after each of the other life cycle methods is called and just before the user can interact with the scene again. The `sceneDidBecomeActive(_:)` method is the perfect place to call the view controller's updateView(), ensuring that the labels display the proper count.

- **Step 7 Test**

  - Run the app in Simulator.
  - The labels for both the app did finish launching and the scene did become active events should show 1 for their counts.
  - Go to the Home screen on Simulator.
  - Return to the app and observe which other labels have changed.
  - Bring up the app switcher, but instead of changing apps, return to the same app. Which labels have changed?
  - When the user quits your app or dismisses a scene, `sceneDidDisconnect(_:)` will be called—but adding a label and counter the same way for this method would be ineffective, as the data would be lost between terminations.
  - Great job! You've made an app that helps you visualize the app life cycle. Be sure to save it to your project folder.

### Lesson 1.3 Model-View-Controller

#### Overview

- At this stage of the course, you're probably beginning to feel more comfortable with how apps work and how to build basic app features. You've seen how even relatively simple apps rely on many files, structures, and classes. Imagine how the code for a medium-sized app might span across hundreds of files in your project.
In this lesson, you'll learn how to organize files, structures, and classes into a design pattern called Model-View-Controller, or MVC. MVC will help you architect the files in your app as well as the interactions and relationships between different types and instances.
- MVC is not a trivial topic. This lesson will help you get started, but you'll continue learning about and reinforcing your understanding of MVC concepts for a long time. You'll learn the intricacies of implementing proper MVC patterns in your app from sample code, from building your own apps, and from mentors and teachers.
- Keep in mind that there's never one right answer to any architecture decision. There are best practices, but style and personal preference will play a role in how you choose to organize your app's projects and relationship models.
- Now that you've started to build more complex projects, you know quite a bit about how different classes and structures work together. And you've probably noticed some patterns emerge.
- For example, you know you can use AppDelegate and SceneDelegate methods to handle different events in the app life cycle. You know you can edit the Main storyboard file in Interface Builder to define views and create user interfaces. You've worked with view controller files that define how a scene should respond when the user taps a button or navigates between screens. And you've seen specific classes that represent data for your app to display.
- So far in this course, you've followed specific instructions that told you what classes or structures to create and which properties or functions to add to them. But before that, someone had to decide how all the pieces were going to work together. How would you decide which new classes or structures to create? How do you know what properties to have? Or which objects should call functions on other objects?
- These are all great questions. And by asking them, you're graduating from someone who's simply following instructions to a programmer who's making decisions.
- For decades, software developers have studied and practiced different ways of answering these questions. As software development has matured, a few common design patterns have emerged. One of the most prevalent patterns is Model-View-Controller, or MVC. MVC assigns objects to one of three roles — model, view, or controller — and helps define the way objects communicate with each other.
- The diagram below provides an overview of how the three types of objects work in relation to one another. Each layer, or type, has specific roles and guidelines for communicating with the other layers.

  - <img src="./resources/overview_mvc.png" alt="Overview MVC" width="400" />

- Before trying to understand how the interactions work, you should understand what each type of object is responsible for. As you read the descriptions below, revisit the diagram and review the interactions between the types of objects.

#### Object - Model

- A model object groups the data you need to represent items or concepts. These items or concepts might be unique to your app, such as a character in a game or a task in a to-do list, or they might represent things in the real world, such as a person and their contact information in an address book, or an item to be purchased from a store.
- Model objects can be related to other model objects. A model object for a car might have a driver property, or that game character might have an array of inventory items.
- In most cases, you'll create model objects by defining structures or classes in your project. And you'll typically define the structures or classes in new Swift files. 
- Model objects are made up of properties that represent attributes of the type, and they sometimes have methods for updating and modifying their own properties.
- **Communication**
  - Model objects are usually created in response to some user interaction with a view or a control. The message to create the model object is transmitted from the view through a controller object, which is most often a subclass of a view controller. That said, model objects

#### Object - View

- You've already learned that views are the visual aspects of the user interface. A view object knows how to draw itself on the screen and can respond to user input. A major purpose of view objects is to display data about an app's model objects and to allow the user to edit that data. 
- Views can be reused or reconfigured to show different instances of model data. For example, a view object in the Contacts app can be used to display information about any contact, and a view in Mail can be used to show any message.
- Views often have an update or configure function that accepts a model object as a parameter; the view uses the model object to update itself to match the data it's meant to display. For example, imagine a table view cell displaying a photo of a player along with the player's current score. The view might have an update(_:) method that assigns the player's image to an image view and the player's score to a label.
- When building an iOS app, you'll typically define the view layer in Interface Builder. Views can then be referenced or used in a view controller class as outlets or actions.
- **Communication**
  - Even though views are commonly used to display data about an app's model objects, views should never own a model object as a property or modify a model object directly. Instead, the view can send a message — for example, that the user has performed an action — to a view controller, and the controller will update the model object.

#### Object - Controller

- A controller object acts as the messenger between views and model objects. For example, when the user interacts with a view, the view sends a message to a view controller, and the view controller can then update the model object. Alternatively, when a model object is created or updated, a view controller can tell its view to redraw or update itself with the new data.
- In fact, the view controller is the most common type of controller for new iOS developers. As you've learned, a view controller controls a view along with its accompanying subviews and usually displays information about one or more model objects. The model object, or collection of model objects, is usually defined as a property on the view controller. When the user interacts with a view or control, an action or block of code is triggered on the view controller, which can then update the model object.
- There are two other types of controllers: model controllers and helper controllers.
- **Model Controllers**
  - Similar to how a view controller controls a view, a model controller helps control a model object or a collection of model objects. There are three common reasons you might want to create a model controller:
    - Multiple objects or scenes need access to the model data.
    - The logic for adding, modifying, or deleting model data is complex.
    - You want to keep the code in your view controllers focused on managing the views.
  - For example, imagine you're working on a simple Notes app that syncs the user's notes to a server. The app has two scenes: a list view, which displays all the notes in a table view, and a detail view, which allows the user to create a new note or read and edit a preexisting note.
  - If you used only view controllers, the list view controller would take on the majority of the work in the app — not only managing the list view but also managing all the model data, including fetching the note data from the server, adding new notes, replacing modified notes, deleting notes, and saving all changes to the server.
  - Instead, you might decide to abstract, or move, all the code that manages notes to a separate model controller called NoteController. This approach allows the view controller to focus only on displaying the notes and leaves the networking and model management to the model controller. 
  - Using model controllers isn't required; in fact, it's not a common practice in small projects. But abstraction is crucial when working on larger projects or working with other developers, because it makes your code simpler, more readable, and easier to maintain.
- **Helper Controllers**
  - Helper controllers are useful anytime you want to consolidate related data or functionality so that it can be accessed by other objects in your app. One common example of a helper controller is a NetworkController, which manages all the network requests in a given app.
  - Controllers should perform a very specific function without depending on other controllers to perform their work. For example, a NoteListViewController might display a list of notes, and a NoteDetailViewController might display the details about an individual note instance and respond to user input to modify or create a note.
- **Communication**
  - Besides mediating the communication between models and views, controller objects can communicate or work directly with other controller objects. Consider the examples above. The initial view controller of a Notes app (NoteListViewController) might be responsible for displaying a list of notes, so it accesses the notes property on a note model controller (NoteController). The note model controller needs to check whether there are any new notes, so it tells the network controller (NetworkController) to check the web service for new data. If the NetworkController finds new notes, it downloads the data and sends it back to the NoteController, which would then update its notes property and send a callback to the NoteListViewController that it has new data, enabling the NoteListViewController to update its view of notes.

#### Example

- Imagine you're tasked with building an app to track photos of meals that the user has eaten, along with notes and an optional rating of each meal. What model objects will you need? What views? What controllers?
- Before moving on, take a moment to consider how you might plan, or architect, this simple app using model, view, and controller objects.
- **Model**
  - If the app is going to track meals, it makes sense to have a model object called Meal that holds the data about each meal. Each meal should have a name, and — according to your feature description — it also needs to have a photo, notes, and a rating.
  - Next, you'll need to consider how your model objects will be displayed in the app — and your decisions will impact the properties to keep on your model objects. For example, if you want to make sure that a list of meals is displayed in the order that the meals were added, you may need to include a timestamp on each meal object.
  - So your model object might be declared as a structure or a class with five properties:

    - ```swift
        struct Meal {
            var name: String
            var photo: UIImage
            var notes: String
            var rating: Int
            let timestamp: Date
        }
      ```

- **View**
  - You can imagine that the app will need at least two views, or scenes. Just like in the Notes example, one scene will display a list of all the meals that the app has tracked, and another scene will display the details of a meal.
  - The meal list scene could display the list in a table view, with the name and photo of each meal in an individual cell. The meal detail scene could then display the name of the meal in the navigation bar, a photo in an image view, notes about the meal in a text view, and the current rating in a segmented control.
  - Each of these two views will need a view controller class that can manage the model data it's displaying and can respond to user interactions.
  - In most cases, you'll use a storyboard to define your views in Interface Builder. You'll assign each view a view controller class, where you can add actions or outlets to respond programmatically to user interaction and to update the view.
  - Occasionally, you may build a view that requires its own class definition. For example, you may have a custom table view cell with a button, which would require you to create a subclass of the cell type and add functionality in the class definition. You'll learn more about custom views with subclasses later.
- **Controller**
  - At a minimum, you'll need a controller for the meal list and meal detail views. For the meal list, you should use a UITableViewController, which displays data in a table, and for the detail view controller you should use a UIViewController. Some of the table view setup that follows will be unfamiliar to you, but don't worry — you'll learn more about UITableViewController and setting up table views in a future lesson.
- **Meal List Table View Controller**
  - `class MealListTableViewController: UITableViewController {...}`
  - The MealListTableViewController should have a property that holds a collection of meals to display.
  - The properties you add to a table view controller subclass — in this case, MealListTableViewController — should reflect anything that makes it different from a plain UITableViewController.
    - `class MealListTableViewController: UITableViewController { var meals: [Meal] = [] }`
  - Because the collection of meals lives on MealListTableViewController, the controller should handle adding and deleting meals from the collection. One approach would be to modify the array directly, but you might choose instead to define functions that add or remove meals from the array.
  - When the user quits the app, they'll expect it to save any meal data they just entered or modified. In this example, MealListTableViewController is the intermediary between the view and the meal model data, so it should take responsibility for saving data to disk, as well as for loading data when the app launches and before the view is displayed.

    - ```swift
        class MealListTableViewController: UITableViewController {
            var meals: [Meal] = []
            func saveMeals() {...} 
            func loadMeals() {...}
        }
      ```

  - The view controller should also respond to any user events. What events might the list view handle?
  - List views usually let the user select an item to display more details about the item. You'd enable that functionality by adding a segue from the list view scene to the detail view scene in the storyboard. You'll also need to add and implement the `prepare` function that passes the selected meal to the detail scene.
  - When all is said and done, you'll likely end up with a view controller that looks similar to this:

    - ```swift
        class MealListTableViewController: UITableViewController {
            var meals: [Meal] = []
            override func viewDidLoad() {
                super.viewDidLoad()
                // Load the meals and set up the table view
            }
            // Required table view methods

            override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {...}

            override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {...}

            // Navigation methods

            @IBSegueAction func showMealDetails(_ coder: NSCoder) -> MealDetailViewController? {
                // Initialize MealDetailViewController with selected Meal and return
            }

            @IBAction func unwindToMealList(sender: UIStoryboardSegue) {
                // Capture the new or updated meal from the MealDetailViewController and save it to the meals property
            }

            // Persistence methods

            func saveMeals() {
                // Save the meals model data to the disk
            }

            func loadMeals() {
                // Load meals data from the disk and assign it to the meals property
            }
        }
      ```

  - Once you've planned the required features of the list view controller, consider the detail view controller. How will the two controllers interact with each other?
- **Meal Detail View Controller**
  - `class MealDetailViewController: UIViewController {...}`
  - The MealDetailViewController will display the details about an individual meal. You can use Interface Builder to set up views for displaying the name, the image, the notes, and the rating, then create outlets to MealDetailViewController. Your view controller should also have a property for the meal that will be displayed.
  - Providing a custom initializer and using@IBSegueAction to display the detail controller from MealListTableViewController allows you to guarantee that a model is provided, so that you do not need to worry about handling optional values.

    - ```swift
        class MealDetailViewController: UIViewController {
            var meal: Meal

            @IBOutlet var nameTextField: UITextField!
            @IBOutlet var photoImageView: UIImageView!
            @IBOutlet var ratingControl: RatingControl!
            @IBOutlet var saveButton: UIBarButtonItem!

            init?(coder: NSCoder, meal: Meal) {
                self.meal = meal
                super.init(coder: coder)
            }

            override func viewDidLoad() {
                super.viewDidLoad()
                // Update view components using the Meal model
            }
        }
      ```

  - The MealDetailViewController should handle updating the view it controls with the information about the model object (the meal) it's meant to display. Many times you can do this right in viewDidLoad(), but you may want to break the process out into its own method — especially if the model can change.
  - The detail view controller might have additional methods to allow the user to add a photo or to modify other properties of the meal. In those scenarios, it will need some way to know whether it's displaying an existing meal or creating a new meal.
  - After accounting for all the tasks the detail view controller is handling, the result might look something like this:

    - ```swift
        class MealDetailViewController: UIViewController, UIImagePickerControllerDelegate {

            @IBOutlet var nameTextField: UITextField!
            @IBOutlet var photoImageView: UIImageView!
            @IBOutlet var ratingControl: RatingControl!
            @IBOutlet var saveButton: UIBarButtonItem!

            var meal: Meal

            init?(coder: NSCoder, meal: Meal) {
                self.meal = meal
                super.init(coder: coder)
            }

            override func viewDidLoad() {
                super.viewDidLoad()
                // Update view components using the Meal model
            }

            // Navigation methods

            override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
                // Update the meal property that will be accessed by the MealListTableViewController to update the list of meals
            }

            @IBAction func cancel(_ sender: UIBarButtonItem) {
                // Dismiss the view without saving the meal
            }
        }
      ```

- **A Reminder**
  - MVC is a tool to help you write clean, maintainable, and organized code. But there's more than one way to implement it. If you were to plan out the same classes for the same app, you might end up with different variable names, different function names, different model objects, and different interfaces for your view hierarchies.
  - When thinking through MVC, developers have their own distinct styles and patterns. Since you're going through this course, you'll tend to follow the patterns used here. But as you grow and gain more experience, you'll come up with your own preferences for how to architect the objects in your apps.
  - You'll also develop your own perspective on the advantages and guidelines behind the MVC design pattern. Look at sample code and previous projects for further guidance. And don't be afraid to ask someone with more experience to mentor you in the best approach to app architecture.

#### Project Organization

- Xcode projects get crowded quickly. Even small apps can have dozens or hundreds of files that define different views, storyboards, structures, classes, protocols, and controllers. Writing navigable code is an important part of building any program. If you aren't careful, your project can quickly become disorganized, which will make it hard to continue working productively.
- The first step in organizing your project is to create clear, descriptive filenames. For example, ProductListTableViewController.swift is much more descriptive than MainVC.swift. Descriptive names for classes, structures, protocols, properties, and functions will also help you stay productive. Some developers worry that they'll have to write long names over and over again, but the autocompletion feature in Xcode makes it easy, so there's no advantage to a short name.
- Another good practice is to create separate files for each of your type definitions, such as classes, structures, protocols, and enumerations. For example, create separate Car.swift, Driver.swift, RaceTrack.swift, and Garage.swift files, rather than one combined Model.swift file.
- One more thing: Write your code as if complete strangers are going to read it. You might think this only matters when you're working with other people, but it also applies to code only you will see. When you need to revise your code six months down the road, you'll appreciate descriptive names, clear function signatures, and details comments.
- Xcode also provides a grouping feature to help you organize code. It's basically a way to group your files into folders within your project, so that you don't end up with one never-ending list of files in the Project navigator.
- Many developers make groups for the following:
  - View controllers
  - Views
  - Models
  - Model controllers
  - Other controllers
  - Protocols
  - Extensions
  - Resources
    - Storyboards
    - Frameworks
- For now, you might want to keep your AppDelegate, SceneDelegate, and Main files on the top level and use a shorter list of groups:
  - Models
  - Other controllers
  - Storyboards
  - View controllers
  - Views

    - <img src="./resources/list_of_group.png" alt="List of Groups" width="200" />

- It's worth noting that the way you organize your project in Xcode may not always change the way your files are saved in the project directories; depending on how groups are created, it might only affect how files are displayed within Xcode itself. In Xcode, this is signified by two different folder icons. A group that is used to organize information logically in your project but that does not represent a folder that you would see in Finder is denoted by a shaded lower-left corner in the folder icon.

  - <img src="./resources/folder_icons.png" alt="Folder Icons" width="200" />

- When creating groups in Xcode's navigator, you will see two options: New Group and New Group without Folder. The latter does not create a folder on disk — it's simply visual organization within Xcode.
- The specifics of how to organize your projects will depend on what you're building. For now, think simple. Allow yourself time to keep things organized so that your projects are easy to navigate. You'll appreciate it later.

#### Lab - Favorite Athletes

- **Object**
  - In this lab, you'll plan out and create an app that uses proper MVC design. Your app will have two screens for displaying the user's favorite athletes. It will allow the user to add new athletes to the list and to edit existing entries.
  - In your student resources folder, open the starter project called "FavoriteAthlete." The app is already set up with a table view for displaying a list of athletes and a form for collecting information about an individual athlete.
- **Step 1 Make a Plan**
  - Based on the app description, think through the following questions and write down your answers in a brief outline:
    - What model object(s) will you need for this app?
    - What properties will the model object(s) need?
    - In addition to the view controller provided in the starter project, what other controllers will you need?
    - What properties and functions will the controllers need?
  - Did you come up with a plan for moving forward? Your design may differ slightly from the steps in this lab, and that's OK. In app design, there's no one right way to do things. But for the purposes of this project, you'll use the following model and controllers:
    - The Athlete model will store information, through properties, for the athlete's name, age, league, and team.
    - The AthleteTableViewController will handle the view associated with displaying the user's list of favorite athletes.
    - The AthleteFormViewController will handle the views and user input associated with entering and editing athlete information.
- **Step 2 Examine the Starter Project**
  - Take a look at the view controllers in the storyboard. You'll see a table view controller that has two segues to a form. One segue has an identifier of AddAthlete and originates from an Add button. The other segue has an identifier of EditAthlete and originates from a table view cell.
  - The form has a Save button that you'll use to trigger an unwind segue to bring the user back to the table view.
  - The table view controller's subclass has already been created and set as AthleteTableViewController. At the moment, you'll see an error in this file. That's because it references a type, Athlete, that hasn't been created yet.
- **Step 3 Create the Model**
  - Create a new Swift file named "Athlete."
  - In the file, create an Athlete struct that has the following properties: name, age, league, and team.
  - Add a computed property description of type String that uses the four properties to return a phrase describing the athlete, as in the following:

    - ```swift
        var description: String {
            return "\(name) is \(age) years old and plays for the \(team) in the \(league)."
        }
      ```

- **Step 4 Create and Implement a View Controller Subclass**
  - Create a new Cocoa Touch Class file that subclasses UIViewController and is named “AthleteFormViewController.” In the Main storyboard, use the Identity inspector to set the class of the form scene to your new subclass.
  - Add a variable property athlete of type Athlete? to your AthleteFormViewController class. Why is this variable optional? It will be nil when the user comes to the screen to create a new athlete, and it will have a value when the user comes to the screen to edit an athlete.
  - Create a custom initializer with the signature init?(coder: NSCoder, athlete: Athlete?). Assign self.athlete to your instance variable and call the super implementation. Satisfy the compilation error that this creates by using Xcode's fix-it suggestion.

    - ```swift
        class AthleteFormViewController: UIViewController {
            var athlete: Athlete?
            
            init?(coder: NSCoder, athlete: Athlete?) {
                self.athlete = athlete
                super.init(coder: coder)
            }
            
            required init?(coder: NSCoder) {
                // fatalError("init(coder:) has not been implemented")
                super.init(coder: coder)
            }
            ...
      ```

  - Create an updateView() method and call it in the viewDidLoad() method.
  - Add outlets from the storyboard to all the text fields in AthleteFormViewController. In the same file, add an action for the Save button.
  - In updateView(), unwrap the athlete property and check whether it contains an Athlete object. If so, use the object to update the text fields with the athlete's information.

    - ```swift
        func updateView() {
            guard let theAthlete = athlete else { return }
            nameTextField.text = theAthlete.name
            ageTextField.text = "\(theAthlete.age)"
            leagueTextField.text = theAthlete.league
            teamTextField.text = theAthlete.team
        }
      ```

  - Inside the the action for the Save button, create an Athlete object by unwrapping the text from the text fields and passing it to the Athlete initializer, as follows:​

    - ```swift
        guard let name = nameTextField.text,
            let ageString = ageTextField.text,
            let age = Int(ageString),
            let league = leagueTextField.text,
            let team = teamTextField.text else {return}
         
        athlete = Athlete(name: name, age: age, league: league, team: team)
      ```

- **Step 5 Finish Implementing AthleteTableViewController**
  - In the starter project for this lab, most of the AthleteTableViewController class has already been implemented. You'll learn how to set up a table view on your own in a later lesson. For now, you simply need to create the functionality for informing the next view controller which athlete was tapped and for receiving information about an added or edited athlete.
  - Create an @IBSegueAction from the AddAthlete segue by Control-dragging from the segue arrow between the two controllers into AthleteTableViewController. Name it `addAthlete` with no arguments. Do the same for the EditAthlete segue, naming the action `editAthlete` and setting Arguments to `Sender`.
  - In the addAthlete method, instantiate and return a new instance of AthleteFormViewController.
  - When the editAthlete method is called, the sender parameter will be the cell that was touched. Since you haven't yet learned about table views, this next part might be tricky, but see if you can follow along. Your Athlete objects are stored in an array called athletes in the table view controller. The index of each athlete in the array corresponds to the index of the table cell displaying that athlete. UITableView provides a method to look up a cell's IndexPath, which you can use to look up the corresponding Athlete in your athletes array.
  - In the editAthlete method, unwrap the `sender` and cast it to a `UITableViewCell`, then use UITableView's `indexPath(for:)` method to get the IndexPath for the cell. The index path method returns an optional, so you can include that call in your unwrapping logic. Once you have the index path, you can access its row property, which translates to the index of the athlete in your athletes array. Your implementation might look like the following:​

    - ```swift
        let athleteToEdit: Athlete?
        if let cell = sender as? UITableViewCell,
          let indexPath = tableView.indexPath(for: cell) {
            athleteToEdit = athletes[indexPath.row]
        } else {
            athleteToEdit = nil
        }
         
        return AthleteFormViewController(coder: coder, athlete: athleteToEdit)
      ```

  - Next, you'll enable the table view controller to receive an Athlete object back from the AthleteFormViewController in the event that a new athlete has been added or an existing athlete has been edited. You will do this with an unwind segue. In AthleteTableViewController, add an @IBAction method that takes a parameter of type UIStoryboardSegue. Inside the body of this method, you'll first need to cast the segue's source view controller as an AthleteFormViewController and unwrap its athlete property with a guard statement.
  - Which index path, if any, was selected before going to AthleteFormViewController? You can use the table view's indexPathForSelectedRow property in combination with conditional binding to discover the path. If the returned index path is successfully unwrapped, the Athlete object coming back is an edited athlete, not a new athlete. You'll need to use the index path to replace the existing athlete in the athletes array. If the returned index path is unsuccessfully unwrapped, the Athlete object coming back is a new athlete—in which case you can simply append the new athlete to the end of the athletes array. Here's how all this works:​

    - ```swift
        @IBAction func unwindToAthleteList(segue: UIStoryboardSegue) {
            guard let athleteFormViewController = segue.source as? AthleteFormViewController,
                  let athlete = athleteFormViewController.athlete else { return }
            if let selectedIndexPath = tableView.indexPathForSelectedRow {
                athletes[selectedIndexPath.row] = athlete
            } else {
                athletes.append(athlete)
            }
        }
      ```

- **Step 6 Perform the Unwind Segue in Storyboard**
  - Finally, you need to create the unwind segue. In the storyboard, Control-drag from the Athlete Form View Controller to the view controller's Exit, then choose your unwind segue: `unwindToAthleteListWithSegue`. Give this segue a name, `segueForm`, by selecting it in the Document Outline and adding the identifier in the Attributes inspector.
  - Now you need to instruct the segue when to execute in the AthleteFormViewController. Do this by calling `performSegue(withIdentifier:sender:)` at the end of your Save button's @IBAction method, passing in the identifier of the unwind segue and self as the sender: `performSegue(withIdentifier: "segueForm", sender: self)`
  - Now run the app and see if you can add and edit your favorite athletes.
  - Congratulations! You've planned and created a simple app using MVC principles. Be sure to save your work to your project folder.

### Lesson 1.4 Scroll Views

- Whenever your app needs to display content or allow manipulation of content that doesn't fit entirely on the screen, it's time for a scroll view. You can see an excellent example in the Photos app. Open Photos on an iOS device and select a photo. At first tap, the picture will fill as much of the screen as possible while maintaining the same aspect ratio. Now imagine you want to inspect a particular detail of the image. Zoom into your picture by double-tapping. Even though the photo is now bigger than the screen — especially if you're using a smaller device — you'll be able to scroll anywhere you like on the image.
- A UIScrollView needs to know two sets of information to function: the position and size of the scroll view, and the size of the content being displayed. Programmatically, these values are stored in the scroll view's frame property and contentSize property, respectively. Both properties can be managed using Auto Layout and Interface Builder.
- For scrolling to happen, the width or height (or both) of the content size must be greater than the frame's width or height. In the layout below, the frame size is represented by the red box. Notice that the content height is greater than the frame height and the content width is the same as the frame width - so this view will only scroll vertically.

  - <img src="./resources/layout_for_scrollView.png" alt="Scroll View" width="300" />

- By adjusting the sizes, you can create scroll views that scroll horizontally, vertically, or both. In addition to the content size requirement, all views you wish to scroll must be subviews of the scroll view.

#### Scroll Views in Interface Builder

- In this lesson, you'll learn how to implement scroll views with the Auto Layout feature of Interface Builder. Imagine you're designing a UI that will allow users to enter their shipping information, perhaps as part of a checkout flow for an online purchase. It will look much like the layout above.
- Start by creating a new Xcode project using the iOS App template. Name the project "ScrollingForm." When creating the project, make sure the interface option is set to "Storyboard" and save it to your project folder.
- **Define Scroll View Frame**
  - In the Main storyboard, find a scroll view in the Objects library and drag it onto the view controller scene. Adjust the size of the scroll view to match the size of the scene's Safe Area.
  - Add constraints to pin all four edges of the scroll view to the edges of the view controller's Safe Area. These Auto Layout constraints apply to the scroll view's Frame Layout Guide and ensure the scroll view size is the same as the view controller's view — whether on an iPhone SE, an iPad Pro, or any device in between. The constraints also tell the scroll view its size and position.
- **Programmatic Constraints to Content Layout Guide**
  - If needed, you can also apply constraints programmatically to a scroll view's Content Layout Guide. In other words, you can not only constrain a scroll view's size and position but also place constraints to control the behavior of a scroll view's content. For example, if you have an image view inside of a scroll view that you want to stay centered when zooming in and out, you can anchor the center of of the image view to the center of the scroll view's Content Layout Guide, as follows:

    - ```swift
        imageView.centerXAnchor.constraints(equalTo: scrollView.contentLayoutGuide.centerXAnchor)
        imageView.centerYAnchor.constraints(equalTo: scrollView.contentLayoutGuide.centerYAnchor)
      ```

  - This is just one example, and while you won't use constraints on a scroll view's Content Layout Guide in this lesson's scrolling form, they can be useful in many scenarios.
- **Define Content View Using Stack View**
  - Next, you'll tell the scroll view the size of the content. For most common layout tasks, the logic to implement a scroll view with Auto Layout will be much easier if you use a dummy view to contain the scroll view’s content. The dummy view's job is to contain the content and, generally, it is not seen by the user. This is sometimes referred to as a content view.
  - To continue your form, you'll use a stack view to contain all your content.
    - Select a vertical stack view from the Object library and add it as a subview of the scroll view.
    - Adjust the size to be the same as the scroll view
    - Pin the edges of the stack view to the Content Layout Guide by Control-dragging from the Stack View in the outline to the Content Layout Guide on the Document Outline. By holding Command while clicking each constraint, you can add them all without the menu closing.

    - <img src="./resources/pin_edges_of_stackView.png" alt="Pin the edges of stack view to the Content Layout Guide" width="400" />

    - In the Document Outline, click the arrow next to Constraints, below Stack View, and expand the outline view's area so you can see the full set of constraints that were just added. Verify that each one's Constant value is set to 0.  The stack view now defines the scroll view's content area.
  - This form should only scroll vertically — not horizontally.
    - Create an Equal Widths constraint between the Stack View and the Frame Layout Guide by Control-dragging from one to the other in the outline.
    - Locate the new constraint in the outline and verify its Constant is 0 and Multiplier is 1. Now the width of the stack view will always be equal to the scroll view's frame, which means it can't scroll horizontally.

#### Creating the Form

- **Start Creating the Form**
  - Think back to the lesson on Auto Layout and stack views. You learned that when a stack view doesn't have a defined width or height, its size is based on its subviews. As you add content to the stack view, it expands to fit all your content. Thus, the content inside the stack view defines the stack view's size. It's only as big as it needs to be.
  - Because stack views grow to fit their content, they are a perfect candidate for holding the content of a scroll view. As the content grows, the scroll view will scroll to meet the needs of the content size.
  - In your ScrollingForm project, you'll add your scrolling content to the stack view. To simplify the process, you'll create a group of views that will allow you to add as many fields as you need. Each group will consist of three views: a UIView (to serve as a background and container), a label, and a text field. The label will tell the user what type of data to enter into the text field. Both will be subviews of the background view.
  - Drag a view from the Object library into the stack view. (Hint: Typing "UIView" in the library's filter is a quick way to find the view object.) Interface Builder is showing that you have layout issues, indicated by a white arrow inside a red circle in the Document Outline, as well as by the red layout indicators in the scene.
  - What's going on? You haven't defined enough size information for the scroll view's content size. For starters, the UIView you added doesn't have an intrinsic, or natural, size. Therefore, the stack view can't determine its size, which means the scroll view can't determine its content size. In the next section, after you've added the rest of your views, you'll make sure that you have enough constraints for the required size definitions.
  - Add a label to the view you just added. Use the alignment guides to place the label in the upper-left of the view.
  - Next, add a text field below the label - again, taking care to use the alignment guides.
  - Adjust the height of the view so that its bottom margin is aligned with the bottom of the text field.
  - Next, adjust the trailing edges of the label and text field so they're aligned with the margin of the view's trailing edge.
- **Add Constraints**
  - Now that your view is nicely aligned, adding constraints should be a breeze. Adding these constraints will resolve the layout issues discussed above by defining the content size of the scroll view. (If you need a quick review, check out the lesson on Auto Layout in Unit 2 of Develop in Swift Fundamentals.)
    - Select the label and click the Add New Constraints tool button. Add a constraint for each edge, '8'.
    - Select the text field and click the Add New Constraints tool button. Since you added a constraint between the text field and the label in the previous step, you only need to add constraints for the three remaining edges: leading: 8, trailing: 8, and bottom: 20
- **Add More Fields to Your Form**
  - If you ran your project now, would you be able to scroll? Why or why not? You'd be correct if you said "no." The content isn't bigger than the scroll view, so scrolling isn't necessary.
  - In the Document Outline, select the view in the stack view and copy it (Command-C). Paste it (Command-V) several times into the stack view, once for every field you want to include in your form. Since your form will collect users' shipping information, you'll probably need the following fields: first name, last name, address line 1, address line 2, city, state, ZIP code, and phone number. Don't forget to modify the labels with descriptions of the field content. 
  - Build and run your app on different devices. On larger devices, you may find there's enough room to display all the views — so the scroll view won't scroll. On smaller devices, such as an iPhone XS, you won't be able to see all the views, but you should be able to scroll to see the rest. If you don't have a physical device small enough, run the app on a small device in Simulator.

#### Keyboard Issues

- When you ran your app on smaller devices, you might have noticed that the keyboard covers some of the text fields while they are being edited.
- Obviously, it's not a good experience if the user can't see the text they're entering. This lesson doesn't provide details about keyboard management with scroll views, but here's a brief explanation that will help you handle the problem.
- iOS sends a notification every time the keyboard is shown. You can add code to listen for the notification and execute a function when the notification is posted. And, because the notification includes the keyboard size, your function can adjust the scroll view so that its contents aren't overlapped by the keyboard. You can also add a listener for another notification that is posted when the keyboard will be hidden, allowing you to reverse the adjustments.
- If you're interested in learning more about notifications, check out the API Reference documentation for [Notification Center](https://developer.apple.com/documentation/foundation/notificationcenter).
- To adjust the size of the content view, you need to create an outlet from the scroll view to its view controller. Then you can add the following code to the ViewController class to fix the keyboard issues:

  - ```swift
      func registerForKeyboardNotifications() {
          NotificationCenter.default.addObserver(self, selector: #selector(keyboardWasShown(_:)), name: UIResponder.keyboardDidShowNotification,
          object: nil)
          NotificationCenter.default.addObserver(self, selector: #selector(keyboardWillBeHidden(_:)), name: UIResponder.keyboardWillHideNotification,
          object: nil)
      }

      @objc func keyboardWasShown(_ notification: NSNotification) {
          guard let info = notification.userInfo,
              let keyboardFrameValue = info[UIResponder.keyboardFrameBeginUserInfoKey]
              as? NSValue else { return }

          let keyboardFrame = keyboardFrameValue.cgRectValue
          let keyboardSize = keyboardFrame.size

          let contentInsets = UIEdgeInsets(top: 0.0, left: 0.0, bottom: keyboardSize.height, right: 0.0)
          scrollView.contentInset = contentInsets
          scrollView.scrollIndicatorInsets = contentInsets
      }

      @objc func keyboardWillBeHidden(_ notification: NSNotification) {
          let contentInsets = UIEdgeInsets.zero
          scrollView.contentInset = contentInsets
          scrollView.scrollIndicatorInsets = contentInsets
      }
    ```

- Now add the following line to viewDidLoad(): `registerForKeyboardNotifications()`
- Build and run your app. You should now be able to see what you're typing in any text field.

#### Content Insets and Scroll Indicator Insets

- To ensure that the keyboard does not overlap your content, you used a content inset to change the size of the content area of the scroll view, making room for the keyboard. You'll use this solution often when you want to add padding to the scroll view content so that controllers, toolbars, and keyboards don’t interfere with the user's experience of the content.
- To add padding, use the contentInset property to specify a buffer area around the content of the scroll view. By adding padding, you're making the scroll view's window into its content smaller without changing the size of the scroll view itself or the size of its content.
- The contentInset property is a UIEdgeInsets struct with fields for top, bottom, left, and right.

  - <img src="./resources/contentInset.png" alt="Content Inset" width="400" />

- But there's a problem. Changing the contentInset value has an unexpected side effect when your scroll view displays scroll indicators. For example, the scroll indicators may be drawn behind the keyboard — an unwanted result. To fix this, you'll need to modify the `scrollIndicatorInsets`. Like the contentInset property, the scrollIndicatorInsets property is defined as a UIEdgeInsets struct. You can remedy the situation by setting the scrollIndicatorInsets values to match the contentInset value.

  - ```swift
      let contentInsets = UIEdgeInsets(top: 0.0, left: 0.0, bottom: keyboardSize.height, right: 0.0)
      scrollView.contentInset = contentInsets
      scrollView.scrollIndicatorInsets = contentInsets
    ```

#### The Scroll View Family

- UIScrollView is the parent class of several other classes in UIKit, including UITableView and UICollectionView. These two prominent classes both inherit all the functionality of UIScrollView classes. As you advance to these new topics, you'll want to keep this relationship in mind, since you'll be able to use all the tools you learned in this lesson when you use table views or collection views.

#### Challenge

- Create a horizontal scroll view that displays three of your favorite images.

#### Lab - I Spy

##### Objective

- The objective of this lab is to implement a scroll view on an image view that will enable users to zoom in and pan an image. Your app will have a single view with a single picture.
- Create a new project called "ISpy" using the iOS App template.

##### Step 1 Set Up the Storyboard

- Add a scroll view to the view controller and resize it to fit in the Safe Area. Add constraints to pin the scroll view edges to the Safe Area of the view controller.
- Add an image view as a subview of the scroll view and resize it to fit the scroll view's bounds. Add constraints to pin the edges of the image view to the Content Layout Guide of the scroll view.

  - <img src="./resources/constraints_imageView.png" alt="Constraints of Image View" width="500" />

- Add an image of your choosing to the Assets folder. Set it as the image of the image view.
- Create outlets from the scroll view and image view to the ViewController file.

##### Step 2 Implement the UIScrollViewDelegate

- To use a scroll view for zooming, your view controller needs to conform to UIScrollViewDelegate. You can read more about UIScrollViewDelegate in the [documentation](https://developer.apple.com/documentation/uikit/uiscrollviewdelegate). But for now, go ahead and add the following line of code to the class declaration: `class ViewController: UIViewController, UIScrollViewDelegate {...}`
- In the viewDidLoad() method, set the scroll view's delegate to be the ViewController instance.
- Implement the `viewForZooming()` delegate method, and return the image view in the body of the method.
- Create a new method `updateZoomFor(size: CGSize)` that will update the zoom of the image. Override `viewDidAppear(_:)` and in the body call `updateZoomFor(size:)`, passing in `view.bounds.size`.
- Inside the `updateZoomFor(size:)` method, use the imageView and the size parameter to calculate the scale. Then set the `minimumZoomScale` property of the scrollView to be that scale, as in the following:

  - ```swift
      let widthScale = size.width / imageView.bounds.width 
      let heightScale = size.height / imageView.bounds.height 
      let scale = min(widthScale, heightScale)
      scrollView.minimumZoomScale = scale
      scrollView.zoomScale = scale
    ```

- The code above starts by calculating the scale necessary to show the entire width and height of the image. It then sets the minimum scale to be the smaller of the two (width or height), so that you won't be able to zoom out beyond that. Finally, it sets the initial zoom scale so that the image fits inside the screen.
- Run the app. You should be able to zoom and scroll in all directions. (Use **Option-drag** to zoom in Simulator.)
- Great work! You've made an app that can scroll and zoom in on an image. Can you think of something you want to build where this functionality might be useful? Be sure to save your app to your project folder for future reference.

  - ```swift
      import UIKit

      class ViewController: UIViewController, UIScrollViewDelegate {
          @IBOutlet var scrollView: UIScrollView!
          @IBOutlet var imageView: UIImageView!

          override func viewDidLoad() {
              super.viewDidLoad()
              scrollView.delegate = self
          }

          func viewForZooming(in scrollView: UIScrollView) -> UIView? {
              return imageView
          }
          
          func updateZoomFor(size: CGSize) {
              let widthScale = size.width / imageView.bounds.width
              let heightScale = size.height / imageView.bounds.height
              let scale = min(widthScale, heightScale)
              scrollView.minimumZoomScale = scale
              scrollView.zoomScale = scale
          }
          
          override func viewDidAppear(_ animated: Bool) {
              super.viewDidAppear(animated)
              updateZoomFor(size: view.bounds.size)
          }
      }
    ```

- Tip: When to use super when overriding iOS method
  - It depends on recommendations from the documentation for the superclass's implementation of that method.
  - `viewDidAppear(_:)` - If you override this method, you must call super at some point in your implementation.

### Lesson 1.5 Table Views

- Enter UITableView, one of the most widely used views in iOS. Table views are perfect for efficiently displaying large or small amounts of information and for managing dynamic or static lists of data.
- In this lesson, you'll learn why table views are so popular among iOS developers. This lesson will focus on dynamic table views; the next table view lesson will cover static table views.
- If you've ever used an iOS device, odds are you've encountered a table view, or UITableView. Because of their utility, table views are one of the most ubiquitous views in all of iOS. In the Music app, table views allow you to navigate through your music hierarchy. In Contacts, table views present your contacts in an indexed list. In Settings, table views organize controls into visually distinct groupings. 
- As you work through this lesson, you'll learn how to work with table views by building an emoji dictionary app.
- Before we jump into table views, you'll need to complete a few setup steps — all things you've learned about in previous lessons. To get started, open Xcode and create a new iOS app project called "EmojiDictionary." When creating the project, make sure the interface option is set to "Storyboard." Delete the ViewController file and the view controller scene in the Main storyboard. Now you have a truly blank app canvas to work with.
- The next step is to create your model. Remember that model objects should represent the real-world object you want to display to the user. In this project, you're going to display information about different emoji, so you'll need to create an Emoji model object. Your emoji structure will have the following properties:
  - symbol, a String that holds the emoji symbol
  - name, a String representing the emoji name
  - description, a String describing the emoji
  - usage, a String describing how the emoji is used or its meaning
- For example, an emoji instance might look like the following:

  - ```swift
      {
          symbol: "🐘",
          name: "Elephant",
          description: "A gray elephant.",
          usage: "good memory"
      }
    ```

- Before looking at the code segment below, create a new Swift file called Emoji.swift and try to create a struct to represent an emoji on your own.

  - ```swift
      struct Emoji {   
          var symbol: String
          var name: String
          var description: String
          var usage: String
      }
    ```

- Now that your model is set up, you can dive into table views.

#### Anatomy of a Table View

- A table view is an instance of the UITableView class. It may be responsible for displaying one data object or tens of thousands of data objects. To accommodate many rows of data, table views present data in a scrolling, single-column list, with the option to divide the rows into sections or groups. Each section can have a header above the first item and a footer below the last item. The table view itself can also have its own header and footer.

##### Table View Controllers

- There are two ways to add a table view to your projects, depending on how you will use the table view.
- One approach is to add a table view instance directly to a view controller's view. In this scenario, the view controller may manage other views in addition to the table view. This means you're responsible for managing the position and size of the table view. You're also responsible for setting the data source and delegate object(s). (You'll learn about the data source object and the delegate object later in this lesson.)
- The second approach is to use a table view controller. UITableViewController is a view controller subclass that manages a single table view instance. In this approach, the table view takes up the entire view, and you can't adjust the table view's size. The table view controller also acts as the data source and delegate of the table view.
- While you may find uses for both approaches, table view controllers provide some additional functionality. If you were to use the first approach and also wanted one of these features, you'd have to write additional code. For example, when using a table view controller, the table view's scroll view will always adjust to accommodate the keyboard when necessary. If you were to use a table view added to a regular view controller, you would have to write code to monitor when the keyboard is presented and adjust the table view's content insets accordingly. 
- Except in special cases, you'll likely find that it's best to use a table view controller.
- Back in your EmojiDictionary project, open the Main storyboard and drag a navigation controller from the Object library onto the scene. You'll notice that a second view controller, a table view controller, automatically came along for the ride.
- Next, set the navigation controller to be the initial view controller by selecting the navigation controller and checking "Is Initial View Controller" in the Attributes inspector. Then, update the title in the navigation bar from "Root View Controller" to "Emoji Dictionary."
- Create a new Cocoa Touch Class called `EmojiTableViewController` as a subclass of `UITableViewController`. Back in your storyboard, select the table view controller and assign `EmojiTableViewController` as its custom class in the Identity inspector(1).
- Build and run you app. At this point, you should see an empty table view inside a navigation controller.

  - <img src="./resources/tableViewController.png" alt="Table View Controller" width="500" />

##### Table View Style

- Table views come in three styles: plain, grouped, and inset grouped.
  - The plain style is the default. In a plain table view, rows can be separated into labeled sections with an optional index along the right edge of the table (such as the alphabet index seen in Contacts). Each section immediately follows the previous section with no spacing, creating an unbroken list.
  - In a grouped table view, rows are displayed in visually distinct groups, or sections, with spacing in between - and without an index option.
  - An inset grouped table view is similar to grouped, but each visual group in inset on both sides of the view with rounded corners, providing a very pronounced separation between sections.

##### Table View Cells

- Every row in a table view is represented with a table view cell, a UITableViewCell instance. Cells are reusable views that can display text, images, or any other UIView. In addition to the cell content, each cell has an optional accessory view (more on accessory views later).

  - <img src="./resources/table_view_cell_non_editing.png" alt="Table View Cell - Non-Editing Mode" width="400" />

- In addition to the default, non-editing mode, table views can enter an editing mode, which enables users to insert, delete, or reorder cells. In editing mode, the cell content size shrinks and any accessory view disappears to allow room for the editing and reorder controls.

  - <img src="./resources/table_view_cell_editing.png" alt="Table View Cell - Editing Mode" width="400" />

- The UITableViewCell class defines three properties for cell content. 
  - `textLabel` and `detailTextLabel` are UILabels for the title and subtitle (or additional detail). 
  - `imageView` is a UIImageView for an image.
- These three properties have existed since the release of the iOS 2.0 SDK in 2009. UIKit has evolved since then, and table view cell configurations were introduced in iOS 14.0. Cell configurations let you describe what a cell should contain, rather than directly updating the cell's views yourself. The cell is responsible for taking the information in a configuration and updating its own views. This extra abstraction gives the cell more control over how to display its contents and lets you focus on the data you want it to display. (Also, the support for the textLabel, detailTextLabel, and imageView properties will be deprecated in a future version of iOS, so you should use cell configurations when possible.)
- To configure a cell, request its defaultContentConfiguration and set the properties of the returned content configuration. Then update the cell's contentConfiguration property with the updated configuration. The configuration has properties for text, secondaryText, image, and more to adjust the presentation of elements in a cell. You'll see cell content configurations in action shortly.
- The UIKit framework defines four standard cell styles that you can select in Interface Builder or programmatically, using the enum UITableViewCell.CellStyle. Each style supports its own combination of properties and has its own layout.
- | Storyboard Name | Programmatic Enum Name | Displays | Description |
  |---|---|---|---|
  | Basic | .default | textLabel, imageView | A label on the left side of the cell with black, left-aligned text; options leading image view. <img src="./resources/table_basic.png" alt="Basic" width="200" />|
  | Subtitle | .subtitle | textLabel, detailTextLabel, imageView | A label across the top with black, left-aligned text; a second label below in smaller, gray, left-aligned text; optional leading image view.  <img src="./resources/table_subtitle.png" alt="Subtitle" width="200" />|
  | Right Detail | .value1 | textLabel, detailTextLabel, imageView | A label on the left side of the cell with black, left-aligned text; a label on the right side with black(storyboard) or gray (programmatic), right-align text; optional leading image view. <img src="./resources/table_right_detail.png" alt="Right Detail" width="200" />|
  | Left Detail | .value2 | textLabel, detailTextLabel | A label on the left side of the cell with small, black, right aligned text, followed by a detail label with small, black (storyboard) or blue (programmatic), left-align text. <img src="./resources/table_left_detail.png" alt="Left Detail" width="200" />|
-  When the table is in default (view-only) mode, cells can display an accessory view, which you can also select in Interface Builder or programmatically, using the enum `UITableViewCell.AccessoryType`. The iOS SDK defines five standard types of accessory views, some of which can respond to user touches, like a button. With a delegate method, you can respond and execute code when the user taps the accessory view. You'll learn about the delegate later in this lesson.
No matter which accessory view is displayed, your code is responsible for responding to the user's tap on a cell or its accessory view. Accessory views are just indicators to the user about the nature of the cell.
  - | Storyboard Name | Programmatic Enum Name | Description | Image |
    |---|---|---|---|
    | None | .none |  No accessory view is displayed; content view takes up the entire cell. | <img src="./resources/accessoryType_none.png" alt="" width="50" /> |
    | Disclosure Indicator | .disclosureIndicator | A chevron. Tapping the cell typically triggers a show segue. The accessory view does not track touches. | <img src="./resources/accessoryType_disclosure.png" alt="" width="50" /> |
    | Detail | .detailButton | An info button. The accessory view tracks touches; tapping it typically displays additional information about the cell's contents. | <img src="./resources/accessoryType_detail.png" alt="" width="50" /> |
    | Detail Disclosure | .detailDisclosureButton | An info button and a chevron. Tapping the cell typically triggers a show segue. The accessory view tracks touches and typically displays additional information about the cell's contents. | <img src="./resources/accessoryType_detail_disclosure.png" alt="" width="50" /> |
    | Checkmark | .checkmark | A checkmark. The accessory view does not track touches. | <img src="./resources/accessoryType_checkmark.png" alt="" width="50" /> |
- Return to your EmojiDictionary project. In the table view controller scene, notice there is one cell under the Prototype Cells header. If you select the cell and open the Attributes inspector, you'll see many adjustable options, including Style. Feel free to select the different styles to see the options. When you're done, select the Subtitle style. You will use the labels to display the emoji symbol and name on the title line and the description as the subtitle. This style gives you the most room for both of your labels.

##### Table View Readability Margins

- By default, table view cells take up the entire width of the table view. This may cause readability issues on large screens: It's hard to read a very long line of text, and short lines can be awkward when the label is far from its accessory view. But there's a fix. With the table view selected in your storyboard, navigate to the `Size inspector` and find and check the `Follow Readable Width` option. Now, the width of your table view cells will stay within readable margins.
- You can also set this option programmatically by setting the `cellLayoutMarginsFollowReadableWidth` property to true. You can do this in the viewDidLoad() method of EmojiTableViewController.

#### Index Paths

- Many of the API methods for table views either accept an index path as a parameter or return an index path. As you might guess, an index path points to a specific row in a specific section of a table view. You can access these values through the index path's row and section properties. Just like array indices, these values are zero-indexed.

  - <img src="./resources/table_index_path.png" alt="Index Paths" width="200" />

#### Arrays and Table Views

- Table views are ideal for displaying a collection of similar data and are typically backed by a collection of model objects. This collection is usually an array — though it's possible to use another collection, such as a dictionary. Why is an array the best match with a table view? First, ordering of an array is guaranteed and stable. Also, arrays have a property (count) to query how many objects they contain, which can then inform the table view how many rows to display. In addition, arrays and index path values are both zero-based, which simplifies accurate addressing.
- To display a list of emoji in EmojiDictionary, you'll need to create an array of Emoji and add it as a property to your EmojiTableViewController. Feel free to create your own list or use the array provided below. (While it could be argued that the plural of “emoji” is “emoji,” for the sake of clarity — always a programmer's goal—“emojis” is the best name for your array.)

  - ```swift
      var emojis: [Emoji] = [
          Emoji(symbol: "😀", name: "Grinning Face", description: "A typical smiley face.", usage: "happiness"),
          Emoji(symbol: "😕", name: "Confused Face", description: "A confused, puzzled face.", usage: "unsure what to think; displeasure"),
          Emoji(symbol: "😍", name: "Heart Eyes", description: "A smiley face with hearts for eyes.", usage: "love of something; attractive"),
          Emoji(symbol: "🧑‍💻", name: "Developer", description: "A person working on a MacBook (probably using Xcode to write iOS apps in Swift).", usage: "apps, software, programming"),
          Emoji(symbol: "🐢", name: "Turtle", description: "A cute turtle.", usage: "something slow"),
          Emoji(symbol: "🐘", name: "Elephant", description: "A gray elephant.", usage: "good memory"),
          Emoji(symbol: "🍝", name: "Spaghetti", description: "A plate of spaghetti.", usage: "spaghetti"),
          Emoji(symbol: "🎲", name: "Die", description: "A single die.", usage: "taking a risk, chance; game"),
          Emoji(symbol: "⛺️", name: "Tent", description: "A small tent.", usage: "camping"),
          Emoji(symbol: "📚", name: "Stack of Books", description: "Three colored books stacked on each other.", usage: "homework, studying"),
          Emoji(symbol: "💔", name: "Broken Heart", description: "A red, broken heart.", usage: "extreme sadness"), 
          Emoji(symbol: "💤", name: "Snore", description: "Three blue \'z\'s.", usage: "tired, sleepiness"),
          Emoji(symbol: "🏁", name: "Checkered Flag", description: "A black-and-white checkered flag.", usage: "completion")
      ]
    ```

##### Cell Dequeueing

- Before diving into displaying this data, you first need to learn about cell dequeueing. It's possible to create a table view that's responsible for displaying hundreds of thousands of model objects. Each of these objects will be displayed by a cell, and each could have multiple views. Every view displayed onscreen must exist in a device's finite memory. If the table view were to try to initialize a cell for each object, a large list would quickly cause the device to run out of memory, resulting in a crash.
- To address this issue, table views only load the visible cells — plus a few more to make sure scrolling stays smooth. As the user scrolls through a table view, cells leave the visual field, and others are displayed at the opposite end of the device. Typically, the cells entering the visual field share the same layout as the ones that left the visual field moments ago. Could you take the cell that just left and reuse it as the cell that's about to enter?
- In fact, this process — called dequeueing — is precisely how table views manage their cells. You can register each cell type in your table view with a `reuseIdentifier` string. The table view can then manage a stockpile of cells for each reuse identifier. When you want a cell, you use the table view instance method `dequeueReusableCell(withIdentifier:for:)`, passing in the reuse identifier for the desired cell type. This method asks the table view instance to provide a matching cell that's available for reuse: `let cell: UITableViewCell = tableView.dequeueResuableCell(withIdentifier: "Cell", for: indexPath)`
- You can register your cells in Interface Builder by selecting the prototype cell and adding a value to the reuse identifier field in the Attributes inspector. In EmojiDictionary, select your table view cell. Then, navigate to the Attributes inspector and add `EmojiCell` as the Identifier. Your cell is now registered.
- Through the process of dequeueing, the table view only creates and loads as many cells as absolutely necessary, rather than one cell for every piece of information displayed. Dequeueing saves memory and allows for a smooth flow when scrolling through table views — providing the best possible user experience.

#### Table View Protocols

- A dynamic table view object must have a data source object and may or may not have a delegate object. Following the MVC design model you learned about in an earlier lesson, the data source acts as an intermediary between the table view and your app's data model. The optional delegate manages the appearance (minus the actual cells) and the behavior of the table view.
- Since the UITableView class itself doesn't have the methods, properties, or data required to configure its own contents, it delegates that responsibility to other objects. The data source and delegate are often, but not always, the same object. Typically this object is your custom subclass of UITableViewController. The responsibilities of the data source and delegate are defined in the UITableViewDataSource and UITableViewDelegate protocols.
- As a reminder, protocols define methods and properties for the objects conforming to the protocol to implement. Now that you understand the general difference between the two table view protocols, you'll explore the specific methods defined by them.

#### Table View Data Source

- The data source, which adopts the `UITableViewDataSource` protocol, is responsible for providing the necessary data to the table view object.
- Take a moment to think about the minimum information a table view would need to display its contents. It needs to know how many things to display (i.e. how many rows are in the table) and what goes in each row (i.e. the contents of each cell).
- To provide this information, you implement the methods outlined in UITableViewDataSource.
- For now, you'll focus on three data source methods that provide the number of sections in the table view, the number of rows in each section, and the cell to display for each index path.

  - ```swift
      optional func numberOfSections(in tableView: UITableView) -> Int
      func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int
      func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell
    ```

- When the table view loads or reloads its data, it queries the data source by calling each of these methods (sometimes multiple times) to request information for the visible rows. As the user scrolls through the table view and different rows become visible, the table view continues to query the data source for information to fill the new rows about to be displayed. The data source is responsible for returning the requested information based on the values of the parameters passed into the methods.
- In the next three sections, you'll explore the parameters of each function and your job, as the programmer, when implementing these functions.
- **Number of Sections**
  - In `numberOfSections(in:)`, you return as an Int the number of sections you want the table view to display. You probably noticed in the code above that this function has the optional modifier. If you choose not to write this function, the table view will display one section.
- **Number of Rows in a Section**
  - Once the table view knows how many sections to display, it needs to know how many rows there will be in each section. The `tableView(_:numberOfRowsInSection:)` method has two parameters: the table view requesting the information and the section in question. Based on these parameters, your job is to return an Int representing the number of rows for the given section. This required method will be called for every section in your table view.
- **Cell for Row at Index Path**
  - The table view now knows how many sections to display and how many rows there should be in each section. `tableView(_:cellForRowAt:)` provides you with an index path, and in the body of the method you dequeue, configure, and return a cell for the table view to display.
  - While different situations will call for different implementations of this method, most will adhere to the following steps:
    1. Fetch the correct cell type by dequeueing a cell.
    2. Fetch the model object to be displayed.
    3. Configure the cell's properties with the model object's properties — in other words, set views (labels, image views, etc.) based on the model object.
    4. Return the fully configured cell.

#### Implement the Data Source

- It's time to implement your table view's data source. Since you're using a UITableViewController subclass, your table view already has its data source assigned as the EmojiTableViewController instance. Open the Swift file for your emoji table view controller. Notice that the three data source methods are already defined for you, but with comments warning you that the implementations are incomplete.
  - The first method that will be called is `numberOfSections(in:)`. EmojiDictionary will have one section, so delete the comment inside the body of the method and return 1. (You could also remove the implementation of numberOfSections(in:) completely, and the table view would assume that it has one section. However, it's a good habit to clearly define your table views rather than relying on default values.)
  - Next, the table view will call `tableView(_:numberOfRowsInSection:)` to determine how many rows to place inside the section. You'll know which section is being requested via the parameter section. In EmojiDictionary, you only have one section. Delete the comment in tableView(_:numberOfRowsInSection:) and add the following to its body: `return emojis.count`
    - The code above returns the count of your emojis array. This means that if you were to allow user input to add to the array of emoji, your table view could be updated at runtime to match.
      - For example, if you added an emoji to the emojis array, the count of the array would increase by one, and your table view would display an additional row.
    - Since the EmojiDictionary table view has only one section, there's no need to use the section parameter of the method. If you were to add more sections in the future, you'd need to add conditional logic to return the correct number of cells in each section.
- Now that the table view knows how many rows to display, the table view will call `tableView(_:cellForRowAt:)` to get a cell for each row. To expose the method, you'll need to uncomment it. Delete the /* before the method and the */ after it. Next, you'll implement the method to follow the dequeueing steps detailed above.
  - First, you'll ask the table view to dequeue a cell. The following line (provided in the template with a placeholder for the reuse identifier) returns a UITableViewCell instance of the same style you registered in Interface Builder with the EmojiCell identifier: `let cell = tableView.dequeueReusableCell(withIdentifier: "EmojiCell", for: indexPath)`
  - The second step is to get the appropriate model object to display in the cell — in this example, that's the Emoji instance associated with that row. Using the indexPath parameter, you can get the array index required to retrieve the correct Emoji. Recall that the index path has two properties to access the cell address: `section` and `row`. Since your table view only has one section, you can ignore the section property, as it will be the same (in this case) every time the method is called.
    - You'll use the `row` property instead. Add the following line: `let emoji = emojis[indexPath.row]`
  - Your third step is to configure the cell with information about the emoji. Using the example cell above, you'll modify the cell's view to display the emoji info. Add the following lines:

    - ```swift
        var content = cell.defaultContentConfiguration()
        content.text = "\(emoji.symbol) - \(emoji.name)"
        content.secondaryText = emoji.description
        cell.contentConfiguration = content
      ```

    - These lines update the cell's content configuration so that a string consisting of the `symbol` and `name` will be rendered in the primary text position and the `description` will be rendered in the secondary text position. (Your usage property will be put to use in the next lesson.)
  - At last, your cell is looking the way you want. The final line from the template returns the configured cell: `return cell`
- You've completed the basic implementation of a table view data source, which means your table view now has enough information to display content.
- Build and run your app. At this point, you should see a filled table view with one row for each emoji in your array.
- Take some time to add more emoji to the emojis array. Build and run your app. Notice that without changing any code your table view was able to handle the additional data and display more cells, demonstrating the power that comes from combining an array with a table view.

#### Table View Delegate

- The second protocol in the table view API is the delegate. The delegate object, which conforms to the UITableViewDelegate protocol, implements methods to modify visible aspects of the table view, manage selections, support an accessory view, and support editing of individual rows. Unlike the data source protocol, the delegate protocol has no required methods.
- Although several of the delegate methods are beyond the scope of this lesson, there are two that are important to learn right now:
  - `tableView(_:accessoryButtonTappedForRowWith:)` — This method is used to respond to user interaction with the accessory view.
  - `tableView(_:didSelectRowAt:)` — This method is used to respond to user interaction with the cell's content area.
- **Accessory Button Tapped for Row**
  - As you learned earlier in the lesson, two of the accessory types track user interaction. Accessory button tapped for row is called if the cell has a detail or a detail disclosure accessory type and the user taps the detail indicator. This gives you the opportunity to respond to the user interaction, and you'll be given the index path of the row that contains the tapped accessory button so you can get the corresponding model object.
- **Did Select Row**
  - In a table view, you can respond to user interaction by implementing the `tableView(_:didSelectRowAt:)` method of the table view delegate. Any time the user taps a cell, the table view will select that row, provided the cell's selection style isn't set to `.none`. By default, the table view changes the background of a selected cell from white to gray and deselects any previously selected cell. The currently selected index path is accessible through the table view property `indexPathForSelectedRow`. Once the selection is completed, `tableView(_:didSelectRowAt:)` is called, and you're provided with the index path of the selected cell.
- **Implement the Delegate**
  - In your EmojiDictionary app, you're using a UITableViewController subclass to manage the table view. That table view controller, as you have seen, was automatically assigned as the table view data source, and it is also assigned as the table view delegate.
  - Add the delegate method `tableView(_:didSelectRowAt:)` to your EmojiTableViewController. In this method, print the emoji symbol and the index path of the row tapped.

    - ```swift
        override func tableView(_ tableView: UITableView, didSelectRowAt
          indexPath: IndexPath) {
            let emoji = emojis[indexPath.row]
            print("\(emoji.symbol) \(indexPath)")
        }
      ```
 
- Build and run your app. As you select rows, you should see the background color change and your message printed in the console.

#### Edit the Table View

- In addition to the data source methods you've already seen, there are others that allow for modifying the data in the table view. For example, `tableView(_:moveRowAt:to:)` reorders the cells displayed by a table view. Cells display the reorder control based on three factors:
  - The `UITableViewCell` class property `showsReorderControl`, a `Bool`, must be set to `true`. 
  - The data source method `tableView(_:moveRowAt:to:)` must be implemented. 
  - The table must be in editing mode.
- When it is displayed, the reorder control appears to the right of the cell's content and allows the user to drag cells to reorder them. The same factors will cause the delete button to display to the left of the content.
- Back in your project, add the following line to the method `tableView(_:cellForRowAt:)` method just before the return statement: `cell.showsReorderControl = true`
- Typically, but not necessarily, a table view will enter editing mode when the user taps an Edit button in the navigation bar. That seems like a good idea for your app. 
  - To enable this functionality, add a bar button item to the left slot of the table view controller's navigation bar. 
  - Choose Edit for the System Item property of the bar button, then add an @IBAction from the Edit button to the table view controller. 
  - Put the table view into editing mode using the setEditing(_:animated:) function; this will toggle the editing mode of the table view on and off:

    - ```swift
        @IBAction func editButtonTapped(_ sender: UIBarButtonItem) {
            let tableViewEditingMode = tableView.isEditing
         
            tableView.setEditing(!tableViewEditingMode, animated: true)
        }
      ```

- Alternatively, you can add an Edit button programmatically that will give you the same functionality without requiring any additional actions.
  - In viewDidLoad(), set the `leftBarButtonItem` to `editButtonItem`, a predefined button that switches the table view's editing mode on and off.

    - ```swift
        override func viewDidLoad() {
            super.viewDidLoad()
            navigationItem.leftBarButtonItem = editButtonItem
        }
      ```

- Next, implement the `tableView(_:moveRowAt:to:)` method. When called, it should remove the data within emojis at `fromIndexPath.row` and add it back at `to.row`:

  - ```swift
      override func tableView(_ tableView: UITableView, moveRowAt fromIndexPath: IndexPath, to: IndexPath) {
          let movedEmoji = emojis.remove(at: fromIndexPath.row)
          emojis.insert(movedEmoji, at: to.row)
      }
    ```

- Build and run your app. Enter editing mode. You should now see the reorder controls. Go ahead and use one to rearrange the cells.
- Maybe you only want users to be able to reorder items, not delete them. To remove the delete indicator, you could add the following method to the table view controller:

  - ```swift
      override func tableView(_ tableView: UITableView, editingStyleForRowAt indexPath: IndexPath) -> UITableViewCell.EditingStyle {
          return .none
      }
    ```

#### Reload Data

- The time will come when you need to reload your table view's data. Imagine, for example, that a user adds a new model object in a separate view controller and then returns to an already loaded table view. The table view won't reflect the addition.
- Table views have a built-in instance method, called `reloadData()`, that forces a refresh. If you want to refresh the table view with new data when a user returns to the view, you could add the call to `reloadData()` in the view will appear method.

  - ```swift
      override func viewWillAppear(_ animated: Bool) {
          super.viewWillAppear(animated)
          tableView.reloadData()
      }
    ```

- With your new skills, you'll be able to confidently and efficiently build a custom display to present whatever model objects your app uses and respond to user interactions with your table view. Make sure to save your EmojiDictionary project as you will continue to build on it in the next table view lesson.

  - ```swift
      import UIKit
      class EmojiTableViewController: UITableViewController {
          var emojis: [Emoji] = [
              Emoji(symbol: "😀", name: "Grinning Face", description: "A typical smiley face.", usage: "happiness"),
              Emoji(symbol: "😕", name: "Confused Face", description: "A confused, puzzled face.", usage: "unsure what to think; displeasure"),
              Emoji(symbol: "😍", name: "Heart Eyes", description: "A smiley face with hearts for eyes.", usage: "love of something; attractive"),
              Emoji(symbol: "🧑‍💻", name: "Developer", description: "A person working on a MacBook (probably using Xcode to write iOS apps in Swift).", usage: "apps, software, programming"),
              Emoji(symbol: "🐢", name: "Turtle", description: "A cute turtle.", usage: "something slow"),
              Emoji(symbol: "🐘", name: "Elephant", description: "A gray elephant.", usage: "good memory"),
              Emoji(symbol: "🍝", name: "Spaghetti", description: "A plate of spaghetti.", usage: "spaghetti"),
              Emoji(symbol: "🎲", name: "Die", description: "A single die.", usage: "taking a risk, chance; game"),
              Emoji(symbol: "⛺️", name: "Tent", description: "A small tent.", usage: "camping"),
              Emoji(symbol: "📚", name: "Stack of Books", description: "Three colored books stacked on each other.", usage: "homework, studying"),
              Emoji(symbol: "💔", name: "Broken Heart", description: "A red, broken heart.", usage: "extreme sadness"),
              Emoji(symbol: "💤", name: "Snore", description: "Three blue \'z\'s.", usage: "tired, sleepiness"),
              Emoji(symbol: "🏁", name: "Checkered Flag", description: "A black-and-white checkered flag.", usage: "completion")
          ]

          override func viewDidLoad() {
              super.viewDidLoad()
              navigationItem.leftBarButtonItem = editButtonItem
          }

          override func numberOfSections(in tableView: UITableView) -> Int {
              return 1
          }

          override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
              return emojis.count
          }

          override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
              let cell = tableView.dequeueReusableCell(withIdentifier: "EmojiCell", for: indexPath)
              let emoji = emojis[indexPath.row]
              
              var content = cell.defaultContentConfiguration()
              content.text = "\(emoji.symbol) - \(emoji.name)"
              content.secondaryText = emoji.description
              cell.contentConfiguration = content
              
              cell.showsReorderControl = true
              
              return cell
          }

          override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
              let emoji = emojis[indexPath.row]
              print("\(emoji.symbol) \(indexPath)")
          }
          override func tableView(_ tableView: UITableView, moveRowAt fromIndexPath: IndexPath, to: IndexPath) {
              let movedEmoji = emojis.remove(at: fromIndexPath.row)
              emojis.insert(movedEmoji, at: to.row)
          }

          override func tableView(_ tableView: UITableView, editingStyleForRowAt indexPath: IndexPath) -> UITableViewCell.EditingStyle {
              return .none
          }
          
          override func viewWillAppear(_ animated: Bool) {
              super.viewWillAppear(animated)
              tableView.reloadData()
          }
      }
    ```

#### Challenge - Table Views

- With your new skills, you'll be able to confidently and efficiently build a custom display to present whatever model objects your app uses and respond to user interactions with your table view. Make sure to save your EmojiDictionary project, as you will build on it in the next table view lesson.

#### Lab - Meal Tracker

- The objective of this lab is to give you a chance to practice implementing a basic table view. You'll create an app that will display a list of foods grouped into three sections, one for each meal of the day.
- Create a new project called "Meal Tracker" using the iOS App template. When creating the project, make sure the interface option is set to "Storyboard."

##### Step 1 - Set Up the Storyboard

- Delete the UIViewController that comes with the storyboard and its associated ViewController file. Add a UITableViewController from the Object library.
- Embed the scene in a navigation controller. Check `Is Initial View Controller` in Attributes inspector of Navigation Controller.
- The table view will default to having one prototype cell. 
  - Change its Style to `Subtitle`.
  - Update the reuse identifier to `Food`.
- Select the table view and change its Style to `Grouped`.
- Add a new Cocoa Touch Class file called `FoodTableViewController` that subclasses UITableViewController. Assign that class to the table view controller in your storyboard.

##### Step 2 - Create Model Objects

- Create two new Swift files. Name one “Meal” and the other “Food.”
- In the Food file, create a Food struct. Give it two properties, `name` and `description`, both of type `String`.
- In the Meal file, create a Meal struct. Give it two properties, `name` and `food`. The name property should be of type `String`, and food should be a type `[Food]`.

##### Step 3 - Implement UITableViewDataSource

- In the FoodTableViewController, remove all the template code except for `numberOfSections(in:)`, `tableView(_:numberOfRowsInSection:)`, and `tableView(_:cellForRowAt:)`.
- Create a computed property, `meals`, of type [Meal].
  - This property will return three Meal objects that each have three Food objects. The meals will represent `breakfast`, `lunch`, and `dinner`.
  - In the body of the property, create three Meal objects. Name them breakfast, lunch, and dinner. Set their food values to [] for now.
  - Return the three Meal objects in an array with the order: breakfast, lunch, dinner.
  - Create three Food objects for the food array on each Meal object. Give each Food item an appropriate name for its corresponding meal.

    - ```swift
        var meals: [Meal] {
            let aBreakfast = Food(name: "Eggs", description: "Scrambled with bacon")
            let bBreakfast = Food(name: "Cooked Rice", description: "Boiled rice by pressed pot with side dishes")
            let cBreakfast = Food(name: "Salad", description: "Cabbage, Cucumber, Almond, and Dressing")
            let breakfast = Meal(name: "Breakfast", food: [aBreakfast, bBreakfast, cBreakfast])
            
            let aLunch = Food(name: "Soup", description: "Mushroom soup")
            let bLunch = Food(name: "Pizza", description: "Pepperoni pizza")
            let cLunch = Food(name: "Pork belly", description: "Grilled port belly")
            let lunch = Meal(name: "Lunch", food: [aLunch, bLunch, cLunch])
            
            let aDinner = Food(name: "Milk", description: "2% Milk")
            let bDinner = Food(name: "Ramen", description: "Shin ramen")
            let cDinner = Food(name: "Seaweed Soup", description: "")
            let dinner = Meal(name: "Breakfast", food: [aDinner, bDinner, cDinner])
            
            return [breakfast, lunch, dinner]
        }
      ```

- In `numberOfSections(in:)`, return the number of meals in your meals array. `return meals.count`
- In `tableView(_:numberOfRowsInSection:)`, access the meal for the given section and return the number of food items in that meal. `return meals[section].food.count`
- In `tableView(_:cellForRowAt:)`, 
  - dequeue a cell with the reuse identifier `Food`.
  - Access the meal for the row using `indexPath.section`.
  - From the returned meal, access the food for the row using `indexPath.row`.
  - Get the cell's default content configuration, update the content configuration's text and secondary text to be the `name` and `description` of the food item, and assign it to the cell's content configuration before returning the cell.

    - ```swift
        override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
            let cell = tableView.dequeueReusableCell(withIdentifier: "Food", for: indexPath)
            let meal = meals[indexPath.section]
            let food = meal.food[indexPath.row]

            var content = cell.defaultContentConfiguration()
            content.text = "\(food.name)"
            content.secondaryText = "\(food.description)"
            cell.contentConfiguration = content

            return cell
        }
      ```

- In `tableView(_:titleForHeaderInSection:)`, return the `name` of the meal that corresponds to the section.

  - ```swift
      override func tableView(_ tableView: UITableView, titleForHeaderInSection section: Int) -> String? {
          return meals[section].name
      }
    ```

- Congratulations! You've made an app that displays a list of meals and food items in a table view. Be sure to save it to your project folder for future reference.

  - ```swift
      import UIKit
      class FoodTableViewController: UITableViewController {
          var meals: [Meal] {
              let aBreakfast = Food(name: "Eggs", description: "Scrambled with bacon")
              let bBreakfast = Food(name: "Cooked Rice", description: "Boiled rice by pressed pot with side dishes")
              let cBreakfast = Food(name: "Salad", description: "Cabbage, Cucumber, Almond, and Dressing")
              let breakfast = Meal(name: "Breakfast", food: [aBreakfast, bBreakfast, cBreakfast])
              
              let aLunch = Food(name: "Soup", description: "Mushroom soup")
              let bLunch = Food(name: "Pizza", description: "Pepperoni pizza")
              let cLunch = Food(name: "Pork belly", description: "Grilled port belly")
              let lunch = Meal(name: "Lunch", food: [aLunch, bLunch, cLunch])
              
              let aDinner = Food(name: "Milk", description: "2% Milk")
              let bDinner = Food(name: "Ramen", description: "Shin ramen")
              let cDinner = Food(name: "Seaweed Soup", description: "")
              let dinner = Meal(name: "Breakfast", food: [aDinner, bDinner, cDinner])
              
              return [breakfast, lunch, dinner]
          }
          
          override func viewDidLoad() {
              super.viewDidLoad()
          }

          override func numberOfSections(in tableView: UITableView) -> Int {
              return meals.count
          }

          override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
              return meals[section].food.count
          }

          override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
              let cell = tableView.dequeueReusableCell(withIdentifier: "Food", for: indexPath)
              let meal = meals[indexPath.section]
              let food = meal.food[indexPath.row]

              var content = cell.defaultContentConfiguration()
              content.text = "\(food.name)"
              content.secondaryText = "\(food.description)"
              cell.contentConfiguration = content

              return cell
          }

          override func tableView(_ tableView: UITableView, titleForHeaderInSection section: Int) -> String? {
              return meals[section].name
          }
      }
    ```

### Lesson 1.6 Intermediate Table Views

- In the previous lesson, you learned how to build basic table views. While the styles and functionality provided by basic table views will get you far, you'll often find you want more customization. In this lesson, you'll learn how to create custom table view cells and use row actions to add and delete table view rows. You'll also learn how you can use table views to display static data without the overhead of implementing the table view's data source.

#### Custom Table View Cells

- Throughout this section, you'll build on the EmojiDictionary project you set up in the previous table view lesson. In that lesson, you used the table view cell styles provided in the iOS SDK. But what if you wanted further customization? For example, table view cells in the Mail app display preview text so the user can read a piece of each message at a glance. In the App Store, cells display app names and an image of the app as well as a button that allows users to download the app right from the table view.
- In EmojiDictionary, the default cell displaying the emoji works — but emoji are all about the symbol. Wouldn't it be nice if the symbol were displayed more prominently?
- Since emoji are characters, it won't work to use the included image view. Instead, you'll build a custom table view cell that adds a third, larger label aligned to the leading edge of the cell to display the emoji.
- To start creating a custom table view cell in the EmojiDictionary project, open the Main storyboard, select the table view cell, and set its Style in the Attributes inspector to `Custom`. You now have a blank cell to work with.
- You can customize this empty cell using the tools provided in Interface Builder in the same way you would customize a scene's main view.
- Add a horizontal stack view to the cell's content view with Spacing set to 8. Use the Add New Constraints tool to constrain the stack view to the margins of the cell (make sure that the "Constrain to margins" checkbox is checked). 
- In the horizontal stack view, add a label and, to the right of the label, a vertical stack view. The horizontal stack view on the canvas is probably very short, which will make it difficult to drag in the labels. Instead, you can drag the labels to the Stack View item in the Document Outline. The left label will contain the emoji, so replace the label text with a sample emoji. (You can bring up the emoji keyboard on your Mac using Control+Command+Spacebar in a text field.) Set the font size to 24 and the Alignment to centered.
- Add two labels to the vertical stack view. Set the vertical stack view's Distribution property to `Fill Equally` so that the two labels share equal space.
- At this point, you'll probably notice that the Auto Layout engine is giving too much width to the emoji label compared to the vertical stack view and its labels.
- **Content Hugging**
  - How much horizontal space should the emoji label use compared to the labels in the vertical stack view? It would be nice if the emoji symbol used as much space as it needed to fit its contents, and then allowed all remaining horizontal space to be dedicated to the vertical labels.
  - In short, the emoji label's width should hug its contents. To achieve this, select the emoji label and open the Size inspector. Under Content Hugging Priority, change the value in the Horizontal field from `251` to `252`.
    - This number is a relative number indicating how high of a priority the Auto Layout engine should place on making the view hug its content. In this case, the Horizontal Content Hugging Priority of each of the other labels is set to 251, so the Auto Layout engine prioritizes shrinking the emoji label to fit its content over shrinking the other labels to fit their content.
  - The end result is that the emoji label is only as wide as it needs to be to display the entire emoji symbol, while the other labels fill the rest of the horizontal space.
  - Touch up your cell's design by setting the top label's font to the `Title 3` Text Style, provided by the system, and the lower label's font to `Subhead`. You can also set the values for these labels to `Name` Label and `Description` Label in the storyboard to help remind you of their purposes.
- **Create Cell Subclass**
  - Now that your cell is visually set up in the storyboard, you'll need to create a custom table view subclass. This will let you create outlets that you can use to configure the cell. Create a new Cocoa Touch Class file named “EmojiTableViewCell” and set its subclass to UITableViewCell.
  - Back in the storyboard, update the identity of the cell to be an EmojiTableViewCell. 
  - In the EmojiTableViewCell class, add outlets for each of the labels: symbolLabel, nameLabel, and descriptionLabel. Also, add a function called update(with emoji: Emoji). This function will take an Emoji instance and use it to update the cell's labels appropriately. Try adding the implementation of this function on your own before looking at the following code:

    - ```swift
        func update(with emoji: Emoji) {
            symbolLabel.text = emoji.symbol
            nameLabel.text = emoji.name
            descriptionLabel.text = emoji.description
        }
      ```

  - Using an `update(with:)` method on a custom table view cell is a common pattern for abstracting, or moving, setup code from the `tableView(_:cellForRowAt:)` method into the cell itself.
  - At this point, your custom cell class is ready for use. The last step will be to update the `tableView(_:cellForRowAt:)` method to use your new cell. 
    - The first line involves the dequeueing process. When you dequeue a cell, the method returns a UITableViewCell instance. You'll need to force - downcast the dequeued cell to your EmojiTableViewCell class so that you can use the update(with:) method you defined earlier. The new line will look like this: `let cell = tableView.dequeueReusableCell(withIdentifier: "EmojiCell", for: indexPath) as! EmojiTableViewCell`
    - The second thing you'll need to do is update how the cell is configured. Instead of using the default cell's text and detail labels, you'll use your recently created `update(with:)` method. Here's the cell configuration part of `tableView(_:cellForRowAt:)`: `cell.update(with: emoji)`, `cell.showsReorderControl = true`
    - Your final `tableView(_:cellForRowAt:)` implementation will now look like this (possibly without the comments):

      - ```swift
          override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
              //Step 1: Dequeue cell
              let cell = tableView.dequeueReusableCell(withIdentifier: "EmojiCell", for: indexPath) as! EmojiTableViewCell
           
              //Step 2: Fetch model object to display
              let emoji = emojis[indexPath.row]
           
              //Step 3: Configure cell
              cell.update(with: emoji)
              cell.showsReorderControl = true
           
              //Step 4: Return cell
              return cell
          }
        ```

  - Build and run your app. You should see your emoji displayed with your custom table view cells.

#### Edit Table Views

- Once the table view enters editing mode, it calls its delegate method `tableView(_:editingStyleForRowAt:)` to determine the editing style of each row. In the previous lesson, you might have used this method to set the editing style of all rows to have no edit control displayed:

  - ```swift
      override func tableView(_ tableView: UITableView, editingStyleForRowAt indexPath: IndexPath) -> UITableViewCell.EditingStyle {
          return .none
      }
    ```

- In addition to `.none`, there are two other styles: `.delete` and `.insert`.
- When your table view enters editing mode, several data source and delegate methods are called in sequence. The steps, including user input, include:
  1. `tableView(_:canEditRowAt:)` — This data source method is used to exclude rows from being edited. Note that this method isn't necessary for most apps.
  2. `tableView(_:editingStyleForRowAt:)` — This delegate method designates the row's editing style. If you don't implement this method, the table view will display the deletion control. Following this method, the table view will be fully in editing mode.
  3. The user taps an editing control. If it's a deletion control, the row displays a Delete button for confirmation.
  4. `tableView(_:commit:forRowAt:)` — This data source method updates your data model to reflect the user's action in step 3. Although this protocol method is marked optional, the data source must implement it if it needs to delete or insert rows. It must do two things:
     - Update the corresponding data source by either deleting the referenced item or adding an item.
     - Send `deleteRows(at:with:)` or `insertRows(at:with:)` to the table view to remove or insert the appropriate rows.
- **Delete Items**
  - Like most apps, EmojiDictionary won't exclude specific rows from being edited. Therefore, you can ignore step 1.
  - For step 2, you'll need to tell the table view which controls, if any, should be displayed. In EmojiDictionary, you'll display the delete control. If you didn't implement the `tableView(_:editingStyleForRowAt:)` delegate method in the first table view lesson, add it now — and if you already added it, ensure that it returns .delete:

    - ```swift
        override func tableView(_ tableView: UITableView, editingStyleForRowAt indexPath: IndexPath) -> UITableViewCell.EditingStyle {
            return .delete
        }
      ```

  - In step 3, the user's action leads to the implementation of `tableView(_:commit:forRowAt:)` in step 4. When the user taps a control, the app must update the model data backing the table view and the table view itself. For your convenience, Apple has included this method header in table view controllers. You just have to remove the `/*` and the `*/`.
  - In the implementation, remove the emoji from the emojis array using the given index path. Next, call the table view's `deleteRows(at:with:)` method. This method takes in an array of index paths, specifying the rows to delete, and a `UITableView.RowAnimation` enum. There are several different row animations. To see a full list, take a look at the [documentation](https://developer.apple.com/documentation/uikit/uitableview/rowanimation). For now, implement the method using `.automatic` for the row animation.

    - ```swift
        override func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath) {
            if editingStyle == .delete {
                emojis.remove(at: indexPath.row)
                tableView.deleteRows(at: [indexPath], with: .automatic)
            }
        }
      ```

- Build and run your app. If you tap the Edit button, the table view should enter editing mode and give you the option to delete rows. If you tap the delete control, that row should animate away.
- Implementing the ability to delete rows also automatically enables swipe to delete. Try swiping on a row from right to left to make sure it animates away.

#### Add and Edit Emoji

- What if you want your users to be able to add or edit the emoji in your dictionary app? Right now, the only solution is to modify the emojis array property — but your users don't have access to your code.
- How could you add this capability? For adding an emoji, you could use the `.insert` control on a blank row. Another approach would be to add an Add (+) button to the navigation bar and present a new view controller where users can input the properties of their addition — and you could use that same view controller to permit emoji editing. Two birds with one view controller!
- This is also a perfect case for a static table view.
- **Static Table Views**
  - As you can guess from the name, you'll use a static table view when the data it displays will be static — you know exactly how many rows and what types of cells are needed, regardless of the specific information it will display. Static table views are great for forms, settings, or anything where the number of rows doesn't change. In your EmojiDictionary project, your static table view will contain four cells, each with a text field to allow input and editing.
  - For static table views, you'll always use a table view controller — which should not implement the data source protocol. Instead, you'll use the viewDidLoad() method to populate the table view's data. When you create a new UITableViewController subclass, you'll need to remember to delete or comment out the data source methods that are provided in the file. Otherwise your static table view will attempt to use the empty data source methods, and you'll end up with an empty table view.
  - In your storyboard, add a navigation controller from the Object library to the scene, which will include a table view controller alongside it.
    - This new table view controller will be backed by a subclass of UITableViewController. Start by creating a new file to define a UITableViewController subclass called `AddEditEmojiTableViewController`. Remember to delete or comment out the table view data source methods in the file.
  - In the Main storyboard set the Root View Controller's Custom Class to `AddEditEmojiTableViewController`. In AddEditEmojiTableViewController, add a property called emoji with a type of Emoji?.
  - Next, create a custom initializer, as you'll be using @IBSegueAction to pass an Emoji when editing.

    - ```swift
        var emoji: Emoji?
        init?(coder: NSCoder, emoji: Emoji?) {
            self.emoji = emoji
            super.init(coder: coder)
        }
      ```

  - This will cause a compilation error, use the Xcode provided fix-it to resolve the issue.
  - Next, go back to your storyboard and add a bar button item to the Emoji Dictionary scene. Set the system item to Add. Create a modal presentation segue from the bar button item to the new navigation controller. Create a second modal presentation segue from the emoji cell to the navigation controller.
    - <img src="./resources/addEmoji.png" alt="Add Emoji Segue" width="500" />
  - According to the Human Interface Guidelines, you should modally present this static table controller when it's being used to create a new Emoji or to edit an existing Emoji. If, however, you're simply displaying data without the ability to edit, it should be pushed.
  - With the segue from the cell to the navigation controller in place, you are no longer interested in knowing when a row is selected — remove the `tableView(_:didSelectRowAt:)` method in EmojiTableViewController.
  - Your new segues both go from the list of emoji to the navigation controller, but for your @IBSegueAction you're interested in initializing the AddEditEmojiTableViewController — the scene after the navigation controller — to pass it an Emoji when editing. You will notice that Xcode has provided a segue between the navigation controller and the AddEditEmojiTableViewController; this is called a relationship segue, and it's what you'll use to create an @IBSegueAction.
  - Control-drag from the segue between the navigation controller and AddEditEmojiTableViewController into EmojiTableViewController and create an action segue with the name `addEditEmoji` and Arguments set to `Sender`.
    - <img src="./resources/addEditEmojiSegueAction.png" alt="Add Edit Emoji Segue Action" width="500" />
  - The new `addEditEmoji(_:sender:)` method will be called when the user taps the Add button or taps a row to edit an emoji. Add the following implementation.

    - ```swift
        if let cell = sender as? UITableViewCell,
          let indexPath = tableView.indexPath(for: cell) {
            // Editing Emoji

            let emojiToEdit = emojis[indexPath.row]
            return AddEditEmojiTableViewController(coder: coder, emoji: emojiToEdit)
        } else {
            // Adding Emoji

            return AddEditEmojiTableViewController(coder: coder, emoji: nil)
        }
      ```

  - When a cell is tapped, the sender parameter will be the cell, and you can use that to find the IndexPath — which translates back to the position in the emojis array. If the sender is not a cell, you can assume it was the Add button, so you create the `AddEditEmojiTableViewController` with a `nil` value for emoji.
  - Build and run your app. You should be able to tap the Add button or a cell and see your new table view. However, it won't be populated, and it is missing a dismiss button. While you can dismiss by swiping down, the Human Interface Guidelines state that it's best to always provide a button as well.
  - In the new table view controller scene, select the table view and open the Attributes inspector. Change the Content to Static Cells.
  - You can now start to build your static table view. In the Attributes inspector, adjust the number of sections to 4 and the Style of the table view to `Grouped`. Next, edit each section's attributes by selecting the section in the Document Outline and adjusting the settings in Attributes inspector. Set the number of rows in each section to 1 and add the following section headers (from top to bottom): “Symbol,” “Name,” “Description,” and “Usage.” Add a text field in each cell, and don't forget to add constraints.
  - Go ahead and test the app so far. Navigate back to your static table view controller, and enter the first text field. When the keyboard is presented, locate the emoji icon in the bottom left. Selecting it will switch to the Emoji keyboard. You can return to your default keyboard by pressing the ABC icon.
- **Pass Data to Static Table View**
  - Now that your static table view is set up, you need to fill in the text fields when emoji is not nil. Create outlets for each text field: `symbolTextField`, `nameTextField`, `descriptionTextField`, and `usageTextField`. In the AddEditEmojiTableViewController's viewDidLoad() method, check whether emoji has a value. If it does, then you're editing an existing Emoji; update the text of each text field with the corresponding emoji properties as well as the view's title. If the property is nil, then the static table view controller was presented to create a new Emoji:

    - ```swift
        if let emoji = emoji {
            symbolTextField.text = emoji.symbol
            nameTextField.text = emoji.name
            descriptionTextField.text = emoji.description
            usageTextField.text = emoji.usage
            title = "Edit Emoji"
        } else {
            title = "Add Emoji"
        }
      ```

  - If you try running on a smaller device, you'll notice that the keyboard initially covers the usageTextField and possibly the descriptionTextField when it slides up. But the table view controller automatically moves the text fields above the keyboard — unlike with a scroll view, where you needed to add code to get this behavior. This is another benefit of using static table view controllers to build input screens.
- **Add Action Buttons with Unwind Segue**
  - Currently, the AddEditEmojiTableViewController can only be dismissed by swiping it down. Add two bar button items to the navigation bar, one on each side of the title. For the button on the left, adjust the System Item to `Cancel`. For the button on the right, adjust it to `Save`. Both of these buttons should trigger the dismissal of the view controller by dismissing it modally.
  - Add an `unwindToEmojiTableView(segue:)` method in EmojiTableViewController. 
    - `@IBAction func unwindToEmojiTableView(segue: UIStoryboardSegue) {}`
  - You'll need a way to determine whether the unwind segue was triggered via the Save or Cancel button. The simplest way to do this is to give the segue initiated by the Save button an identifier. 
    - Connect the Save button to this segue by Control-dragging from the Save button to the view controller's Exit icon, then choose the method. Locate the segue in the Document Outline and open the Attributes inspector to set the Identifier to `saveUnwind`. Now do the same for the Cancel button, but do not worry about providing an identifier for the segue.
    - <img src="./resources/save_segue.png" alt="Save Segue" width="300" />
  - Build and run your application to verify that the AddEditEmojiTableViewController properly dismisses.
- **Update Save Button**
  - The Save button should only be enabled if every text field contains a value. The simplest way to enable and disable the button is to have a single method that checks every text field for a value and enables the button only if all four fields are not empty. Create an outlet for the Save button called `saveButton`, then implement a method that will check the text of every field.

    - ```swift
        func updateSaveButtonState() {
            let symbolText = symbolTextField.text ?? ""
            let nameText = nameTextField.text ?? ""
            let descriptionText = descriptionTextField.text ?? ""
            let usageText = usageTextField.text ?? ""
            saveButton.isEnabled = !symbolText.isEmpty && !nameText.isEmpty && !descriptionText.isEmpty && !usageText.isEmpty
        }
      ```

  - You should call this method in viewDidLoad() so that the button is disabled after being modally presented or enabled if you pushed to this view controller.

    - ```swift
        override func viewDidLoad() {
            super.viewDidLoad()
         
            if let emoji = emoji {
                symbolTextField.text = emoji.symbol
                nameTextField.text = emoji.name
                descriptionTextField.text = emoji.description
                usageTextField.text = emoji.usage
                title = "Edit Emoji"
            } else {
                title = "Add Emoji"
            }
         
            updateSaveButtonState()
        }
      ```

  - How do you continually check to see if the Save button should be enabled or disabled? You can call `updateSaveButtonState` after each key press. Create an @IBAction in code that will call the method.

    - ```swift
        @IBAction func textEditingChanged(_ sender: UITextField) {
            updateSaveButtonState()
        }
      ```

  - Next, open your storyboard and the assistant editor. Select a text field in the static table view and open the Connections inspector. Drag from the `Editing Changed` event to the `textEditingChanged(_:)` method. Repeat this step for the other text fields. 
  - Build and run the app and try adding text into the four fields. The Save button should only be enabled when all four fields contain text.
  - This is a good start, but it's not perfect: The symbolTextField should only allow a single emoji character. Input validation is a very important topic in software development — invalid inputs can cause your software to do unexpected things. 
  - Add the following method that checks whether the provided UITextField contains a single emoji character.

    - ```swift
        func containsSingleEmoji(_ textField: UITextField) -> Bool {
            guard let text = textField.text, text.count == 1 else {
                return false
            }
         
            let isCombinedIntoEmoji = text.unicodeScalars.count > 1 && text.unicodeScalars.first?.properties.isEmoji ?? false
            let isEmojiPresentation = text.unicodeScalars.first?.properties.isEmojiPresentation ?? false
         
            return isEmojiPresentation || isCombinedIntoEmoji
        }
      ```

  - This implementation uses methods, properties, and concepts you probably aren't familiar with, but you can often find solutions to small - scoped problems like this in reference documentation or on the internet.
  - Update the updateSaveButtonState() method to use this new validation.

    - ```swift
        func updateSaveButtonState() {
            let nameText = nameTextField.text ?? ""
            let descriptionText = descriptionTextField.text ?? ""
            let usageText = usageTextField.text ?? ""
            saveButton.isEnabled = containsSingleEmoji(symbolTextField) && !nameText.isEmpty && !descriptionText.isEmpty && !usageText.isEmpty
        }
      ```

  - Build and run the app to test out your input validation code. Confirm that the Save button is disabled unless the Name, Description, and Usage fields contain text and the Symbol field contains a single emoji character.

- **Save Emoji**
  - When the Save button is pressed, the saveUnwind segue is performed. Before the segue is triggered, you should use the text field values to construct a new Emoji instance and set it to your emoji property. In the unwind segue, the property can be used to update the emojis collection.
  - In `AddEditEmojiTableViewController`, add the `prepare(for segue:)` method. Ensure that the `saveUnwind` segue is being performed (you don't want to do any work when Cancel is pressed), then update the emoji property. Since you've performed validation with the Save button, don't be too concerned that you have to handle the optional values from text fields — your validation ensures they'll never be empty.

    - ```swift
        override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
            guard segue.identifier == "saveUnwind" else { return }
         
            let symbol = symbolTextField.text!
            let name = nameTextField.text ?? ""
            let description = descriptionTextField.text ?? ""
            let usage = usageTextField.text ?? ""
            emoji = Emoji(symbol: symbol, name: name, description: description, usage: usage)
        }
      ```

  - Back in the `unwindToEmojiTableView(segue:)` method, verify that the saveUnwind segue was triggered. If so, check whether the table view still has a selected row. If it does, then you're unwinding after editing a particular Emoji. If it does not, then a new Emoji is being created. To add a new entry into emojis, you'll need to calculate the index path for the new row and add the item to the end of the collection. Update the table view accordingly.

    - ```swift
        @IBAction func unwindToEmojiTableView(segue: UIStoryboardSegue) {
            guard segue.identifier == "saveUnwind",
                let sourceViewController = segue.source as? AddEditEmojiTableViewController,
                let emoji = sourceViewController.emoji else { return }
         
            if let selectedIndexPath = tableView.indexPathForSelectedRow {
                emojis[selectedIndexPath.row] = emoji
                tableView.reloadRows(at: [selectedIndexPath], with: .none)
            } else {
                let newIndexPath = IndexPath(row: emojis.count, section: 0)
                emojis.append(emoji)
                tableView.insertRows(at: [newIndexPath], with: .automatic)
            }
        }
      ```

  - Build and run your app. You should now be able to add new emoji or edit existing ones.
- **Automatic Row Height**
  - If you enter a long piece of text into descriptionTextField, you may have noticed that the text is truncated. The first step toward resolving this issue is to set the bottom label's number of lines to 0 in the Attributes inspector. But if you build and run the app, there's no change. What's going on?
  - Even though the description label's text can cover multiple lines, the cell height still defaults to 44. You could implement `tableView(_:heightForRowAt:)` and manually calculate the height that a cell should be to display all the text, but that would be very cumbersome and error-prone. Instead, you can tell the table view that its row height should be determined automatically based on the contents of the cell.
  - To begin, add the following lines of code into the `viewDidLoad()` method of EmojiTableViewController. This will tell the table view that it needs to calculate the cell height but also give it a sensible estimate for how tall the average cell will be, to improve performance.

    - ```swift
        tableView.rowHeight = UITableView.automaticDimension
        tableView.estimatedRowHeight = 44.0
      ```

  - If you have constraints properly placed on both the top and bottom of your views, UITableView can automatically calculate the height that each cell needs to be to display all the content inside its views. However, if you want to avoid warnings in Interface Builder, you might still need to help the Auto Layout engine know which view's content to prioritize.
- **Compression Resistance**
  - Interface Builder does not have access to your code, so it doesn't know that the cell height will be calculated automatically. From its perspective, you have a cell with a constant height and two labels. The Auto Layout engine is unsure what should happen if the text length extends past the current size of the label, so it's up to you to resolve the ambiguity.
  - Every view has a compression resistance value that represents how resistant the view is to shrinking. This is similar to the content hugging priority in that it is a relative value. The default is 750. In the Size inspector, set the Name Label's Vertical Compression Resistance to 751, telling the Auto Layout engine that it has a higher priority than everything else — including the cell itself — to avoid shrinking. Set the Description Label's Vertical Compression Resistance to 752 so that it is resistant to shrinking as well (a value of 751 would create ambiguity between the two labels).
  - Now, since the vertical stack view still has a compression resistance value of 750, the Auto Layout engine understands that the stack view should grow to accommodate a larger amount of text. The stack view height increase will be taken into account when the table view calculates the cell height via UITableView.automaticDimension.
  - Build and run you app, supplying long text into the descriptionTextField. You should see the cell height increase.

#### Lab - Favorite Books

- **Object**
  - The objective of this lab is to implement intermediate table view features into an app that keeps track of your favorite books.
  - You'll find a starter project called “FavoriteBooks” in your student resources folder. Take a minute to look it over and run it to see how it behaves. You'll notice that it contains a table view controller that segues to a regular view controller. The regular view controller contains a form that allows the user to enter details about a book. In this lab, you'll replace the form with a static table view, add the capability to delete books from the main list of books, and create a custom table view cell to better display the details of each book in the main list.
- **Step 1. Create a Static Table View**
  - Drag a table view controller onto the storyboard. With the table view selected, use the Attributes inspector to set the Content to `Static Cells` and the Style to `Grouped`.
  - You'll be creating a form that models your existing form, where each text field is in its own cell and each cell is in its own section. You'll want five sections. In the storyboard, you can set the section header titles to “Title,” “Author,” “Genre,” and “Length.” The last section will be for the Save button.
  - Place a text field in each of the top four cells and a button in the last cell — setting appropriate constraints.
  - Create a new Cocoa Touch Class file named “BookFormTableViewController” and make sure it subclasses UITableViewController.
  - Your static table view doesn't need most of the boilerplate code that's included with this new subclass. Go ahead and delete all of it except for viewDidLoad().
  - Set the class of your table view controller in the storyboard and create outlets for your text fields and an action for your Save button.
- **Step 2 Replace BookFormViewController ​with BookFormTableViewController**
  - To transfer the functionality of BookFormViewController to BookFormTableViewController, you can copy and paste the entire implementation of BookFormViewController over — but you will need to rewire the text field outlets and save action to your new scene in the storyboard.
  - The `saveButtonTapped(_:)` method performs an unwind segue. For the unwind segue to work, you'll need to connect it to your BookFormTableViewController in the storyboard and give the segue the `UnwindToBookTable` identifier. You'll also need to update the implementation of `prepareForUnwind(segue:)` in BookTableViewController: Where segue.source is cast to BookFormViewController, this now needs to be BookFormTableViewController.
    - In the storyboard, Control-Drag from BookFormTableViewController to the Exit and select `prepareForUnwindWithSegue`. Click the Unwind segue in the Document Outline and type the Identifier in the Attribute Inspector to `UnwindToBookTable`.
  - In the storyboard, remove the original form view controller and add the table view controller in its place. Remember the segues that existed with the view controller you just deleted? Re-create the same two segues to this table view controller and wire up the Edit segue to the `editBook(_:sender:)` @IBSegueAction in BookTableViewController.
    - Control-Drag from the Add button and BookCell to BookFormTableViewController, and select Show in the Action segue.
    - Type the identifier in the Attribute inspector to `AddBook` and `EditBook` respectively.
    - To insert a segue action, Control-Drag from EditBook segue in the storyboard to BookTableViewController and the Name is editBook.
    - Copy and past the code into it.

      - ```swift
          guard let cell = sender as? UITableViewCell, let indexPath = tableView.indexPath(for: cell) else {
              return nil
          }
          
          let book = books[indexPath.row]
          
          return BookFormTableViewController(coder: coder, book: book)
        ```

  - Run the app to make sure it behaves as it did before, only with a different visual form.
- **Step 3 Delete Books**
  - What if the user wants to remove a book from the list? Go to your BookTableViewController and add the table view method `tableView(_:commit:forRowAt:)`.
  - To this method, add control flow that will check whether the editing style is `.delete`; if it is, remove the Book object in the Books array that corresponds to the index path in question, then remove the table view row at the same index path.

    - ```swift
        override func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath) {
            if editingStyle == .delete {
                books.remove(at: indexPath.row)
                tableView.deleteRows(at: [indexPath], with: .automatic)
            }
        }
      ```

  - Run the app and confirm that you can delete books from the list.
- **Step 4 Create a Custom Book Cell**
  - Maybe you want to customize the display of your books' information. In your BookTableViewController, change the Style of the prototype cell to Custom.
  - You can now configure the cell however you'd like. For example, at the top, you might want a label for the book title; under it, you might want labels for author, genre, and length that are somewhat smaller and slightly indented. Or you could have six labels, in addition to the title label, arranged into three rows of two, where the labels on the left say “Author,” “Genre,” and “Length,” and the labels on the right contain the actual values corresponding to author, genre, and length. Go ahead and design your own custom prototype cell. Make sure it displays all the information contained in a Book object.
  - To create outlets for your labels, you'll need a custom UITableViewCell subclass. Create a new file that subclasses UITableViewCell named “BookTableViewCell.”
  - In the storyboard, set the type of your custom cell to be a BookTableViewCell, then create outlets for the labels.

    - ```swift
        @IBOutlet var titleTextField: UILabel!
        @IBOutlet var authorTextField: UILabel!
        @IBOutlet var genreTextField: UILabel!
        @IBOutlet var lengthTextField: UILabel!
      ```

  - In BookTableViewCell, create a method called `update(with book: Book)` that will set the labels properly for each book cell.

    - ```swift
        func update(with book: Book) {
            titleTextField.text = book.title
            authorTextField.text = book.author
            genreTextField.text = book.genre
            lengthTextField.text = book.length
        }
      ```

  - Back in your BookTableViewController, make some slight changes to the implementation for `tableView(_:cellForRowAt:)`. 
    - First, force-cast the cell that's dequeued as a BookTableViewCell. Then, instead of setting cell.textLabel?.text and cell.detailTextLabel?.text, you can just call your custom cell's update(with:) method and pass in the Book object.

      - ```swift
          override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
              let cell = tableView.dequeueReusableCell(withIdentifier: "BookTableViewCell", for: indexPath) as! BookTableViewCell

              let book = books[indexPath.row]
              cell.update(with: book)
              cell.showsReorderControl = true
              
              return cell
          }
        ```

  - Run the app and see how the appearance of your cells has changed.
  - Congratulations! The intermediate table view concepts you just added to FavoriteBooks will provide a much better experience to the user. Make sure you save your final product in your project folder.

### Lesson 1.7 Saving Data

- As a programmer, you'll need to design your apps so that they save user data between launches. Imagine if a user wrote a note in the Notes app only to have it disappear when they relaunched the app. Whether ten seconds or ten months later, users expect all their data to still be there.
- In a previous lesson, you learned about MVC as a way to design the classes, structures, and flow of data in your app. You'll recall that model data represents the information in your app, views display the information, and a controller object acts as the intermediary, or communication layer, between the model and the views.
- When you add the ability to save data, you're adding a storage layer to your MVC architecture. In most cases, you'll use controller objects to access the storage layer.
With iOS, you have several ways to persist, or save, information in your app. In this lesson, you'll learn one approach: saving information to the device by archiving model data to a file saved on the disk. In the future, you'll learn about more advanced data storage tools, such as SQLite and Core Data.
- Regardless of the approach you choose, your behind-the-scenes data store will sit in the same place in the MVC diagram. As your app is running, it pulls information from the data store, works with it, and saves it back to the data store when the data is modified or when the app closes.

#### Encoding and Decoding with Codable

- In a previous lesson, you learned about the `Codable` protocol. The Codable protocol declares two methods that a class must implement so that its instances can be encoded and decoded, or serialized, into data that can be written to a file on disk. This lesson will walk you through encoding an object into a special format called a Property List, or plist, which is similar to a representation of a Dictionary in a file that can be saved to disk.
- In the earlier lesson, you learned that an object should be responsible for encoding and decoding its own instance variables. For a model object to be encoded or decoded, it must adopt the Codable protocol.
- Here's an example declaration of a Note model object that adopts the Codable protocol: `class Note: Codable {...}`
- Once an object conforms to the Codable protocol, an Encoder object can be used to encode the object as a data representation that can be saved to disk. Similarly, a Decoder object can be used to turn that encoded data into its corresponding model object. The `encode(_:)` method on an Encoder returns a Data object, and the `decode(_:from:)` method on a Decoder takes a Data object and returns an instance of the Codable object in question. (Data is a Swift structure that represents data stored as bytes. Data provides instance methods for writing to and reading from a file.)
- Conforming to the Codable protocol is simple. A class must adopt two methods: `init(from:)` and `encode(to:)`, which take a Decoder and an Encoder as a parameter, respectively. However, in most cases the Swift compiler makes it even easier to conform to the Codable protocol. If a model object only declares properties of types that already conform to Codable, an automatic conformance is triggered that satisfies all the protocol requirements.
- In other words, if all your object's properties are Codable, then all your object needs to do to adopt the Codable protocol is to include Codable in its declaration.
- The discussion that follows works through an example of encoding an object as Data and writing it to a file. To practice implementing Codable using the example, create a new iOS playground in Xcode (File > New > Playground) called “PersistencePractice.”

  - ```swift
      struct Note: Codable {
          let title: String
          let text: String
          let timestamp: Date
      }
    ```

- The code above declares a simple Note model object, similar to the objects you've declared previously. It also adopts the Codable protocol. Notice that the compiler is not displaying any errors. String and Date are Swift types that conform to Codable, so despite not explicitly having the two required methods for the protocol, Note automatically conforms to Codable.
- Create an instance of Note that can be encoded: `let newNote = Note(title: "Grocery run", text: "Pick up mayonnaise, mustard, lettuce, tomato, and pickles.", timestamp: Date())`
- Now look at the following example to see how to use an Encoder object to encode a value to a plist.

  - ```swift
      let propertyListEncoder = PropertyListEncoder()
      if let encodedNote = try? propertyListEncoder.encode(newNote) {
          print(encodedNote)
      }
    ```

- You might remember from the earlier lesson on protocols that PropertyListEncoder's `encode(_:)` method is a throwing function, requiring you to use either the do-try-catch syntax or the keyword `try?`. Because `try?` is used in this example, `encode(_:)` will simply return optional Data instead of throwing any errors. If you look at the console, you will see that printing encodedNote prints the number of bytes stored in the Data object. You have successfully encoded newNote.
- Decoding your data appears similar to the previous code, but is done in reverse and uses a PropertyListDecoder. Put the following lines of code inside your if-let block, below the existing print statement:

  - ```swift
      let propertyListDecoder = PropertyListDecoder()
      if let decodedNote = try? propertyListDecoder.decode(Note.self, from: encodedNote) {
          print(decodedNote)
      }
    ```

- You first create a PropertyListDecoder and then use it to decode encodedNote into a Note object. The `decode(_:from:)` method took as parameters `Note.self` and the note data. **Passing in Note.self passes in the actual Swift type of Note instead of any specific Note object**, which simply lets the method know what type of object it is trying to create from the data. PropertyListDecoder's `decode(_:from:)` method is also a throwing function, so the code uses `try?` and unwraps the resulting optional. By printing decodedNote to the console, you can see that you have successfully decoded the data.
- At this point your whole playground should look as follows:

  - ```swift
      struct Note: Codable {
          let title: String
          let text: String
          let timestamp: Date
      }
       
      let newNote = Note(title: "Grocery run", text: "Pick up mayonnaise, mustard, lettuce, tomato, and pickles.", timestamp: Date())
       
      let propertyListEncoder = PropertyListEncoder()
      if let encodedNote = try? propertyListEncoder.encode(newNote) {
          print(encodedNote)
       
          let propertyListDecoder = PropertyListDecoder()
          if let decodedNote = try?
            propertyListDecoder.decode(Note.self, from: encodedNote) {
              print(decodedNote)
          }
      }
    ```

#### Writing Data to a File

- Now what's left is to write your data to a file once it has been encoded and to read from that file before decoding. To do this, you'll need to learn a bit about the iOS file system.
- **Sandboxing and the Documents Directory**
  - In many operating systems, applications are given access to read and write files anywhere on the disk. So an application like Numbers might be able to open a spreadsheet from any folder, and a web browser might be able to write files to any directory.
  - This behavior is inherently insecure. A poorly written app could accidentally (or intentionally) delete important data that it has no business touching. Imagine if you downloaded an app to edit a music file, and it deleted all your photos.
  - Around the time iOS came on the scene, some operating systems introduced the concept of “sandboxing,” meaning giving programs access only to resources that they'd created or that they specifically requested access to. In this model, each app has its own environment where it can create, modify, or delete data — its own sandbox — and it doesn't have access to resources outside of the sandbox.
  - iOS apps work in the sandbox model. The operating system may allow access to resources outside of your app, but only when your app receives explicit permission from the user. For example, your app can load the user's contacts only after the user has given it permission to access them.
  - As part of the sandbox model, your app has a few directories that it can use to save data. One of those directories is called the Documents directory, and it's where you're allowed to save and modify information related to your app.
  - Another feature of the sandbox security model is that the file path to the Documents directory will change each time your app is loaded into memory. You can think of a file path as an address, similar to a URL, for locating the data. The contents of the directory will stay the same, but the address changes — preventing a variety of potential security issues.
  - How can you work with files within your sandboxed app? The Foundation framework defines a FileManager class that's used for interacting with the files on disk. It has a function that gives your app access to the Documents directory (wherever it is) and enables it to read and write files in that directory.
  - Back in your playground file, add the following line: `let documentsDirectory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!`
  - Option-click the urls method to view its declaration. You can see that the function returns an array of URLs for the specified directory in the requested domains. For a list of available iOS directories, check out the [documentation for the FileManager.SearchPathDirectory enumeration](https://developer.apple.com/documentation/foundation/filemanager/searchpathdirectory#).
  - For this lesson, you're interested in the `.documentDirectory` — which you'll pass as the search parameter. The `.userDomainMask` refers to the user's home folder, the folder that holds the user's apps and all their data.
  - The result of the function you entered above is an array of URL objects, which point to the directories that match your search. But there's only one Documents directory, so you request the first result of the search to assign to the variable.
  - As a result, the documentsDirectory variable will hold a URL that points to the directory, or folder, where you can read and write data. Now you need a full path that provides a filename and extension as well, which you can get by appending those elements to the directory:

    - ```swift
        let documentsDirectory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
        let archiveURL = documentsDirectory.appendingPathComponent("notes_test").appendingPathExtension("plist")
      ```

  - If you look at the results sidebar for these two directories, you'll see that the first ends in /Documents/ and the second ends in Documents/notes_test.plist. This is the full path where you will write your Note object's data to file.
- **Writing the Data**
  - Now that you know how to access the Documents directory, you can use methods on Data to write directly to and from files in that directory.
  - Back in your playground, remove your previous code for encoding and decoding. Then, at the end of what is left, create a new PropertyListEncoder and encode newNote without unwrapping the result. Your code should look like the following:

    - ```swift
        struct Note: Codable {
            let title: String
            let text: String
            let timestamp: Date
        }
         
        let newNote = Note(title: "Grocery run", text: "Pick up mayonnaise, mustard, lettuce, tomato, and pickles.", timestamp: Date())
         
        let documentsDirectory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
        let archiveURL = documentsDirectory.appendingPathComponent("notes_test").appendingPathExtension("plist")
         
        let propertyListEncoder = PropertyListEncoder()
        let encodedNote = try? propertyListEncoder.encode(newNote)
      ```

  - Now that you have a file path at which to save encodedNote, you can use the `write(to:options:)` method on Data to write to that path: `try? encodedNote?.write(to: archiveURL, options: .noFileProtection)`
  - Notice that `write(to:options:)` is also a throwing function and requires the `try?` keyword. This method will create a file at the specified URL with the data in encodedNote. By adding `.noFileProtection` as a parameter, you allow the file to be overwritten in the future should your Note object change and need to be saved again.
  - To retrieve the data from the file, you can initialize a Data object using its throwing initializer `init(contentsOf:)` and pass it the URL at which the data is stored. You can then decode the data:

    - ```swift
        let propertyListDecoder = PropertyListDecoder()
        if let retrievedNoteData = try? Data(contentsOf: archiveURL),
            let decodedNote = try? propertyListDecoder.decode(Note.self, from: retrievedNoteData) {
            print(decodedNote)
        }
      ```

  - You have successfully encoded, saved, loaded, and decoded your note.

- **Saving an Array of Model Data**
  - In many cases, your app will need to archive and unarchive collections of model data, not just a single instance. For example, you'll want the app to save all the user's notes, not just a single note.
  - To persist an array of model objects, you'll use the same methods you used previously, just on an array of objects instead of a single object. Encoder and Decoder objects can encode and decode arrays as long as their elements conform to Codable.
  - To put this concept into practice, enter several notes in your playground. Add the notes to a [Notes] array, then update your code to encode the array and write the resulting data to file.

    - ```swift
        let note1 = Note(title: "Note One", text: "This is a sample note.", timestamp: Date())
        let note2 = Note(title: "Note Two", text: "This is another sample note.", timestamp: Date())
        let note3 = Note(title: "Note Three", text: "This is yet another sample note.", timestamp: Date())
        let notes = [note1, note2, note3]
         
        let documentsDirectory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
        let archiveURL = documentsDirectory.appendingPathComponent("notes_test").appendingPathExtension("plist")
         
        let propertyListEncoder = PropertyListEncoder()
        let encodedNotes = try? propertyListEncoder.encode(notes)
         
        try? encodedNotes?.write(to: archiveURL, options: .noFileProtection)
         
        let propertyListDecoder = PropertyListDecoder()
        if let retrievedNotesData = try? Data(contentsOf: archiveURL),
            let decodedNotes = try? propertyListDecoder.decode(Array<Note>.self, from: retrievedNotesData) {
            print(decodedNotes)
        }
      ```

  - That's it! You've saved a collection of notes to a file and loaded the data from the file back into your program. Persistence is a key feature for many apps you'll build in the future.
- **Translating from a Playground to a Project**
  - In this lesson, you've created a playground to implement basic persistence using the Codable protocol.
  - How does this practice translate into a full Xcode project? Where does all the code go? When implementing persistence in your projects, keep three things in mind:
    1. Your model objects should implement the Codable protocol, similar to your implementation in the playground.
    2. Writing to and reading from files can be implemented as static methods on the model. Be sure to make these method names descriptive, like `saveToFile()` and `loadFromFile()`. Any time you add, remove, or modify your model data, you'll want to update the file. You can accomplish this by calling your `saveToFile()` in appropriate locations. When your app launches, you'll want to load data from the disk. To do so, call your `loadFromFile()` method at an early stage of your app launch. This could be in the `AppDelegate` or in the `viewDidLoad()` method of your first view controller.
    3. You'll want to save your objects in the correct app delegate life cycle events, such as when the app enters the background or is terminated.

#### Lab - Remember Your Emojis

- The objective of this lab is to use the Codable protocol, the FileManager, and methods on Data to persist information between app launches. Start with the EmojiDictionary project that you developed in earlier table view lessons. Take a few minutes to refamiliarize yourself with the code. If you run the app, you will see that any additions and changes you make to the default list of emoji are lost when the app is restarted. In this lab, you'll add persistence to your Emoji objects.

##### Step 1 Implement Saving on Emoji

- To persist your app's information across app launches, you need to be able to write its information to a file on disk. Start by making your Emoji struct adopt the Codable protocol in its declaration.
- Remember that you can implement saving and loading as static methods on the model. Add the static method signature for a `saveToFile(emojis:)` function in the Emoji class. It should take an array of Emoji objects as a parameter. Now add a static `loadFromFile()` method. It shouldn't take any parameters, but should return an array of Emoji objects.
- In each of these methods, you'll use either a PropertyListEncoder to encode your Emoji object or a PropertyListDecoder to decode it. But to write data to a file, you need to have a file path. Add a static property archiveURL to your Emoji class that returns the file path for Documents/emojis.plist. If you need a refresher on how to do this, go back and read this lesson's section on sandboxing.
- Now that you have a path for saving your Emoji object, fill in the method bodies of saveToFile(emojis:) and loadFromFile(), using Emoji.archiveURL as the file path. Your saveToFile(emojis:) method should save the supplied emojis array by encoding it and then writing it to Emoji.archiveURL. Your loadFromFile() method should read the data from the file, decode it as an [Emoji] array, and return the array.
- Create a static sampleEmojis method that returns a predefined [Emoji] collection. You can use the list assigned to emojis in EmojiTableViewController as your list of items.

##### Step 2 Save and Load When Appropriate

- Update emojis to be initialized to an empty collection rather than a large sample collection. When the viewDidLoad() method is called, you should check the Documents directory for any previously saved Emoji objects using loadFromFile(). If they're found, assign them to emojis. If not, assign emojis to Emoji.sampleEmojis.
- Take a moment to think about when it might be appropriate to save your Emoji objects.
In this case, the central spot for your data is the emojis array on the EmojiTableViewController—which means it would be appropriate to call saveToFile(emojis:) whenever the emojis property is changed.
- Next, think about when it might be appropriate to load your archived Emoji objects. Again, in this simple case, there's really only one point where the archived data will need to be unarchived—when the first view loads. You should already be calling this method in the first view controller's viewDidLoad().
- By now, your emoji data should be properly saving and loading. Run your app. Try adding a new emoji and tapping the Save button. Then close the app, reopen it, and observe whether the emoji persists in the table view. Repeat this process for editing an already existing emoji.
- On `Emoji.swift`

  - ```swift
      import Foundation

      struct Emoji: Codable {
          var symbol: String
          var name: String
          var description: String
          var usage: String
          
          static var documentsDirectory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
          static let archiveURL = documentsDirectory.appendingPathComponent("emojis").appendingPathExtension("plist")
          
          
          static func saveToFile(emojis: [Emoji]) {
              let propertyListEncoder = PropertyListEncoder()
              let encodedEmojis = try? propertyListEncoder.encode(emojis)
              
              try? encodedEmojis?.write(to: archiveURL, options: .noFileProtection)
          }
          
          static func loadFromFile() -> [Emoji]? {
              let propertyListDecoder = PropertyListDecoder()
              guard let retrivedEmojisData = try? Data(contentsOf: archiveURL) else {return nil}
              return try? propertyListDecoder.decode(Array<Emoji>.self, from: retrivedEmojisData)
          }
          
          static func sampleEmojis() -> [Emoji] {
              let emojis: [Emoji] = [
                  Emoji(symbol: "😀", name: "Grinning Face", description: "A typical smiley face.", usage: "happiness"),
                  ...
              ]
              
              return emojis
          }
      }
    ```

- On `EmojiTableViewController.swift`

  - ```swift
      import UIKit
      class EmojiTableViewController: UITableViewController {
          var emojis = [Emoji]()

          override func viewDidLoad() {
              super.viewDidLoad()              
              if let savedEmojis = Emoji.loadFromFile() {
                  emojis = savedEmojis
              } else {
                  emojis = Emoji.sampleEmojis()
              }
              
              navigationItem.leftBarButtonItem = editButtonItem
              tableView.rowHeight = UITableView.automaticDimension
              tableView.estimatedRowHeight = 44.0
              
          }

          @IBSegueAction func addEditEmoji(_ coder: NSCoder, sender: Any?) -> AddEditEmojiTableViewController? {
              if let cell = sender as? UITableViewCell,
                let indexPath = tableView.indexPath(for: cell) {
                  let emojiToEdit = emojis[indexPath.row]
                  return AddEditEmojiTableViewController(coder: coder, emoji: emojiToEdit)
              } else {
                  return AddEditEmojiTableViewController(coder: coder, emoji: nil)
              }
          }
          
          override func numberOfSections(in tableView: UITableView) -> Int {
              return 1
          }

          override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
              return emojis.count
          }

          override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
              let cell = tableView.dequeueReusableCell(withIdentifier: "EmojiCell", for: indexPath) as! EmojiTableViewCell
              let emoji = emojis[indexPath.row]
              
              cell.update(with: emoji)
              cell.showsReorderControl = true
              
              return cell
          }

          override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
              let emoji = emojis[indexPath.row]
              print("\(emoji.symbol) \(indexPath)")
          }
          override func tableView(_ tableView: UITableView, moveRowAt fromIndexPath: IndexPath, to: IndexPath) {
              let movedEmoji = emojis.remove(at: fromIndexPath.row)
              emojis.insert(movedEmoji, at: to.row)
              
              tableView.reloadData()
              Emoji.saveToFile(emojis: emojis)
          }

          override func tableView(_ tableView: UITableView, editingStyleForRowAt indexPath: IndexPath) -> UITableViewCell.EditingStyle {
              return .delete
          }
          
          override func viewWillAppear(_ animated: Bool) {
              super.viewWillAppear(animated)
              tableView.reloadData()
          }
          
          override func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath) {
              if editingStyle == .delete {
                  emojis.remove(at: indexPath.row)
                  tableView.deleteRows(at: [indexPath] , with: .automatic)
                  Emoji.saveToFile(emojis: emojis)
              }
          }
          @IBAction func unwindToEmojiTableView(segue: UIStoryboardSegue) {
              guard segue.identifier == "saveUnwind",
                    let sourceViewController = segue.source as? AddEditEmojiTableViewController,
                    let emoji = sourceViewController.emoji
              else { return }
              
              if let selectedIndexPath = tableView.indexPathForSelectedRow {
                  emojis[selectedIndexPath.row] = emoji
                  tableView.reloadRows(at: [selectedIndexPath], with: .none)
              } else {
                  let newIndexPath = IndexPath(row: emojis.count, section: 0)
                  emojis.append(emoji)
                  tableView.insertRows(at: [newIndexPath], with: .automatic)
              }
              
              Emoji.saveToFile(emojis: emojis)
          }
      }
    ```

- Congratulations! Persisting data isn't an easy thing to learn, but it's a concept you'll probably use in every app you build. Be sure to save this new version of your app to your project folder.

### Lesson 1.8 System View Controllers

- If you're an iOS user, you're likely already familiar with interfaces that allow you to access, present, and share content in an iOS app. When you enter your password for App Store purchases, use the Camera app to take a profile picture, or share an article through the Messages app, you're probably using a system view controller.
As a developer, you can use these familiar view controllers to extend the functionality of your apps. For example, you might want to access a user's photo library, share data to other apps, or respond to certain user choices.
- In this lesson, you'll learn the following:
  - How to use UIActivityViewController to share content to other installed apps
  - How to use SFSafariViewController to present information from the internet without exiting the app
  - How to use UIAlertController to present new information and options, as well as how to respond to the user's actions
  - How to use UIImagePickerController to access the device's camera and photo library and allow the user to select an image
  - How to use MFMailComposeViewController to send emails from within the app
- You'll work through the lesson by creating a project that uses these system view controllers and combines them to provide a better, more familiar user experience. The number of view controllers in that list might feel a bit overwhelming, but you'll learn that it's quite easy to build them into your app.

#### Create the Project

- Create a new project called "SystemViewControllers" using the iOS App template. When creating the project, make sure the interface option is set to "Storyboard."
- In the Main storyboard, drag an image view from the Object library to the view controller. Add a photo to the Asset Catalog and set your image view to display it, as you have done in previous lessons. Later in this lesson, you'll learn how to use different system view controllers to share the image. When you learn about the image picker view controller, you'll allow the user to set the image from the photo library.
- Next, add four buttons and set their text to reflect the system view controllers they'll be calling: “Share,” “Safari,” “Camera,” and “Email.” (You might want to use a stack view to help manage the layout.)
- Create an outlet for the image view called imageView. Set your image and create four actions, one for each button tap.
- If you have an iOS device available for testing, you might want to use it for this lesson instead of Simulator. Running this app on your device will allow you to access your own photos and share content with other apps on the device. You can switch from Simulator to a physical device with the schemes menu in the top-left corner of Xcode.

#### Share with the Activity Controller

- Have you ever been using an app and seen an image that you wanted to send to a friend? Have you ever wondered how you were able to share content between different apps? You can add sharing to your app easily using the `UIActivityViewController`. Fairly simple to implement, an activity controller provides a familiar experience that allows the user to share content from one app with other apps on their device.
- You'll begin by creating an instance of UIActivityViewController when the Share button is tapped. The initializer for an activity view controller asks for a parameter called `activityItems`, an array of type `Any`. The sharing interface will show any app that can accept the activity items you've included in the array.
- To share an item — such as text, an image, a URL, or other content — you'll pass it into the activityItems array. In this case, you will unwrap the image view's image and include it in the array of activity items in the initializer.
- The initializer also has a parameter for `applicationActivities` that represents any custom services that your app might support. For this project, you'll set it to `nil`. Once the activity view controller has been created, don't forget to present it to the user. Here's how this works:

  - ```swift
      @IBAction func shareButtonTapped(_ sender: UIButton) {
          guard let image = imageView.image else { return }
          let activityController =  UIActivityViewController(activityItems: [image],
            applicationActivities: nil)
          activityController.popoverPresentationController?.sourceView = sender
          present(activityController, animated: true, completion: nil)
      }
    ```

- Why is the `popoverPresentationController` property being modified? On an iPad, a UIActivityViewController will be presented inside a popover, and all popovers emanate from a particular view. Popovers are best presented from the button that triggered the presentation — in this case, sender. This line of code will have no effect on smaller iOS devices.
- Run your project and tap the Share button. If you're running the project in Simulator, you may only see one or two apps. But if you're running on your device and you have a lot of apps, you may discover quite a number of apps where you can share your content.
- You can experiment with the array of activity items. What happens when you replace the image with a string? What about when you include an image and a string? You'll find that the available apps change to match the types of activity items you want to share.

#### Use Safari Services to Display Web Content

- You've just learned how to take content from your app and share it with other apps on your device. What if you wanted to build an app that would present your user with content from the web while keeping them inside your app? `SFSafariViewController` allows you to open a webpage on a Safari web browser inside your app.
- Have a look at the [documentation for SFSafariViewController](https://developer.apple.com/documentation/safariservices/sfsafariviewcontroller) to learn about some of its features, which include a safe and familiar interface for browsing the web, a read-only address field, Reader, AutoFill, Fraudulent Website Detection, and content blocking.
- While you're in the documentation, see if you can identify which framework allows you to use Safari view controllers. Most of what you've worked with until now has been in either the Foundation or UIKit framework. SFSafariViewController, however, is in the `SafariServices` framework. To open a Safari view controller, you'll need to import this framework into your project by adding it to the top of your view controller file, beneath the line that imports UIKit.

  - ```swift
      import UIKit
      import SafariServices
    ```

- Now you're ready to present your own Safari view controller. You may have noticed in the documentation that the Safari view controller has a read-only address field, which means you'll need to specify which website the sure will be visiting when you initialize the view controller.

- To make an instance of Safari view controller, you have three things to do. 
  - First, create a URL from a string; this is the site you'll present to the user. You'll notice the `URL(string:)` initializer returns an optional, so you'll need to unwrap it. 
  - Next, create an instance of SFSafariViewController with the URL you just created. 
  - And last, present the Safari view controller to the user. The code is quite simple. Here's how to make it happen:

  - ```swift
      @IBAction func safariButtonTapped(_ sender: UIButton) {
          if let url = URL(string: "https://www.apple.com") {
              let safariViewController = SFSafariViewController(url: url)
              present(safariViewController, animated: true, completion: nil)
          }
      }
    ```

- Run the app and tap the Safari button. You've successfully opened a Safari view controller in your app. Tapping the Done button in the upper-left corner or swiping right from the edge of the screen will navigate back to the original view controller. You can use any URL when presenting a Safari view controller.

#### Present an Alert Controller

- You've now shared content through the activity controller, and you've presented websites with the Safari view controller. But so far, all the content has been predetermined. How can you open up the experience to the user? How can you use system view controllers to allow them to interact with your app?
- A great way to present your user with different options is with the `UIAlertController`. You've seen alert controllers before. iOS presents low battery alerts when you are running low and software update alerts when a new version of the operating system is available. You can use alert controllers to capture the attention of the user and give them options to choose from. As the programmer, you can determine the text of the alert, the available options, and the code that runs when the user chooses each option.
- In the [documentation for UIAlertController](https://developer.apple.com/documentation/uikit/uialertcontroller), read through the different symbols and consider how you might configure an alert controller.
- For this app, you'll use the alert controller to ask the user whether they would like to take a new photo or select an existing photo from their photo library to set as the new image for the image view. So you'll start by implementing the alert controller inside the `cameraButtonTapped` function.
- When you initialize a UIAlertController, you specify a title, message, and a preferred presentation style — which also determines the placement of the alert. Choosing `.alert` will place the alert in the center of the screen, while `.actionSheet` will place the alert at the bottom of the screen.
- Try it out. Create an alert controller and set the style to `.actionSheet`. Add an alert action to cancel, or dismiss, the alert controller. And, finally, present the alert controller to the user. Just like the activity view controller, you should set the sourceView property so that the popover presents in the proper location on an iPad. You can put it all together with the following code:

  - ```swift
      @IBAction func cameraButtonTapped(_ sender: UIButton) {
          let alertController = UIAlertController(title: "Choose Image Source", message: nil, preferredStyle: .actionSheet)

          let cancelAction = UIAlertAction(title: "Cancel", style: .cancel, handler: nil)
          alertController.addAction(cancelAction)
          alertController.popoverPresentationController?.sourceView = sender

          present(alertController, animated: true, completion: nil)
      }
    ```

- You're now presenting the alert, but how can you respond to the user's action? Create a UIAlertAction and give it the title "Camera" and a style of .default. The alert action has a property called handler, which is where you'll write the code that will be executed if the user selects that option. The handler is a closure, or block of code, that will execute if its action has been selected. You'll learn more about closures in a future lesson.
- For now, when you encounter a closure parameter, you can select the placeholder and press Return, which will automatically generate the closure syntax you'll need to add the code you want to execute. You can see an example of the closure syntax in the next code sample.
- Imagine that you want the handler to print out the user's selection. Once you've created that alert action, you'll add it to the alert controller. Create another button that will allow the user to select a picture from the photo library and add it to the alert controller.
- Here's how all that will work:

  - ```swift
      @IBAction func cameraButtonTapped(_ sender: UIButton) {
          let alertController = UIAlertController(title: "Choose Image Source", message: nil, preferredStyle: .actionSheet)
       
          let cancelAction = UIAlertAction(title: "Cancel", style: .cancel, handler: nil)
          let cameraAction = UIAlertAction(title: "Camera", style: .default, handler: { action in 
              print("User selected Camera action")
          })
       
          let photoLibraryAction = UIAlertAction(title: "Photo Library",
            style: .default, handler: { action in
              print("User selected Photo Library action")
          })
       
          alertController.addAction(cancelAction)
          alertController.addAction(cameraAction)
          alertController.addAction(photoLibraryAction)
          alertController.popoverPresentationController?.sourceView = sender
       
          present(alertController, animated: true, completion: nil)
      }
    ```

- Run the project on your device and present the alert controller. Select one of the action alerts, and notice that the string prints to the console. Now that you've presented an alert and responded to the user's actions, you're ready to step it up a notch.

#### Access the Camera

- You've used an activity controller to share content, presented web content through a Safari view controller, and even responded to a user's actions from an alert controller. Now you can use these same tools to work with user - generated content by accessing the user's photos and camera.
- When might you want to access a device's camera? Maybe you're building a photo - sharing app or an app where users can choose a profile picture. To access the user's camera or photo library, you'll use `UIImagePickerController`.
- To use an image picker controller, you must adopt two protocols:
  - `UIImagePickerControllerDelegate` will transfer the selected image's information back into your app.
  - `UINavigationControllerDelegate` will handle the responsibility for dismissing the image picker view.
- Update your ViewController class to adopt both `UIImagePickerControllerDelegate` and `UINavigationControllerDelegate`: `class ViewController: UIViewController, UIImagePickerControllerDelegate, UINavigationControllerDelegate {...}`
- In your `cameraButtonTapped` action, create an instance of `UIImagePickerController` and set your view controller as the delegate. Place this code above the alert controller initializer.

  - ```swift
      @IBAction func cameraButtonTapped(_ sender: UIButton) {
          let imagePicker = UIImagePickerController()
          imagePicker.delegate = self
       
          //...
      }
    ```

- With the delegate set, it's time to present the user with a choice of photo sources. In this case, you'll use the handlers for the alert action items you created in the previous section.
- You'll only want to present the user with the options they can actually choose. For example, if the app is running on Simulator, you only want to present the Photo Library option, because Simulator does not have a camera. If you try to open the camera in Simulator, the app will crash with a fatal error.
- You can use the class method `UIImagePickerController.isSourceTypeAvailable(_:)`, which will return a Bool indicating whether the source type can be used on that device.
- Now create an alert controller and check whether the camera and photo library are available source types. If one or both are supported, create alert actions accordingly. Add the actions to the alert controller, remembering to set the source type in the action's handler. Finally, present the image picker. Your code should resemble the following:

  - ```swift
      @IBAction func cameraButtonTapped(_ sender: UIButton) {
          let imagePicker = UIImagePickerController()
          imagePicker.delegate = self
       
          let alertController = UIAlertController(title: "Choose Image Source", message: nil, preferredStyle: .actionSheet)
       
          let cancelAction = UIAlertAction(title: "Cancel", style: .cancel, handler: nil)
          alertController.addAction(cancelAction)
       
          if UIImagePickerController.isSourceTypeAvailable(.camera) {
              let cameraAction = UIAlertAction(title: "Camera", style: .default, handler: { action in
                  imagePicker.sourceType = .camera
                  self.present(imagePicker, animated: true, completion: nil)
              })
              alertController.addAction(cameraAction)
          }
       
          if UIImagePickerController.isSourceTypeAvailable(.photoLibrary) {
              let photoLibraryAction = UIAlertAction(title: "Photo Library", style: .default, handler: { action in
                  imagePicker.sourceType = .photoLibrary
                  self.present(imagePicker, animated: true, completion: nil)
              })
              alertController.addAction(photoLibraryAction)
          }
       
          alertController.popoverPresentationController?.sourceView = sender
       
          present(alertController, animated: true, completion: nil)
      }
    ```

- Build and run your app. What happened when you selected the photo library or camera? If you ran the app on an iOS device and followed all the steps above, your app probably crashed when you chose the camera. Take a look in the console, and you'll see an error message that reads: `This app has crashed because it attempted to access privacy-sensitive data without a usage description. The app's Info.plist must contain an NSCameraUsageDescription key with a string value explaining to the user how the app uses this data.`
- This message appears because your app must request permission before trying to access the user's private information — in this case, the camera. The operating system handles presenting the user with the option to allow access. The `NSCameraUsageDescription` key is used to tell the user why your app wants to access their camera.
- Open the Info file in the Project navigator and enter a new key for `NSCameraUsageDescription`
- For the value, enter how you'll use the user's data — for example, `To share photos from the camera.` (1)
  - <img src="./resources/new_info.png" alt="new Info - NSCameraUsageDescription" width="400" />
- Now the operating system can ask the user for permission and present the camera accordingly. Run the app again. You'll notice an alert asking permission to access your camera.
- Go ahead and select a photo from the library. What happened? Not much; you're just back at your view controller's view. You'll need a way to grab the selected photo and bring it into your app. You do this by implementing the delegate method `imagePickerController(_:didFinishPickingMediaWithInfo:)`. This method tells the delegate that the user has picked a photo (or other media), and it includes the photo in the info dictionary. Add the method to your view controller:

  - ```swift
      @IBAction func cameraButtonTapped(_ sender: UIButton) {...}
       
      func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
       
      }
    ```

- In the body of this method, you can access the media through the info dictionary. The dictionary keys are of type UIImagePickerController.InfoKey and give you access to information regarding the user's image picking session. For now, you'll simply use the `.originalImage` key to get the image selected by the user. However, you should look at the [documentation for UIImagePickerController.InfoKey](https://developer.apple.com/documentation/uikit/uiimagepickercontroller/infokey) to see what other information is available.
- Because the info dictionary is of the type `[UIImagePickerController.InfoKey: Any]`, the value for the original image key won't be of type UIImage. You can typecast the value to be a UIImage — but remember that by doing so you will make it an optional `(UIImage?)`. Once you've unwrapped the optional image, you can set it to display in your view controller. To dismiss the image picker, simply call `dismiss(animated:completion:)` at the end. Here's how it should look:

  - ```swift
      @IBAction func cameraButtonTapped(_ sender: UIButton) {...}
       
      func imagePickerController(_ picker: UIImagePickerController,  didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
          guard let selectedImage = info[.originalImage] as? UIImage else { return }
       
          imageView.image = selectedImage
          dismiss(animated: true, completion: nil)
      }
    ```

- Build and run the app. You can now open the photo library and select a photo or take a picture with the camera for your view controller to display.

#### Send Email from Your App

- The last system view controller you'll learn about in this lesson is the `MFMailComposeViewController`, which allows you to send emails from within your app. The mail compose view controller is in the `MessageUI` framework, which provides an interface for sending email and text messages. Add an import statement to the top of your view controller, beneath the other import statements.
- Have a look at the [documentation for MFMailComposeViewController](https://developer.apple.com/documentation/messageui/mfmailcomposeviewcontroller#) before moving forward.
- Just like the other view controllers, you're going to present the MFMailComposeViewController from the relevant button's action. But before doing anything: Is the user's device capable of sending email? Similar to the verification you performed with the image picker, you'll use the function `canSendMail()` to determine whether the device has available mail services. If not, you'll print a message to the console and return out of the function:

  - ```swift
      @IBAction func emailButtonTapped(_ sender: UIButton) {
          guard MFMailComposeViewController.canSendMail() else {
              print("Can not send mail")
              return
          }
      }
    ```

- You may have noticed in the documentation the symbol for accessing the delegate. Similar to the `UIImagePickerControllerDelegate`, the `mailComposeDelegate` is responsible for dismissing the mail compose view controller at the appropriate time. To set the delegate, you'll need to adopt the `MFMailComposeViewControllerDelegate` protocol: `class ViewController: UIViewController, UIImagePickerControllerDelegate, UINavigationControllerDelegate, MFMailComposeViewControllerDelegate {...}`
- Now that your view controller conforms to the protocol, you're ready to implement the mail compose view controller. Create an instance of `MFMailComposeViewController` and set the view controller as the `mailComposeDelegate`:

  - ```swift
      @IBAction func emailButtonTapped(_ sender: UIButton) {
          guard MFMailComposeViewController.canSendMail() else { return }
       
          let mailComposer = MFMailComposeViewController()
          mailComposer.mailComposeDelegate = self
      }
    ```

- Notice that the guard statement returns silently if canSendMail returns false. This wouldn't be a good design in a real world app. You should disable — or just not show — the email option if the user can't send email on their device (for example, on an iPhone with no email account set up). Or you could display a message explaining the situation.
- You can go even further with email functionality. With the `MFMailComposeViewController`, you can configure different parts of an email in your code: the recipients, the subject, the message body, and even attachments. One of the parameters used in setting the message body is the parameter `isHTML`, a Bool for checking whether your message should be interpreted as plain text or HTML. In this email, you won't be sending HTML, so the `isHTML` parameter is set to false. Once the email's details are configured, present the mail compose view controller to the user. Here's how the code goes:

  - ```swift
      mailComposer.setToRecipients(["example@example.com"])
      mailComposer.setSubject("Look at this")
      mailComposer.setMessageBody("Hello, this is an email from the app I made.", isHTML: false)
       
      if let image = imageView.image, let jpegData = image.jpegData(compressionQuality: 0.9) {
          mailComposer.addAttachmentData(jpegData, mimeType: "image/jpeg", fileName: "photo.jpg")
      }

      present(mailComposer, animated: true, completion: nil)
    ```

- When the user has finished sending an email, they'll need a way to dismiss the mail compose view controller and return to the app. You can use the delegate method `mailComposeController(didFinishWith:)` to dismiss the view:

  - ```swift
      @IBAction func emailButtonTapped(_ sender: UIButton) {...}
       
      func mailComposeController(_ controller: MFMailComposeViewController, didFinishWith result: MFMailComposeResult, error: Error?) {
          dismiss(animated: true, completion: nil)
      }
    ```

- If you have an iOS device, try running the app to see this system view controller at work. (Simulator doesn't have an email app, so tapping the button there won't do anything.)

- Wrap-Up
  - Now that you've successfully implemented the activity view controller, Safari view controller, alert controller, image picker controller, and mail compose view controller, you're ready to incorporate one or all of them into your apps.
  - In addition to extending the functionality of your apps, these system view controllers add an element of familiarity to the apps you create and provide a sense of continuity across the iOS platform.
- Challenge
  - Read the [documentation for MFMessageComposeViewController](https://developer.apple.com/documentation/messageui/mfmessagecomposeviewcontroller) and implement the message compose view controller in your app.

- ViewController.swift

  - ```swift
      import UIKit
      import SafariServices
      import MessageUI

      class ViewController: UIViewController, UIImagePickerControllerDelegate, UINavigationControllerDelegate, MFMailComposeViewControllerDelegate {
          @IBOutlet var imageView: UIImageView!
          
          override func viewDidLoad() {
              super.viewDidLoad()
          }

          @IBAction func shareButtonTapped(_ sender: UIButton) {
              guard let image = imageView.image else {return}
              let activityController = UIActivityViewController(activityItems: [image], applicationActivities: nil)
              activityController.popoverPresentationController?.sourceView = sender
              present(activityController, animated: true, completion: nil)
          }
          @IBAction func safariButtonTapped(_ sender: UIButton) {
              if let url = URL(string: "https://www.apple.com") {
                  let safariViewController = SFSafariViewController(url: url)
                  present(safariViewController, animated: true, completion: nil)
              }
          }
          @IBAction func cameraButtonTapped(_ sender: UIButton) {
              let imagePicker = UIImagePickerController()
              imagePicker.delegate = self
              let alertController = UIAlertController(title: "Choose Image Source", message: nil, preferredStyle: .actionSheet)
          
              let cancelAction = UIAlertAction(title: "Cancel", style: .cancel, handler: nil)
              alertController.addAction(cancelAction)
          
              if UIImagePickerController.isSourceTypeAvailable(.camera) {
                  let cameraAction = UIAlertAction(title: "Camera",
                    style: .default, handler: { action in
                      imagePicker.sourceType = .camera
                      self.present(imagePicker, animated: true, completion: nil)
                  })
                  alertController.addAction(cameraAction)
              }
          
              if UIImagePickerController.isSourceTypeAvailable(.photoLibrary) {
                  let photoLibraryAction = UIAlertAction(title: "Photo Library",
                    style: .default, handler: { action in
                      imagePicker.sourceType = .photoLibrary
                      self.present(imagePicker, animated: true, completion: nil)
                  })
                  alertController.addAction(photoLibraryAction)
              }
          
              alertController.popoverPresentationController?.sourceView = sender
          
              present(alertController, animated: true, completion: nil)
          }
          func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
              guard let selectedImage = info[.originalImage] as? UIImage else { return }
              imageView.image = selectedImage
              dismiss(animated: true, completion: nil)
          }
          @IBAction func emailButtonTapped(_ sender: UIButton) {
              guard MFMailComposeViewController.canSendMail() else {
                  print("Can not send mail")
                  return
              }
              
              let mailComposer = MFMailComposeViewController()
              mailComposer.mailComposeDelegate = self
              mailComposer.setToRecipients(["example@example.com"])
              mailComposer.setSubject("Look at this")
              mailComposer.setMessageBody("Hello, this is an email from the app I made.", isHTML: false)
              
              if let image = imageView.image, let jpegData = image.jpegData(compressionQuality: 0.9) {
                  mailComposer.addAttachmentData(jpegData, mimeType: "image/jpeg", fileName: "photo.jpg")
              }
              present(mailComposer, animated: true, completion: nil)
          }
          
          func mailComposeController(_ controller: MFMailComposeViewController, didFinishWith result: MFMailComposeResult, error: Error?) {
              dismiss(animated: true, completion: nil)
          }
      }
    ```

#### Lab - Home Furniture Sharing

- The objective of this lab is to use system view controllers in an app that lists rooms and furniture and allows users to share furniture items with other apps on their device. Make sure to build and run your app on a physical iOS device if you have one — rather than on Simulator — so you'll have more apps that permit image and text sharing.
- Your app has been started for you. Open the project called “HomeFurniture” in your student resources folder. Take a few minutes to get familiar with the project. Run the app and navigate to the detail screen of a furniture item. Notice that the button in the middle of the screen says “Choose Photo,” but that it doesn't do anything when you tap it. The button at the bottom left also does nothing. Your task in this lab is to add functionality to these two buttons.

##### Step 1 Add a UIAlertController

- To present the right `UIImagePickerController` state, you'll need to know whether the user wants to choose an existing image from the device's photo library or take a new image using the camera. To enable the user's choice, you'll use a `UIAlertController` to present an action sheet from the bottom of the screen.
- In the `choosePhotoButtonTapped(_:)` method in `FurnitureDetailViewController`, you'll need to create and present your action sheet. First, create an instance of a UIAlertController. You can make the title and message parameters `nil`, but you'll need to set the preferredStyle to `UIAlertControllerStyle.actionSheet`.
- Next, create a `UIAlertAction` for canceling the alert and add it to your instance of `UIAlertController`. As you've learned, a handler executes code after the button is tapped. By default, a button tapped in a `UIAlertController` will dismiss the alert, so the `handler` parameter can be `nil`.
- You'll need to add two more `UIAlertAction` instances: one for choosing an image from the photo library and another for using the camera to take a new image. Use the autocompletion feature in Xcode to get the right syntax for including the `handler` parameter on these actions. You can leave the bodies of the handlers empty for now, but you'll need to put in code later to ensure the image picker uses the right source. Be sure to add these actions to your alert controller.
- Present the alert controller at the end of the method.

  - ```swift
      @IBAction func choosePhotoButtonTapped(_ sender: Any) {
          let alertController = UIAlertController(title: "Choose Image Source", message: nil, preferredStyle: .actionSheet)

          let cancelAction = UIAlertAction(title: "Cancel", style: .cancel, handler: nil)
          alertController.addAction(cancelAction)
          
          let cameraAction = UIAlertAction(title: "Camera", style: .default, handler: {})
          alertController.addAction(cameraAction)

          let photoLibraryAction = UIAlertAction(title: "Photo Library", style: .default, handler: {})
          alertController.addAction(photoLibraryAction)
          
          present(alertController, animated: true, completion: nil)
      }
    ```

##### Step 2 Add a UIImagePickerController

- Now that you have your alert controller set up, you'll need to add the functionality for presenting the image picker. Make an instance of a `UIImagePickerController` before the line of code that creates your alert controller (but still in your `choosePhotoButtonTapped(_:)` method).
- Set your image picker's delegate property to self. If your view controller doesn't conform to the `UINavigationControllerDelegate` and `UIImagePickerControllerDelegate` protocols, you'll get an error at this point. You can fix the error by adding these protocols to the class declaration.
- In the handlers for your actions, set the corresponding source type on your image picker and present the image picker.
- Right now, if you run the app on Simulator and try to present the camera, the app will crash (since Simulator has no camera). To avoid giving users the option to choose a source that doesn't exist, add an if statement before your image-picking actions to check whether the source type is available. For example:

  - ```swift
      if UIImagePickerController.isSourceTypeAvailable(.photoLibrary) {
          let photoLibraryAction = UIAlertAction(title:
            "Photo Library", style: .default, handler: { (_) in
              imagePicker.sourceType = .photoLibrary
          })
          alertController.addAction(photoLibraryAction)
      }
    ```

- There are two more pieces of image-picking functionality to write. You'll need to provide a way to retrieve the image the user has selected as well as a way to dismiss the image picker if the user decides against picking an image. Start by implementing two delegate methods, `imagePickerController(_:didFinishPickingMediaWithInfo:)` and `imagePickerControllerDidCancel(_:)`, in your view controller.
- In `imagePickerControllerDidCancel(_:)`, simply dismiss the image picker with `dismiss(animated: true, completion: nil)`.
- In `imagePickerController(_:didFinishPickingMediaWithInfo:)`, you'll use the key `.originalImage` to retrieve the image from the info dictionary, then you'll unwrap the result and cast it as a UIImage.
- Once you've retrieved the image, call the image's `jpegData(compressionQuality:)` method with a value of `0.9` and set the `imageData` property on `furniture` to the result. The `UIImage` instance method `jpegData(compressionQuality:)` returns a `Data` representation of the image. Then dismiss the image picker and call `updateView()` in the completion.
- To use the camera on a device, your app will need explicit permission from the user. Enter the reason you need access in the Info file by adding the `NSCameraUsageDescription` key with a brief phrase.
- Run the app. Navigate to the detail view for one of the furniture items and try selecting an image.

  - ```swift
      @IBAction func choosePhotoButtonTapped(_ sender: Any) {
          let imagePicker = UIImagePickerController()
          imagePicker.delegate = self

          let alertController = UIAlertController(title: "Choose Image Source", message: nil, preferredStyle: .actionSheet)
          let cancelAction = UIAlertAction(title: "Cancel", style: .cancel, handler: nil)
          alertController.addAction(cancelAction)
          
          if UIImagePickerController.isSourceTypeAvailable(.camera) {
              let cameraAction = UIAlertAction(title: "Camera", style: .default, handler: { (_) in
                  imagePicker.sourceType = .camera
                  self.present(imagePicker, animated: true, completion: nil)
              })
              alertController.addAction(cameraAction)
          }
          
          if UIImagePickerController.isSourceTypeAvailable(.photoLibrary) {
              let photoLibraryAction = UIAlertAction(title: "Photo Library", style: .default, handler: { (_) in
                  imagePicker.sourceType = .photoLibrary
                  self.present(imagePicker, animated: true, completion: nil)
              })
              alertController.addAction(photoLibraryAction)
          }
          
          present(alertController, animated: true, completion: nil)
      }

      func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
          guard let selectedImage = info[.originalImage] as? UIImage else {return}
          
          if let furniture = furniture, let jpegData = selectedImage.jpegData(compressionQuality: 0.9) {
              furniture.imageData = jpegData
              updateView()
          }
          dismiss(animated: true, completion: nil)
      }
      
      func imagePickerControllerDidCancel(_ picker: UIImagePickerController) {
          dismiss(animated: true, completion: nil)
      }
    ```

##### Step 3 Share Furniture with the UIActivityViewController

- From the `actionButtonTapped(_:)` method, use the `UIActivityViewController` to add the ability to share the image and description from the `Furniture` item.
- Great work! Your user can now choose an image from an image source and add it to the app. They can also share information with other apps on their device. Be sure to save this project to your project folder.

  - ```swift
      @IBAction func actionButtonTapped(_ sender: UIButton) {
          guard let furniture = furniture else {return}
          
          let activityController = UIActivityViewController(activityItems: [furniture.imageData, furniture.name, furniture.description], applicationActivities: nil)
          activityController.popoverPresentationController?.sourceView = sender
          
          present(activityController, animated: true, completion: nil)
      }
    ```

##### Issue - showing up text on a label on the simulator, but not showing it up on an actual device

- If Dark mode is enabled on the actual device, the text color will appear white.

### Lesson 1.9 Complex Input Screens

- Take a look at the standard iOS apps preinstalled on your device. You'll notice that they display various input screens for collecting information. A good example is the interface for adding a new event in Calendar. This input screen relies on several types of controls — including text fields, date pickers, and switches — neatly organized in a table view controller. Another example is the interface for entering a new contact in the Contacts app.
- Whether tapping interface elements or using the keyboard, inputting information can be a tedious process. When you design views that require complex or large amounts of user input, your end goal should always be to make the process easier for users.

#### Model

- Data input screens are tightly coupled with an app's model data. Imagine you're building an app to track cars on a dealer's lot. You'd probably need an input screen for adding details about each car. You'd have a text field to input the VIN, two date pickers to input the date acquired and the date sold, and maybe a segmented control to indicate whether the car is new or used.
- As you progress through this lesson, you'll build an app for the fictional Hotel Manzana. Hotel staff will use the app to register guests when they arrive at the hotel. As you build the app, try to configure your scenes in Interface Builder to look exactly the way you want your input screens to be displayed. This will minimize the amount of customization code you'll need to do later, while giving you a visual representation of the forms as you're creating them.
- To start, create an Xcode project using the iOS App template and name it “Hotel Manzana." When creating the project, make sure the interface option is set to "Storyboard." Go ahead and delete the ViewController file and the associated scene in the storyboard; this app doesn't need them.
- Your client at Hotel Manzana has given you a list of the information they'd like to collect: the guest's first name, last name, and email; the check-in and check-out dates; the number of adults and children in the room; whether the guest wants Wi-Fi access; and the guest's room choice. Hotel Manzana has three different room choices: a room with two queen beds, a room with one king bed, and a suite with two king bedrooms. Each room type has a name, a short name (fewer than three characters), and price. You'll also include an ID number in your data model - invisible to the user - to make it easy to check whether two room type instances are equal.
- Before reading ahead, try to plan out the necessary models for your client's app. Maybe you came up with something like the following:

  - ```swift
      struct Registration {
          var firstName: String
          var lastName: String
          var emailAddress: String
       
          var checkInDate: Date
          var checkOutDate: Date
          var numberOfAdults: Int
          var numberOfChildren: Int
       
          var wifi: Bool
          var roomType: RoomType
      }
       
      struct RoomType: Equatable {
          var id: Int
          var name: String
          var shortName: String
          var price: Int
       
          //Equatable Protocol Implementation for RoomType

          static func ==(lhs: RoomType, rhs: RoomType) -> Bool {
              return lhs.id == rhs.id
          }
      }
    ```

- Remember that there's never one and only one right answer in programming. Your approach might be one of many workable solutions. For example, in this exercise, you may have assigned the properties slightly different names or types. What matters is that you have a way to store everything your app will be tracking.
- For the purposes of this lesson, though, you'll work with the structures defined above.

#### Input Screens

- An input screen is a form with controls for inputting data. As you've learned, a good option for a form is a table view, with each cell representing a different piece of data to be entered. With table views, you can group information into logical sections and use section headers and footers to give instructions. And because table views scroll, you can fit in more inputs than you can on regular view controllers.
- Most table views used for form input have static cells. Static cells allow the table view controller subclass to have outlets directly on each of the input controls, making it much simpler to access the control's values. In contrast, a dynamic table view requires a table view cell subclass to access the views on a cell.
- To build the input screen for Hotel Manzana, you'll use a table view controller.
- In your app's storyboard, add a navigation controller with a table view controller as the root view controller. Set the navigation controller as the initial view controller. Set the navigation item title for the table view controller to "New Guest Registration". In the Attributes inspector, set the table view Content to `Static Cells` and the Style to `Grouped`.
- With the table view appropriately configured, you'll need to add and configure your cells. The type of cells is highly dependent on the data you're trying to collect. Based on the type, you'll use the appropriate control to collect that data. For a refresher on controls, you can check out the “Controls in Action” lesson.
- <img src="./resources/input_screen.png" alt="Input Screen" width="200" />

#### Collect Strings

- For Hotel Manzana, you'll need to collect three pieces of String data: the guest's first name, last name, and email address. You'll generally use text fields for collecting strings, with the placeholder text property indicating the text field's purpose. If your string might take multiple lines, a text view could be a better choice (even though it's not technically a control).
- In the storyboard, add a text field to each of three cells, one for each string you're collecting. Pin all four edges of the text fields and set the placeholder text in the Attributes inspector.
- To implement the logic behind your input screen, you'll need a class file to add your code. Add a new Cocoa Touch Class called `AddRegistrationTableViewController` that inherits from `UITableViewController`. Since you're implementing a static table view, you don't need to implement the data source. In fact, if you'd like, you can delete the boilerplate data source code that comes with a `UITableViewController` subclass.
  - Delete `numberOfSections` method and `tableView(numberOfRowsInSection:)` method, or update the return
- Make sure the identity of the table view controller in the storyboard is set to your newly created table view controller subclass.
  - Click `New Guest Registration` on the Document Outline, choose Class to `AddRegistrationTableViewController` in the Identity inspector.
- Next, add outlets for the text fields, so you can reference them in code. Here's how that works:

  - ```swift
      class AddRegistrationTableViewController: UITableViewController {
          @IBOutlet var firstNameTextField: UITextField!
          @IBOutlet var lastNameTextField: UITextField!
          @IBOutlet var emailTextField: UITextField!
          //...
      }
    ```

- In the storyboard, add a bar button item to the right bar button item slot on the AddRegistrationTableViewController's navigation bar. In the Attributes inspector, set the new button to the `Done` system item.
- Hook up an @IBAction to your `Done` button and name the function `doneBarButtonTapped(_:)`. Add code to this method to print any data in the text fields when the button is tapped.

  - ```swift
      @IBAction func doneBarButtonTapped(_ sender: UIBarButtonItem) {
          let firstName = firstNameTextField.text ?? ""
          let lastName = lastNameTextField.text ?? ""
          let email = emailTextField.text ?? ""
       
          print("DONE TAPPED")
          print("firstName: \(firstName)")
          print("lastName: \(lastName)")
          print("email: \(email)")
      }
    ```

- You've now collected the first three pieces of data from your input view.

#### Collect Dates

- The Hotel Manzana app will also collect two dates. Dates are actually a very complex type. With the date type, you're representing a single point in time (both the day and the time of day), and that representation has several variable characteristics, like a time zone and a calendar (for example, Gregorian or Hebrew). How can you communicate a date in a way that's meaningful to all users?
- Apple has many tools for clearly presenting dates, one of which is the date picker. The date picker makes date input easy to implement. When collecting a date, you'll typically use two cells. The first cell is a right detail cell (or a similar custom cell), where the title label describes the date (such as “Check-In Date,” “Appointment Date,” or “Inspection Date”) and the detail label shows the currently selected date. The second cell contains the date picker.
  - <img src="./resources/date_picker.png" alt="Date Picker" width="400" />
- In your storyboard, add a new section with two sets of date input cells: one set for the check-in date and one set for the check-out date. Each set should have two cells: one to display the selected date and one to display the date picker.
- Referencing the previous image, add two date pickers from the Object library and use Auto Layout to position them. You'll want to set the constraints relative to the content view rather than the margins for the date pickers. The layout constraints and the intrinsic size of the date pickers will cause the rows to size automatically to the correct size.
- Next, set each date picker's `Preferred Style` to `Wheels` and `Mode` to `Date` in the Attributes inspector. Setting the Mode to Date changes the display of the date picker so that the user doesn't have the option to enter a time.
- For the label cells (rows 0 and 2), click the `Table View Cell` and set the `Style` to `Right Detail`. Set the font size for all of the labels to 17 points. Set the title labels to read "Check-In Date" and "Check-Out Date," and set the detail labels to some random date. You'll configure the date label's text programmatically in the next steps.
- With your date input cells set up in the storyboard, you'll need to write code to update the date labels to match the user's input. As with your text fields, you'll need to have access to the date pickers and to the labels that will update. Add outlets for those four views. (You don't need outlets for the Check-In Date and Check-Out Date labels, since they won't change.)

  - ```swift
      class AddRegistrationTableViewController: UITableViewController {
          @IBOutlet var checkInDateLabel: UILabel!
          @IBOutlet var checkInDatePicker: UIDatePicker!
          @IBOutlet var checkOutDateLabel: UILabel!
          @IBOutlet var checkOutDatePicker: UIDatePicker!
          //...
      }
    ```

- With most controls, you can respond when the control's value changes. In the Hotel Manzana app, your code will need to update the detail label every time the date selected in a date picker changes. Before you add your @IBAction method, write an instance function called `updateDateViews()` in `AddRegistrationTableViewController`.
- On the Hotel Manzana form, you'll want the date to read as a string. The date type can be a little tricky to represent, but date formatting can help you out. Using `Date`'s `formatted` method, you can create string representations of dates. The way the formatted string will read is controlled by the parameters that you pass to the `formatted` method.
- The `formatted` method with no arguments will produce a date in a default format. You can supply additional arguments to show only the date or time and to adjust the style (such as displaying the month using a number or a spelled-out name, or omitting seconds from the time display).
- For the Hotel Manzana app, the `.abbreviated` option will work well for dates. Because the hotel doesn't care about the time of day, you can use the `.omitted` option for the time.
- In `updateDateViews()`, add the following code to update the date label text:

  - ```swift
      checkInDateLabel.text = checkInDatePicker.date.formatted(date: .abbreviated, time: .omitted)
      checkOutDateLabel.text = checkOutDatePicker.date.formatted(date: .abbreviated, time: .omitted)
    ```

- Next, you'll configure your date pickers to only allow valid input and to initialize the `date` property when the view loads. Date pickers have a `minimumDate` property and a `maximumDate` property, which work together to prevent the user from selecting a date outside the range. In the Hotel Manzana app, it would make sense to set a minimum date for both pickers: the current day for the check-in date, and one day ahead of the check-in date picker's date for the check-out date.
- The check-in minimum date doesn't change — it's always today — so you can set it in `viewDidLoad()`. You'll also initialize the check-in date in `viewDidLoad()`. However, the check-out minimumDate will change based on the check-in date. You'll set its minimumDate and date in the `updateDateViews()` method. Here's how all of that works together:

  - ```swift
      // In viewDidLoad()
      let midnightToday = Calendar.current.startOfDay(for: Date())
      checkInDatePicker.minimumDate = midnightToday
      checkInDatePicker.date = midnightToday
       
      // At the top of updateDateViews()
      checkOutDatePicker.minimumDate = Calendar.current.date(byAdding: .day, value: 1, to: checkInDatePicker.date)
    ```

- Take a closer look at this code. What's a Calendar? Without knowing the details, you should be able to tell that you're getting the current calendar, which lets you calculate dates using the `date(byAdding:value:to:)` method. In updateDateViews(), you're using that method to **add 1 day to the value of checkInDatePicker.date**.
- Now that the updateDateViews() method is complete, remember you'll need to add an action to both date pickers. In the implementation of the function, call the updateDateViews() method:

  - ```swift
      @IBAction func datePickerValueChanged(_ sender: UIDatePicker) {
          updateDateViews()
      }
    ```

Because your static table view doesn't rely on a data source, it won't “load” when it's first displayed. So it's a good idea to also call the updateDateViews() method in viewDidLoad().
- Remember the doneBarButtonTapped(_:) method? Add some code that will print the date properties of your new controls:
 
@IBAction func doneBarButtonTapped(_ sender: UIBarButtonItem) {
    let firstName = firstNameTextField.text ?? ""
    let lastName = lastNameTextField.text ?? ""
    let email = emailTextField.text ?? ""
    let checkInDate = checkInDatePicker.date
    let checkOutDate = checkOutDatePicker.date
 
    print("DONE TAPPED")
    print("firstName: \(firstName)")
    print("lastName: \(lastName)")
    print("email: \(email)")
    print("checkIn: \(checkInDate)")
    print("checkOut: \(checkOutDate)")
}

- Build and run your app. You should be able to use date pickers to input your date information, and your labels should update accordingly.

- The table view was not displayed.
  - It was because of `numberOfSections` returned `0`.
  - Since you're implementing a static table view, you don't need to implement the data source. In fact, if you'd like, you can delete the boilerplate data source code that comes with a `UITableViewController` subclass.
  - Delete `numberOfSections` method and `tableView(numberOfRowsInSection:)` method, or update the return.

- **Adjust Cell Heights**
  - Have you noticed that your two date picker cells take up a significant portion of an iPhone screen? Date pickers can make it difficult to scroll a table view. Since swiping up on the date picker scrolls the picker wheel, it won't scroll the table view. While this is expected if the user intentionally swipes up on the date picker, it's a large enough control that a user may intend to scroll the table view and inadvertently scroll the picker.
  - How do other apps handle this problem? In Calendar, the date pickers are hidden until the user selects the label row. Plus, the date picker appears right below the labels. When one date picker is shown, any others collapse - so you'll see only one date picker at a time.
  - You will implement a similar interface here, using the table view delegate methods `tableView(_:heightForRowAt:)`, to adjust the height, and `tableView(_:didSelectRowAt:)`, to respond to user interaction and dynamically modify the row's height at runtime. But first, you'll need a few variables to track the state of your views. Add the following properties to your `AddRegistrationTableViewController` class:

    - ```swift
        let checkInDatePickerCellIndexPath = IndexPath(row: 1, section: 1)
        let checkOutDatePickerCellIndexPath = IndexPath(row: 3, section: 1)
         
        var isCheckInDatePickerVisible: Bool = false {
            didSet {
                checkInDatePicker.isHidden = !isCheckInDatePickerVisible
            }
        }
         
        var isCheckOutDatePickerVisible: Bool = false {
            didSet {
                checkOutDatePicker.isHidden = !isCheckOutDatePickerVisible
            }
        }
      ```

  - What's going on in the code above? The first two properties store the index path of the date pickers for easy comparison in the delegate methods. The next two properties store whether or not the date pickers will be shown and then appropriately show or hide them when the properties are set. (They don't adjust the cell height, though. That's coming next.) Both date pickers start as not shown.
  - Now that you have variables to track the visibility of your date pickers, you can implement the `tableView(_:heightForRowAt:)` method. This table view delegate method queries for the row's height when your table view rows are displayed. As the developer, your job is to return the height based on the index path.
  - How will you implement `tableView(_:heightForRowAt:)` in the Hotel Manzana app? By using a switch on indexPath, you can include cases for the two date picker rows with a where clause for each to further define when the case is matched. (A where clause works like an if clause.) Specifically, you'll want to return a height of 0 for the date picker cells when they are not shown. Then you'll use the default for all other situations to let the cells determine their height automatically. Here's what that looks like:

    - ```swift
        override func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
            switch indexPath {
            case checkInDatePickerCellIndexPath where isCheckInDatePickerVisible == false:
                return 0
            case checkOutDatePickerCellIndexPath where isCheckOutDatePickerVisible == false:
                return 0
            default:
                return UITableView.automaticDimension
            }
        }
      ```

  - The rows with the date pickers also need to have a specific estimated row height and can't use `UITableView.automaticDimension`. You can return the correct estimated row height for each row using the `tableView(_:estimatedHeightForRowAt:)` method. The exact value returned for the date cell rows isn't important (as long as it's not 0), because Auto Layout will ensure the correct height is used. A quick check of the Size inspector for the date pickers' cells shows that 190 is a good estimate for the row height when the cells are visible. Notice that you don't want to use the where clause in this case:

    - ```swift
        override func tableView(_ tableView: UITableView, estimatedHeightForRowAt indexPath: IndexPath) -> CGFloat {
            switch indexPath {
            case checkInDatePickerCellIndexPath:
                return 190
            case checkOutDatePickerCellIndexPath:
                return 190
            default:
                return UITableView.automaticDimension
            }
        }
      ```

  - Build and run your app. Because the default value of the visibility variables for the date picker is false, you'll no longer see the date pickers. To show them, you'll need to respond to the user tapping the date label cell.

#### Show or Hide Date Pickers

- As you learned in the first table view lesson, you can respond to a user's cell selection with the delegate method `tableView(_:didSelectRowAt:)`.
- When the user taps a cell in the Hotel Manzana app, you'll first need to deselect the cell — that is, remove its gray highlight. This is a common pattern in the `tableView(_:didSelectRowAt:)` method when tapping the cell performs an action rather than a navigational push/modal, because it wouldn't make sense for the cell to remain highlighted after the action is performed. In the case of a navigation event, it's nice to leave the cell highlighted so when the user returns to the list they'll briefly see the highlighted cell to remind them what they had selected. `UITableViewController` subclasses handle this automatically for you, and you can see the subtle effect in apps like Settings. In the case of an app that allows multiple cell selection, that indication should be managed through accessory views, not the highlighted state of the cell.
- It may be helpful to create properties for `checkInDateLabelCellIndexPath` and `checkOutDateLabelCellIndexPath` to compare with the selected IndexPath.

  - ```swift
      let checkInDateLabelCellIndexPath = IndexPath(row: 0, section: 1)
      let checkOutDateLabelCellIndexPath = IndexPath(row: 2, section: 1)
    ```

- If the index path corresponds to one of the date label cells, you'll toggle the appropriate date picker and update the table view. The requirements of this feature boil down to the following:
  - When both pickers are not visible, selecting a label toggles the visibility of the corresponding picker.
  - When one picker is visible, selecting its own label toggles its visibility; selecting the other label toggles the visibility of both pickers.
- This can be represented by a relatively concise if/else if statement. Finally, when the visibility of a picker is toggled, you must instruct the table view to update itself so that the height for each row is recalculated. This can be done by calling the following methods: `tableView.beginUpdates()` and `tableView.endUpdates()`
- Here is one way to write the logic:

  - ```swift
      override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
          tableView.deselectRow(at: indexPath, animated: true)

          if indexPath == checkInDateLabelCellIndexPath && isCheckOutDatePickerVisible == false {
              // Check-in label selected, check-out picker is not visible, toggle check-in picker
              isCheckInDatePickerVisible.toggle()
          } else if indexPath == checkOutDateLabelCellIndexPath && isCheckInDatePickerVisible == false {
              // Check-out label selected, check-in picker is not visible, toggle check-out picker
              isCheckOutDatePickerVisible.toggle()
          } else if indexPath == checkInDateLabelCellIndexPath || indexPath == checkOutDateLabelCellIndexPath {
              // Either label was selected, previous conditions failed meaning at least one picker is visible, toggle both
              isCheckInDatePickerVisible.toggle()
              isCheckOutDatePickerVisible.toggle()
          } else {
              return
          }

          tableView.beginUpdates()
          tableView.endUpdates()
      }
    ```

- Build and run your app. At this point, in addition to guest information, you should be able to enter check-in and check-out dates while displaying only one date picker at a time.

- The datepicker's height didn't work well. I modified `tableVew(_,heightForRowAt)`

  - ```swift
      override func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
          switch indexPath {
          case checkInDatePickerCellIndexPath:
              if isCheckInDatePickerVisible {
                  return 232
              } else {
                  return 0
              }
          case checkOutDatePickerCellIndexPath:
              if isCheckOutDatePickerVisible {
                  return 232
              } else {
                  return 0
              }
          default:
              return UITableView.automaticDimension
          }
      }
    ```

#### Collect Numbers

- The next type of data you'll collect is numbers. You can use different controls — steppers, sliders, even text fields — depending on the kind of numbers your app will collect.
  - For example, you'll use a stepper when you need precise control of discrete and incremental adjustments to the value.
  - You'll use a slider when you want continuous adjustments over a defined range.
  - And you'll use text fields when a keypad might be more useful to input the number, such as for entering a very large number — and then use a number formatter (similar to a date formatter) to convert the string into a number.
- In the Hotel Manzana app, the staff needs to know how many adults and children are staying at the hotel. Since both these values can be represented with discrete increments of 1, steppers are a good choice. In this case, you'll need two cells, one for entering the number of adults and another for the number of children. Each cell will have three objects: a label describing the entry, another label displaying the value of the stepper, and the stepper itself.
- Using the following image as an example, add a new section with two cells to your table view.
  - <img src="./resources/collect_numbers.png" alt="Collect Numbers" width="500" />
- Next, you'll need outlets for the value labels and steppers:

  - ```swift
      @IBOutlet var numberOfAdultsLabel: UILabel!
      @IBOutlet var numberOfAdultsStepper: UIStepper!
      @IBOutlet var numberOfChildrenLabel: UILabel!
      @IBOutlet var numberOfChildrenStepper: UIStepper!
    ```

- To update your new views, add a method called `updateNumberOfGuests`. This method will be called initially to synchronize the views and again any time the stepper value changes.

  - ```swift
      func updateNumberOfGuests() {
          numberOfAdultsLabel.text = "\(Int(numberOfAdultsStepper.value))"
          numberOfChildrenLabel.text = "\(Int(numberOfChildrenStepper.value))"
      }
    ```

- To respond to a change in stepper value, add an `@IBAction` that calls your `updateNumberOfGuests` function. You'll also call `updateNumberOfGuests` in `viewDidLoad()` to set up the views correctly on the first load.

  - ```swift
      @IBAction func stepperValueChanged(_ sender: UIStepper) {
          updateNumberOfGuests()
      }
      override func viewDidLoad() {
          //...
          updateNumberOfGuests()
          //...
      }
    ```

- Back in your `doneBarButtonTapped(_:)` method, add logic to print the entered values:

  - ```swift
      @IBAction func doneBarButtonTapped(_ sender: UIBarButtonItem) {
          // ...
          let numberOfAdults = Int(numberOfAdultsStepper.value)
          let numberOfChildren = Int(numberOfChildrenStepper.value)
       
          // ...
          print("numberOfAdults: \(numberOfAdults)")
          print("numberOfChildren: \(numberOfChildren)")
      }
    ```

- Build and run your app. You should now be able to adjust the number of guests for each registration and see the labels update accordingly. You should also be able to click the Done button to see the console print the values you entered.

#### Collect Binary Input

- At Hotel Manzana, guests have the option of purchasing Wi-Fi access for $10 per day. This is an example of binary data, where there are two options: yes or no. Switches are perfect for inputting this type of data. For example, in Calendar, you use a switch to indicate whether an event is an all-day event or a timed event.
- The cell will have three objects: a label describing the control, another label listing the price, and the switch itself. Using the image as a reference, add a new section with one cell to your table view.
  - <img src="./resources/collect_binary.png" alt="Collect Binary Input" width="500" />
- Next, add an outlet for the switch to `AddRegistrationTableViewController` so you can access the value when the user taps the Done button: `@IBOutlet var wifiSwitch: UISwitch!`
- What happens when the value of the switch changes? Add an empty action for now. You'll have the opportunity to add functionality for this part of your UI in the challenge at the end of the lesson.

  - ```swift
      @IBAction func wifiSwitchChanged(_ sender: UISwitch) {
          //implemented later
      }
    ```

- Finally, add logic to your `doneBarButtonTapped(_:)` method to print the state of the switch when the user submits the form.

  - ```swift
      @IBAction func doneBarButtonTapped(_ sender: UIBarButtonItem) {
          // ...
          let hasWifi = wifiSwitch.isOn
       
          // ...
          print("wifi: \(hasWifi)")
      }
    ```

- Build and run your app. Verify that you're able to see the correct binary choice displayed in the console.

#### Collect Predefined Options

- What if you need to collect input from a set of predefined options? For example, an e-commerce app may need to know the purchaser's state for shipping, or a college registration app may ask students to select multiple classes for the next term. You could use a segmented control if (and only if) all of these conditions are met: the number of options is small (say, fewer than five), all options have a short name, and the user will choose only one option. But that's a fairly narrow use case.
- What if you need to allow for multiple selections? What if you have a large number of options or they require long descriptions? In these cases, you'll often use an additional table view to present the options and a custom protocol to communicate the user's choice(s). To make it easier to change the displayed choices, programmers will often use dynamic table views for these selection table views.
- If you haven't already added the definition for the RoomType struct introduced at the beginning of this lesson, add a new Swift file to the project named "RoomType" and add the struct definition:

  - ```swift
      struct RoomType: Equatable {
          var id: Int
          var name: String
          var shortName: String
          var price: Int

          // Equatable Protocol Implementation for RoomType
          static func ==(lhs: RoomType, rhs: RoomType) -> Bool {
              return lhs.id == rhs.id
          }
      }
    ```

- Back at the Hotel Manzana, staff will record the guest's room choice on check-in. Add an array of room types as a type property on `RoomType`.

  - ```swift
      static RoomType: Equatable {
          // ...

          static var all: [RoomType] {
              return [
                  RoomType(id: 0, name: "Two Queens", shortName: "2Q", price: 179),
                  RoomType(id: 1, name: "One King", shortName: "K", price: 209),
                  RoomType(id: 2, name: "Penthouse Suite", shortName: "PHS", price: 309)
              ]
          }
      }
    ```

- In the storyboard, add a new `table view controller`, which you'll use to display the room choices and to allow staff to collect the guest's choice. Add a Cocoa Touch Class file for your new controller called “SelectRoomTypeTableViewController.swift.” Set the identity of the new table view controller to the new subclass you just created.
- Next, set up your prototype cell. Change its Style to `Right Detail`. You'll use the title label for the room type, the detail label for the price, and the accessory view to indicate selection. Set the reuse identifier to `RoomTypeCell`.
  - <img src="./resources/room_type_cell.png" alt="Room Type Cell" width="500" />
- In `SelectRoomTypeTableViewController`, implement the data source. Use the newly created array in the RoomType class to populate the table view. If you need a refresher, check out the first lesson on table views.

  - ```swift
      // MARK: - Table view data source
       
      override func numberOfSections(in tableView: UITableView) -> Int {
          return 1
      }
       
      override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
          return RoomType.all.count
      }

      override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
          let cell = tableView.dequeueReusableCell(withIdentifier: "RoomTypeCell", for: indexPath)
          let roomType = RoomType.all[indexPath.row]

          var content = cell.defaultContentConfiguration()
          content.text = roomType.name
          content.secondaryText = "$ \(roomType.price)"
          cell.contentConfiguration = content
          return cell
      }
    ```

- Finally, in Interface Builder, move the arrow designating the initial view controller to `SelectRoomTypeTableViewController`, then run the app. You should now be able to see your populated table view.

#### Implement Selection

- Remember using `tableView(_:didSelectRowAt:)` to make changes when the user selected a date label cell? You'll use the same method to respond to the user's selection in `SelectRoomTypeTableViewController`.
- Start by adding a variable to `SelectRoomTypeTableViewController` to hold the currently selected room type. Make it an optional, because it's possible the staff won't have collected the room choice yet: `var roomType: RoomType?`
- Next, add and implement the `tableView(_:didSelectRowAt:)` delegate method. Start the implementation by deselecting the selected row (removing the gray highlight). Then set the roomType property to the room type that corresponds to the index path. Finally, reload your table view.

  - ```swift
      override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
          tableView.deselectRow(at: indexPath, animated: true)
          roomType = RoomType.all[indexPath.row]
          tableView.reloadData()
      }
    ```

- To indicate the currently selected room type, you'll use the checkmark accessory type. In `tableView(_:cellForRowAt:)`, add logic to configure the cell's accessory type. If the room type for the row is equal to the selected room type, the accessory type is .checkmark. If not, the accessory type is .none. Here's how to write the logic:

  - ```swift
      if roomType == self.roomType {
          cell.accessoryType = .checkmark
      } else {
          cell.accessoryType = .none
      }
    ```

- Build and run your app. If you still have the initial view controller set to SelectRoomTypeTableViewController, you'll be able to see your populated table view and to select a room type.

#### Communicate Selection

- What happens next? Once the selection has been made, you'll need to pass the guest's room choice back to the main table view controller. To do this, you'll create a custom protocol. Recall that protocols define functions and properties that another class will implement. In this case, your selection table view will define a function that's implemented by the main table view controller. The function gives the main table view controller access to any of the parameters of the function — and it allows you write to custom code.
- In the Hotel Manzana app, you'll add a protocol to the `SelectRoomTypeTableViewController` class file called `SelectRoomTypeTableViewControllerDelegate`. In the delegate, define a function called `selectRoomTypeTableViewController(_:didSelect:)` that takes in two parameters: the `SelectRoomTypeTableViewController` and a `RoomType` instance.
- Notice how the naming of this delegate method follows a pattern similar to the ones you're familiar with from implementing `UITableViewDelegate`. When defining delegate and data source methods, it's good practice to include relevant items in the passed arguments. This helps conformers better identify what the method is for in their implementation. Had you chosen to define the delegate method as `didSelect(roomType:)`, it might not have been clear that this method will be called by a `SelectRoomTypeTableViewController` when a room is selected, and someone might have been tempted to start calling the method for other reasons.
- Since another class will implement the protocol, you'll need to define a property to hold the reference to the implementing instance:

  - ```swift
      protocol SelectRoomTypeTableViewControllerDelegate: AnyObject {
          func selectRoomTypeTableViewController(_ controller: SelectRoomTypeTableViewController, didSelect roomType: RoomType)
      }
       
      class SelectRoomTypeTableViewController: UITableViewController {
          weak var delegate: SelectRoomTypeTableViewControllerDelegate?
          //...
      }
    ```

- Now, when the user selects a room type, you can call your new delegate method. In `didSelectRow`, add a call to `selectRoomTypeTableViewController(_:didSelect:)`:

  - ```swift
      override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
          tableView.deselectRow(at: indexPath, animated: true)
          let roomType = RoomType.all[indexPath.row]
          self.roomType = roomType
          delegate?.selectRoomTypeTableViewController(self, didSelect: roomType)
          tableView.reloadData()
      }
    ```

- Next, you'll add `SelectRoomTypeTableViewController` to the app's navigation hierarchy, displaying the view controller when the user taps the associated cell.
- In the `AddRegistrationTableViewController` scene, add a new section with one cell and set the cell's Style to `Right Detail`. You'll use the title label to display a description of the requested information (“Room Type”) and the detail label to display the guest's choice (the room type's name). Set the Accessory to `Disclosure Indicator`. The small chevron indicates that a tap of this cell will push to a new view controller with options. You'll also add an outlet for the detail label, because it will update based on the selected room type: `@IBOutlet var roomTypeLabel: UILabel!`
- Add a property to hold the selected room type: `var roomType: RoomType?`
- Next, add a function to update the room type labels:

  - ```swift
      func updateRoomType() {
          if let roomType = roomType {
              roomTypeLabel.text = roomType.name
          } else {
              roomTypeLabel.text = "Not Set"
          }
      }
    ```

- Add a call to the update room type function in `viewDidLoad()`.
- These properties and methods allow for the selected room type to be displayed and to be updated as the user makes a different selection. The call to `updateRoomType()` in the `viewDidLoad()` method initializes the table view when it first loads.
- Next, you'll conform to the custom protocol of the `SelectRoomTypeTableViewController` and implement the required method, `selectRoomTypeTableViewController(_:didSelect:)`. In the implementation, set the `roomType` property of the `AddRegistrationTableViewController` and update the room type labels. Here are those two steps:

  - ```swift
      class AddRegistrationTableViewController: UITableViewController, SelectRoomTypeTableViewControllerDelegate {
          func selectRoomTypeTableViewController(_ controller: SelectRoomTypeTableViewController, didSelect roomType: RoomType) {
              self.roomType = roomType
              updateRoomType()
          }
          //...
      }
    ```

- Now you'll need to add a show segue from the room type cell to the `SelectRoomTypeTableViewController`. If your initial view controller is still `SelectRoomTypeTableViewController`, change it back to the navigation controller.
- Create an `@IBSegueAction` in `AddRegistrationTableViewController` from your new show segue called `selectRoomType`, with no arguments.
- Finally, in `AddRegistrationTableViewController`, implement the `selectRoomType(_:)` method to set the delegate property and the roomType property of the `SelectRoomTypeTableViewController` if a selection has already been made:

  - ```swift
      @IBSegueAction func selectRoomType(_ coder: NSCoder) -> SelectRoomTypeTableViewController? {
          let selectRoomTypeController = SelectRoomTypeTableViewController(coder: coder)
          selectRoomTypeController?.delegate = self
          selectRoomTypeController?.roomType = roomType
       
          return selectRoomTypeController
      }
    ```

- You've now set up your custom protocol. Here's a review of how it works to communicate the user's choice of room type:
  1. The user taps a cell to make a selection, triggering `didSelectRow`.
  2. The `tableView(_:didSelectRowAt:)` method calls the delegate method `selectRoomTypeTableViewController(_:didSelect:)`, using two things: 
     1. the receiver stored in the delegate property (your `AddRegistrationTableViewController` instance) 
     2. and the index path to the selected room type (which will be used as the parameter of the `selectRoomTypeTableViewController(_:didSelect:)` method).
  3. In the `AddRegistrationTableViewController`, the `selectRoomTypeTableViewController(_:didSelect:)` method provides access to the room type parameter and updates the `AddRegistrationTableViewController` property that's holding the selected room type. This is the implementation of the method called in step 2.
- Before you test your app, don't forget to add the logic to print the room selection when the Done button is tapped:

  - ```swift
      @IBAction func doneBarButtonTapped(_ sender: UIBarButtonItem) {
          //...
          let roomChoice = roomType?.name ?? "Not Set"
          //...
          print("roomType: \(roomChoice)")
      }
    ```

- Build and run your app. You should now see your completed form, and you should be able to select a room type and see it displayed in both table views.

#### Create a New Model Object Instance

- Now that you have an elegant and orderly custom input screen, you'll need to use the entered data to generate your model object. For Hotel Manzana, you'll create a new Registration object when the staff submits the guest information. This model object can be stored using Codable or converted to JSON to transfer to a web server. (You'll learn about JSON in the next unit.) And then it can be passed around to view controllers for manipulation and display.
- An easy way to make a model object instance is to add a computed property to the table view controller of your form. This property will create and return a model object from all the view outlets and view controller properties. You can then use the computed property in an unwind segue to add the new model object to your storage system.
- If you haven't already added the definition for the Registration struct introduced at the beginning of this lesson, add a new Swift file to the project named "Registration" and add the struct definition:

  - ```swift
      struct Registration {
          var firstName: String
          var lastName: String
          var emailAddress: String
       
          var checkInDate: Date
          var checkOutDate: Date
          var numberOfAdults: Int
          var numberOfChildren: Int
       
          var wifi: Bool
          var roomType: RoomType
      }
    ```

- In your `AddRegistrationTableViewController`, add a computed property called registration that returns a `Registration?`. Use the code from the `doneBarButtonTapped(_:)` method to help you initialize a new registration object. If the `roomType` isn't set, return `nil`; otherwise, return a valid Registration object. Here's how to work that:

  - ```swift
      var registration: Registration? {
          guard let roomType = roomType else { return nil }
       
          let firstName = firstNameTextField.text ?? ""
          let lastName = lastNameTextField.text ?? ""
          let email = emailTextField.text ?? ""
          let checkInDate = checkInDatePicker.date
          let checkOutDate = checkOutDatePicker.date
          let numberOfAdults = Int(numberOfAdultsStepper.value)
          let numberOfChildren = Int(numberOfChildrenStepper.value)
          let hasWifi = wifiSwitch.isOn
       
          return Registration(firstName: firstName,
                              lastName: lastName,
                              emailAddress: email,
                              checkInDate: checkInDate,
                              checkOutDate: checkOutDate,
                              numberOfAdults: numberOfAdults,
                              numberOfChildren: numberOfChildren,
                              wifi: hasWifi,
                              roomType: roomType)
      }
    ```

- At this point in the Hotel Manzana app, you'll be able to quickly generate your model object — collecting all the data submitted through the input screen — by simply referencing the `registration` property.

#### Incorporate the Form into a Workflow

- Your input screens are very nice. But if you think about how the hotel staff might use this app, you can imagine that these input screens represent a small part of their overall workflow. For example, after the staff registers a guest, they'd probably like to see and access the information entered.
- These registration objects might be displayed in a table view for quick reference. To incorporate your `AddRegistrationTableViewController` into a full app workflow, you'll be adding a table view that displays the registrations entered and allows the staff to access the `AddRegistrationTableViewController` input screen you built.
- Back in your storyboard, add a navigation controller from the Object library. Set the initial view controller to be the new navigation controller. This table view will display all the registrations that the staff have submitted. Add a Cocoa Touch Class file called “RegistrationTableViewController.swift” and update the new table view controller's identity to match the new class.
- To display each registration, select the prototype cell and set its Style to `Subtitle` and its identifier to `RegistrationCell`. In your new `RegistrationTableViewController` scene, set the navigation item title to Registrations and add an Add bar button item to the right slot of the navigation bar. To display the input screen, add a modal segue from the Add bar button to the navigation controller of the `AddRegistrationTableViewController`.
- In the `RegistrationTableViewController`, add a property that holds an array of Registration objects and set its default value to be empty: `var registrations: [Registration] = []`
- Use the array of Registration objects to implement the data source for the `RegistrationTableViewController`. The registration cells will display the full name of the guest on the title label. The subtitle label will display the check-in date, the check-out date, and the room type. Before using the following code, practice implementing the data source on your own.
- Here's the code for the registration table view data source:

  - ```swift
      override func numberOfSections(in tableView: UITableView) -> Int {
          return 1
      }
       
      override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
          return registrations.count
      }
       
      override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
          let cell = tableView.dequeueReusableCell(withIdentifier: "RegistrationCell", for: indexPath)
       
          let registration = registrations[indexPath.row]
       
          var content = cell.defaultContentConfiguration()
          content.text = registration.firstName + " " + registration.lastName
          content.secondaryText = (registration.checkInDate..<registration.checkOutDate).formatted(date:​ .numeric, time: .omitted) + ": " + registration.roomType.name
          cell.contentConfiguration = content
       
          return cell
      }
    ```

- The assignment of the `content.secondaryText` is using the formatted method to format multiple dates. In this case, you're using a range to capture and format both `checkInDate` and `checkOutDate`. It's a bit cleaner than writing the same formatting code for both dates separately and concatenating the strings.
- Next, add an unwind segue method to `RegistrationTableViewController` so that the `AddRegistrationTableViewController` can return to the `RegistrationTableViewController`. In the implementation, get the source view controller (`AddRegistrationTableViewController`) to access the registration property. If this property isn't `nil`, you'll add it to the registrations array and reload the table view:

  - ```swift
      @IBAction func unwindFromAddRegistration(unwindSegue: UIStoryboardSegue) {
          guard let addRegistrationTableViewController =
            unwindSegue.source as? AddRegistrationTableViewController,
          let registration = addRegistrationTableViewController.registration else
            { return }
       
          registrations.append(registration)
          tableView.reloadData()
      }
    ```

- For the last step, you'll need to adjust the actions sent by the Done bar button on the `AddRegistrationTableViewController`. In the storyboard, select the Done button and open the Connections inspector. In Sent Actions, click the small “x” to delete the action. Instead, hook the unwind segue to the Done button by Control-dragging from the button to the Exit icon of the view controller and choosing `unwindFromAddRegistrationWithUnwindSegue`.
- At this point, you don't need the `doneBarButtonTapped(_:)` method — so go ahead and delete it. (The purpose of the `doneBarButtonTapped(_:)` method was to allow you to test your input screen as you were building it. Now that you've created the Registration object, you no longer need the print statements.)
- What if the user wants to cancel an entry? Drag a bar button item from the Object library to the left bar button slot in the `AddRegistrationTableViewController` scene. Set its System Item to `Cancel`. Add an action that will dismiss the view controller when the button is tapped:

  - ```swift
      @IBAction func cancelButtonTapped(_ sender: Any) {
          dismiss(animated: true, completion: nil)
      }
    ```

- Build and run your app. You should now be able to add new registration objects and see them appear in the table view.
- Nice work! User input can become very difficult to handle. Learning to properly gather and manage this information is an incredibly important part of becoming an app developer. As you come up with your own app ideas, be sure to consider how you can use what you learned in this lesson to better handle user input.

#### Complex Input Screens - Challenge

- Update the `RegistrationTableViewController` with a segue that allows the user to select and view the details of a registration in the `AddRegistrationTableViewController` scene.
- Update the AddRegistrationTableViewController to disable or enable the Done button based on having all of the required information for a reservation. (Hint: Disable the button if self.registration is nil.)
- Add a new section to the table view to display a summary of charges that updates dynamically as the staff enters information.
  - <img src="./resources/complex_input_challenge.png" alt="Complex Input Screens Challenge" width="300" />

#### Lab - Employee Roster

- The objective of this lab is to create a screen that accepts complex user input. You'll use a date picker and a custom delegate to build an employee roster that keeps track of employee information.
- The majority of the app has already been built. You'll find a starter project in your student resources folder. Take a few minutes to look through the project and see what has already been implemented. Go ahead and run the app. You might notice that you can add a new employee and edit existing employees, but you can't edit an employee's birthday or type. You'll add functionality to these fields as you go through the lab.

##### Step 1. Add Birthday Input

- To capture or edit an employee's birthday, you'll need a date picker. In the storyboard, add a third cell to the first section of the `EmployeeDetailTableViewController`. 
  - Drag a date picker to this cell and constrain its top, leading, trailing, and bottom edges to the cell. Set the date picker's Preferred Style to `Wheels` and Mode to `Date` in the Attributes inspector. Create an outlet from the date picker to `EmployeeDetailTableViewController` called `dobDatePicker`.
- You'll want to hide the cell until the user selects it. Start by adding a property to `EmployeeDetailTableViewController` that keeps track of whether the picker should be visible. 
  - Add `isEditingBirthday` and set its initial value to `false`. Add a `didSet` to the property that calls `tableView.beginUpdates()` and `tableView.endUpdates()`. These steps will ensure that the table view calls its delegate methods whenever the value of `isEditingBirthday` changes.
- In addition, you'll need to expand or hide the date picker cell every time the value of `isEditingBirthday` changes. 
  - Start by declaring the table view delegate method `tableView(_:heightForRowAt:)`. In the body of the method, return the height for the cell at the specified index path. 
  - If `indexPath` is equal to the date picker cell's index path and `isEditingBirthday` is `false`, return `0` to hide the picker. Otherwise return `UITableView.automaticDimension` to allow the cells to size themselves.
- Has the user selected the birthday cell on the table view? You now need a way to change the value of `isEditingBirthday` based on the user's selection. 
  - Start by declaring the table view delegate method `tableView(_:didSelectRowAt:)`. In the body of the method, deselect the selected row (regardless of the row) using `tableView.deselectRow(at: indexPath, animated: true)`. 
  - If the index path is the same as the date label's index path, you'll set `isEditingBirthday` to the opposite of what it was previously using `isEditingBirthday.toggle()`. This is a good place to change the text color of the `dobLabel` to `.label` and to use the `formatted` method on the date to set the text on `dobLabel` to the date on the date picker.
- Now, open the storyboard and the assistant editor and create an action from the date picker to `EmployeeDetailTableViewController`. 
  - Set the Event to `Value Changed` so that the action will be called any time the user changes the date on the date picker. In the body of this method, set the text of `dobLabel` to the new selected date.
- Finally, look at the action method saveButtonTapped. Currently, it creates a new Employee object using `Date()`, which returns the current date. Change this method to use the date from the date picker.
- Run the app. Ensure that you can now open the date picker and that the selected date is saved to the new Employee object.

  - ```swift
      @IBOutlet var dobDatePicker: UIDatePicker!
      
      var isEditingBirthday: Bool = false {
          didSet {
              tableView.beginUpdates()
              tableView.endUpdates()
          }
      }

      @IBAction func saveButtonTapped(_ sender: Any) {
          // ...
          let employee = Employee(name: name, dateOfBirth: dobDatePicker.date, employeeType: .exempt)
          // ...
      }

      @IBAction func dobDatePickerChanged(_ sender: UIDatePicker) {
          dobLabel.text = dobDatePicker.date.formatted(date: .numeric, time: .omitted)
      }
      override func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
          if indexPath == IndexPath(row:2, section: 0) && isEditingBirthday == false {
              return 0
          } else {
              return UITableView.automaticDimension
          }
      }

      override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
          tableView.deselectRow(at: indexPath, animated: true)
          if indexPath == IndexPath(row: 1, section: 0) {
              isEditingBirthday.toggle()
              dobLabel.textColor = .label
          }
      }
    ```

##### Step 2. Add Input Screen for Employee Type

- Drag another table view controller from the Object library onto the canvas. This table view will contain a list of the different employee types and allow the user to check one of them.
- Create a show segue from the employee type cell to your new table view controller.
In your new table view controller, set the cell Style to `Basic` and give the cell an identifier.
- Create a new file called “EmployeeTypeTableViewController.swift” that subclasses `UITableViewController`. In the storyboard, set the new table view controller's class to be `EmployeeTypeTableViewController`.
- In `EmployeeTypeTableViewController`, add a property `employeeType` of type `EmployeeType?` to keep track of whether an employee type has been chosen.
- Use the array of `EmployeeType` objects, `EmployeeType.allCases`, to fill in the body of `tableView(_:numberOfRowsInSection:)`. This array is generated by way of the `CaseIterable` protocol on `EmployeeType`.
- In `tableView(_:cellForRowAt:)`, dequeue a cell with the identifier you set in the storyboard. Get the cell's default content configuration and set its text to be the string returned from the `description` property on the correct `EmployeeType` object. Then set the cell's `contentConfiguration` property to the updated content configuration. If the cell's `EmployeeType` object is the same as the one in the `employeeType` property, it's the selected employee type and should have a checkmark next to it; otherwise, it should not. Use the following code to set the checkmark:

  - ```swift
      let type = EmployeeType.allCases[indexPath.row]
       
      var content = cell.defaultContentConfiguration()
      content.text = type.description
      cell.contentConfiguration = content
       
      if employeeType == type {
          cell.accessoryType = .checkmark
      } else {
          cell.accessoryType = .none
      }
    ```

- Finally, implement the table view delegate method `tableView(_:didSelectRowAt:)` to set the employeeType property to the EmployeeType object that corresponds to the selected row and reload the table view.
- Run the app and navigate to the screen you just created. When you select a row, a checkmark should appear next to it. If you select a different row, the prior checkmark should disappear and a new checkmark should appear next to the row your just selected.

  - ```swift
      class EmployeeTypeTableViewController: UITableViewController {
          var employeeType: EmployeeType?
          
          override func viewDidLoad() {
              super.viewDidLoad()
          }

          override func numberOfSections(in tableView: UITableView) -> Int {
              return 1
          }

          override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
              return EmployeeType.allCases.count
          }

          override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
              let cell = tableView.dequeueReusableCell(withIdentifier: "employeeTypeCell", for: indexPath)

              let type = EmployeeType.allCases[indexPath.row]
              
              var content = cell.defaultContentConfiguration()
              content.text = type.description
              cell.contentConfiguration = content
              
              if employeeType == type {
                  cell.accessoryType = .checkmark
              } else {
                  cell.accessoryType = .none
              }
              return cell
          }

          override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
              tableView.deselectRow(at: indexPath, animated: true)
              let employeeType = EmployeeType.allCases[indexPath.row]
              self.employeeType = employeeType
              tableView.reloadData()
          }
      }
    ```

##### Step 3. Add Custom Delegate for Employee Type

- But wait. Your selections in the employee type screen still don't transfer back to the previous screen. You'll need to create a custom delegate protocol to make this work. Declare a protocol named `EmployeeTypeTableViewControllerDelegate` in the `EmployeeTypeTableViewController` file, above its class declaration.
- Inside your new protocol, declare a method `employeeTypeTableViewController(_:didSelect:)` that takes the controller and an `EmployeeType` object and doesn't return anything. Any object that conforms to your custom protocol will be forced by the compiler to implement this method.

  - ```swift
      protocol EmployeeTypeTableViewControllerDelegate: AnyObject {
          func employeeTypeTableViewController(_ controller: EmployeeTypeTableViewController, didSelect employeeType: EmployeeType)
      }
    ```

- In `EmployeeTypeTableViewController`, add a property delegate of type `EmployeeTypeTableViewControllerDelegate?`: `var delegate: EmployeeTypeTableViewControllerDelegate?`
- In `tableView(_:didSelectRowAt:)`, call the delegate's `didSelect(employeeType:)` method after the code that sets the `employeeType` property.

  - ```swift
      override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
          tableView.deselectRow(at: indexPath, animated: true)
          let employeeType = EmployeeType.allCases[indexPath.row]
          self.employeeType = employeeType
          delegate?.employeeTypeTableViewController(self, didSelect: employeeType)
          tableView.reloadData()
      }
    ```

- You now have code that will tell the delegate that a new employee type was selected, but you have no code assigning the delegate. The delegate will be the employee detail screen that presented the employee type screen. Create an `@IBSegueAction` with no arguments for the segue between the `EmployeeDetailTableViewController` and `EmployeeTypeTableViewController` scenes named `showEmployeeTypes`. In its implementation, initialize a new instance of `EmployeeTypeTableViewController` with the provided `coder`, set the delegate property to `self`, and return the new instance. 

  - ```swift
      @IBSegueAction func showEmployeeType(_ coder: NSCoder) -> EmployeeTypeTableViewController? {
          let employeeTypeTableViewController = EmployeeTypeTableViewController(coder: coder)
          employeeTypeTableViewController?.delegate = self
          
          return employeeTypeTableViewController
      }
    ```

- This should cause the compiler to throw an error, because `EmployeeDetailTableViewController` doesn't conform  to `EmployeeTypeTableViewControllerDelegate`. Fix this by adding `EmployeeTypeTableViewControllerDelegate` to the class declaration and implementing the `employeeTypeTableViewController(_:didSelect:)` method.
- `EmployeeDetailTableViewController` will also need a property `employeeType` of type `EmployeeType?` to keep track of the selected employee type. Add this now.
- In `employeeTypeTableViewController(_:didSelect:)`, assign the `employeeType` passed into the method to `self.employeeType`, update the text of `employeeTypeLabel`, and make sure the text color of `employeeTypeLabel` is now `.black` (since an employee type has been selected).

  - ```swift
      class EmployeeDetailTableViewController: UITableViewController, UITextFieldDelegate, EmployeeTypeTableViewControllerDelegate {
          func employeeTypeTableViewController(_ controller: EmployeeTypeTableViewController, didSelect employeeType: EmployeeType) {
              self.employeeType = employeeType
              employeeTypeLabel.text = employeeType.description
              employeeTypeLabel.textColor = .black
          }

          var employeeType: EmployeeType?

          // ...

          override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
              guard let employeeTypeTableViewController = segue.destination as? EmployeeTypeTableViewController else {return}
              
              employeeTypeTableViewController.employeeType = employee?.employeeType
              employeeTypeTableViewController.delegate = self
          }
    ```

- In the `saveButtonTapped` action method, unwrap `employeeType` just after the code that unwraps the text from `nameTextField`. Then pass the unwrapped `EmployeeType` object into the initializer that's creating a new `Employee`.

  - ```swift
      @IBAction func saveButtonTapped(_ sender: Any) {
          guard let name = nameTextField.text, let employeeType = employeeType else {return}
          
          let employee = Employee(name: name, dateOfBirth: dobDatePicker.date, employeeType: employeeType)
          delegate?.employeeDetailTableViewController(self, didSave: employee)
      }
    ```

- Finally, since you've updated the requirements to save, you should update the `updateSaveButtonState()` method to only enable the Save button if `employeeType` contains a value. Also, call `updateSaveButtonState()` within `employeeTypeTableViewController(_:didSelect:)`.

  - ```swift
      private func updateSaveButtonState() {
          let shouldEnableSaveButton = (nameTextField.text?.isEmpty == false) && ((employeeType?.hashValue) != nil)
          saveBarButtonItem.isEnabled = shouldEnableSaveButton
      }
    ```

- Run the app and confirm that you can create a full Employee object that stores an employee's name, birthday, and type.
- Congratulations! You added complex input features to the Employee Roster, giving the user a much smoother and simpler experience. Be sure to save your final product to your project folder.

##### Challenge - User Experience Enhancement

- The date of the birth date picker defaults to today's date, but it's highly unlikely that you'll be adding an employee who was just born! What default values could reduce the required scrolling of the date picker? Hint: Employees are probably between the ages of 16 and 65 and can be born in any month and on any day.

  - ```swift
      func updateView() {
          if let employee = employee {
              // ...
          } else {
              navigationItem.title = "New Employee"
              dobDatePicker.maximumDate = Calendar.current.date(byAdding: .year, value: -16, to: Date())
              dobDatePicker.minimumDate = Calendar.current.date(byAdding: .year, value: -65, to: Date())
          }
      }
    ```

### Guided Project: List

- In this unit, you learned how to display lists using table views, build screens that allow for complex input, and save data to disk. In this guided project, you'll combine those skills to create an app that manages a list and stores it away for later retrieval. You can add, modify, and delete items in the list using a custom input screen. Depending on what type of items are on your list, you can choose from various controls to give your users the best possible form of input.

#### Part One - Project Planning

- Think of a collection of items that you'd like to manage with an app. Maybe you have a collection of baseball cards that you trade with your friends, and you need an easy way to add and remove cards from the list. Or maybe you want to keep track of homework assignments: What are their due dates, which have you finished, and which have been turned in to the teacher?
- In this guided project, you'll work with a to-do list of action items. Each to-do has a due date and a set of notes; it can be marked as complete, but remains in the list until the user deletes it. A more advanced to-do manager might include ways to sync the list across multiple devices or to notify the user as the due date approaches, but those capabilities are beyond the scope of this unit. You'll find examples of ways to enhance your app at the end of the guided project.
- Features
  - Nearly every app that manages a list of items will handle the following actions:
    1. Display the list
    2. Add items to the list
    3. Edit existing items on the list
    4. Delete items from the list
    5. Automatically save the list to disk
- What type of items will the list display? In most cases, your answer will affect the complexity of the second and third features. For example, if your app will manage a baseball card collection, it might need to store an image of each card, the card's quality, the card's market value, and the price paid for the card. The more properties you include per item, the more methods of input you'll need to build into your app.
- **Workflow**
  - What iOS apps include similar features to the list above? Take a look at the built-in Mail app. It displays a list of email messages and provides an easy way to delete messages. On the list screen, there's a button to create a new email, which will be added to the Sent folder once it's been sent. If you tap an email in the list of drafts, the interface to edit the draft is identical to the interface for creating a new email message.
  - Contacts is another app with a similar feature set. Contacts are displayed in a table, and at the top of the table view is a button for adding a new contact. Tapping a name in the table shows more details about each contact and provides an interface for editing contact info.
  - As you can see, there's nothing revolutionary about this type of app. Even though you might like to create something totally fresh and unique, it's actually a better idea to follow an approach that's easy and familiar to your users.
  - The app you'll build should adhere to these simple specs:
    - When the app opens, it displays a list of items.
    - The list screen has controls for adding and deleting items on the list.
    - Tapping an item displays more details about the item.
    - The detail screen permits editing.
    - The app saves the latest version of the list.
- **Controllers**
  - Are you ready to imagine the interface elements and how you'll put them together?
  - Since you'll be displaying items in a table view, you'll use a `UITableViewController` to manage the displaying, editing, and deletion of cells. What about a controller to manage the collection of items? You could create an `ItemController`, or something similar, to manage the array. But since this app will have a limited number of screens, the `UITableViewController` should be able to handle the collection that its table view represents. The same controller will be responsible for saving the items to disk whenever the list is updated — after inserting, editing, or deleting.
  - One last thing: If you include an image for each item, you'll need a `UIImagePickerController` to allow the user to add a photo to the table or to edit an existing one. In any case, your interface won't have more than a few controllers.
- **Views**
  - Depending on how much information you choose to display in each table cell, you may need to create a `UITableViewCell` subclass so that you can add additional labels, images, or controls to the cell. On the screen for adding and editing list items, you'll use a set of iOS input controls that best correspond to the item's properties. For example, if each item in the collection has a date associated with it, you'll want to use a date picker. If the item needs a description, you'll want to include a text field or text view, depending on the length of the description.
- **Models**
  - Your app is centered around one model — the particular type of item it will display and manage. This will be a ToDo model that will include a title, a due date, some extra notes, and a way to mark each to-do as complete. Since your collection must be stored to disk for future retrieval, your objects will need to adopt the Codable protocol that you learned about in this unit.

#### Part Two - Set Up Project and Display Model

- Create an Xcode project using the iOS App template and name it "ToDoList." When creating the project, make sure the interface option is set to "Storyboard." You've used this template many times in earlier lessons, so you know it handles a lot of up-front work for you. It starts you with a storyboard, an app delegate, and a custom view controller. Specifically, the Main storyboard has set the initial view controller to be of type ViewController, a subclass of UIViewController.
- But in the ToDoList app, your first screen will display a table, not a plain white screen. So you'll need to delete the existing view controller and add a UITableViewController inside a UINavigationController. These first few steps will configure the initial screen appropriately and prepare you to display a model in the table.
- Open the Main storyboard and drag a navigation controller from the Object library onto the canvas. Conveniently, this step also adds a UITableViewController with the UINavigationController attached. You can now delete ViewController, since you'll be creating new files with the appropriate subclass.
- If you were to build and run the app, you'd still see the single white screen. To fix this, adjust the initial view controller in the storyboard by dragging the entry point arrow to the navigation controller. Once you've made this connection, it's safe to delete the storyboard's original view controller.
- Now, if you run your app, you'll see an empty table with the title “Root View Controller” in the navigation bar. You can adjust the title by selecting the table view controller's navigation item in the Document Outline and updating its title in the Attributes inspector. Change the title to `My To-Dos`.
- Select the navigation controller's navigation bar and check the `Prefers Large Titles` box in the Attributes inspector. This will make your title display left-aligned and in a larger font.
- **Define the Model**
  - Before you can begin displaying your items in a table, you'll need to define the model. From the Xcode menu bar, choose File > New > File, then select "Swift File." You can name the file anything you like, but it's a good practice to give it the same name as your model. In this guided project, you'll use a type called `ToDo`.
  - Open the Swift file and create your ToDo struct: `struct ToDo { }`
  - What properties will each of your model objects require? If the model object is Homework, it may have a dueDate property of type Date and a subject property used in a custom Subject enumeration. If your model includes a UIImage property, you'll need to change the import Foundation line at the beginning of the Swift file to import UIKit. (The Foundation framework doesn't contain any platform-specific classes, and UIImage is specific to iOS.)
  - An example of the properties for a to-do list are shown below.

    - ```swift
        struct ToDo: Equatable {
            let id = UUID()
            var title: String
            var isComplete: Bool
            var dueDate: Date
            var notes: String?
         
            static func ==(lhs: ToDo, rhs: ToDo) -> Bool {
                return lhs.id == rhs.id
            }
        }
      ```

  - Note the use of `UUID()`. This is a system-provided type that is a universally unique value that can be used to identify types. Anytime you create one, you're theoretically guaranteed to get a different value, making it a reliable unique identifier. Apple's implementation conforms to [RFC 4122 version 4](https://datatracker.ietf.org/doc/html/rfc4122), if you're interested in learning more.
- **Configure the Table View Controller**
  - Now that you've defined the model, you need to create the `UITableViewController` subclass to display a collection of models. Create the new subclass and name it something that lends itself to the list you're creating. If you're not deviating from these instructions, name it `ToDoTableViewController`.
  - In addition to displaying the list in the table, the table view controller will manage the collection of items. Create an empty array of your model objects inside the class definition: `var toDos = [ToDo]()`
  - Does your table view have multiple sections to consider? In the Contacts app, cells are separated into sections based on the first letter of the first name or the first letter of the last name. In Mail, there are no separate sections for cells. Your app is probably more like Mail, so all cells will be displayed in a single section — hence, you can delete the `numberOfSections(in:)` delegate method.
  - If all model objects will be displayed in a cell and all cells are in the same section, `tableView(_:numberOfRowsInSection:)` can simply return the number of objects in your collection. In other words, there should be one table view cell for each entry in your array of models.

    - ```swift
        override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
            return toDos.count
        }
      ```

  - The next method to override is `tableView(_:cellForRowAt:)`. This method is called whenever a cell is about to be displayed onscreen, and the method provides you the IndexPath that you'll use to determine which cell you're dealing with. Ignoring custom cells for now, you'll follow the standard implementation for this method that you learned in previous lessons:
    1. Dequeue a cell for the given index path.
    2. Get the model out of the array that corresponds to the cell being displayed.
    3. Update the cell's properties appropriately and return the cell from the method.
  - In the following code snippet, a standard `UITableViewCell` is dequeued, and its textLabel text is set to the title property of the corresponding ToDo. Even though the cell doesn't display other properties of the ToDo, it will help you verify that you've set up the table view controller correctly.

    - ```swift
        override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
            let cell = tableView.dequeueReusableCell(withIdentifier: "ToDoCellIdentifier", for: indexPath)

            let toDo = toDos[indexPath.row]
            var content = cell.defaultContentConfiguration()
            content.text = toDo.title
            cell.contentConfiguration = content
            return cell
        }
      ```

  - Your table view controller subclass is not being utilized yet, because the storyboard is using a regular `UITableViewController`. Open the Main storyboard and select the table view controller. Using the Identity inspector, set the Custom Class to the `UITableViewController` subclass — in this case, `ToDoTableViewController`.
  - Now try building and running. You should see your table view controller inside a navigation controller with your own custom title.
- **Supply Initial Data**
  - The feature list for this project includes adding items to your collection and saving them to disk. But it doesn't make sense to write these features before you can see anything in the table. For now, you can populate the ToDo array with some initial data if no data is retrieved from disk.
  - First, write a static method in the ToDo structure that retrieves the array of items stored on disk, if there are any, and returns them. You'll write the body of this method later, when it comes time to store and retrieve items from disk. For now, simply return nil.
  
    - ```swift
        static func loadToDos() -> [ToDo]?  {
            return nil
        }
      ```

  - Next, write a static method that populates the array with sample data. You can create as many model objects as you wish, but don't go crazy — the user will eventually clear out these fake models to make room for their actual data. And don't worry about the values you give each property. With sample data, you just need enough information to distinguish one item from the next.
  - In this example, each ToDo has been given a different title property, since the cell will need to display this property:

    - ```swift
        static func loadSampleToDos() -> [ToDo] {
            let toDo1 = ToDo(title: "To-Do One", isComplete: false, dueDate: Date(), notes: "Notes 1")
            let toDo2 = ToDo(title: "To-Do Two", isComplete: false, dueDate: Date(), notes: "Notes 2")
            let toDo3 = ToDo(title: "To-Do Three", isComplete: false, dueDate: Date(), notes: "Notes 3")
         
            return [toDo1, toDo2, toDo3]
        }
      ```

  - Now that you have some model data to see, build and run your project. You'll find that the table still isn't displaying any information. Why not?
  - As you learned in earlier units, breakpoints are excellent for debugging problems, because they let you inspect the code as it's being executed. Try placing breakpoints in two of your `UITableViewController` methods: `tableView(_:numberOfRowsInSection) and tableView(_:cellForRowAt)`. Then run your app again. What do you notice?
  - The first method, which determines how many cells should be displayed, is called. But there are no items in the array, so the method returns 0 — and the second method is never called. Had the first method returned 3, `tableView(_:cellForRowAt:)` would have been called three times, and three items would have been displayed.
  - What you'll need to do is add some conditional logic inside viewDidLoad(). Try to load items from disk; if there aren't any, use the `loadSampleToDos()` method to fill the array with data:

    - ```swift
        override func viewDidLoad() {
            super.viewDidLoad()
         
            if let savedToDos = ToDo.loadToDos() {
                toDos = savedToDos
            } else {
                toDos = ToDo.loadSampleToDos()
            }
        }
      ```

  - One more thing. If you build and run the app now, it will crash with an assertion failure: Terminating app due to uncaught exception 'NSInternalInconsistencyException', reason: 'unable to dequeue a cell with identifier `ToDoCellIdentifier` - must register a nib or a class for the identifier or connect a prototype cell in a storyboard'. In `tableView(_:cellForRowAt:)`, the app was unable to dequeue a cell with the specified identifier, because the identifier hasn't been added to the storyboard.
  - Open the Main storyboard and select the prototype cell in the table view. In the Attributes inspector, use the same identifier string that you supplied in code. With the identifier in place, the sample data should now display.

#### Part Three - Add and Delete Controls

- Your app can now display data. It's time to create controls so that users can add items to the list and delete existing ones. To add items, you can use the + button — which you've probably noticed in the upper-right corner of the navigation bar on many iOS apps. A tap of the button will modally present a view controller that allows the user to enter model data. For quick removal of an item, you can use a cell's swipe-to-delete functionality. But you might also want to place an Edit button in the upper-left corner of the navigation bar, in case the user isn't familiar with swipe-to-delete. More on this later.
- **Add Items**
  - Open the Main storyboard and select the table view controller. Drag a bar button item from the Object library to the upper-right corner of the navigation bar's navigation item. To set the button to a + symbol instead of text, open the Attributes inspector and change the `System Item` to `Add`.
  - Tapping the + button should modally present a new view controller, which will use a static table view contained in a navigation controller. Drag a new navigation controller from the Object library onto the canvas. As before, this step will add both a navigation controller and a table view controller.
  - Control-drag from the + button to the new navigation controller and choose `Present Modally` in the popover. Take a moment to update the title of the new table view controller, making it clear that it's for creating new items. In this project, the title is “New To-Do.”  Select the new table view controller's navigation item and, from the `Large Title` dropdown, select `Never`. This will help make it clear to the user that this page is a sub-page of the primary to-do list.
- **Delete Items**
  - To add swipe-to-delete functionality to your table view controller's cells, you'll need to override the `tableView(_:canEditRowAt:)` method. You can use the `indexPath` property to choose which cells are editable. In this app, since all cells can be edited, you'll simply return `true`.
  - Open the `ToDoTableViewController` class definition and add (or uncomment) the following implementation:

    - ```swift
        override func tableView(_ tableView: UITableView, canEditRowAt indexPath: IndexPath) -> Bool {
            return true
        }
      ```

  - When the cell is swiped, a red Delete button appears to the right of the cell. You'll need to configure exactly how deletion will take place. Override the `tableView(_:commit:forRowAt:)` method. Inside its implementation, verify that the Delete button triggered the method call, then delete the model from the array and from the table view.

    - ```swift
        override func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath) {
            if editingStyle == .delete {
                toDos.remove(at: indexPath.row)
                tableView.deleteRows(at: [indexPath], with: .automatic)
            }
        }
      ```

  - There's one last thing to complete the delete functionality: an Edit button. You could add a bar button item from the Object library and set its `System Item` property to `Edit`, but there's a better alternative. You can create an intelligent Edit button that will place the entire table view into editing mode, displaying - delete buttons to the left of each cell. When tapped, the button will switch its text from “Edit” to “Done” — and back again at the next tap.
  - Add the following line of code to the viewDidLoad() method of your table view controller: `navigationItem.leftBarButtonItem = editButtonItem`
  - Build and run your app and try clicking the Edit button. Notice how the button updates its title to Done and places the table view in edit mode. And when you tap a delete button, the item is removed from the collection.

#### Part Four - Static Table View Interface

- In this section, you'll learn the steps necessary for the user to populate the ToDo items. What kind of controls do you need to create? The answer depends on the properties you've included in your model.
- Before you begin to think through the controls, you'll need to adjust some attributes of the table view (not the table view controller) of New To-Do Controller. In the Attributes inspector, switch the `Content` type from `Dynamic Prototypes` to `Static Cells`. Then switch the `Style` to `Grouped`, so that each section you add is clearly separated from the other sections.
- How many sections will you need in the static table view? It depends on how many controls you'll add to the screen. You can easily adjust the number of sections using the `Sections` property in the Attributes inspector, then modify the `Rows` property in each section to change the number of rows. For the to-do list, each section will have only one row.
- The result interface is as follow:
  - <img src="./resources/static_tableview_interface.png" alt="" width="200" />
- **Save and Cancel Buttons**
  - After the user has entered a new item, they'll probably expect to tap a Save button to confirm its entry. Or, if they decide against an entry, they may look for a Cancel button. Whether saving or canceling, they'll expect to return to the table view.
  - You've learned that most modally presented view controllers need to include some sort of control to dismiss the current view and return to the screen from which it was presented. In this case, you'll return to the table view using an unwind segue.
  - Begin by adding a method in your `ToDoTableViewController` class that allows future button taps to unwind to this view controller. You'll probably recognize the following unwind segue from previous lessons: `@IBAction func unwindToToDoList(segue: UIStoryboardSegue) { }`
  - Open the Main storyboard and add two bar button items to the top of the new table view controller. Update the left bar button's System Item attribute to `Cancel` and the right bar button's System Item to `Save`. Then Control-drag from each button to the Exit control at the top of the table view controller and select the `unwindToToDoListWithSegue` method in the popover. To distinguish the two segues, give the unwind segue connected to the Save button an identifier of `saveUnwind`.  You don't need to enter an identifier for the Cancel button, because nothing additional needs to happen when it's tapped.
- **Basic Information**
  - Each ToDo has two basic pieces of information: a title and whether or not the to-do is complete. You can use a `UITextField` to add the `title` property. For `isComplete`, use a `UIButton` with two alternative images: an empty circle and a circle with a checkmark.
  - To create the cell that will contain the `title` and `isComplete`, select the section from the Document Outline and update its Header to “Basic Information.” Next, drag a `button` and a `text field` from the Object library onto the table view cell.
  - In the Attributes inspector for the button, set the Type to `Custom`, set the Style to `Default`, and set the Title to an `empty string`. Then set the State Config property to `Selected` and set the Image property to the `checkmark.circle.fill` system image. Under the Selected Symbol Configuration section, set Configuration to `Point Size` and Point Size to `24`. Last, change the State Config property back to `Default`, then set the Image property to the `circle` system image and use the same Symbol Configuration settings that you just used for the Selected state.
  - In the Attributes inspector for the text field, set the Placeholder to “Remind me to...” to clarify the purpose of the field.
- **Due Date**
  - The due date cells are a bit more dynamic than the basic information cell. When the screen is first displayed, there will be one visible cell that displays “Due Date” on the left and a short date and time on the right. When the cell is tapped, the date picker cell expands to reveal a UIDatePicker that allows the user to adjust the date and time. Selecting the cell again hides the date picker. 
  - In this section, you'll focus only on adding the labels and controls to the due date cell. You'll tackle hiding and showing the picker later.
  - Begin by adding a new section to your table view in the storyboard. This section should have two cells. Set the first cell's Style to `Right Detail` and configure it(set the font size for the labels to `17` points). Set the second cell's Style to `Custom` and add a date picker to it from the Object library. Add constraints to the top, trailing, bottom, and leading edges of the picker. Set the constraints to the superview, not to the margins. Use the Attributes inspector to set the Preferred Style of the date picker to `Wheels`.
- **Notes**
  - The ToDoList app will allows the user to add optional notes to a ToDo item.
  - To create this cell, select the section in the Document Outline and update its Header to “Notes.” Next, select the cell in the Document Outline and update its Row Height to `200`. Finally, drag a text view from the Object library onto the table view cell. Add four constraints to the text view that align its top, leading, bottom, and trailing edges to the edges of the cell. You can remove the placeholder text in text view using the Attributes inspector.
- **Additional Controls**
  - The checkmark button, date picker, and text view are just a few of the controls that you could include in your list tracker app. For example, if your app is managing a collection of baseball cards, you might want a cell with a `UISegmentedControl` that allows the user to select the card's condition: Mint, Near Mint, Fair, etc. Or maybe you want to allow the user to associate an image with the model object, which would require a `UIImagePickerController` for selecting an image from their photo library. Refer to the “Controls in Action” and “System View Controllers” lessons if you need help configuring these controls.

#### Part Five - Connect the Static Table View to Code

- Take a step back to review everything you've built so far. You have a table view controller that displays a list of items, and you've defined the properties for the model. Items from the list can be deleted, and you've created the interface for inputting data. Now it's time to add some code to make the buttons, labels, and controls in the static table view update properly.
- **Add View Controller Subclass**
  - Start by creating an additional subclass of UITableViewController called `ToDoDetailTableViewController` so that you can add outlets to the controls, read their values, add actions, and create a new model with the data that the user supplies. Since this is a static table view, you can remove the data source methods provided by Xcode's template.
    - Remove `numberOfSections(in:)` and `tableView(_:numberOfRowsInSection)`.
  - Now return to the Main storyboard. Update the Custom Class of the static table view controller to match the name of the class you just created. With the class set properly, you can use the assistant editor to create outlets, one for each control or view that you'll update in code or read values from to create your model object.
  - The outlets created for the ToDo cells are as follows:

    - ```swift
        @IBOutlet var titleTextField: UITextField!
        @IBOutlet var isCompleteButton: UIButton!
        @IBOutlet var dueDateLabel: UILabel!
        @IBOutlet var dueDateDatePicker: UIDatePicker!
        @IBOutlet var notesTextView: UITextView!
      ```

- **Disable the Save Button**
  - Every ToDo requires a title. If there's no text in the title text field, the user shouldn't be able to save the item.
  - To disable the Save button, start by creating an outlet in code: `@IBOutlet var saveButton: UIBarButtonItem!`
  - Next, write a helper method that updates the Save button depending on whether or not text exists in the text field. If the string is empty, disable the button; otherwise, enable it:

    - ```swift
        func updateSaveButtonState() {
            let shouldEnableSaveButton = titleTextField.text?.isEmpty == false
            saveButton.isEnabled = shouldEnableSaveButton
        }
      ```

  - You'll need to call this method in `viewDidLoad()` so that the button is disabled as soon as the user brings up the view controller to add a new item.

    - ```swift
        override func viewDidLoad() {
            super.viewDidLoad()
            updateSaveButtonState()
        }
      ```

  - The `updateSaveButtonState()` method should be called after each keyboard tap in the text field, ensuring that the state of the button is always up to date. Open the assistant editor, then Control-drag from the text field to an available spot in the view controller class definition. Create a new action called `textEditingChanged` that will fire whenever the `Editing Changed` control event takes place.
  - In the definition of `textEditingChanged`, call `updateSaveButtonState()`:

    - ```swift
        @IBAction func textEditingChanged(_ sender: UITextField) {
            updateSaveButtonState()
        }
      ```

  - You've now prevented the user from creating a ToDo without a title.
- **Dismiss Keyboard on Return**
  - If you run your app at this point, you'll find that when you are editing the title text field, the keyboard blocks the notes text view and part of the date picker. It would be great if the user could tap the Return key on the keyboard to dismiss the keyboard, making it easier to interact with the other controls.
  - With the assistant editor open, Control-drag again from the text field to an available spot in the view controller class definition. Create an action that will fire whenever the `Primary Action Triggered` control event takes place. In the case of `UITextField`, the action will fire when Return is tapped. When the action occurs, resign the text field from its role as the first responder:

    - ```swift
        @IBAction func returnPressed(_ sender: UITextField) {
            sender.resignFirstResponder()
        }
      ```

- **Switch Button Image**
  - Is the ToDo complete or not? Earlier, you set two images to reflect isCompleteButton's Default state and its Selected state. When the user taps the button, you want the image to toggle between the empty circle and the checkmark image.
  - Add an action to the button that changes the isSelected property whenever it's tapped: Control-drag from the button to an available space in the view controller definition and create an action called `isCompleteButtonTapped`.

    - ```swift
        @IBAction func isCompleteButtonTapped(_ sender: UIButton) {
            isCompleteButton.isSelected.toggle()
        }
      ```

  - Build and run the app. Clicking this button in the static table view will switch the isCompleteButton image from selected to unselected and vice versa.
- **Update Date Label**
  - The text for `dueDateLabel` should reflect the value the user entered in the date picker. As with the Save button, you'll update the value in `viewDidLoad()` so it's correct before being displayed to the user.
  - You'll use the formatted method on Date to generate the date string. Previously you have used `formatted(date:time:)`, but you can have even greater control over the way the date is formatted by passing a style argument to provide detailed settings for each field. In this case you'll use the `.dateTime` style and customize it to display the date and time in the most succinct format possible. For example, using: `date.formatted(.dateTime.month(.defaultDigits).day().year(.twoDigits).hour().minute())` would cause "January 1st, 1970, at 12:00am" to appear as "1/1/70, 12:00 AM." Note that the order of the fields does not matter as they will be ordered according to the locale in use at runtime.
  - Next, write a helper method in your detail table view controller to update `dueDateLabel` with the date passed into the method as a parameter. This method should be called in `viewDidLoad()` and whenever the date picker value changes:

    - ```swift
        override func viewDidLoad() {
            super.viewDidLoad()
            updateDueDateLabel(date: dueDateDatePicker.date)
            updateSaveButtonState()
        }
         
        func updateDueDateLabel(date: Date) {
            dueDateLabel.text = date.formatted(.dateTime.month(.defaultDigits).day().year(.twoDigits).hour().minute())
        }
      ```

  - To create an action that's fired whenever the user changes the date picker, Control-drag from the date picker to an available space in the view controller class definition, then tie the action to the `Primary Action Triggered` event.

    - ```swift
        @IBAction func datePickerChanged(_ sender: UIDatePicker) {
            updateDueDateLabel(date: sender.date)
        }
      ```

  - Now the due date label will display a string of text that matches the value in the date picker.
- **Update the Date Picker Starting Value**
  - At the moment, the date picker displays the current date and time when the view controller is first displayed. Does that makes sense for something that hasn't been done? A more reasonable starting value might be 24 hours from now.
  - In `viewDidLoad()`, you can set the date picker's date before updating the date label. The Date class makes it easy to calculate a date that's 24 hours in the future. Starting with `Date()`, which creates a date with the current date and time, you can use the `addingTimeInterval(_:)` method to add any number of seconds. How many seconds are in 24 hours? Multiply 24 by 60 to convert hours into minutes, then multiply by 60 again to convert minutes into seconds.

    - ```swift
        override func viewDidLoad() {
            super.viewDidLoad()
            dueDateDatePicker.date = Date().addingTimeInterval(24*60*60)
            updateDueDateLabel(date: dueDateDatePicker.date)
            updateSaveButtonState()
        }
      ```

  - Now when the static table view is displayed, the date picker will display a more logical starting value. Build and run the app to test your date picker.
- **Expand and Collapse the Date Picker Cell**
  - Having the date picker open all the time can make scrolling problematic for the table view. To avoid this, you can hide the picker until the user taps the date label cell, the same way you did in an earlier lesson.
  - To set the height of each cell dynamically, you can override the `tableView(_:heightForRowAt:)` method, rather than using the values you supplied in Interface Builder.
  - What's the right height for each cell? When a cell is visible, you should allow Auto Layout to handle the size, but when the date picker should be hidden, the height for that cell should be 0.
  - Your app will need a way to know if the date picker is hidden or not. Its initial state should be hidden, so you'll add a Bool property to the detail table view controller class and set it to `true`. While you are there, add properties to store the index paths for the date label and date picker cells:

    - ```swift
        var isDatePickerHidden = true
        let dateLabelIndexPath = IndexPath(row: 0, section: 1)
        let datePickerIndexPath = IndexPath(row: 1, section: 1)
        let notesIndexPath = IndexPath(row: 0, section: 2)
      ```

  - Next, you'll use a switch statement in the `tableView(_:heightForRowAt:)` delegate method to handle the date picker and notes index paths accordingly. In the case where the index path is the picker cell's and the picker is hidden, return `0`. For the notes index path, return `200`. Otherwise, you'll return `UITableView.automaticDimension` so that the other cells can size themselves.

    - ```swift
        override func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
            switch indexPath {
            case datePickerIndexPath where isDatePickerHidden == true:
                return 0
            case notesIndexPath:
                return 200
            default:
                return UITableView.automaticDimension
            }
        }
      ```

  - You'll also need to provide an estimated row height for each row in the table. In the case of the date picker index path, the value does not need to be exact-216 is close to the value in the Size inspector for the cell and is a good choice. For the notes index path, return `200`. Otherwise, you'll return `UITableView.automaticDimension`.

    - ```swift
        override func tableView(_ tableView: UITableView, estimatedHeightForRowAt indexPath: IndexPath) -> CGFloat {
            switch indexPath {
            case datePickerIndexPath:
                return 216
            case notesIndexPath:
                return 200
            default:
                return UITableView.automaticDimension
            }
        }
      ```

  - Next, your app will need to respond to the user tapping the Due Date cell, which will change `isDatePickerHidden` to `false`. You can override `tableView(_:didSelectRowAt:)` to handle the tap, but you're only concerned with this one cell. Similar to the previous method, you can check whether the index path matches up with the label cell's. If it does, your code will need to toggle `isDatePickerHidden`, then update the cell height. To animate a cell height adjustment, an easy trick is to call a table view's `beginUpdates` and `endUpdates` methods without any code between the two. This is also a good time to set the label to the date value on the picker.

    - ```swift
        override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
            tableView.deselectRow(at: indexPath, animated: true)
            if indexPath == dateLabelIndexPath {
                isDatePickerHidden.toggle()
                updateDueDateLabel(date: dueDateDatePicker.date)
                tableView.beginUpdates()
                tableView.endUpdates()
            }
        }
      ```

  - Build and run your app. You should have a Save button that ensures the user provides a title before saving, a text field that dismisses the keyboard when Return is tapped, a functional checkmark button that switches images on each tap, and a date picker view that shows and hides with an animation.

#### Part Six - Create and Save the Model

- What have you created so far? You have a table view controller that displays your list of items, but the list is using some initial data rather than data created through the static table view. You've added controls that enable the user to set values for each property in your model, but the Save button dismisses the modal view controller without actually saving the data. In this section, you'll create a new model object using data from the controls and update your list accordingly.
- **Read Data from Controls**
  - At the moment, your Save button performs an unwind segue when tapped, but you need to do some more work before the segue is performed. In earlier lessons, you used the `prepare(for:sender:)` method to pass information from one view controller to another. You'll use the same method here.
  - First, on the `ToDoDetailTableViewController`, you'll need to verify that the `saveUnwind` segue is being performed, because you don't want to do any extra work when the Cancel button is tapped. Next, you'll read the values from the appropriate controls, store them into constants, and pass the values into your model's initializer:

    - ```swift
        override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
            super.prepare(for: segue, sender: sender)
         
            guard segue.identifier == "saveUnwind" else { return }
         
            let title = titleTextField.text!
            let isComplete = isCompleteButton.isSelected
            let dueDate = dueDateDatePicker.date
            let notes = notesTextView.text
        }
      ```

  - Is it safe to force-unwrap the title's text property? If you set up the Save button properly, it should only be enabled when there's text in the title text field. Why does it need to be unwrapped at all? Recall that the title property of a ToDo is non-optional, so the constant needs to be of type String, not String?.
- **Pass Data Across the Unwind Segue**
  - Think about how you could pass information across the `prepare(for:sender:)` method. The segue has a destination property, and in your to-do app you could access the destination view controller's array of models and add a new entry to the array.
  - However, the static table view controller is only responsible for one ToDo at a time. It doesn't have to know that it was presented from a table view controller or that the data will eventually be saved to disk. It's always a good practice to keep the concerns of each view controller separate.
  - Since the static table view controller will deal with one model at a time, you'll add an optional model property to the class definition. (It's optional because the property will be `nil` until the Save button is tapped and the property can be given a value.)
    - `var toDo: ToDo?`
  - Next, you'll add a line to the bottom of `prepare(for:sender:)` that sets the property to a value: `toDo = ToDo(title: title, isComplete: isComplete, dueDate: dueDate, notes: notes)`
  - Now that the model is stored in a property, how does the list view controller access the property and add it to the collection? The unwind segue defined in the first table view controller can access data from the view controller that's being dismissed.
  - When the `unwindToToDoList(_:)` method is called, it has a segue parameter, similar to `prepare(for:sender:)`. Again, you'll want to verify that the `saveUnwind` segue is being called; otherwise, the Cancel button has been tapped and there isn't any work to perform. Next, you'll check to see whether a model object exists in the segue's source (the view controller that triggered the segue). If a model object exists, add it to the array, then add a table cell that represents the new data.

    - ```swift
        @IBAction func unwindToToDoList(segue: UIStoryboardSegue) {
            guard segue.identifier == "saveUnwind" else { return }
            let sourceViewController = segue.source as! ToDoDetailTableViewController
         
            if let toDo = sourceViewController.toDo {
                let newIndexPath = IndexPath(row: toDos.count, section: 0)
         
                toDos.append(toDo)
                tableView.insertRows(at: [newIndexPath], with: .automatic)
            }
        }
      ```

  - Notice that you needed to calculate the index path so that you could add a row at the correct position. Why would you use `count` for the row of the index path? Imagine your array contains 0 items. When you add a cell, you'd insert it in the first row — row 0. If your array contains 1 item, you'd insert it below the existing one, so in row 1. Using the array's count value guarantees that you add a cell at the bottom of the table view.
  - Build and run your app. When you click the Save button, creating a new entry in your collection, a new cell will be displayed.

#### Part Seven - Editing Details

- When your user selects a cell to view more about a particular item, they might also want to edit the information. The static table view you've created already provides an interface for editing, so this feature is on its way to completion.
- **Present Details**
  - Open the Main storyboard and Control-drag from the cell in the to-do list view controller to the static table view controller's navigation controller. Choose the `Present Modally` segue. Next, find the segue between the navigation controller and `ToDoDetailTableViewController`. Create an `@IBSegueAction` by Control-dragging from the segue into `ToDoTableViewController`'s implementation, giving it the argument "Sender" and naming it `editToDo(_:sender:)`.
  - When you tap a ToDo in the list, your segue action will be called, and you're responsible for creating the `ToDoDetailTableViewController` that will be presented. You need to pass the model object from the list to the detail view controller. Using the sender parameter, which will be the tapped cell when the user selects one, you can determine the index path and therefore the ToDo that was selected.
  - This segue action will also be called when the Add button is tapped. Before you added the segue action, the system created the `ToDoDetailTableViewController` for you, but now you'll want to return an empty `ToDoDetailTableViewController` in this case.

    - ```swift
        @IBSegueAction func editToDo(_ coder: NSCoder, sender: Any?) -> ToDoDetailTableViewController? {
            let detailController = ToDoDetailTableViewController(coder: coder)
         
            guard let cell = sender as? UITableViewCell,​ let indexPath = tableView.indexPath(for: cell) else {
                // if sender is the add button, return an empty controller
                return detailController
            }
         
            tableView.deselectRow(at: indexPath, animated: true)
         
            detailController?.toDo = toDos[indexPath.row]
         
            return detailController
        }
      ```

  - The view controller has the data, but if you build and run your app, you'll see that the data isn't displayed in the `ToDoDetailTableViewController`. You'll need to make some edits before the views are updated to reflect the provided data.
- **Update the Static Table View Controller**
  - You originally created the static table view controller to add new items, but you can modify it to allow editing as well. When the view controller is initially loaded, your app will need to determine whether the screen is being used to create a new item or to edit an existing one. How will it know?
  - When the user taps the + button, the static table view controller is displayed, and the optional model property is still `nil`. However, if the user taps a cell, the property contains a model, because it was passed from one controller to another in `editToDo(_:)`.
  - So what's your next move? You'll need to update `viewDidLoad()` to handle the presence (or lack) of data. If the view controller is passed an item, update all the controls to match the model values. If the view controller doesn't have any data, then `viewDidLoad()` will configure the interface for easy model creation, including updates to the text field text, date picker date, navigation bar title, and notes text. Here's how that works:

    - ```swift
        override func viewDidLoad() {
            super.viewDidLoad()
            let currentDueDate: Date
            if let toDo = toDo {
              navigationItem.title = "To-Do"
              titleTextField.text = toDo.title
              isCompleteButton.isSelected = toDo.isComplete
              currentDueDate = toDo.dueDate
              notesTextView.text = toDo.notes
            } else {
              currentDueDate = Date().addingTimeInterval(24*60*60)
            }
         
            dueDateDatePicker.date = currentDueDate
            updateDueDateLabel(date: currentDueDate)
            updateSaveButtonState()
        }
      ```

  - When the user taps the save button, the `prepare(for:sender:)` method is called. Currently this method creates a new ToDo from the current state of the controls. But when an existing ToDo is being edited, the existing ToDo should be updated. Replace the unconditional creation of the ToDo: `toDo = ToDo(title: title, isComplete: isComplete, dueDate: dueDate, notes: notes)`
  - with code that updates the existing toDo if it is not `nil`:

    - ```swift
        if toDo != nil {
            toDo?.title = title
            toDo?.isComplete = isComplete
            toDo?.dueDate = dueDate
            toDo?.notes = notes
        } else {
            toDo = ToDo(title: title, isComplete: isComplete, dueDate: dueDate, notes: notes)
        }
      ```

  - Build and run your app and tap a cell in your list. The static table view should display a new title, and all the controls should match the data the cell represented.
- **Update the Unwind Segue Logic**
  - Currently, two things happen when the unwind segue is triggered from the Save button: A new item is added to the end of the array, and a new cell is added to the table view. But when an existing model is edited, the old model data in the array should be replaced, and the corresponding cell should be reloaded to display the latest information. Since the unwind segue expects model data to be present — regardless of adding or editing — how will the app know whether to add a new item or update an existing one?
  - Take a look at the table view to see whether a cell is still selected when the unwind segue takes place. A selected cell means the unwind segue is happening after editing. If no cell is selected, the + button (not a cell) was tapped, which indicates a new entry to the collection.
  - After you safely unwrap the model data, you can use if-let syntax to unwrap the ToDo's index in your array. If the ToDo is in the array, you will receive its index and use it to update the corresponding model and cell. Otherwise, append the model to the end of the collection and add a new cell, as you did earlier. Here's the code for that logic:

    - ```swift
        @IBAction func unwindToToDoList(segue: UIStoryboardSegue) {
            guard segue.identifier == "saveUnwind" else { return }
            let sourceViewController = segue.source as! ToDoDetailTableViewController
         
            if let toDo = sourceViewController.toDo {
                if let indexOfExistingToDo = toDos.firstIndex(of: toDo) {
                    toDos[indexOfExistingToDo] = toDo
                    tableView.reloadRows(at: [IndexPath(row: indexOfExistingToDo, section: 0)], with: .automatic)
                } else {
                    let newIndexPath = IndexPath(row: toDos.count, section: 0)
                    toDos.append(toDo)
                    tableView.insertRows(at: [newIndexPath], with: .automatic)
                }
            }
        }
      ```

  - Build and run your app. You can use the sample data to try editing items on the list or create your own items. When you use the Save button to unwind to the list, the table should always display the most up-to-date model data.

#### Part Eight - Create a Custom UITableViewCell

- You can use the static table view controller to save any changes to an item, but what if you wanted to make a quick change to an item right from the list? For example, it would be convenient for the user if a to-do could be marked as complete without having to present the editing screen, toggle the checkmark, and press the Save button. You can simplify the workflow by using a custom UITableViewCell with its own set of controls and labels.
- **Create a Cell Subclass**
  - Create a new Cocoa Touch Class file, choosing `UITableViewCell` as the subclass. Call your new subclass `ToDoCell`.
  - Currently, the `ToDoTableViewController` uses instances of `UITableViewCell` to represent each item in your list. Now that you've created a subclass of `UITableViewCell`, you'll need to switch to using the new cell. In your storyboard, select the prototype cell, then use the Identity inspector to change its class.
  - Navigate to the to-do table view controller class and locate the `tableView(_:cellForRowAt:)` method. Currently, the first line calls `dequeueReusableCell`, which will return a `UITableViewCell`. Use the following code to conditionally downcast this cell to your custom class so that it matches the type of cell defined in the storyboard: `let cell = tableView.dequeueReusableCell(withIdentifier: "ToDoCellIdentifier", for: indexPath) as! ToDoCell`
  - If you've updated everything successfully, you can build and run your app, and everything should continue to work. The next step is to add new controls to enable the user to quickly update the information in the cell.
- **Design the Cell**
  - In your storyboard, you'll want a button that allows the user to mark the ToDo as complete. Select the cell and confirm that its Style is `Custom`. The checkmark button is identical to the one you created in the static table view controller, so you can copy (Command-C) it from the New To-Do scene and paste (Command-V) it into the prototype cell in the to-do list. Add constraints to the button that center it vertically in the cell and align it to the leading edge with at least 8 pixels of spacing.
  - Previously, you used the textLabel of a `UITableViewCell` to display the to-do’s title, but this new button will partially cover that label. Go ahead and add your own label, positioning it to the right of the checkmark button. Be sure to add appropriate constraints. 
  - Use the assistant editor to create outlets for your new views to `ToDoCell`, which will enable you to update the label text and the button status.

    - ```swift
        @IBOutlet var isCompleteButton: UIButton!
        @IBOutlet var titleLabel: UILabel!
      ```

  - With the outlets in place, you can update their properties whenever a cell is about to be displayed. Locate the `tableView(_:cellForRowAt:)` method again. Switch it from using the cell's textLabel to your new `titleLabel`. In addition, add a line of code that sets the checkmark button to selected based on the model's `isComplete` property — just as you did with the checkmark in the static table view. Here's how it all works:

    - ```swift
        override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
            let cell = tableView.dequeueReusableCell(withIdentifier: "ToDoCellIdentifier", for: indexPath) as! ToDoCell
         
            let toDo = toDos[indexPath.row]
            cell.titleLabel?.text = toDo.title
            cell.isCompleteButton.isSelected = toDo.isComplete
         
            return cell
        }
      ```

  - If you followed the advice above to copy the checkmark button from the one in the static table view controller, it will have a Touch Up Inside action associated with it. You'll want to remove this action by selecting the `isCompletedButton`, opening the Connections inspector, and clicking the "x" next to the Touch Up Inside action to remove it.
  - Build and run your app. You should see the title of the ToDo displayed next to the checkmark button. At this point, the checkmark button reflects the isComplete status of the ToDo. It can't yet be tapped, but you can edit one of the to-dos toggle its completion state, then return to the list to confirm that the checkmark disappears.
- **Add a Cell Delegate Method**
  - You'll need to add a method in your custom cell that handles the checkmark button tap, but what should it do? You'll have to update the isComplete boolean, but the cell doesn't have any knowledge of the ToDo model, nor does it know its own index in the table view.
  - But there's good news. The table view controller knows all these details. In fact, the only thing it needs to know is which cell's checkmark was tapped. If the cell had a delegate, it could notify the delegate when its checkmark button is tapped.
  - In the storyboard, use the assistant editor to create an action that's called when the checkmark button is tapped.
  - Above the definition of your cell subclass, define a protocol with a method that passes the cell back to the delegate:

    - ```swift
        protocol ToDoCellDelegate: AnyObject {
            func checkmarkTapped(sender: ToDoCell)
        }
      ```

  - Add a delegate property to your cell subclass so that the cell has something to inform: `weak var delegate: ToDoCellDelegate?`
  - In the action tied to the checkmark button, inform the delegate that the button tap has occurred, passing in self as the parameter to the method:

    - ```swift
        @IBAction func completeButtonTapped(_ sender: UIButton) {
            delegate?.checkmarkTapped(sender: self)
        }
      ```

  - The cell's delegate should be the table view controller, which doesn't conform to the protocol you defined earlier. Update the view controller's class definition so that it can be set as the delegate: `class ToDoTableViewController: UITableViewController, ToDoCellDelegate`
  - To satisfy the conditions of the protocol, this view controller must implement all of its methods. In this case, there's only one method defined in the protocol. Add the method declaration, but leave the method body blank: `func checkmarkTapped(sender: ToDoCell) { }`
  - Whenever a cell is dequeued, the table view controller should set itself as the cell's delegate. Locate the `tableView(_:cellForRowAt:)` method and set the delegate property of the cell accordingly: `cell.delegate = self`
  - What have you implemented so far? Add breakpoints in the complete button action and in the empty `checkmarkTapped(_:)` method, then build and run the app. Tap the cell's checkmark button, triggering the action and the first breakpoint. The method will inform the delegate (the table view controller) that its checkmark was tapped and will pass itself through the delegate method. Let the code continue executing, and you'll trigger the breakpoint in `checkmarkTapped`.
  - All that's left is to determine the index path of the cell, which you'll use to toggle the `isComplete` boolean of the corresponding `ToDo`. Once you've updated the model, reload the cell so that its appearance is up to date with the data. Here's the code:

    - ```swift
        func checkmarkTapped(sender: ToDoCell) {
            if let indexPath = tableView.indexPath(for: sender) {
                var toDo = toDos[indexPath.row]
                toDo.isComplete.toggle()
                toDos[indexPath.row] = toDo
                tableView.reloadRows(at: [indexPath], with: .automatic)
            }
        }
      ```

  - Build and run your app again. The cell's button should now be fully functional.
  - While more complicated than handling actions on cells in static table views, this is a necessary and common pattern for handling actions performed on cells in dynamic table views.

#### Part Nine - Codable

- Your app is now capable of listing, adding, editing, and deleting items in a collection. You're almost done. You still need your app to save the collection to disk and to retrieve it when the app is opened at some point in the future.
As you've learned and practiced in earlier lessons, persistence requires you to add Codable support to your model data. Once your model adopts this protocol, you'll add logic in the appropriate places to save items whenever necessary.
- **Add Support for Archiving and Unarchiving**
  - Start by adding Codable to your model's type definition: `struct ToDo: Equatable, Codable { ... }`
  - You may notice a warning on the line defining the constant id informing you that it will not be decoded because it is defined as an immutable property — by assigning it a value where it's declared, you've prevented the automatic implementation of Codable conformance from setting it.
  - To resolve the issue, first change the declaration of id by removing the assignment of `UUID()`. It should now look like this: `let id: UUID`. Of course, now you'll get an error everywhere the `ToDo` memberwise initializer is used, because it now requires an `id` argument. You could update your code to add a new UUID value for each initialization of `ToDo`, but it'll be cleaner to create a custom initializer with an identical signature to the current memberwise initializer — `init(title:isComplete:dueDate:notes:)` — and assign a value to id there.
  - The beginning of the `ToDo` struct should now look like:

    - ```swift
        struct ToDo: Equatable, Codable { 
            let id: UUID
            var title: String
            var isComplete: Bool
            var dueDate: Date
            var notes: String?
         
            init(title: String, isComplete: Bool, dueDate: Date, notes: String?) {
                self.id = UUID()
                self.title = title
                self.isComplete = isComplete
                self.dueDate = dueDate
                self.notes = notes
            }
            ...
        }
      ```

  - `ToDo` objects can now be encoded and decoded, but your app still needs to include logic that tells it where and when to store the data.
  - Typically, you'll want to save your data somewhere in your app's Documents directory. Since this directory is accessible to your app and can't be modified by another app, it's a safe place to store your list. You can use the `FileManager` class to locate your app's Documents directory, create a subfolder for archiving data, and store that path to a constant.
  - To ensure that the constants are only defined one time and are not tied to a particular instance of your model, use the `static` keyword. Add the following constant definitions to your model, making sure to define `ArchiveURL` with the appropriate subfolder string:

    - ```swift
        static let documentsDirectory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
        static let archiveURL = documentsDirectory.appendingPathComponent("toDos").appendingPathExtension("plist")
      ```

  - Now that you have a path defined for your data store, you can replace the body of the `loadToDos()` method with the appropriate logic to load data from disk. Use a `PropertyListDecoder` and the methods on Data that you learned in a previous lesson to unarchive the data and return it:

    - ```swift
        static func loadToDos() -> [ToDo]?  {
            guard let codedToDos = try? Data(contentsOf: archiveURL) else {return nil}
            let propertyListDecoder = PropertyListDecoder()
            return try? propertyListDecoder.decode(Array<ToDo>.self, from: codedToDos)
        }
      ```

  - Before you can attempt to read model data from disk, you have to provide a way to save it to disk. Write a `static` method that uses a `PropertyListEncoder` to encode a [ToDo] collection and then uses the `write(to:options:)` method on Data to store it in the Documents directory:

    - ```swift
        static func saveToDos(_ toDos: [ToDo]) {
            let propertyListEncoder = PropertyListEncoder()
            let codedToDos = try? propertyListEncoder.encode(toDos)
            try? codedToDos?.write(to: archiveURL, options: .noFileProtection)
        }
      ```

  - When is the appropriate time to write your collection of models to disk? You could say it's whenever the collection changes in some way — and that's mostly correct. The only exceptions are when you fill the array with sample data or when you load the data from disk. Otherwise, you want your app to save data any time there's a change in the collection.
  - The collection can be changed in several ways. The first is when the user swipes a cell to delete the item. In this case, call the `save()` method after you've removed the item from the array:

    - ```swift
        override func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath) {
            if editingStyle == .delete {
                toDos.remove(at: indexPath.row)
                tableView.deleteRows(at: [indexPath], with: .automatic)
                ToDo.saveToDos(toDos)
            }
        }
      ```

  - The collection can also be updated when the user taps the Save button (regardless of whether an item was added or edited). Therefore, call the save method whenever you perform the segue marked with the `saveUnwind()` identifier:

    - ```swift
        @IBAction func unwindToToDoList(segue: UIStoryboardSegue) {
            // ...

            ToDo.saveToDos(toDos)
        }
      ```

  - Finally, in the custom cell delegate method, you'll need to save the collection of items when the user taps the button in the cell. Each button tap changes the `isComplete` property of the ToDo from true to false or visa versa.

    - ```swift
        func checkmarkTapped(sender: ToDoCell) {
            if let indexPath = tableView.indexPath(for: sender) {
                // ...

                ToDo.saveToDos(toDos)
            }
        }
      ```

  - Build and run your app. Make some changes to your collection, triggering the save functionality. Then stop the app and relaunch it to trigger the `load()` method.
- **Wrap-Up**
  - Congratulations on completing a custom list app! The procedures you learned to display, create, modify, and save data in this project will be extremely valuable throughout your iOS development career. As you continue to develop more apps, you'll begin to recognize common requirements among projects, and you'll be able to implement features with greater ease.
  - Do you think all the steps in this project are more or less reusable? If not, try working through the project again, but using different items in the collection. You'll be amazed at how much of the code fits neatly into a new app.
- **Stretch Goals**
  - The features you built in this project are common across many iOS apps. Would you like to build some features that are less common? Would you like to add your own unique flavor to your app? Here are some additional features you might try to implement:
    - Add the ability to share the details of an item using the Mail app.
    - Add a search field at the top of your list that allows the user to filter a large list of items by specifying a portion of the item's title.
- **Summary**
  - Wow! You've just learned three of the most important skills for building a common type of app: how to list data in a table view, how to architect your app using the MVC design pattern, and how to save data between app launches. Along the way, you've also learned about how apps are launched, how to build scroll views, how to use some of the common system view controllers, and how to build complex forms to collect information from a user.
  - Now that you know how to build table views, persist data, and use MVC to architect your app, you can start building many more complex apps that store data locally. At this point, you should also be able to start picking up new frameworks and tools by looking at the documentation. There are many new types of views and classes to explore and integrate into your apps, each one building on the skills you already have.
  - Take some time to experiment and learn about something new. Are you interested in building an app with Maps, or an app that uses location services? Want to play music or video? Go look up MapKit, CoreLocation, or AVFoundation. You'll find that “many of the frameworks and tools out there work very similarly to those you've already learned.
  - In the next unit, you'll learn how to level up your apps by fetching and sending information over the internet. You're well on your way to being a seasoned iOS developer. Congratulations!

## Unit 2 Working With The Web

- Most apps connect to web services to fetch or send information that is used in the app. In this unit, you'll learn how to create and send network requests to send and receive data.
- To begin, you'll learn about closures, which are blocks of code that can be used like variables, which allow you to write code that will be executed at a later time. You'll use closures to write code that will execute after network requests are finished. You'll also learn how to use closures to work with collections and create animations in your app.
- Then you'll work through three lessons that explain how the web works, how to request information from a web service, how to turn that information into structures or classes you can use within your app, and how to make your app run smoothly with long running networking operations.

### Lesson 2.1 Closures

- Closures can be a complicated topic for new developers, but many of the frameworks for building apps use them extensively. When understood and used correctly, they can be a very powerful tool for writing clean, reusable, high-performance code. You don't need to master closures at this stage, but you do need to be familiar with how to work with them.
- This lesson will introduce you to closures and show you how to define them, how to use them as function arguments, and how to use some of the common functions that take closures as arguments. You'll learn more about closures in future lessons, as specific concepts apply to the projects you'll be building.
- Closures are self-contained blocks of functionality that can be passed around as values and executed later on in your code. This may sound familiar, as functions are also self-contained blocks of functionality. What's the difference? You can think of a function as a special, named case of a closure. Closures typically don't have a name and can be stored as a variable or passed as an argument. In fact, some developers refer to closures as “anonymous functions,” because they're like functions without a name.

#### Syntax

- The syntax for declaring and using closures can be difficult to understand. It's helpful to start by reviewing the syntax of a function:

  - ```swift
      func sum(numbers: [Int]) -> Int {
        // Code that adds together the numbers array
        return total
      }
    ```

- The function above has a name (sum), an argument (numbers) that's an array of Int values, and a return type Int. The code that's executed is written inside a pair of braces.
- Now compare that with the syntax of a closure:

  - ```swift
      let sumClosure = { (numbers: [Int]) -> Int in
        // Code that adds together the numbers array
        return total
      }
    ```

- The closure above is assigned to a constant named `sumClosure`. The closure, which is written inside a pair of braces, is a block of code that takes an array of `Int` values (`numbers`) and has a return type `Int`. Because the entire closure is inside the braces, you add the `in` keyword to indicate the beginning of the code to be executed when calling the closure.
- You can then use the closure to perform the action by calling `sumClosure`.

  - ```swift
      let sum = sumClosure([1, 2, 3, 4])
      print(sum) // 10
    ```

- All values in Swift have a type, including closures. You define the type by specifying a parameter list and a return type, or the lack of either, at the start of the block of code. Just like functions, there are four types of closures:
  1. A closure with no parameters and no return value:

     - ```swift
        let printClosure = { () -> Void in
        print("This closure does not take any parameters and does not return a value.")
        }
       ```

  2. A closure with parameters and no return value:

     - ```swift
        let printClosure = { (string: String) -> Void in
        print(string)
        }
       ```

  3. A closure with no parameters and a return value:

     - ```swift
        let randomNumberClosure = { () -> Int in
        // Code that returns a random number
        }
       ```

  4. A closure with parameters and a return value (as in the earlier example):

     - ```swift
        let randomNumberClosure = { (minValue: Int, maxValue: Int) -> Int in
        // Code that returns a random number between `minValue` and `maxValue`
        }
       ```

#### Passing Closures as Arguments

- Many commonly used collection functions and UIKit objects take closures as arguments. Now that you know how to declare a closure, you can learn about passing a closure into a function.
- Imagine you're working on an app for playing music. Your task is to build a screen that displays a playlist. Since tracks may arrive in any order, your playlist will need to sort them based on one of three user preferences: track number, track name, or star rating.
- Your solution is the built-in `sorted(by:)` array function, an instance method on arrays that takes a closure as an argument. Given an array of tracks, you can pass the function a closure that takes two track numbers and determines whether the first track should appear before or after the second track when sorted:

  - ```swift
      let sortedTracks = tracks.sorted { (firstTrack, secondTrack) -> Bool in
          return firstTrack.trackNumber < secondTrack.trackNumber
      }
    ```

- When you're first learning how to write closure syntax, it may feel cumbersome to figure out where to position the curly braces and parentheses. Xcode does a great job of helping you with this! As you are writing out the method name, press the Return key to autocomplete it, and the cursor will highlight the first parameter. Press Return again, and Xcode will simplify and autofill the closure syntax. All that's left for you to do is provide names for each of the parameters.
- The `sorted(by:)` function will run the instruction in the passed - in closure on every pair of objects in the array and return a Bool value. If the closure returns true, the first track will stay in front of the second track; if it returns false, the first track will move behind the second track. The function will repeat until the entire array returns true.
- What makes closures so powerful is that you can pass any logic to the `sorted(by:)` function. So now you can pass a closure that sorts by track name or by star rating:

  - ```swift
      let sortedTracks = tracks.sorted { (firstTrack, secondTrack) -> Bool in
          return firstTrack.starRating < secondTrack.starRating
      }
    ```

- Did you notice that firstTrack and secondTrack do not include type information? Unlike the closures shown previously, `sorted(by:)` is a method being called by a `[Track]` collection, and the Swift compiler can infer that the two parameters within the closure will be of type Track.

#### Additional Syntactic Sugar

- The Swift compiler is very smart. Because it knows the types of every constant, variable, and closure, it can make assumptions that allow you to write more concise code. Swift allows you to combine type safety with closure syntax to help you move from syntax - heavy statements to very simple, concise statements. Programmers refer to such concise code as “syntactic sugar.”
- Right now, you shouldn't write code that's more concise than you can handle, but as you become more familiar with closures and their syntactic sugar, it will become second nature to write code in the simplest way possible.
- Starting with Xcode's autocompleted closure syntax, take a look at how you can simplify the closure one step at a time.

  - ```swift
      let sortedTracks = tracks.sorted { (firstTrack, secondTrack) -> Bool in
          return firstTrack.starRating < secondTrack.starRating
      }
    ```

- Swift can infer that the closure must return a Bool, so you can remove the return type:

  - ```swift
      let sortedTracks = tracks.sorted { (firstTrack, secondTrack) in
        return firstTrack.starRating < secondTrack.starRating
      }
    ```

- Because the `sorted(by:)` function expects two instances to compare, Swift knows that you need to pass the closure two tracks. The compiler provides placeholder names for the closure parameters that you can use instead of their actual names. Placeholder names begin with `$` followed by the index of the parameter. So you can access the first parameter using `$0`, the second using `$1`, and so on. Now you can drop the parameter names and in: `let sortedTracks = tracks.sorted { return $0.starRating < $1.starRating }`
- When a closure has one line, Swift will automatically return the result, so you can remove the return keyword: `let sortedTracks = tracks.sorted { $0.starRating < $1.starRating }`
- You can see how Swift allows you to trim your code from three lines to one line. As you become more comfortable with Swift closures, you will start to prefer this concise style.
- One last interesting bit: Remember that closures are effectively functions. When a method accepts a closure as a parameter, it will also accept a function with a signature that matches the closure requirements. If your `Track` type conformed to `Comparable`, you could reduce this even further.

  - ```swift
      struct Track: Comparable {
          var trackNumber: Int
       
          static func < (lhs: Track, rhs: Track) -> Bool {
              return lhs.trackNumber < rhs.trackNumber
          }
      }

      let tracks = [Track(trackNumber: 3), Track(trackNumber: 2), Track(trackNumber: 1), Track(trackNumber: 4)]
       
      let sortedTracks = tracks.sorted(by: <)
    ```

- The `<` function takes two parameters of type Track and returns a Bool — exactly the type of `sorted(by:)`'s parameter.
- **Trailing Closure Syntax**
  - You may be wondering why the `by` argument label was omitted during Xcode's autocompletion of the closure. If you were being as verbose as possible, the `sorted(by:)` method would be called like this:

    - ```swift
        let sortedTracks = tracks.sorted(by: { (firstTrack, secondTrack) -> Bool in
            return firstTrack.starRating < secondTrack.starRating
        })
      ```

- When a closure is the last argument of a function, you can move the closing parenthesis after the second-to-last argument, drop the name of the final argument, and leave the curly braces at the end. When the function's only argument is a closure, the parentheses can be omitted entirely, along with the argument name. This is called “trailing closure syntax.”
- Imagine you have a function `performRequest(url: String, response: (Data) -> Void)`. The first argument is a URL string, and the last is a `response` closure that contains the data at the specified URL.
  - `func performRequest(url: String, response: (code: Int) -> Void) { }`
- Here's what it would look like to call the function using trailing closure syntax. The closing parenthesis is moved to the previous argument, the response argument label is dropped, and the curly braces hang nicely off the end.

  - ```swift
      performRequest(url: "https://www.apple.com") { (data) in
          print(data)
      }
    ```

#### Collection Functions Using Closures

- Swift includes some useful functions for iterating over collections that take a closure argument to define common actions. 
  - You'll learn about `map()`, which allows you to create an array from another array by transforming each value in the collection. 
  - You'll learn about `filter()`, which takes an array and creates another with only the objects that meet your defined set of rules. 
  - And you'll learn about `reduce()`, which lets you combine many values into a single value.
- These functions are extremely powerful and can be chained to create advanced logic. When you're working with arrays, think about whether one of these collection functions can help you write more concise code.
- **Map**
  - The `map()` function is an instance method that can be used on an array to create a new array. It takes a closure parameter that tells it what to do to each object before adding it to the new array. In this example, you're starting with an array of first names, but you want an array of full names.
  - You can use a for-in loop to create the new array:

    - ```swift
        // Initial array
        let names = ["Johnny", "Nellie", "Aaron", "Rachel"]
         
        // Creates an empty array that will be used to store the full names
        var fullNames: [String] = []

        for name in names {
          let fullName = name + " Smith"
          fullNames.append(fullName)
        }
      ```

  - Or you can use the `map()` function. The closure will be executed on each object in the `firstNames` array to create what will be added to the `fullNames` array.

    - ```swift
        let names = ["Johnny", "Nellie", "Aaron", "Rachel"]
         
        // Creates a new array of full names by adding "Smith" to each first name
        let fullNames = names.map { (name) -> String in
            return name + " Smith"
        }
      ```

  - A shortened version of the map() function works just as well. Recall that $0 is a placeholder for the closure's first parameter—in this case, the current object in the array.

    - ```swift
        let names = ["Johnny", "Nellie", "Aaron", "Rachel"]
         
        // Creates a new array of full names by adding "Smith" to each first name
        let fullNames = names.map { $0 + " Smith" }
      ```

  - Each of these examples will result in the same array of full names.
  - You can use any of these three approaches to transform the array of firstNames into fullNames. As a new programmer, you might find the for-in loop to be the easiest to think through. As you feel more comfortable with Swift and closure syntax, you'll be drawn to the more concise `map()` function. Experienced programmers prefer concise code because it feels cleaner and is easier to read.
  - Closure syntax can be difficult. It's helpful to pause and break down the syntax of each example. Look at the example above. Try to identify the function call, the parameter, and the closure syntax.
    - What is the function call? (map())
    - What is the closure parameter? ({ $0 + " Smith" })
    - What does the $0 represent? (The individual object in the array that's being worked with.)
- **Filter**
  - The `filter()` function creates a new array with only the objects from the starting array that match a specific use case. Filter takes a closure parameter that returns `true` or `false` to determine whether the object should be included in the new array. In this example, you want to filter out all the numbers that are greater than 20.
  - Here's the code using a traditional for-in loop:

    - ```swift
        let numbers = [4, 8, 15, 16, 23, 42]
        var numbersLessThan20: [Int] = []
         
        for number in numbers {
            if number < 20 {
                numbersLessThan20.append(number)
            }
        }

        print(numbersLessThan20)
        Console Output:
        [4, 8, 15, 16]
      ```

  - And here it is using the `filter()` function:

    - ```swift
        let numbers = [4, 8, 15, 16, 23, 42]
        let numbersLessThan20 = numbers.filter { (number) -> Bool in
            return number < 20
        }
         
        print(numbersLessThan20)
        Console Output:
        [4, 8, 15, 16]
      ```

  - And now with the shortened syntax of filter():

    - ```swift
        let numbers = [4, 8, 15, 16, 23, 42]
        let numbersLessThan20 = numbers.filter { $0 < 20 }
        print(numbersLessThan20)
        Console Output:
        [4, 8, 15, 16]
      ```

  - Again, each example yields the same array.
  - As with the `map()` example, you can use any of these three approaches to filter the array of numbers into `numbersLessThan20`. As you learn more about Swift and closure syntax, you'll find `filter()` is more concise and results in cleaner code. For now, use what's comfortable for you.
  - Again, to help familiarize yourself with closure syntax, pause and break down the syntax of each example. Try to identify the function call, the parameter, and the closure syntax. In the example just above:
    - What is the function call? (filter())
    - What is the closure parameter? ({ $0 < 20 })
    - What does the $0 represent? (The individual object in the array that is being worked with.)
- **Reduce**
  - The `reduce()` function combines all the values in an array into one value. It takes a starting value and a closure that dictates how to combine the items. The code in this example takes an array of numbers and adds them together.
  - Here's a potential solution using a for-in loop:

    - ```swift
        let numbers = [8, 6, 7, 5, 3, 0, 9]
        var total = 0
         
        for number in numbers {
            total = total + number
        }
      ```

  - And another using the `reduce()` function:

    - ```swift
        let numbers = [8, 6, 7, 5, 3, 0, 9]
        let total = numbers.reduce(0) { (currentTotal, newValue) -> Int in
            return currentTotal + newValue
        }
      ```

  - Or the shortened syntax of `reduce()`. Recall that $0 is a placeholder for the current object, and $1 is a placeholder for the next object in the array.

    - ```swift
        let numbers = [8, 6, 7, 5, 3, 0, 9]
        let total = numbers.reduce(0) { $0 + $1 }
      ```

  - Similar to passing the `<` function in the filter example, you could pass the `+` function to closure parameter on `reduce()`, since its signature matches the requirements of the closure parameter.

    - ```swift
        let numbers = [8, 6, 7, 5, 3, 0, 9]
        let total = numbers.reduce(0, +)
      ```

  - Once again, pause and break down the syntax of each example. Try to identify the function call, the parameter, and the closure syntax. For the example using the shortened syntax of reduce():
    - What is the function call? (reduce())
    - What is the starting value in the function call? (0)
    - What is the closure parameter? ({ $0 + $1 })
    - What does the $0 represent? (The value of all the items that have been reduced so far.)
    - What does the $1 represent? (The value of the new item that you are reducing into the total.)

#### Closures Capture Their Environment

- In a previous lesson, you learned about scope and that any constant or variable declared within braces is defined in that local scope and isn't accessible by any other scope. Closures work somewhat differently. They're often written using values or functions that are defined in the same scope as the closure — but not within the closure braces.
- Consider the following closure that animates the backgroundColor from its current color to red:

  - ```swift
      animate {
        self.view.backgroundColor = .red
      }
    ```

- What if the view property were removed from the screen during this animation? Would the animation stop? Would the code crash, since view would no longer be an accessible property? No worries. Swift is smart. Based on the context, a closure can access, or capture, surrounding constants and variables. This means that, if the view would typically be deallocated when it's removed from the screen, the animate closure will keep the view in memory until the closure is complete.
- This intelligence enables closures to use those constants and variables safely and to modify them within the closure block. Similarly, Swift wants to make it clear that the owner of the view property must also be kept in memory until the animation completes, so it requires the `self` keyword.

### Lesson 2.2 Extensions

- Extensions allow you to add functionality to a type that's already defined. You can use extensions to create new properties or methods for Swift standard types, such as `Int`, `String`, `Bool`, and `Array`, or custom types that you've defined for your app. Extensions are also useful for organizing your code into logical chunks, such as the code needed for a type to adopt a protocol.
- In this lesson, you'll learn how and why to use extensions.
- Most often, you'll use extensions for two things: adding functionality to types you can't edit directly — such as types defined in the Swift standard library — and organizing complex code in a logical way.
- What kinds of functionality might you want to add to an existing type? You can add computed properties, define methods, provide new initializers, or conform to a protocol.
- Keep in mind that when you extend an existing type, any new functionality will be available not just on all future instances of that type but also on all existing instances anywhere in your program.
- To define an extension, use the `extension` keyword, followed by the name of the type you're extending. You'll declare the new functionality inside the braces.

  - ```swift
      extension SomeType {
        // New functionality to add to SomeType goes here
      }
    ```

#### Adding Computed Properties

- In an earlier lesson, you added computed properties to a Temperature type to allow you to easily read back different temperature measurements from the same instance. You can use an extension to add new computed properties to a type that already exists.
- The following example extends the `UIColor` type with a `random` static computed property that returns a random color.

  - ```swift
      extension UIColor {
          static var random: UIColor {
              let red = CGFloat.random(in: 0...1)
              let green = CGFloat.random(in: 0...1)
              let blue = CGFloat.random(in: 0...1)
       
              return UIColor(red: red, green: green, blue: blue, alpha: 1.0)
          }
      }
    ```

- Now you can access the `random` property on any `UIColor` instance in the program using `UIColor.random`.

#### Adding Instance or Type Methods

- You can also use extensions to add new instance or type methods in a simple, clean way.
- Imagine you're tasked with building a feature that allows the user to pluralize words.
  - Apple -> Apples
  - Song -> Songs
  - Person -> People
  - Tennis court -> Tennis courts
- You could extend the `String` class with a `pluralize()` method that changes a string to its pluralized version. Adding a method to an extension is identical to adding a method to a type definition. Just use the `func` keyword, followed by parentheses that contain any parameters, and optionally define a return type.
- In this example, you're updating the `String` in place, meaning you're performing an action on the current instance, not returning a separate copy. Recall that when you programmatically modify a structure's value in one of its methods, you must include the `mutating` keyword. This is true for extensions to structures as well.

  - ```swift
      extension String {
        mutating func pluralize() {
          // Complex code that modifies the current value (self) to be plural
        }
      }

      let apple = "Apple"
      apple.pluralized()
      print(apple)

      Console Output:
      Apples
    ```

- You can even use an extension to add your own custom initializers to existing types. The code below is an example that only allows you to initialize a `String` that obeys some common best practices for choosing a password:

  - ```swift
      extension String {
          init?(passwordSafeString: String) {
              guard passwordSafeString.rangeOfCharacter(from: .uppercaseLetters) != nil &&
                  passwordSafeString.rangeOfCharacter(from: .lowercaseLetters) != nil &&
                  passwordSafeString.rangeOfCharacter(from: .punctuationCharacters) != nil &&
                  passwordSafeString.rangeOfCharacter(from: .decimalDigits) != nil  else {
                      return nil
              }

              self = passwordSafeString
          }
      }
    ```

#### Organizing Code

- You can imagine that organizing code is extremely important when working on any non-trivial program. Many developers work on apps with hundreds or thousands of files that define classes, structures, enumerations, view controllers, and custom controls for the project. Each definition can grow to hundreds or thousands of lines.
- As you begin working on larger projects, you will use extensions as a way to organize your code.
- One way you might use extensions is to restrict your base type definition to contain only the properties represented by that type and pull all your methods out into an extension of the type:

  - ```swift
      struct Restaurant {
        let name: String
        var menuItems: [MenuItem]
        var servers: [Employee]
        var cookingStaff: [Employee]
        var customers: [Customer]
      }
       
      extension Restaurant {
       
        func add(employee: Employee, to group: StaffGroup) {...}
        func remove(employee: Employee, from group: StaffGroup) {...}
        func add(menuItem: MenuItem)
        func remove(menuItem: MenuItem)
       
        func welcome(guest: Customer) {...}
        func serve(item: MenuItem, to guest: Customer) {...}
        func prepareCheck(for guest: Customer) -> Check {...}
       
        // Other methods related to the Restaurant
      }
    ```

- Note that subclasses can't override methods that are defined in an extension. Consider this when writing extension methods for a class and give some thought to how you or others may be using your class in the future. Also, while separating your code like this can be very helpful, be careful not to take it too far. For example, you could carry this to an extreme and put every separate function into an extension, but that would harm readability more than help it.
- Many developers use extensions to adopt a protocol as a way of logically organizing their code. Imagine you have a `UIViewController` subclass that contains a `UITableView`. To populate the table view, you must implement the required `UITableViewDataSource` methods:

  - ```swift
      class EmployeeViewController: UIViewController, UITableViewDataSource {
          // EmployeeViewController implementation methods ...

          public func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
              return companyAssets.count
          }

          public func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
              // Configure and return cell
          }
      }
    ```

- If it would help the readability of your code, you could break out the `UITableViewDataSource` method implementations into their own section or even their own file using an extension:

  - ```swift
      class EmployeeViewController: UIViewController {
          // EmployeeViewController implementation methods ...
      }

      extension EmployeeViewController: UITableViewDataSource {
          public func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
              return companyAssets.count
          }
       
          public func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
              // Configure and return cell
          }
      }
    ```

- For most developers, deciding how to group things in extensions is a matter of style and preference. As you gain more experience, you'll develop preferences of your own.

### Lesson 2.3 Practical Animation

- Take a look through the most popular iOS apps, and you'll see elegant, subtle animations used to create a connection with onscreen content. When implemented in the right places and at the right moments, animation can provide feedback, enhance the sense of direct manipulation, or help users visualize the results of their actions.
- In this lesson, you'll learn how to use the UIView class and closures to add animations that improve the presentation and functionality of your apps.
- Animations are used heavily in iOS to help users understand what's happening and to tell them something about the data they're viewing.
- Imagine you're using a shopping app. You'll see one screen with a list of products and another screen with a checkout workflow. When you tap a product on the list, what do you expect to happen? Most users would expect to see a new scene with details about the selected product, and they'd expect it to animate horizontally (and with great subtlety) onto the screen from the right side of the device — the default behavior when using a show segue in a navigation controller. They also expect to be returned to the product list by tapping the Back button in the top-left corner or by swiping from left to right.
- When you tap a button to view the cart, what do you expect to happen? Most users would expect the new scene to animate vertically from the bottom of the device onto the screen. This modal transition subtly tells the user that they've shifted contexts, from shopping to checking out. Dismissing the view would return the user back to the shopping context.
- Animating between different views in an app gives the user a sense of orientation — and familiarity — within the app.
- Animation can also be used to convey the character or personality of your app. This is where your animation technique matters. Take a look at the following animations for adding a new item to a list.
- From a functionality point of view, both of these apps communicate the same thing: a new item on the list. But because they animate in different ways, the two apps feel quite distinct. The animation on the left could work well in a game or in a more whimsical app, while the animation on the right is more subtle and would work better in a professional or productivity app.

#### Why Animate?

- Animations should direct the user's attention, keep the user oriented, or connect to user behaviors.
- **Direct the User's Attention**
  - An animation can make your user aware that something in the app has changed, or it can draw their attention to a new feature or area of the screen. For example, when a badge is applied in the Home screen, an animation draws attention to the fact that the app has new information or new content.
- **Keep the User Oriented**
  - As an app developer, you want your users to know where they are and where they've been. Animations can help you orient them as they move through your app.
  - Consider the iOS Calendar app as an example. As you navigate from the year view to the month view and finally to the day view, the animation appears to lift the month or day to the current plane, pushing the rest of the year or month offscreen. As you navigate back to the previous level, the current day or month collapses back into the larger context. These animations keep the user oriented and communicate how they got to each screen.
- **Connect to User Behaviors**
  - You may be the app developer, but you still want your user to feel like they own the app experience. You can use animations to connect the user's behavior to what's happening onscreen — so they know that their actions are causing a result.
  - As an example, try activating Notification Center. As you swipe downward, the appearance of Notification Center reflects the position of your finger. By clearly indicating how you're controlling the view, Notification Center has subtly informed you that you can swipe upward to hide it.

#### What Can Be Animated?

- Core Animation is a set of tools that enable app developers to animate views. UIKit leverages Core Animation to support a set of basic animations that include many of the animations you'll want to create.
- In this lesson, you'll focus on animating UIView objects using some higher-level syntax for interacting with Core Animation. For more advanced animations, check out the [Core Animation Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40004514-CH1-SW1).
- The `UIView` class defines several properties that can be animated:
  - alpha
  - backgroundColor
  - bounds
  - center
  - frame
  - transform
- You're familiar with several of these properties already; as you experiment with different animations in this lesson, feel free to reference the documentation for UIView to learn or refresh your memory about what each of the properties represents.
- So far in this course, you've primarily used Interface Builder to design your views, but you can also work with views programmatically. Ordinarily, when you set a property's value the view will update immediately and without an animation. To animate the change, ​you'll change the property's value in an animation closure, and Core Animation will handle the rest.

#### Animation Closures

- The UIView class defines several class methods that accept a closure within which you can update the animatable properties on your views. The method then handles the calculations to animate the changes over time.
- The methods vary in the number of parameters they accept. They all allow you to specify the duration of the animation, and two also allow the animation to be performed after a specified delay. One has properties to make the animation's timing curve correspond to the motion of a physical spring. And three allow another closure to run on completion of the animation. Use the method with the parameters that match your needs.
 
  - [animate(withDuration:animations:)](https://developer.apple.com/documentation/uikit/uiview/1622418-animate)

    - ```swift
        class func animate(
            withDuration duration: TimeInterval,
            animations: @escaping () -> Void
        )
      ```

  - [animate(withDuration:animations:completion:)](https://developer.apple.com/documentation/uikit/uiview/1622515-animate)

      - ```swift
          class func animate(
              withDuration duration: TimeInterval,
              animations: @escaping () -> Void,
              completion: ((Bool) -> Void)? = nil
          )
        ```

  - [animate(withDuration:delay:options:animations:completion:)](https://developer.apple.com/documentation/uikit/uiview/1622451-animate)

    - ```swift
        class func animate(
            withDuration duration: TimeInterval,
            delay: TimeInterval,
            options: UIView.AnimationOptions = [],
            animations: @escaping () -> Void,
            completion: ((Bool) -> Void)? = nil
        )
      ```

  - [animate(withDuration:delay:usingSpringWithDamping:​initialSpringVelocity:options:animations:completion:)](https://developer.apple.com/documentation/uikit/uiview/1622594-animate)

    - ```swift
        class func animate(
            withDuration duration: TimeInterval,
            delay: TimeInterval,
            usingSpringWithDamping dampingRatio: CGFloat,
            initialSpringVelocity velocity: CGFloat,
            options: UIView.AnimationOptions = [],
            animations: @escaping () -> Void,
            completion: ((Bool) -> Void)? = nil
        )
      ```

- **Animate with Duration**
  - The first method requires only the two parameters that are shared by all four methods. The first parameter, `duration`, represents the number of seconds allocated for the animation. The second parameter, `animations`, is a closure that contains changes to the view's animatable properties.
  - For example, the following code will animate a change to the `alpha` property over the course of two seconds. The change is made within a trailing closure, which you learned about in a previous lesson.

    - ```swift
        UIView.animate(withDuration: 2.0) {
            view.alpha = 0.3
        }
      ```

  - How simple is that? All you had to do is tell the function the end result, and Core Animation handles rendering each frame. As you can see, you can create a sophisticated animation with only a few lines of code.
  - Go ahead and try it for yourself. Create a new playground in Xcode called “FlyingSquares.” You'll use a feature called the live view, which allows you to add views and experiment with some of the animation features you'll learn about in this lesson.
  - Each page in an Xcode playground can have its own live view for displaying a view hierarchy in the assistant editor. To use the live view feature, you'll need to import the `PlaygroundSupport` framework, which gives you access to special playground properties. One of the objects in this framework is called `PlaygroundPage`. Use the `PlaygroundPage.current` property to access the current playground page.
  - In your playground, create a new `UIView` with a 500-by-500-point frame and set it to the `PlaygroundPage.current.liveView` property.

    - ```swift
        import UIKit
        import PlaygroundSupport
         
        let liveViewFrame = CGRect(x: 0, y: 0, width: 500, height: 500)
        let liveView = UIView(frame: liveViewFrame)
        liveView.backgroundColor = .white
         
        PlaygroundPage.current.liveView = liveView
      ```

  - Open the assistant editor and you'll see a square, white view. To verify that the live view is working correctly, try changing the background color and watch the live view change.
  - Now create a 100-by-100-point view and add it as a subview to the live view you just created. You'll use this smaller view to experiment with the different animation methods in this lesson.

    - ```swift
        let smallFrame = CGRect(x: 0, y: 0, width: 100, height: 100)
        let square = UIView(frame: smallFrame)
        square.backgroundColor = .purple
         
        liveView.addSubview(square)
      ```

  - Once the live view refreshes, you'll see the new small view in the top-left corner of the live view. Now you're ready to play with some animations.
  - Add your first animation by using the `UIView.animate(withDuration:animations:)` method to change the background color over three seconds.

    - ```swift
        UIView.animate(withDuration: 3.0) {
            square.backgroundColor = .orange
        }
      ```

  - Experiment with a few of the different animatable properties of the square instance. Try to make the small view larger and move it to the center of the live view. (Hint: Use the frame or center properties to move the view.)

    - ```swift
        UIView.animate(withDuration: 3.0) {
            square.frame = CGRect(x: 100, y: 100, width: 300, height: 300)
        }
      ```

  - Once you get the view to move, create another animation that returns the square back to its original size, position, and color.

    - ```swift
        UIView.animate(withDuration: 3.0) {
            square.backgroundColor = .orange
            square.frame = CGRect(x: 100, y: 100, width: 300, height: 300)
        }
        UIView.animate(withDuration: 3.0, delay: 3.0) {
            square.backgroundColor = .purple
            square.frame = CGRect(x: 0, y: 0, width: 100, height: 100)
        }
      ```

  - Were you able to get the second animation to work? If so, nice work! Otherwise, read the following section to figure out how to use a completion handler to execute code after an animation finishes.
- **Add a Completion Handler**
  - In the previous example, you tried to animate the size, position, and color of the view back to their original values.
  - Why didn't this approach work? The main reason is that the second `animate` call is executed immediately after the first call — so Core Animation skipped the first animation and started executing the new instructions. One way to fix this is to call the second animation after the first animation is completed, not after it is called. You can do this by using the `UIView.animate(withDuration:animations:completion:)` method.
  - Similar to the `animations` parameter, `completion` is also a closure. However, the `completion` closure is executed after your animations have completed (as the name suggests). You can use this closure to run any action you'd like to occur after the animation has finished, including starting another animation.
  - The following code will animate the background color, update the frame, and, using the completion handler, return the view back to its original size, position, and color:

    - ```swift
        UIView.animate(withDuration: 3.0, animations: {
            square.backgroundColor = .orange
            square.frame = CGRect(x: 150, y: 150, width: 200, height: 200)
        }) { (_) in
            UIView.animate(withDuration: 3.0) {
                square.backgroundColor = .purple
                square.frame = smallFrame
            }
        }
      ```

  - Replace your `animate(withDuration:animations:)` method with the animate`(withDuration:animations:completion:)` method, and experiment with performing other actions after the animation has completed.
- **Add a Delay or Custom Options**
  - The third common method used for animations is `UIView.animate(withDuration:delay:options:animations:completion:)`, which introduces two new parameters. The `delay` parameter represents the number of seconds to delay the start of the animation. The `options` parameter is an array of predefined options for customizing animations. For example, you can set your animation to repeat, or you can set a timing curve for the animation. Check out the documentation for available [UIViewAnimationOptions](https://developer.apple.com/documentation/uikit/uiview/animationoptions).
  - The following code moves square from the top-left corner to the bottom-right corner over three seconds after a two-second delay, and repeats the animation indefinitely. It uses no completion closure.

    - ```swift
        UIView.animate(withDuration: 3.0, delay: 2.0, options: [.repeat], animations: {
            square.backgroundColor = .orange
            square.frame = CGRect(x: 400, y: 400, width: 100, height: 100)
        }, completion: nil)
      ```

  - Take some time to experiment with the different animation options.
- **Spring Animations**
  - The fourth method used for animation can really polish your user's interactions — when used with restraint. As you use iOS, you may notice that some interactions and animations feel “bouncy” and can add a spark of joy when you see them. Try the following to move the square from the top-left corner to the center and double its size.

    - ```swift
        UIView.animate(withDuration: 1.3, delay: 0.0, usingSpringWithDamping: 0.5, initialSpringVelocity: 0.3, options: [], animations: {
            square.frame = CGRect(x: 150, y: 150, width: 200, height: 200)
        }, completion: nil)
      ```

  - Notice how the square oscillates as it settles into place, as if it's attached to a spring. This effect can enhance your animations and make your app feel more playful. Try adjusting the duration, damping, and initial velocity values to see how they affect the animation. You may find that it is hard to get these values just right, and certain combinations can lead to unexpected results.
  - Consider using these animations in your apps where it makes sense, but remember not to overdo it. Your users could quickly become irritated by the effect — but just the right touch can change the entire feel of your apps.

#### The Transform Property

- Did you recognize most of animatable properties on the list at the beginning of this lesson? There's probably one that's new to you: the `transform` property. `transform` is an instance of the structure `CGAffineTransform` and can be used to change a view's scale, rotate a view, or move a view without calculating changes to the view's frame.
- Unless you're really into advanced mathematics, this structure can seem a bit intimidating. An affine transform is a three-by-three matrix that maps each point in an untransformed view to another point, resulting in a transformed view.
- Because these animations are so common, iOS has helpful initializers for generating transforms that scale, rotate, or translate your views. And you won't need an advanced math degree to use them.
- | Type | Initializer | Parameter Description |
  |:---:|:---:|:---:|
  | Scale | init(scaleX: CGFloat, y: CGFloat) | The factors by which to scale your view. |
  | Rotate | init(rotationAngle: CGFloat) | The angle (in radians) by which to rotate your view, with a positive value indicating counterclockwise rotation. |
  | Translate | init(translationX: CGFloat, y: CGFloat) | The value by which to move (shift) your view. |
- Instead of manually updating the frame, you can use these transforms to animate a change in scale, rotation, or position. Basic transforms may resemble the following:

  - ```swift
      /* Double the height and width of the view */
      let scaleTransform = CGAffineTransform(scaleX: 2.0, y: 2.0)
       
      /* Rotate the view 180 degrees */
      let rotateTransform = CGAffineTransform(rotationAngle: .pi)
       
      /* Move the view 200 points to the right and 200 points down */
      let translateTransform = CGAffineTransform(translationX: 200, y: 200)
    ```

- You'll notice that the `rotationAngle` parameter in the example above is given the value `.pi`. The rotation value is expressed in radians, a standard unit of measure for angles. 180 degrees is the same as π radians.
- It's also possible to combine transform instances by concatenating them. The following code combines the three transforms in the previous example into a single transform that's assigned to the square view:

  - ```swift
      UIView.animate(withDuration: 2.0) {
          square.backgroundColor = .orange
       
          let scaleTransform = CGAffineTransform(scaleX: 2.0, y: 2.0)
          let rotateTransform = CGAffineTransform(rotationAngle: .pi)
          let translateTransform = CGAffineTransform(translationX: 200, y: 200)
          let comboTransform = scaleTransform.concatenating(rotateTransform)​.concatenating(translateTransform)
       
          square.transform = comboTransform
      }
    ```

- There's one more property of the `CGAffineTransform` type that may be useful: `identity`. You use the `identity` property to undo any `CGAffineTransform` animations you have implemented. The following code uses the completion handler to undo the `CGAffineTransform` animations:

  - ```swift
      UIView.animate(withDuration: 2.0, animations: {
          square.backgroundColor = .orange
       
          let scaleTransform = CGAffineTransform(scaleX: 2.0, y: 2.0)
          let rotateTransform = CGAffineTransform(rotationAngle: .pi)
          let translateTransform = CGAffineTransform(translationX: 200, y: 200)
          let comboTransform = scaleTransform.concatenating(rotateTransform)​.concatenating(translateTransform)
       
          square.transform = comboTransform
      }) { (_) in
          UIView.animate(withDuration: 2.0, animations: {
              square.transform = CGAffineTransform.identity
          })
      }
    ```

- Note that the color is still orange because applying the `identity` property will only undo the `CGAffineTransform` changes.
- If you're interested in exploring the math involved with these affine transforms, start by checking out the CGAffineTransform Documentation.

#### Animate Constraints

- You have seen that you can animate the properties of a UIView. You can also animate constraints through the constant property on `NSLayoutConstraint` (the class all constraints are instances of). To modify a constraint, you need to have a reference to it in code. Similar to creating outlets to views, you can create an outlet for a constraint.
  - `@IBOutlet var widthConstraint: NSLayoutConstraint!`
- To update a constraint, modify the constant value. In the following example, imagine you have a view whose width is defined by `widthConstraint`. When the `buttonTapped(_:)` method is called, the constraint's `constant` is set to `320`, which will update the width the next time that the screen is redrawn.

  - ```swift
      @IBAction func buttonTapped(_ sender: UIButton) {
        widthConstraint.constant = 320
      }
    ```

- To animate the change in width, iOS needs to be given the instruction to adjust the width over time. To do this, call `layoutIfNeeded()` on the parent view inside an animation closure. The `layoutIfNeeded()` method tells the receiver to lay out its subviews, and doing so within an animation will adjust the layout over a given time.

  - ```swift
      @IBAction func buttonTapped(_ sender: UIButton) {
        widthConstraint.constant = 320
       
        UIView.animate(withDuration: 0.5) {
          self.view.layoutIfNeeded()
        }
      }
    ```

- When should you animate constraints instead of modifying the frame or transform properties? Changing the view properties will not modify the position and size of other views on the screen. If, however, you have a series of views whose positions and sizes are based on one another, and you want all the views to update to reflect new constraint values, you should animate the constraints.

#### Animation In Practice

- While it may be tempting to add animations to every view and every action, it's important to have some restraint. In Unit 4, you'll learn more about the Human Interface Guidelines, including those for animations. Here's a quick preview:
  - Use animation judiciously, as excessive or gratuitous animation can be distracting.
  - Strive for realism and credibility. Users will accept some artistic license, but onscreen movement should make sense. If the user will reveal a view by swiping down from the top of the screen, they should be able to dismiss it by swiping up. Also, consider how things move in the real world. For example, a car accelerates gradually to the desired speed, then decelerates gradually to come to a stop. Consider using an ease in/out timing curve so that your animated objects mirror this real-world experience of acceleration.
  - Use consistent animation. iOS users are accustomed to smooth transitions, fluid changes in device orientation, and physics-based scrolling. Unless you’re creating an immersive experience, such as in a game, you'll want to use the built-in animations.
  - Make animations optional. When the option to reduce motion is enabled in accessibility preferences, your app should minimize or eliminate animations. Use the `UIAccessibilityIsReduceMotionEnabled()` function to check whether reduced motion is enabled.

#### Build An Animation Wireframe

- To practice your new animation knowledge, you'll create a wireframe — just the views, without actual functionality — of the Now Playing screen in the Music app.
- If you have the Music app on an iOS device, take a few minutes to play around with it; otherwise, take a moment to notice the animations in the previous video.
- Notice that when you tap the play/pause button, reverse button, or forward button, a circle appears — the finger shadow as it depresses the button — and the icon gets smaller. This animation works to tell the user that the button was tapped.
- Also take note of the album art. When you switch between playing music and pausing music, the album art changes size. The album is large and at full width when playing, and smaller but still centered when paused. This animation keeps the user oriented by signaling that the buttons are affecting the current selection. It also helps inform the user that the song has started or stopped playing.
- You're now familiar with how animations should be used and some of the code you'll use to build them. You'll apply that knowledge and practice creating animations as you build this wireframe:
- Not counting the view controller's view, how many subviews are there? What is the finger shadow? Including the stack view containing the buttons, you'll have eight views: an image view, a stack view with three buttons, and three views for the finger shadows.
- **Set Up the View**
  - Create a new Xcode project using the iOS App template and name it "MusicWireframe." Once created, open the Main storyboard.
  - Build a view similar to the one in the earlier video as if the track were playing. Add an image view from the Object library, and use Auto Layout to constrain its top, leading, and trailing edges to the top, leading, and trailing edges of the view margin. Also constrain the view's aspect ratio to 1:1. Since you won't have actual album art to display, adjust the background color so that the view is visible.
    - <img src="./resources/music_wireframe_image.png" alt="Music Wireframe Image" width="800" />
  - Next, add the stack view and the buttons. In the Attributes inspector, set the Style to `Default`, delete the buttons' Title text, and set their Image properties to use the system images called `backward.fill`, `play.fill`, and `forward.fill`. Use the Symbol Configuration to adjust their `point size` appropriately. For the center button, configure the `play.fill` image for the `Default` state and the `pause.fill` image for the `Selected` state, then mark the button's State as `Selected` in the Attributes inspector's `Control` section so that the pause image is visible by default.
  - Use Auto Layout and the stack view to neatly display the buttons below the image view. Your stack view should have the following attributes: `Horizontal` Axis, `Fill` Alignment, and `Equal Spacing` Distribution.
  - Now add three UIView instances to the view. These will act as the finger shadow. Adjust their color to `Light Gray Color`. Next, add constraints to `center` the views behind each button.
    - Move the view to the center of the button. To move shortly, Command + Drag the view. Control + Drag from the view to the button. Command + select center vertically and center horizontally.
  - Set the aspect ratio on each view to `1:1` so that they remain square, then set the height equal to the corresponding button with a multiplier of 1.5 – making them slightly larger.
    - In Add New Constraints, select Aspect Ratio, and then edit the ratio to 1:1 in Size inspector Aspect Ratio.
    - Control + Drag from the view to the button, select Equal Height and modify the multiplier to 1.5 in Size inspector Vertical.
  - <img src="./resources/music_wireframe_buttons.png" alt="Music Wireframe Buttons" width="800" />
  - Build and run your app to test your constraints. You should see your views laid out correctly.
- **Add Outlets**
  - Next, you'll need access to all the views you'll change. Add an `@IBOutlet` to ViewController for each of these views:

    - ```swift
        @IBOutlet var albumImageView: UIImageView!
        @IBOutlet var reverseBackground: UIView!
        @IBOutlet var playPauseBackground: UIView!
        @IBOutlet var forwardBackground: UIView!
        @IBOutlet var reverseButton: UIButton!
        @IBOutlet var playPauseButton: UIButton!
        @IBOutlet var forwardButton: UIButton!
      ```

- **Initialize Views**
  - Building the circular finger shadow view can be a little tricky. You'll need to make the view appear as a circle, and you'll want to hide the view until the user taps the button.
  - While many view properties are adjustable in Interface Builder, this is one scenario where it is easier to build the view programmatically. You'll need to do some of the initialization in the `viewDidLoad()` method.
  - To make a square view render as a circle, you'll need to set two properties: the layer's corner radius and `clipsToBounds`. The layer is used by Core Animation to render the view on the screen; setting the corner radius will round the corners of the view, and the `clipsToBounds` property determines whether the view's visible layer should be confined to the bounds of the view. You can learn more about these properties in the [documentation for UIView](https://developer.apple.com/documentation/uikit/uiview).
  - To make a circle, set the corner radius of the layer to half the height of the view and set the `clipsToBounds` property to `true`. Update your `viewDidLoad()` method to make each of the background views circular:

    - ```swift
        override func viewDidLoad() {
            super.viewDidLoad()
         
            reverseBackground.layer.cornerRadius = reverseBackground.frame.height / 2
            reverseBackground.clipsToBounds = true
         
            playPauseBackground.layer.cornerRadius = playPauseBackground.frame.height / 2
            playPauseBackground.clipsToBounds = true
         
            forwardBackground.layer.cornerRadius = forwardBackground.frame.height / 2
            forwardBackground.clipsToBounds = true
        }
      ```

  - Build and run the app to see that your background views are now circles.
  - Finally, set the background views' alpha value to 0 to make them completely transparent when the view loads.
  - Since you'll be doing the same adjustments to each view, you could opt to use the `forEach(_:)` method on an array of the views, which applies a closure to all its items:

    - ```swift
        override func viewDidLoad() {
            super.viewDidLoad()
         
            [reverseBackground, playPauseBackground,
              forwardBackground].forEach { view in
                view?.layer.cornerRadius = view!.frame.height / 2
                view?.clipsToBounds = true
                view?.alpha = 0.0
            }
        }
      ```

  - You can use a `Bool` property value to keep track of whether the song is playing, which will impact your properties. If you'd like, you can also use property observers to automatically update the play/pause button's selection state and, therefore, its image.

    - ```swift
        class ViewController: UIViewController {
            var isPlaying: Bool = true {
                didSet {
                    playPauseButton.isSelected = isPlaying
                }
            }
            //...
        }
      ```

  - If you build and run your app, you should see a very similar screen to your last build, but the finger shadow views are no longer visible. If you'd like to see your circular views, you can comment out the line (or lines) adjusting the alpha. (As you move on, don't forget to uncomment that code.)
- **Add Album Art Animation**
  - Now it's time to focus on the animation of the album art. As the user plays or pauses the song, the album art will change: from full size when playing to smaller when paused. To respond to the user tapping the play/pause button (and changing the state of the animation), you'll need to add an `@IBAction` to the button.
  - Add a new action for the play/pause button: `@IBAction func playPauseButtonTapped() {}`
  - When the button is tapped, you'll toggle the `isPlaying` property and adjust the image view for its new state. If the song is playing, you'll animate the view to its full size — with a little bit of spring to the animation. Otherwise, when the song is not playing, you'll reduce the image view to 80 percent of its original size — with no spring this time.
  - Which property should you animate here? If you said the `transform` property, you'd be right. But you may have said `frame` or `bounds`. While you could use either of these properties to adjust the size, it would be easier to use a scale affine transform, which will allow you to adjust the size by a specified factor.
  - Remember the `CGAffineTransform` type's `identity` property? You'll use that to undo previous transforms and return the image to its original size. The following code shows how to apply the `identity` property. Of course, as with all animations, you'll also need to define a duration.

    - ```swift
        @IBAction func playPauseButtonTapped() {
            isPlaying.toggle()
         
            if isPlaying {
                UIView.animate(withDuration: 0.8, delay: 0, usingSpringWithDamping: 0.6, initialSpringVelocity: 0.1, options: [], animations: {
                    self.albumImageView.transform = CGAffineTransform.identity
                }, completion: nil)
            } else {
                UIView.animate(withDuration: 0.5) {
                    self.albumImageView.transform = CGAffineTransform(scaleX: 0.8, y: 0.8)
                }
            }
        }
      ```

  - Build and run your app. At this point, you should be able to tap the play/pause button and see the image view animate as you change from playing to paused.
- **Add Button Animations**
  - Are you ready to add the animations that make the buttons look like the user is physically pressing them? Check out the Music app again and see what happens when you touch the button and when you release it. As you release, the finger shadow grows (even as it's fading away), and the icon returns to its normal size.
  - To accomplish these animations, you'll need to add two actions: one that responds to the touch-down control event and one that responds to the touch-up event. Since these actions will be similar across the three buttons, you can repeat them. But how will you know which button was tapped and which finger shadow to animate? You'll have the sender be an argument to the action function.
  - Start by adding two actions to the reverse button:
    - | Connection | Name | Type | Event | Arguments |
      |---|---|---|---|---|
      | Action | touchedDown | UIButton | Touch Down | Sender |
      | Action | touchedUp | UIButton | Touch Up Inside | Sender |

    - ```swift
        @IBAction func touchedDown(_ sender: UIButton) {}
         
        @IBAction func touchedUpInside(_ sender: UIButton) {}
      ```

  - Next, connect the other two buttons to the same two methods, making sure to connect the correct touch event. It might be helpful to use the assistant editor to drag from the Connections inspector to the view controller class file. 
  - At this point, you could add print statements to test that your button actions are hooked up and firing correctly. The reverse and forward buttons should have two actions, and the play/pause button should have three (touchedDown, touchedUpInside, playPauseButtonTapped).
  - You may notice that there are two methods called on a touch-up event for the play/pause button. While not a typical configuration, each method in this case is responsible for different actions/animations, so it's reasonable to separate the functions.
  - Before you can animate the button views, you'll need to figure out which button was tapped and get the appropriate finger shadow (background) view. Using a switch statement, determine which button was tapped and store the appropriate background view in a constant:

    - ```swift
        @IBAction func touchedDown(_ sender: UIButton) {
            let buttonBackground: UIView
         
            switch sender {
            case reverseButton:
                buttonBackground = reverseBackground
            case playPauseButton:
                buttonBackground = playPauseBackground
            case forwardButton:
                buttonBackground = forwardBackground
            default:
                return
            }
        }
      ```

  - Now that you know the views you'll animate, it's time to make the animation call and update the properties in the closure.
  - For touchedDown, the button will scale by a factor of 80 percent on both sides, and the background view will have an alpha of 30 percent—just enough to give a subtle appearance.

    - ```swift
        @IBAction func touchedDown(_ sender: UIButton) {
            let buttonBackground: UIView
         
            switch sender {
            case reverseButton:
                buttonBackground = reverseBackground
            case playPauseButton:
                buttonBackground = playPauseBackground
            case forwardButton:
                buttonBackground = forwardBackground
            default:
                return
            }
         
            UIView.animate(withDuration: 0.25) {
                buttonBackground.alpha = 0.3
                sender.transform = CGAffineTransform(scaleX: 0.8, y: 0.8)
            }
        }
      ```

  - Now see if you can write code for the touch-up action. In this animation, the background view will disappear (alpha = 0) and expand slightly (say, 120 percent on each side), and the button will return to its original size (use the identity type property). In addition, when the animation is complete (and therefore the background view is invisible), you'll return the view to its original size. Remember that you'll need to determine which button and which background view to animate.

    - ```swift
        @IBAction func touchedUpInside(_ sender: UIButton) {
            let buttonBackground: UIView
         
            switch sender {
            case reverseButton:
                buttonBackground = reverseBackground
            case playPauseButton:
                buttonBackground = playPauseBackground
            case forwardButton:
                buttonBackground = forwardBackground
            default:
                return
            }
         
            UIView.animate(withDuration: 0.25, animations: {
                buttonBackground.alpha = 0.0
                buttonBackground.transform = CGAffineTransform(scaleX: 1.2, y: 1.2)
                sender.transform = CGAffineTransform.identity
            }) { (_) in
                buttonBackground.transform = CGAffineTransform.identity
            }
        }
      ```

  - At this point, you should have a working wireframe—complete with animations on the image view, as well as on the buttons and their background views. Nice work!

#### Lab - Enter to Win a Contest

- The objective of this lab is to help you understand when is a good time to use an animation. You'll create an animation that will help your app be more intuitive for the user.
- Create a new Xcode project called “Contest” and save it to your projects folder.
- The app will have the user input an email address to be entered into a contest to win a prize. When the user taps the button to submit their email address, they will be taken to another screen that says they've been entered into the contest.
- If the user taps the submit button but has not entered an email address, the app will stay on the input view, and the text field will animate to grab the attention of the user — helping them see that they need to fill in the text field to enter the contest.

##### Step 1. Set Up the Storyboard

- On the first view controller of the app, have a label at the top to tell the user that they need to enter an email address to be entered in the contest.
- Add a text field for the user to input their email address.
- Add a button to submit the entry to the contest.
- Add another view controller. Create a segue from the text field of the first view controller to the second view controller. Give the segue an identifier.
- On the second view controller, add a label to say that the user has successfully entered the contest.
- Create an `@IBOutlet` for the email text field. Create an `@IBAction` for the submit button.

##### Step 2. Create the animation

- When the user enters an email address, they should be moved to the next screen. Otherwise, the text field should shake a little, indicating to the user that they need to enter an email address.
- Using what you know of the `animate(withDuration:)` methods, determine how you will compose the animation. Try using `CGAffineTransform(translation)`.
- Using the text field's `transform` property, move the text field slightly, perhaps to the right.
- Use the `identity` property to return the text field to its original state.

##### Step 3. Implement the animation

- In the `@IBAction` for the submit button, create an `if-else` statement.
- If the text field is an empty string, then use the animation code written in Step 2 on the text field.
- Else, when the text field has a value, perform the segue with the identifier set in Step 1.

  - ```swift
      @IBAction func submitButtonTapped(_ sender: UIButton) {
          if let text = emailTextField.text, !text.isEmpty {
              performSegue(withIdentifier: "Succeed", sender: nil)
          } else {
              UIView.animate(withDuration: 0.1, animations: {
                  self.emailTextField.transform = CGAffineTransform(translationX: 10, y: 10)
              }) { (_) in
                  UIView.animate(withDuration: 0.1, animations: {
                      self.emailTextField.transform = CGAffineTransform(translationX: -10, y:-10)
                  }, completion: { (_) in
                      self.emailTextField.transform = CGAffineTransform.identity
                  })
              }
          }
      }
    ```

- Great job! You've successfully created an animation that helps the user understand how to use your app. Animations can be a powerful resource for you to use in your apps. Continue to practice and use animations. Be sure to save your project to your projects folder for future reference.

##### Challenge — Extra Shaking

- There's one more animation API that was not discussed: `animateKeyframes(withDuration:delay:options:animations:completion:)`. Study the [documentation for this method](https://developer.apple.com/documentation/uikit/uiview/1622552-animatekeyframes) and try using it to enhance the shake animation by shaking the text field back and forth a few times.

  - ```swift
      UIView.animateKeyframes(withDuration: 0.6, delay: 0.0, options: .calculationModeCubic, animations: {
          UIView.addKeyframe(withRelativeStartTime: 0.0, relativeDuration: 0.25) {
              self.emailTextField.transform = CGAffineTransform(translationX: 10, y: 10)
          }
          UIView.addKeyframe(withRelativeStartTime: 0.25, relativeDuration: 0.25) {
              self.emailTextField.transform = CGAffineTransform(translationX: -10, y: -10)
          }
          UIView.addKeyframe(withRelativeStartTime: 0.5, relativeDuration: 0.25) {
              self.emailTextField.transform = CGAffineTransform(translationX: 10, y: 10)
          }
          UIView.addKeyframe(withRelativeStartTime: 0.75, relativeDuration: 0.25) {
              self.emailTextField.transform = CGAffineTransform(translationX: -10, y: -10)
          }
      }, completion: { (_) in
          self.emailTextField.transform = CGAffineTransform.identity
      })
    ```

### Lesson 2.4 Working with the Web: HTTP and URL

- So far in this course, you've built apps that allow the user to create data or to use data from their device. But there's an entire world of information out there on the internet. How can you enable users to access it or to send information over the internet directly from your app?
- In this lesson, you'll learn the basics of how web data is sent and received, how URLs work, and how to fetch data for use in your app. Along the way, you'll build a playground that uses a URLSession to fetch and display a photo. In future lessons, you'll learn how to turn data you fetch from the web into custom model data, where to put code that performs network requests, and best practices for displaying web data in your app.
- Take a minute to think about some of your favorite apps. How many of them work without a Wi-Fi or cellular data connection? Most apps rely on a network connection to fetch or send data. Safari, Mail, Messages, and the App Store all require an internet connection to browse the web, fetch emails, send and receive messages, or check out the latest apps.
- As an app developer, you'll undoubtedly want to build apps that require a network connection to communicate with a web service — whether fetching information to display or sending data that the user has entered. So how do you do it?
- Before digging into the specifics of writing network requests in your app, it's important to understand some basics about how the web works. You'll learn about the HTTP and HTTPS protocols, the most common ways to send information around the web; how URLs are crafted to tell a web server what data you're requesting; and how to use the native `Foundation` APIs to create data tasks for sending and receiving data.

#### The Basics

- When you open a website, you send a network request that travels over the internet until it arrives at a web server — a specialized computer that's programmed to listen for your requests and to respond to them. The web address you use determines which server receives the request and what response you'll get. So when you enter apple.com in your web browser's address bar, you'll see Apple's home page — and you won't see it when you go to any other web address.
- You probably already know that a web address is also called a URL, which stands for uniform resource locator. But have you ever wondered what all the pieces of those sometimes long URLs are?
- A URL consists of two required parts:
  - http://apple.com
    - http:// : Protocol
    - apple.com :  Domain
- URLs can get more specific and more complex by including subdomains, ports, paths, or query parameters.
  - https://itunes.apple.com/us/app/keynote/id409183694?mt=2
    - https:// : Protocol
    - itunes : Subdomain
    - apple.com : Domain
    - us/app/keynote/id409183694 : Path
    - mt=12 : Query
- Even though URLs may appear long and complicated, you'll find they're easier to understand once you break them down.
- **Protocol**
  - Similar to protocols in iOS, a web protocol specifies how the browser and the server communicate with each other. In general, websites use the HTTP or HTTPS protocol. HTTP stands for Hypertext Transfer Protocol, and HTTPS stands for Hypertext Transfer Protocol Secure. Hypertext refers to text or images that link to other hypertext files, and the protocol specifies how to send and request hypertext information. For many years, HTTP was the more common of these two protocols, but most websites have moved, or are moving, to the more secure HTTPS.
- **Subdomain**
  - A subdomain can be used to point to a more specific server or to redirect to another URL.
- **Domain Name**
  - A domain name is a unique reference to a specific website and always includes a host name (like “apple” or “itunes”) and a top-level domain (like “.com” or “.org”). A domain name points to specific servers that handle all the requests that go to that domain.
- **Port**
  - Ports are a standard feature of the Internet Protocol (IP), on which HTTP and HTTPS are built. Every computer connecting to the internet via IP can expose multiple ports, each with its own number. Different ports are associated with different protocols and types of data. By default, websites use the standard ports assigned to web protocols — port 80 (HTTP) or port 443 (HTTPS) — so browsers will first check those ports if the URL doesn't specify a port number. You won't usually see the port number in a URL for that reason.
- **Path**
  - The path refers to a specific file or subdirectory on a server. For example, if you navigate to https://apple.com/iphone/ the server will respond with the HTML that displays information about iPhone products.
- **Query Parameters**
  - A query provides the server with even more specifics about a request. Queries work like dictionaries in Swift, with keys and values that give the server parameters for generating a page with the desired information. For example, if you navigate to https://www.apple.com/us/search/iPhone?sel=explore&page=2, you're telling the Search page to display the second page of results for the keyword “iPhone” under the Explore tab.
  - You can think of a URL almost like a function in Swift. Just as a function is called on a specific instance, can take parameters, and returns a value, a URL points to a specific server, can take parameters in the form of a path and queries, and returns information.
- **Request Type, Headers, and Body**
  - The most important part of a network request is the URL, which defines where the request should go and passes parameters that the server can use to return the correct information. Network requests can also have a request type, headers, and a body.
- **Request Type**
  - The request type is also known as the HTTP method, and the two most common are GET and POST. As you'd expect, GET is used to request information from a server, and POST is used to send a body of information to a server. When you load a website, you're sending a GET request. When you submit a form on a website, you're sending a POST request.
- **Headers**
  - Like the path and query parameters, the headers of a network request work in the same way as a dictionary: with keys and values that tell the server how to handle the request. (The URL doesn't include the headers, because the user doesn't usually need to see them.)
  - One common header parameter, called `User-Agent`, is a string that identifies the system performing the request. The full `User-Agent` header from a network request sent by Safari might look like the following: `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_3) AppleWebKit/602.4.8 (KHTML, like Gecko) Version/10.0.3 Safari/602.4.8`
  - Responses to your request will also have header values. Header values usually describe the data that's being returned, but may include information about how the browser or client should use the information — for example, if the information can be saved and reused later, or when the information will be outdated.
  - Headers are also used for authentication, which can help you log in to a protected server, or for cookies, which enable the server to keep track of your interactions on the website.
- **Body**
  - The body of a request includes the data that was sent or received. When you receive a response to a GET request, it will also have a body. For example, when you request the information for a website, the resources that make up the site — its HTML, CSS, and images — are all included as bodies of the network request.
  - When your app sends a POST request, it will send a body of information — whether plain text, JSON, or a file — to be posted to the server.

#### Create A URL

- Now that you know what a request is and a little bit about how a response might look, you can start learning the code for creating and sending requests in Swift.
- Create a new playground called “Working with the Web.” You'll use the playground to learn how to construct URLs in Swift, and you'll create and execute your first network request. Playgrounds are a great place to experiment with writing networking code and printing the results.
- You'll use the `URL` type in Swift to construct and modify the URL you'll use in your network requests. Creating a `URL` object is simple: Use the `init?(string: String)` initializer, which returns an optional URL.
- In your playground, create a `URL` object that points to `https://www.apple.com` by adding this line of code: `let url = URL(string: "https://www.apple.com")!`
- Why does that code use the `!` operator? You've already learned that the failable initializer will return an optional `URL` instance. If you look at the documentation for the initializer, you'll find a discussion that explains when the initializer will fail: `Returns nil if a URL cannot be formed with the string (for example, if the string contains characters that are illegal in a URL, or is an empty string).`
- In fact, URLs can contain any of the following characters: 
  - ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-._~:/?#[]@!$&'()*+,;=`.
- As long as you're drawing only from that list of characters, you're set to force - unwrap the optional `URL`. If the unwrap fails, your program will crash — and you'll know there's something wrong with the string you're using to initialize the URL.
- Now that you know how to initialize a `URL` instance, you can experiment with printing out its different properties. For example, use the `scheme` property on `url` to print the HTTPS protocol, use `host` to print the domain, or use `query` to print the list of query parameters. (Note that if you try to print a property with an optional type, such as `query`, directly, the playground will warn you that it has implicitly coerced the expression. You can get around these errors with judicious use of force-unwrapping.)

  - ```swift
      let url = URL(string: "https://www.youtube.com/watch?v=ss1eqNJd8kA")!

      print("Protocol: \(url.scheme)")
      print("Port: \(url.port)")
      print("Domain: \(url.host)")
      print("Path: \(url.path)")
      print("Query Parameters: \(url.query)")

      /* Output */
      Protocol: Optional("https")
      Port: nil
      Domain: Optional("www.youtube.com")
      Path: /watch
      Query Parameters: Optional("v=ss1eqNJd8kA")
    ```

- Take a moment to navigate around your favorite website, looking at how the URL changes from page to page. Can you break it down into parts? If so, you're well on your way to creating and executing network requests.

#### Create and Execute a Network Request

- Now that you've experimented with how to declare simple `URL` instances in Swift, you'll use the `URLSession` class to create and execute your first network request.
- Any app can access a shared `URLSession` instance by accessing the `shared` type property. This session is in charge of managing all the network requests you send, and it allows you to run code when each request is finished.
- **Request Data**
  - Start by requesting data using the shared `URLSession`'s `data(from:delegate:) async throws` method.
  - The autocompletion placeholders should show you that the `from` parameter needs a `URL` instance and the delegate is an optional `URLSessionTaskDelegate`. The second parameter is useful if you need to keep tabs on the session as data is received, but in this case you won't need one. In fact, this parameter has a default `nil` argument, so you can leave it out altogether — enabling you to write this very simple code: `let (data, response) = URLSession.shared.data(from: url)`
  - Note that the method returns a Swift `tuple type`.
    - A `tuple` type is a comma-separated list of types enclosed in parentheses. A tuple type can be used as the return type of a function to enable the function to return a single tuple containing.
  - Tuples are a way for functions to return more than one value. Here, `data`, of type `Data`, represents the body of the response, or the data that you requested from the server. The `response`, of type `URLResponse`, represents information about the response itself, including a status code and any included header fields.
  - But you're not done — you have a handful of compiler errors to address, starting with `'async' call in a function that does not support concurrency.` Option-click the `data` method and check out the full function signature:

    - ```swift
        func data(from url: URL, delegate: URLSessionTaskDelegate? = nil)
          async throws -> (Data, URLResponse)
      ```

  - What does `async` mean? Up to now, all the code you've written has run synchronously. You've assumed, correctly, that each line of code will execute immediately after the one before it. Swift and iOS also support `asynchronous` code. 
    - `Asynchronous` code runs on separate processor threads, with user interface - related code running on the main thread and all session data tasks running on a background thread.
  - A function that's marked as async can suspend execution of your code, allow other code to execute, and then resume your code at a later time. Asynchronous code execution is also known as `concurrency`.
    - `Concurrency` is the idea that a program may run different parts of code - such as networking or animation code - on different threads, or queues, at the same time.
  - It's important to understand something about how network requests work to understand why concurrency is critical to working with `URLSession`.
  - You know that a network request is sent across the internet to a server, which then sends a response back to your app. While this exchange might be extremely fast, it isn't guaranteed to be fast. The server might respond with a very large file, or you might have a slow network connection. Either issue could cause the network request to last multiple seconds or tens of seconds — a very long time in the world of code.
  - The Swift concurrency system uses `actors` to control how asynchronous code runs. 
    - A Swift `actor` is a reference type. actors allow only one task to access their mutable state at a time, which makes it safe for code in multiple tasks to interact with the same instance of an actor.
  - iOS creates a main actor that runs all your user interface code by default. You can run most of your code on the main actor without any issues. But if you execute a time-consuming operation (like sending a network request and waiting for the response) synchronously on the main actor, you block it from running anything else — including all the code in `UIKit` that handles touches and draws views. Your app will appear frozen until the network request completes.
  - That's where asynchronous code comes into play. To avoid blocking the main actor while waiting on long-running work, a `URLSession` sends your request and receives data asynchronously. The main actor can continue updating the user interface and responding to user input while your suspended code waits for the response to your network request.
  - To call an `async` function, either the function that calls it must also be marked `async` or you must use a `Task`. Playground code has no enclosing function at all, so you'll need to put your code in a task, which is the way asynchronous code runs. You initialize a task with a closure containing the code you want it to run. Update the playground by putting your call to `data(from:delegate:)` in a `Task` (and note how trailing closure syntax makes your code easy to read).

    - ```swift
        Task {
            let (data, response) = URLSession.shared.data(from: url)
        }
      ```

  - The previous error is gone. Progress! Now you see that Swift wants you to mark the call with the `await` keyword. 
    - The Swift `await` keyword is used when calling a function that is marked with the `async` keyword, indicating that the caller is aware that the function may suspend execution to run tasks asynchronously.
  - `await` marks the suspension point in your code where the task may pause its execution while an async function performs work. Add the keyword now:

    - ```swift
        Task {
            let (data, response) = await URLSession.shared.data(from: url)
        }
      ```

  - Now there's a new error: `Call can throw, but it is not marked with 'try' and the error is not handled.` If you look again at the full method signature, you'll see that in addition to being `async`, `data(from:delegate:)` is marked as a throwing function.
    - A throwing function is a special type of Swift function that can return specific types of errors.
  - The function will throw an error if one occurs on the client side while running the network request. If so, you can handle it in your code if you choose.
  - Add the `try` keyword to indicate that you're prepared to deal with errors. While you're at it, print out the data you receive.

    - ```swift
        Task {
            let (data, response) = try await URLSession.shared.data(from: url)
         
            print(data)
        }
      ```

  - Success! Your code should now compile with no errors.
- **Execute the Playground to Send the Request**
  - When you run the playground, the Task will be created and the data method will be executed while the task is suspended. Once the data function has completed, the task will continue executing. If there were no errors, the data and response components of the tuple will be assigned values, and the print statement will execute.
  - Generally, this is all you need to execute a network request. The Swift concurrency model allows you to write code that looks very much like synchronous code, but enables the data method to wait on the network connection without stopping everything in its tracks.
  - If you wrote everything correctly, the value of the `Data` returned from the request will be printed to the console. You should see something like the following: `Console Output: 72386 bytes`
- **Interpret the Response**
  - At this point you've probably noticed a warning that "'response' was never used." The `response` from a data request can be used to check that the server returned an appropriate response to the request. You may have seen websites return "error 404" when something goes wrong. HTTP requests return a code of 200 if the request is processed normally. Update your code to check the response as follows:

    - ```swift
        let url = URL(string: "https://www.apple.com")!
        Task {
            let (data, response) = try await URLSession.shared.data(from: url)
         
            if let httpResponse = response as? HTTPURLResponse, httpResponse.statusCode == 200 {
                print(data)
            }
        }
      ```

  - In a real app, you'd use an `else` statement to handle errors returned by the server.
- **Process the Data**
  - Is this the first time you've worked with the `Data` type? `Data` is basically a bag of bits that could represent just about anything — in this case, the body of the response. To get an idea of what `Data` looks like, update your code to use the older `NSData` type and print that to the console instead. Here's how it works:

    - ```swift
        let url = URL(string: "https://www.apple.com")!
        Task {
            let (data, response) = try await URLSession.shared.data(from: url)
         
            if let httpResponse = response as? HTTPURLResponse, httpResponse.statusCode == 200 {
                print(data as NSData)
            }
        }
        /* Console Output: */
        {length = 72386, bytes = 0x0a090a0a 0a090a0a 090a0a0a 090a0a0a ... 090a0a0a 0a090a0a }
      ```

  - You'll notice the result has a truncated bytes property with incomprehensible values. If the result weren't truncated, it would go on for many pages! This is how computers talk, but — since you're a human — you'll want to convert the data into a more legible string. Fortunately, you can use the `String(init: Data, encoding: String.Encoding)` initializer to return a `String` from `Data`, as long as the initializer can parse the data correctly.
  - This lesson won't cover the various types of string encoding; just use the `.utf8` option for the `encoding` parameter. Add the initializer as a new line to your if-let statement and update your print statement to use the new string.

    - ```swift
        let url = URL(string: "https://www.apple.com")!
        Task {
            let (data, response) = try await URLSession.shared.data(from: url)
         
            if let httpResponse = response as? HTTPURLResponse, httpResponse.statusCode == 200, let string = String(data: data, encoding: .utf8) {
                print(string)
            }
        }
      ```

  - Scroll around in the console to see the HTML for the https://www.apple.com home page.
  - You may be wondering why you'd want your iOS app to fetch a website's HTML. That's a great question. In fact, fetching HTML isn't very useful at all. HTML describes how a website should be displayed in a browser — not in an iOS app.
  - More often than not, you'll want to use a network request to fetch raw information (instead of the website's HTML), which you can serialize into model objects and display in your app, just as you've done in previous apps throughout this course.
  - One of the most common formats on the web is called JavaScript Object Notation, or JSON. You'll do some work with JSON in the next lesson. For now, you'll learn how to request information from a web service, or API, that returns JSON. To wrap up this lesson, you'll print the JSON to the console, just like you did with the HTML in the earlier example.

#### Work With An API

- Throughout this course, you've likely seen the acronym API a number of times. API stands for "application programming interface." An API is a set of routines, protocols, or tools for building software applications. Each framework or class you've used as you worked through the lessons, labs, and projects in this course could be considered an API. However, in the context of working with the web, an API usually refers to a website or service that enables information to be sent or received via network requests.
- Many websites and organizations offer APIs that allow developers to work with different types of information. For example, the NASA Astronomy Picture of the Day (APOD) API serves up a beautiful image daily, along with a brief description of the image — powering the popular [Astronomy Picture of the Day](https://apod.nasa.gov/apod/astropix.html) website. By requesting data from the APOD API, you can build an app that displays the same photo and the same description.
- Later in this unit, you'll build your own NASA APOD app. In this lesson, you'll learn how the API works and how to fetch and print the data from the API.
- **API Basics**
  - You now know that a URL includes a domain name, a path, and optional query parameters. The path and query parameters are passed to the server, which processes them to respond with the appropriate information. Many APIs work exclusively by reading the information you pass to the server in the path and query parameters.
  - Web services that make their data available publicly will typically share documentation that details how to construct a URL to request data and will show an example of how the response is structured.
  - Open the documentation for the APOD API, available on NASA's website at https://api.nasa.gov. The page also contains documentation for some of NASA's other APIs, so make sure you locate the APOD section. You can explore the other APIs later.
  - On the NASA website, you'll find a description of the APOD service, a sample image, a description of the HTTP request, a list of the optional query parameters, and a sample query that includes a reference to an `api_key`. API keys are a common tool that web services use to monitor who is using the API and to make sure they're using the API appropriately. Many web services will provide a demo API key that you can use for a small number of requests. Others require you to register for (and possibly pay for) an API key, so they can monitor usage and contact you if you're going over specific limits or using the API incorrectly.
  - Navigate to the Authentication section of the NASA API documentation. You'll see the rate limits, or caps on how many requests you can make in a given time period. If you register for an API key, you can perform 1,000 requests per hour. If you use the demo key, you're limited to 30 requests per hour and 50 requests per day. If you exceed the limit, the API will return an error instead of the data you requested.
  - Given that limit, you should set your playground to run manually so that you won't send a new request to the API every time you modify your code.
- **Create the API Request**
  - Go ahead and create your first request to the APOD API. Update the URL in your network request to match the sample query in the documentation. The following code uses the demo API key. If you're completing this lesson in a classroom with other students, you may find that you need to register for your own free API key. If you do register for a key, replace DEMO_KEY with your key.

    - ```swift
        let url = URL(string: "https://api.nasa.gov/planetary/apod?api_key=DEMO_KEY&date=2018-11-27")!
        Task {
            let (data, response) = try await URLSession.shared.data(from: url)
         
            if let httpResponse = response as? HTTPURLResponse, httpResponse.statusCode == 200,
                let string = String(data: data, encoding: .utf8) {
                    print(string)
            }
        }
      ```

  - The base URL will always fetch information for today's photo. You can experiment with the URL by adding a date query, using the YYYY-MM-DD format, to access photos from any day after 1995-06-16.

#### Modify a URL with URL Components

- This optional section explores how to use the `URLComponents` type to create URLs without having to manually create a full URL string to pass into the `URL(string:)` initializer.
- You learned earlier in this lesson that there are restrictions on what can and can't be included in a URL. Looking back at the list of valid URL characters, you'll notice that it doesn't include the space character. There is a workaround for this problem: If you need to build a URL string that contains an invalid character, you can use a special format called percent-encoding, which represents certain characters with the percent sign (`%`) followed by a number. For example, a space is percent-encoded as `%20`.
- More generally, you'll often need to generate the different parts of a URL dynamically—especially if the URL will have query parameters.
- Fortunately, there's no need to memorize all the invalid characters and their percent-encoded replacements, nor how to construct valid query strings or other parts of a URL. The `Foundation` framework defines a type called `URLComponents` that will help you parse, read, or create all the parts of a URL in a safe, accurate way. `URLComponents` is especially useful when creating queries for URL instances.
- Create a new `URLComponents` instance that uses the base URL of the NASA APOD API (https://api.nasa.gov/planetary/apod). Then add `URLQueryItems` for the `api_key` and a `date`.
- A `URLQueryItem` is simply a single key/value pair, much like a `[String:String]` dictionary with one element. It would be a little bit easier to type if you could supply a dictionary directly — and maybe even easier to read. Use the `map()` function to map a `[String: String]` to an array of `URLQueryItems` to pass to the `queryItems` property on `URLComponents`.

  - ```swift
      var urlComponents = URLComponents(string: "https://api.nasa.gov/planetary/apod")!
      urlComponents.queryItems = [
          "api_key": "DEMO_KEY",
          "date": "2013-07-16"
      ].map { URLQueryItem(name: $0.key, value: $0.value) }
       
      Task {
          let (data, response) = try await URLSession.shared.data(from: urlComponents.url!)
       
          if let httpResponse = response as? HTTPURLResponse,
              httpResponse.statusCode == 200,
              let string = String(data: data, encoding: .utf8) {
                  print(string)
          }
      }
    ```

- The response you receive with your programmatically generated URL should look similar to your previous result. You can use this same technique in other projects when you need to dynamically generate URLs with queries — which you'll do in later lessons.

#### Wrap-up (HTTP and URL)

- You made it! It's no simple task to understand how networking works and how to perform a simple network request using Swift code. You'll continue building on your skills in the next two lessons. First, you'll learn how to take the data returned from an API and convert it into model objects. Next, you'll learn where to put the networking code in an Xcode project and how to update the user interface with data fetched from an API.

#### Lab - iTunes Search (Part1)

- The objective of this lab is to use URLSession to pull data down from the iTunes API and display it in the console. You'll use a search query dictionary to configure a URL, which the URLSession will use to fetch and print the correct data.
- Create a new playground called “iTunes Search.”

##### Step 1 Review the iTunes Search API

- Take a few minutes to review the [documentation for the iTunes Search API](https://performance-partners.apple.com/search-api). Find the base URL for search requests, and pay particular attention to the types of queries you can make.
- Look at the different parameter keys listed in the documentation and their corresponding values. The different keys (term, media, limit, etc.) will be used when you create your query dictionary. Think of some parameters that you might want to include in your query.
- Create a query `[String: String]` dictionary that looks for your favorite movie or for songs by your favorite music artist. Make sure to use the exact keys and expected values listed in the API documentation. At a minimum, you'll want to use the `term` and `media` keys.
- Now that you have your query dictionary, create a variable to hold the base URL for the iTunes Search API: https://itunes.apple.com/search.
- Create an instance of `URLComponents` with the base URL and add `queryItems` by mapping your query dictionary to an array of `URLQueryItems`.

##### Step 2 Pulling Data from the Web

- Now that you have your `URLComponents` configured correctly, you'll use its `url` property to fetch your data from the web.
- Use the shared `URLSession` and call its `data` method for the specified URL. The return value will give you a tuple containing a `Data` and a `URLResponse` value. Provide appropriate names for the placeholder values.
- Don't forget to prefix the call with `try` `await` and wrap it in a `Task { .. }` so that it's compatible with the playground.
- You're now ready to check whether the `data` function has completed with valid data. Create a string from the data that will display the data's contents, then print that string to the console.

  - ```swift
      import UIKit

      var urlComponents = URLComponents(string: "https://itunes.apple.com/search")!

      urlComponents.queryItems = [
          "term": "jack+johnson",
          "limit": "1"
      ].map { URLQueryItem(name: $0.key, value: $0.value) }

      Task {
          let (data, response) = try await URLSession.shared.data(from: urlComponents.url!)
          
          if let httpResponse = response as? HTTPURLResponse,
            httpResponse.statusCode == 200,
            let string = String(data: data, encoding: .utf8) {
              print(string)
          }
      }
    ```

- Excellent work. You've built a simple script that creates a proper URL using `URLComponents` from a query dictionary and fetches that data from a new API you haven't seen or worked with before. Be sure to save the playground to your project folder for future reference.

### Lesson 2.5 Working with the Web: Decoding JSON

- In the last lesson, you learned how to create and send a network request using the `URLSession` class. You also learned how URLs are constructed and how they can communicate a request to a web server. You requested data from the NASA Astronomy Picture of the Day (APOD) API, and you downloaded and printed the server's response.
- But what format did the response come in? Many modern web services return data in the JavaScript Object Notation (JSON) format.
- In this lesson, you'll learn how to read basic JSON. You'll also learn how to convert JSON into Swift types and your own custom model objects by using the `Codable` protocol and a `JSONDecoder`.
- You know how to pull data from the web, and you know that data is often returned in the form of JavaScript Object Notation, usually called JSON. But what is JSON? JSON defines the way objects are stored and structured in JavaScript, a popular language for building web services. JSON is the de facto format for passing information around the web.

#### JSON Basics

- Working with JSON data is very similar to working with basic Swift types, like numbers, strings, Booleans, dictionaries, and arrays. For example, here's how JSON might represent a Person object:

  - ```json
      {
          "name": "Daren Estrada",
          "favorite_show": {
              "title": "The Morning Show",
              "release_year": 2019
          }
      }
    ```

- How does this JSON map to a Swift type? Let's break down how the JSON data is presented:
  - Curly braces `{ }` encapsulate a dictionary of keys and values.
  - Square brackets `[ ]` encapsulate an array.
  - Text inside quotation marks `" "` is a string value.
  - Numbers and Booleans are written without quotation marks.
- JSON data can have several nested layers — which, at first glance, may look quite complex. But if you examine the data one layer at a time, you can identify each individual part.
- Look at the first layer from the previous example:

  - ```json
      {
          "name": "Daren Estrada",
          "favorite_show": {...} 
      }
    ```

- Does this pattern look familiar? You'll probably notice that the data is stored in a key-value pair, just like in a dictionary. Here you see a dictionary of type [String: Any], where the value for the key "name" is a string, but the value for the key "favorite_show" is another dictionary. Because the two values are different types, you'd consider the values to be of type Any.
- What would an array of objects look like in JSON? Check out another representation of JSON data:

  - ```json
      {
          "tvShows": [
              {
                  "title": "The Morning Show",
                  "release_year": 2019
              },
              {
                  "title": "Defending Jacob",
                  "release_year": 2020
              }
          ]
      }
    ```

- You'll notice that each show's data is structured as a [String: Any] dictionary. The next layer of data (the value for the key tvShows) contains an array in JSON. What type is the array? Consider the type of objects it contains—an array of dictionaries that have String keys and String or Int values (which can both be represented by Any)—or, in Swift, [[String: Any]]

#### Decoding into Swift Types

- Remember the NASA APOD API you worked with in the last lesson? Now that you can read JSON, you'll do some more work with the JSON data that the API returned.
- Go ahead and open your “Working with the Web” playground. It should have a network request, directed at the NASA APOD API, that fetches information about a picture. Take a look at the following example and break down the parts of the JSON:

  - ```json
      {
        "date": "2017-02-13",
        "explanation": "Juno just completed its fourth pass near Jupiter...",
        "hdurl": "https://apod.nasa.gov/apod/image/1702/JupiterSouth_JunoPeach_1200.jpg",
        "media_type": "image",
        "service_version": "v1",
        "title": "Cloud Swirls around Southern Jupiter from Juno",
        "url": "https://apod.nasa.gov/apod/image/1702/JupiterSouth_JunoPeach_960.jpg"
      }
    ```

- The JSON is a simple [String: String] dictionary that is only one level deep, making it the perfect practice ground for decoding JSON data into standard Swift types or your own custom model objects. 
- In the following steps, you'll update the code to use a JSONDecoder to convert the response to native Swift types.

#### Convert JSON Data to Swift Types

- Instances of `JSONDecoder` have a `decode(_:from:)` method that accepts a parameter of type `Type` and a parameter of type `Data`. You can pass in a class type by using the class name followed by `.self`. For example, passing in a string type would look like `String.self`. The method then attempts to decode the `Data` object passed into the method into the desired type. For the method to work, the type that you've passed in must adopt the `Codable` protocol, and the `Data` must be in a format that can be decoded into that `Codable` type. If either of these requirements is not met, then your code will not compile.
- `JSONDecoder`'s `decode(_:from:)` method is a throwing function, which you have encountered before. Remember that by pairing a throwing function with the do-try-catch syntax, you can capture and address each specific error that the method can throw. More simply, if you precede a throwing function with `try?`, the function will return an optional value. If there's no error, the optional will hold the expected value; if there is an error, the optional will be `nil`.
- In your playground, use a `JSONDecoder` to decode the data returned to you from the API into a [String: String] dictionary. Since String already adopts the `Codable` protocol, you do not need to do anything special involving `Codable`. Here's how your code should look:

  - ```swift
      Task {
          let (data, response) = try await URLSession.shared.data(from: urlComponents.url!)
       
          let jsonDecoder = JSONDecoder()
          if let httpResponse = response as? HTTPURLResponse,
              httpResponse.statusCode == 200,
              let photoDictionary = try? jsonDecoder.decode([String: String].self, from: data) {
              print(photoDictionary)
          }
      }

      /* Console Output: */
      ["media_type": "image", "hdurl": "https://apod.nasa.gov/apod/image/1307/moon_zond8_1735.jpg", "date": "2013-07-16", "url": "https://apod.nasa.gov/apod/image/1307/moon_zond8_960.jpg", "title": "The Moon from Zond 8", "copyright": "\nGalspace\n", "explanation": "Which moon is this? Earth\'s. Our Moon\'s unfamiliar appearance is due partly to an unfamiliar viewing angle as captured by a little-known spacecraft -- the Soviet Union\'s Zond 8 that circled the Moon in October of 1970. Pictured above, the dark-centered circular feature that stands out near the top of the image is Mare Orientale, a massive impact basin formed by an ancient collision with an asteroid. Mare Orientale is surrounded by light colored and highly textured highlands. Across the image bottom lies the dark and expansive Oceanus Procellarum, the largest of the dark (but dry) maria that dominate the side of the Moon that always faces toward the Earth. Originally designed to carry humans, robotic Zond 8 came within 1000 km of the lunar surface, took about 100 detailed photographs on film, and returned them safely to Earth within a week.   Follow APOD on: Facebook (Daily) (Sky) (Spanish) or Google Plus (Daily) (River)", "service_version": "v1"]
    ```

- Since String already adopts Codable, JSONDecoder has a mapping that it can follow to convert JSON data with string keys and string values into a Swift dictionary with keys of type String and values of type String. Your console should show output similar to what you saw before, but now surrounded by the square brackets indicating a Swift dictionary instead of JSON's curly braces.

#### Decoding into Custom Model Objects

- Now that you've learned how to decode JSON data into native Swift types, you can prepare to decode the data into your own custom model objects. Take a look at the example data again:

  - ```json
      {
        "date": "2017-02-13",
        "explanation": "Juno just completed its fourth pass near Jupiter...",
        "hdurl": "https://apod.nasa.gov/apod/image/1702/JupiterSouth_JunoPeach_1200.jpg",
        "media_type": "image",
        "service_version": "v1",
        "title": "Cloud Swirls around Southern Jupiter from Juno",
        "url": "https://apod.nasa.gov/apod/image/1702/JupiterSouth_JunoPeach_960.jpg"
      }
    ```

- How might you represent this JSON data in a custom model object? You can decide what properties your model object will have and what type each property will need to be.
- For this example, you'll create a structure called `PhotoInfo` that will have `title`, `description`, `url`, and optional `copyright` properties. Your structure will look something like this:

  - ```swift
      struct PhotoInfo {
          var title: String
          var description: String
          var url: URL
          var copyright: String?
      }
    ```

- When deciding if a property on your model object should be required or optional, consider how the API response may change. In this example, the NASA APOD API sometimes includes a copyright key, but not always. When the returned keys may or may not exist, those values should be optional on your type.
- You'll use this model object just like you've done in previous projects: to represent the information you want to manage or display in your app.
- **Initialize Model Objects Using `Codable`**
  - Take a moment to think about what you're working with here. After performing the network request and decoding the JSON data into native Swift types, you have a [String: String] dictionary. Now you'll want to similarly convert the data into a `PhotoInfo` object that you can use throughout your project. How might you approach this problem?
  - The `Codable` protocol has two requirements that define a set of rules that can be applied to a class or structure to easily convert data from one type to another. (This protocol is actually a simple combination of the `Encodable` and `Decodable` protocols, which have one requirement each.) In this case, the `JSONDecoder` class can be used to translate information from JSON into your custom model objects.
  - By declaring that your type adopts `Codable`, you allow the compiler to autogenerate implementations for these two protocol requirements. By default, encoders and decoders work with `Codable` types using the two autogenerated methods. The first is `init(from:)`, a requirement of the `Decodable` protocol, which is used to initialize the new type of data from the existing data. The second is `encode(to:)`, a requirement of the `Encodable` protocol, which is “used to convert the current type to a new type of data.
  - The autogenerated `Codable` protocol method implementations match the property names and values in your type with the keys and values of the encoded type. The APOD JSON data contains key/value pairs for some data that `PhotoInfo` does not use (date, service version, and media type); what happens with those? By default, decoders that work with `Codable` objects will simply ignore keys that do not have a property counterpart. On the other hand, if you have properties on your model that are not in the JSON data, you'll need to either provide default values for them or make them optional—in which case they'll be set to `nil`.
  - Also, your `PhotoInfo` type has a property with a different name than the corresponding key in the JSON data (`description` versus `explanation`), which will require you to implement custom keys. You could do this by creating custom implementations of the required protocol methods, but there's an easier approach that will cover many use cases. To map non-matching keys to your properties, you can declare a nested enumeration named `CodingKeys` that adopts the `CodingKey` protocol. This provides enough information for the compiler to autogenerate methods that account for the differences in key names between your type and the data associated with it.
  - For example, the following code defines a `Report` struct that adopts the `Codable` protocol. By default, the encoded data would use the property names as the keys — but because a `CodingKeys` enumeration is defined, encoding and decoding would use the keys defined there.

    - ```swift
        struct Report: Codable {
            let creationDate: Date
            let profileID: String
            let readCount: Int
         
            enum CodingKeys: String, CodingKey {
                case creationDate = "report_date"
                case profileID = "profile_id"
                case readCount = "read_count"
            }
        }
      ```

  - You'll need to declare a `CodingKeys` enumeration inside `PhotoInfo` to map non-matching keys to your properties. Each case should correspond to a property and have the same name as the property. For cases where the property name and the name of the key in the data are different, you'll assign an associated String with a value matching the key name in the data. For example, `PhotoInfo` has a property called `description` that should map to the NASA APOD API explanation key, so your enum should include a case for `description` that is set equal to `"explanation"`. The keys with matching names do not need an associated value.
  - Also, be sure to make `PhotoInfo` adopt `Codable`. When finished, your structure should look like the following:

    - ```swift
        struct PhotoInfo: Codable {
            var title: String
            var description: String
            var url: URL
            var copyright: String?
         
            enum CodingKeys: String, CodingKey {
                case title
                case description = "explanation"
                case url
                case copyright
            }
        }
      ```

  - Now that you've provided the compiler with the information it needs, it will handle decoding the JSON data into your `PhotoInfo` type.
- **Update the Request Completion Handler**
  - Your PhotoInfo type is now ready to be decoded by `JSONDecoder`. Next, you'll need to update the completion handler in your network request so that it uses the new type instead of printing the json value. Add a line to your if-let check to initialize an instance of `PhotoInfo`, then print the value:

    - ```swift
        Task {
            let (data, response) = try await URLSession.shared.data(from:
              urlComponents.url!)
         
            let jsonDecoder = JSONDecoder()
            if let httpResponse = response as? HTTPURLResponse,
                httpResponse.statusCode == 200,
                let photoInfo = try? jsonDecoder.decode(PhotoInfo.self,
                  from: data) {
                print(photoInfo)
            }
        }

        /* Console Result */
        PhotoInfo(title: "The Moon from Zond 8", description: "Which moon is this? Earth\'s. Our Moon\'s unfamiliar appearance is due partly to an unfamiliar viewing angle as captured by a little-known spacecraft -- the Soviet Union\'s Zond 8 that circled the Moon in October of 1970. Pictured above, the dark-centered circular feature that stands out near the top of the image is Mare Orientale, a massive impact basin formed by an ancient collision with an asteroid. Mare Orientale is surrounded by light colored and highly textured highlands. Across the image bottom lies the dark and expansive Oceanus Procellarum, the largest of the dark (but dry) maria that dominate the side of the Moon that always faces toward the Earth. Originally designed to carry humans, robotic Zond 8 came within 1000 km of the lunar surface, took about 100 detailed photographs on film, and returned them safely to Earth within a week.   Follow APOD on: Facebook (Daily) (Sky) (Spanish) or Google Plus (Daily) (River)", url: https://apod.nasa.gov/apod/image/1307/moon_zond8_960.jpg, copyright: Optional("\nGalspace\n"))
      ```

  - JSON data can be tricky, and this lesson is only the tip of the iceberg. As you start working with other web services and APIs, you'll encounter more complex, nested JSON data with multiple types. It's not uncommon to reach the point where you expect everything to work — but find that your custom model object refuses to decode correctly. In those instances, here are some things you might try:
    - Check the JSON structure to make sure you're writing the correct code.
    - If using one, check your `CodingKeys` enumeration. The keys have to match exactly. Decoding URLs is particularly tricky, because some APIs use "url" and others use "URL". Some prefix the URL with a word or other key, and then use different capitalization, such as "hdUrl", "hdURL", or "hd_url". Always double-check your keys.
    - Check your network connection. You won't get the correct response if you aren't connected to the internet.
    - Check your URL and verify the response body contains JSON in the format you are expecting.

#### Where To Put Your Code

- If you've reached this point, your playground has code that creates and sends a network request to the NASA APOD API. When the request is complete, your code decodes the JSON into your custom model object, and finally prints the model object.
- If you were to move this code into an Xcode project, where would you put it? You'd likely put the model object code into a `PhotoInfo.swift` file. But what about the networking code?
- This is one place where coding is an art as much as it is a science. There are many different styles and patterns you could follow, but the bottom line is this: Put the code where it makes sense in your project. For a simple project, the right place might be in the view controller that will display the data. For a somewhat more complex but not-too-large project, you might put it in a static method on your model object itself. In a very complex project, it might be in a model controller type for fetching and managing the model data. There's no one right answer.
- Later in this lesson, you'll see examples of these three strategies. But there's a more basic issue with your code's organization that you'll take care of first.
- **Move Your Networking Code to a Function**
  - At the moment, you're using a playground to create and run the network request, which means the network request runs as top-level code — code that's executed automatically when the playground runs. Take a look at your playground, and you'll see that the networking code is not part of a function. This isn't very conducive to being executed repeatedly in a project.
  - How would you translate the networking code in your playground into a function that fetched a `PhotoInfo` object? What would you name the function? What would it return? How would the function work with this asynchronous approach to running a network request?
  - Start by creating a new function called `fetchPhotoInfo()`, with no parameters, that returns a `PhotoInfo` object. Move all your networking - related code into the function, including the `baseURL`, `query`, `url`, `task`, and `task.resume` lines of code. (Note that the “date” key has been left out of the query in this implementation. It's an optional parameter; the API will use today's date if it's not included.)

    - ```swift
        func fetchPhotoInfo() -> PhotoInfo {
            var urlComponents = URLComponents(string: "https://api.nasa.gov/planetary/apod")!
            urlComponents.queryItems = [
                "api_key": "DEMO_KEY"
            ].map { URLQueryItem(name: $0.key, value: $0.value) }
         
            let (data, response) = try await URLSession.shared.data(from: urlComponents.url!)
         
            let jsonDecoder = JSONDecoder()
            if let httpResponse = response as? HTTPURLResponse,
                httpResponse.statusCode == 200,
                let photoInfo = try? jsonDecoder.decode(PhotoInfo.self, from: data) {
                print(photoInfo)
            }
        }
      ```

  - You'll note a couple of compiler error saying something like the following: `'async' call in a function that does not support concurrency Errors thrown from here are not handled`
  - The compiler offers a fix-it for the first issue that will add async to the function definition. You can do this manually or click the Fix button to fix the error. Your function definition will now look like: `func fetchPhotoInfo() async -> PhotoInfo {`
  - The second issue can be addressed by adding the `throws` keyword to the function definition so that any errors thrown within your function will be propagated to the caller. The `throws` keyword always follows the `async` keyword. Your function definition should now look like: `func fetchPhotoInfo() async throws -> PhotoInfo {`
  - You'll note a new compiler error that says something like the following: `Missing return in a function expected to return PhotoInfo`
  - Where can you return the object? You'll have to return the PhotoInfo instance after the data has been fetched from the server and successfully decoded. Go ahead and update your code as follows, replacing the call to print with a return statement:

    - ```swift
        let jsonDecoder = JSONDecoder()
        if let httpResponse = response as? HTTPURLResponse,
            httpResponse.statusCode == 200,
            let photoInfo = try? jsonDecoder.decode(PhotoInfo.self, from: data) {
            return photoInfo
        }
      ```

  - Did that solve the problem? `Missing return in a function expected to return PhotoInfo`
  - Well, this is getting tricky. The problem is that the `PhotoInfo` instance is only returned when the data is successfully fetched from the server and successfully decoded. The function signature guarantees a returned `PhotoInfo` instance — unless it throws an error. You'll need to throw an error if you weren't able to decode a `PhotoInfo` from the server response.
  - To create a custom error, define an enum that adopts the `Error` and `LocalizedError` protocols, then add cases for the error conditions you need to report. Update your code to add your own error:

    - ```swift
        enum PhotoInfoError: Error, LocalizedError {
            case itemNotFound
        }
      ```

  - The `JSONDecoder` will already throw an error if it's not able to decode the data, so you can rearrange your code to throw the new error if the `statusCode` from the `URLSession` is not 200 before decoding the JSON and returning the result:

    - ```swift
        func fetchPhotoInfo() async throws -> PhotoInfo {
            var urlComponents = URLComponents(string: "https://api.nasa.gov/planetary/apod")!
            urlComponents.queryItems = [
                "api_key": "DEMO_KEY"
            ].map { URLQueryItem(name: $0.key, value: $0.value) }
         
            let (data, response) = try await URLSession.shared.data(from: urlComponents.url!)
         
            guard let httpResponse = response as? HTTPURLResponse,
                  httpResponse.statusCode == 200 else {
                throw PhotoInfoError.itemNotFound
            }
         
            let jsonDecoder = JSONDecoder()
            let photoInfo = try jsonDecoder.decode(PhotoInfo.self, from: data)
            return(photoInfo)
        }
      ```

  - You can now call your function to verify its operation. Since your function is defined as `throws` and `async`, you will need to `try` and `await` the call. If the call is successful, you'll want to print the results along with a message indicating success. You'll also want to handle any errors that result from calling your function. To handle errors from a throwing function, wrap the call in a do/catch statement. Finally, since the playground is running synchronously, you will need to embed the call in a `Task { ... }`:

    - ```swift
        Task {
            do {
                let photoInfo = try await fetchPhotoInfo()
                print("Successfully fetched PhotoInfo: \(photoInfo)")
            } catch {
                print("Fetch PhotoInfo failed with error: \(error)")
            }
        }

  - This example highlights the power of Swift's concurrency model. You've implemented a function that asynchronously accesses a network resource, and yet the resulting code looks very much like it would if it had been synchronous code. You used the standard Swift error handling model to propagate errors to the caller and were able to let the compiler help ensure that your implementation of the function met the contract that the function definition offers.
  - Great job! You've now abstracted the network call into a function that will access the fetched `PhotoInfo` instance when the request is completed. Your console output should begin with `Successfully fetched PhotoInfo:`.

  - ```swift
      import UIKit

      struct PhotoInfo: Codable {
          var title: String
          var description: String
          var url: URL
          var copyright: String?
          
          enum CodingKeys: String, CodingKey {
              case title
              case description = "explanation"
              case url
              case copyright
          }
      }

      enum PhotoInfoError: Error, LocalizedError {
          case itemNotFound
      }

      func fetchPhotoInfo() async throws -> PhotoInfo {
          var urlComponents = URLComponents(string: "https://api.nasa.gov/planetary/apod")!
          urlComponents.queryItems = [
              "api_key": "DEMO_KEY",
              "date": "2013-07-16"
          ].map { URLQueryItem(name: $0.key, value: $0.value) }
          
          let (data, response) = try await URLSession.shared.data(from: urlComponents.url!)
          
          guard let httpResponse = response as? HTTPURLResponse,
                httpResponse.statusCode == 200 else {
              throw PhotoInfoError.itemNotFound
          }
          
          let jsonDecoder = JSONDecoder()
          let photoInfo = try jsonDecoder.decode(PhotoInfo.self, from: data)
          return photoInfo
      }

      Task {
          do {
              let photoInfo = try await fetchPhotoInfo()
              print("Successfully fetched PhotoInfo: \(photoInfo)")
          } catch {
              print("Fetch PhotoInfo failed with error: \(error)")
          }
      }
    ```

#### Decide Where the Function Should Live

- Now that your network call is nicely organized in a function, you can return to the question of where it should be written in your Xcode projects. Should it go on the view controller that performs the network request? Should it be a static function on the `PhotoInfo` type? Should you build a model controller that performs the fetch for you?
- All three are valid options, depending on the scope of your project. Take a look at how each approach works:
- **Add to the View Controller**

  - ```swift
      class PhotoViewController: UIViewController {
       
          override func viewDidLoad() {
              super.viewDidLoad()
       
              Task {
                  do {
                      let photoInfo = try await fetchPhotoInfo()
                      updateUI(with: photoInfo)
                  } catch {
                      displayError(error)
                  }
              }
          }
       
          func updateUI(with photoInfo: PhotoInfo) {...}
          func displayError(_ error: Error) {...}
          func fetchPhotoInfo() async throws -> PhotoInfo {...}
      }
    ```

- **Static Function on the PhotoInfo Type**

  - ```swift
      extension PhotoInfo {
          static func fetchPhotoInfo() async throws -> PhotoInfo {...}
      }
      class PhotoViewController: UIViewController {
       
          override func viewDidLoad() {
              super.viewDidLoad()
       
              Task {
                  do {
                      let photoInfo = try await PhotoInfo.fetchPhotoInfo()
                      updateUI(with: photoInfo)
                  } catch {
                      displayError(error)
                  }
              }
          }
       
          func updateUI(with photoInfo: PhotoInfo) {...}
          func displayError(_ error: Error) {...}
      }
    ```

- **Build a Model Controller**

  - ```swift
      class PhotoInfoController {
          func fetchPhotoInfo() async throws -> PhotoInfo {...}
      }
      class PhotoViewController: UIViewController {
          let photoInfoController = PhotoInfoController()
       
          override func viewDidLoad() {
              super.viewDidLoad()
       
              Task {
                  do {
                      let photoInfo = try await
                        photoInfoController.fetchPhotoInfo()
                      updateUI(with: photoInfo)
                  } catch {
                      displayError(error)
                  }
              }
          }
       
          func updateUI(with photoInfo: PhotoInfo) {...}
          func displayError(_ error: Error) {...}
      }
    ```

- As you learn more about writing networking code and architecting your apps, you'll run into the various pros and cons of each of these patterns. At this stage, however, you can use any of them in your code.

#### Wrap-Up Working with the Web: Decoding JSON

- Excellent work! Decoding JSON into custom model objects isn't easy, but you worked through it. You'll continue building on your skills in the next lesson, where you'll put together a full project and update the user interface with data fetched from an API.

#### Lab - iTunes Search (Part 2)

- The purpose of this lab is to get familiar with decoding JSON data into your custom model objects. You'll start with the “iTunes Search” playground you created in the last lesson. Your task with this lab will be to decode the data you retrieved into a custom model object.

##### Step 1 Create Your Model Object

- Set the "term" in your queryItems to "Apple" to return more than one result. Run the playground and look at the printed results in the console. You can use the following Data extension method to make the printed JSON easier to read.

  - ```swift
      extension Data {
          func prettyPrintedJSONString() {
              guard
                  let jsonObject = try? JSONSerialization.jsonObject(with: self, options: []),
                  let jsonData = try? JSONSerialization.data(withJSONObject: jsonObject, options: [.prettyPrinted]),
                  let prettyJSONString = String(data: jsonData, encoding: .utf8) else {
                      print("Failed to read JSON Object.")
                      return
              }
              print(prettyJSONString)
          }
      }
       
      ...
       
      Task {
          let (data, response) = try await URLSession.shared.data(from: urlComponents.url!)

          if let httpResponse = response as? HTTPURLResponse,
            httpResponse.statusCode == 200 {
              data.prettyPrintedJSONString()
          }
      }
    ```

- Notice that there's a `results` key with an array of objects. You're going to create a struct model for these objects, so take a moment to identify what properties your model will have. Some of the properties might include `trackName`, `artistName`, `kind`, `description`, and the URL for the artwork.
- Once you've identified the properties that will make up your object, create a `StoreItem` structure with those properties. Make your `StoreItem` adopt `Codable`, and use a `CodingKeys` enum to map the JSON keys to better-suited property names on your model.
- You might have noticed that an item can have different types of descriptions. Some items have a description key, while others have a `longDescription` key. Your model object makes no distinction between the two. If the API sends down a value with the description key, that is what you should assign to your model. However, if it sends down a value with the longDescription key and nothing with the description key, you should use that instead. If neither value exists, you'll just assign description to an empty string.
- ​​To do this, you will need a `CodingKey` for each. But the Encodable protocol doesn't allow you to add cases to CodingKeys that don't match one of your properties, so you can't have a longDescription case. To get around this, create a second nested enumeration called `AdditionalKeys` that adopts `CodingKey` and has a case for `longDescription`. Unfortunately, using this strategy will prevent the compiler from autogenerating Codable method requirements. You'll need to create your own initializer that assigns each value from the JSON payload to its respective property. You can use `try?` when decoding the value for `description`, and if it fails you can decode the value associated with `longDescription` and assign it to the `description` property.​
- The manual decoding process is relatively straightforward. You start by creating a container of values keyed by your `CodingKeys` enum using the `container(keyedBy:)` method on Decoder. Since you'll also have the second nested enum, `AdditionalKeys`, you'll create a second container keyed by that enum when the value for `description` is `nil`. Take a look at the following `init(from:)` implementation:

  - ```swift
      init(from decoder: Decoder) throws {
          let values = try decoder.container(keyedBy: CodingKeys.self)
          name = try values.decode(String.self, forKey: CodingKeys.name)
          artist = try values.decode(String.self, forKey: CodingKeys.artist)
          kind = try values.decode(String.self, forKey: CodingKeys.kind)
          artworkURL = try values.decode(URL.self, forKey: CodingKeys.artworkURL)
       
          if let description = try? values.decode(String.self, forKey: CodingKeys.description) {
              self.description = description
          } else {
              let additionalValues = try decoder.container(keyedBy: AdditionalKeys.self)
              description = (try? additionalValues.decode(String.self, forKey: AdditionalKeys.longDescription)) ?? ""
          }
      }​
    ```

- The `values` constant is assigned to the returned container keyed by `CodingKeys`, then each model property is assigned to its respective decoded value. Each one of these calls can `throw` if the type does not match or the key does not exist in the encoded value. Finally, an attempt is made to decode `description` using the `values` container; if that fails, a new container `additionalValues` is created, keyed by `AdditionalKeys`. The `longDescription` case is used to decode a value, and if that also fails the `description` property is set to an empty string.
- Now that you have a `StoreItem` model, think about how you'll get values to create model instances from the JSON response. Remember, they're nested in a results key. What you're after is a [StoreItem] — an array of your models. What you might have come up with is a separate model for the search response itself, perhaps `SearchResponse`. This would be a simple struct only requiring one property, results:​

  - ```swift
      struct SearchResponse: Codable {
          let results: [StoreItem]
      }
    ```

##### Step 2 Create a Function to Fetch Items

- Now that you have a `SearchResponse` object, you're ready to fetch the data that you'll decode into your `StoreItem` models.
  - Create a new `fetchItems` function that takes a query dictionary and returns an array of `StoreItems`. The function should be marked as `async` and `throws`.
  - `func fetchItems(matching query: [String: String]) async throws -> [StoreItem]`
- The function should create a URL with the queries and issue a new request with the URL. When the request is complete, the function should check the status code from the request and throw an error if it's not equal to 200. Then it should create a `JSONDecoder`, unwrap the data, and decode it into a `SearchResponse` object. Finally, it should return the `results` from `SearchResponse`.
- Follow these steps to implement the `fetchItems` function:
  - Move your `urlComponents` variable inside the function.
  - Create an enum that adopts the `Error` and `LocalizedError` protocols with at least one case.
  - Set the `queryItems` from the passed in `query` parameter.
  - Call the `data` method on the shared URL session and assign the results to a tuple.
  - Check the status code of the request and throw an error if it's not equal to 200.

##### Step 3 Decode the JSON

- A sample solution is provided at the end of this lab's instructions. Use the sample code for the task's completion handler as a reference if needed, but resist copying and pasting it. You'll learn much more if you try to solve the problem independently and then, if needed, read the code, analyze it, and understand it — and only then type it into your project.
  - Inside the task's closure, check for an `error`. If one is present, call the completion handler with the error.
  - Check whether you've received valid data back from the API. If you have, try and decode it into a `SearchResponse` object.
  - If you successfully unwrapped the data and decoded the `SearchResponse` object, pass the results property on SearchResponse through the completion handler. Remember that the results property on SearchResponse has the array of individual StoreItem objects.
  - If the JSON fails to decode properly, catch the error and call the completion handler with the error.

##### Step 4 Call Your Function

- With the `StoreItem` struct, the intermediary `SearchResponse` struct, the `fetchItems` function, and your search query, you're ready to call your function and create your model objects.
  - Call the `fetchItems` function and pass in your query. Check for errors and print the decoded [StoreItem] array in a do/catch statement. Remember to use a Task to wrap your function call.
- You did it! You've fetched data and decoded it into your own custom model object. You'll use code just like this for fetching data to display in your apps. Be sure to save your playground to your project folder for future reference.

  - ```swift
      import UIKit

      extension Data {
          func prettyPrintedJSONString() {
              guard
                  let jsonObject = try? JSONSerialization.jsonObject(with: self, options: []),
                  let jsonData = try? JSONSerialization.data(withJSONObject: jsonObject, options: [.prettyPrinted]),
                  let prettyJSONString = String(data: jsonData, encoding: .utf8) else {
                  print("Failed to read JSON Object.")
                  return
              }
              print(prettyJSONString)
          }
      }

      struct StoreItem: Codable {
          var name: String
          var artist: String
          var kind: String
          var description: String
          var artworkURL: URL
          
          init(from decoder: Decoder) throws {
              let values = try decoder.container(keyedBy: CodingKeys.self)
              
              name = (try? values.decode(String.self, forKey: CodingKeys.name)) ?? "** No Name **"
              artist = (try? values.decode(String.self, forKey: CodingKeys.artist)) ?? "** No Artist Name **"
              kind = (try? values.decode(String.self, forKey: CodingKeys.kind)) ?? "** No Kind **"
              artworkURL = (try? values.decode(URL.self, forKey: CodingKeys.artworkURL)) ?? URL(string: "https://itunes.apple.com")!

              if let description = try? values.decode(String.self, forKey: CodingKeys.description) {
                  self.description = description
              } else {
                  let additionalValues = try decoder.container(keyedBy: AdditionalKeys.self)
                  description = (try? additionalValues.decode(String.self, forKey: AdditionalKeys.longDescription)) ?? "** No Description **"
              }
          }
          enum  CodingKeys: String, CodingKey {
              case name = "trackName"
              case artist = "artistName"
              case kind
              case description
              case artworkURL = "artworkUrl100"
          }
          
          enum AdditionalKeys: String, CodingKey {
              case longDescription
          }
      }

      enum StoreItemError: Error, LocalizedError {
          case itemsNotFound
      }
      struct SearchResponse: Codable {
          let results: [StoreItem]
      }

      func fetchItems(matching query: [String: String]) async throws -> [StoreItem] {
          var urlComponents = URLComponents(string: "https://itunes.apple.com/search")!
          urlComponents.queryItems = query.map { URLQueryItem(name: $0.key, value: $0.value) }
          
          let (data, response) = try await URLSession.shared.data(from: urlComponents.url!)
          
          guard let httpResponse = response as? HTTPURLResponse,
            httpResponse.statusCode == 200 else {
              throw StoreItemError.itemsNotFound
          }
          
      //    print(data.prettyPrintedJSONString())
          let jsonDecoder = JSONDecoder()
          let searchResponse = try jsonDecoder.decode(SearchResponse.self, from: data)
          return searchResponse.results
      }

      let query = [
          "term": "Apple",
          "limit": "4",
      //    "media": "ebook",
      //    "attribute": "authorTerm",
      //    "lang": "en_us"
      ]

      Task {
          do {
              let storeItem = try await fetchItems(matching: query)
              print("Successfully fetched StoreItem: \n")
              storeItem.forEach {
                  print("""
                  Name: \($0.name)
                  Artist: \($0.artist)
                  Kind: \($0.kind)
                  Description: \($0.description)
                  Artwork URL: \($0.artworkURL)

                  """)
              }
          } catch {
              print("Fetch StoreItem failed with error: \(error)")
          }
      }
    ```

### Lesson 2.6 Working with the Web: Concurrency

- In the previous lesson, you learned how to decode JSON into native Swift types and custom model objects, how to write your own completion handler to handle asynchronous code, and a little bit about how you might approach adding the code to an Xcode project.
- In this lesson, you'll take the data you receive from a network request, decode it, and display it in your own app. You'll also download and set your first image for display. To make this all happen, you'll learn about the concurrency system in iOS and how to make sure code that updates the user interface is executed in the right place.
- As you went through the previous two lessons, you used asynchronous programming techniques to handle network requests. You used `await` to call an `async` method, and wrapped your asynchronous code in a `Task`. You also created your own asynchronous throwing method.
- In this lesson, you'll build an app that pulls data from the NASA Astronomy Picture of the Day (APOD) API and displays the information and the photo in the app. As you build this app, you'll encounter a couple of new issues related to running asynchronous requests, and you'll learn how to address them.

#### Get Started

- Create a new project called "SpacePhoto" using the iOS App template. When creating the project, make sure the Interface option is set to Storyboard. Open the Main storyboard and use a stack view to display an image view and two labels vertically. The image view will display today's photo from the APOD API, the first label will display the description, and the second label will display any available copyright information.
  - <img src="./resources/space_photo_get_start.png" alt="Space Photo Get Start" width="400" />
- Embed the scene in a navigation controller. The title of the photo will be set as the title label in the navigation bar.
- You might consider adding the stack view to a scroll view to accommodate a potentially long photo description. Keep in mind, however, that the purpose of this lesson is to update the app's user interface with the results of a network request — so the particulars of the interface aren't important to the project.
- Add outlets for the image view and labels to your ViewController file.

#### Import Your Playground Code

- Go ahead and open the “Working with the Web” playground you built in the previous lessons, then migrate the code into the Xcode project. If you didn't complete the previous lesson, no worries. You can copy the code snippets in the instructions that follow.
- As you learned in the previous lesson, there are many ways to structure the networking code in your app. The sample code in this lesson will add the network request method to a `PhotoInfoController` structure.
- Create a PhotoInfo.swift file and copy and paste your PhotoInfo structure into the file:

  - ```swift
      struct PhotoInfo: Codable {
          var title: String
          var description: String
          var url: URL
          var copyright: String?
       
          enum CodingKeys: String, CodingKey {
              case title
              case description = "explanation"
              case url
              case copyright
          }
      }
    ```

- Create a `PhotoInfoController.swift` file and define a new `PhotoInfoController` class. Add your `PhotoInfoError` enum and your `fetchPhotoInfo()` method to it. The contents of your class should resemble the following:

  - ```swift
      enum PhotoInfoError: Error, LocalizedError {
          case itemNotFound
      }
       
      func fetchPhotoInfo() async throws -> PhotoInfo {
          var urlComponents = URLComponents(string: "https://api.nasa.gov/planetary/apod")!
          urlComponents.queryItems = [
              "api_key": "DEMO_KEY",
          ].map { URLQueryItem(name: $0.key, value: $0.value) }
       
          let (data, response) = try await URLSession.shared.data(from: urlComponents.url!)
       
          guard let httpResponse = response as? HTTPURLResponse,
                  httpResponse.statusCode == 200 else {
              throw PhotoInfoError.itemNotFound
          }
       
          let jsonDecoder = JSONDecoder()
          let photoInfo = try jsonDecoder.decode(PhotoInfo.self, from: data)
          return(photoInfo)
      }
    ```

#### Make The Network Request

- Update your ViewController to call the `fetchPhotoInfo()` method and update the `title` and `labels` with the result.
- You'll want to add a property for an instance of the `PhotoInfoController` that can be used to execute the request. Create the following property within the definition of ViewController: `let photoInfoController = PhotoInfoController()`
- Add the `fetchPhotoInfo()` method to `viewDidLoad()`. Use a do/catch statement to handle any errors, update the `title` of the view controller, and set values for your label outlets. It's best to clear out the text for the `descriptionLabel` and `copyrightLabel` before the network request runs. Otherwise, the labels will display the text that was set in Interface Builder. You can also use a system - provided image for the image view, to indicate that something will eventually be placed there or that something has failed. (You'll notice you don't yet have any image data. You'll get to that shortly.)

  - ```swift
      override func viewDidLoad() {
          super.viewDidLoad()
       
          title = ""
          imageView.image = UIImage(systemName: "photo.on.rectangle")
          descriptionLabel.text = ""
          copyrightLabel.text = ""
       
          do {
              let photoInfo = try await photoInfoController.fetchPhotoInfo()
              self.title = photoInfo.title
              self.descriptionLabel.text = photoInfo.description
              self.copyrightLabel.text = photoInfo.copyright
          } catch {
              self.title = "Error Fetching Photo"
              self.imageView.image = UIImage(systemName: "exclamationmark.octagon")
              self.descriptionLabel.text = error.localizedDescription
              self.copyrightLabel.text = ""
          }
      }
    ```

- The compiler will indicate `'async' call in a function that does not support concurrency` and offer an option to fix the issue: `Add 'async' to function 'viewDidLoad()' to make it asynchronous`. However, the fix won't work in this case, because `viewDidLoad()` is overridden from `UIViewController`, and the superclass method is not marked `async`. (Recall that asynchronous code can only be called from a function marked `async` or from a `task`.) You've already encountered this problem when using asynchronous code from a playground. The solution is the same: Use a task as a bridge to the asynchronous network code. Update your code to use a `Task` to wrap the asynchronous code.
- You'll notice that the above code updates the user interface within the `Task` closure. This is possible because a task guarantees that all of its code will execute in the context of the actor that created it — in this case, the `MainActor`.
- Here's the bottom line: You can expect the code you write in a task to suspend at `async` calls, but you can also assume that the code resumes in the same context. As a rule, you can safely update your UI from code within a task if it's running in a method of a `UIViewController` subclass. (If you're curious, look up `UIViewController` in the documentation. You'll see that its declaration reads `@MainActor class UIViewController : UIResponder`.)
- If you want, you can annotate your `ViewController` class with `@MainActor` to make this relationship explicit:

  - ```swift
      @MainActor
      class ViewController: UIViewController {
      ...
      }

- But since `UIViewController` is already annotated this way, your subclass inherits this property and there's no need for you to annotate it.
- Swift actors and `async` functions are the recommended technology for managing concurrent processing. Grand Central Dispatch is another iOS technology that can be used manage concurrency. While you won't be using Grand Central Dispatch in this course, it can be a useful tool and is worth understanding.

#### Grand Central Dispatch

- Threads manage the flow of executing code. In the early days of computing, all code ran on a single thread of execution. Modern operating systems such as iOS provide multiple threads to allow multiple flows of code to run simultaneously. (With today's multicore processors, threads can run on different cores, but even a single core can run multiple threads to provide the illusion of simultaneous execution.) iOS creates a main thread to run all user interface code.
- Grand Central Dispatch (GCD) is an iOS technology that lets you write asynchronous code by dispatching it to a system of queues, which execute the code on one or more threads managed by GCD, so you don't have to create and maintain the threads yourself. GCD provides one main queue, which runs on the main thread and always has the highest priority, and other background queues that execute at varying levels of priority. The background queues can be used to execute long-running operations that would otherwise clog or block the main queue, making the app appear frozen to the user.
- Older `URLSession` methods require a closure, which runs on a background queue when the request is complete, to process their results. If you used one of these older methods and needed to update user interface elements from that closure, you'd need to send your user interface code to the main queue, where it would be processed at the highest priority level. To do so, you'd use a DispatchQueue to run a block of code on the main queue. Here's what the code looks like:

  - ```swift
      DispatchQueue.main.async {
          // Code here will be executed on the main queue
      }
    ```

- Not to bad, right? The `DispatchQueue` class works with the different system queues. `main` refers to the main queue, and `async` tells the main queue to run the following block as soon as possible.
- Because you've been using `async` methods and tasks in your code, you haven't had to deal with these mechanics. But you may encounter code that doesn't use Swift concurrency, where you'll need to use Grand Central Dispatch.
- You've only scratched the surface of working with the concurrency tools in iOS. But knowing just a small amount enables you to work with code that runs asynchronously. You'll learn more about Swift concurrency, Grand Central Dispatch, and other iOS tools as you build more complex apps in the future.

#### Fetch And Display The Photo

- At this point, your app should correctly fetch data from the APOD API and display the information on your scene. Now, how will you fetch and display the photo?
- One approach to solving a problem like this is to trace a path from the information you have to the information you want. In this case, the path is quite short. Your `PhotoInfo` model object has a `url` property that points to the photo you want to display. So the solution would be to download the data, initialize a `UIImage`, and set it to the image view.
- Downloading the data for an image on the internet is exactly the same as downloading data for a website or from a web service. You'll start by using the shared URL session to initialize a data task pointed at the image's URL. Next, you'll initialize the `UIImage` and set the image view once the data is returned from the URL session.
- Before you write that code, you'll need to make a couple of changes to your view controller. Take a minute to consider the experience you want the user to have, because that objective will determine where you trigger each network call and where you update the user interface.
- Note that you'll need to execute two network requests when the view loads. One will fetch the photo information, and one will fetch the photo itself. In this case, you'll probably want to update the user interface only after you've successfully fetched both the information and the image. But you might organize your code differently if you prefer to display information from one request before the other request is finished.
- So what changes should you make to your code? There are many approaches that you could take, and part of being a software developer is identifying multiple solutions and determining which fits best in your project. One approach you might take is adding a new method to `PhotoInfoController` that fetches the image, then calling that method after you've received the image's URL from your `fetchPhotoInfo()` function.
- Before looking at the following sample code, try to refactor the view controller to fetch and set the image on your own. It can be challenging to write code without a reference, but the practice is worth it. If you aspire to develop apps independently, you'll need to be able to puzzle through this kind of problem.
- **Sample Code**
  - Here's what a new method to fetch the image in `PhotoInfoController` might look like:

    - ```swift
        import UIKit

        class PhotoInfoController {
            enum PhotoInfoError: Error, LocalizedError {
                case itemNotFound
                case imageDataMissing
            }
            
            func fetchPhotoInfo() async throws -> PhotoInfo {...}

            func fetchImage(from url: URL) async throws -> UIImage {
                let (data, response) = try await URLSession.shared.data(from: url)
             
                guard let httpResponse = response as? HTTPURLResponse, httpResponse.statusCode == 200 else {
                    throw PhotoInfoError.imageDataMissing
                }
             
                guard let image = UIImage(data: data) else {
                    throw PhotoInfoError.imageDataMissing
                }
             
                return image
            }
        }
      ```

  - Your refactored ViewController might look like the following:

    - ```swift
        override func viewDidLoad() {
            super.viewDidLoad()
         
            title = ""
            imageView.image = UIImage(systemName: "photo.on.rectangle")
            descriptionLabel.text = ""
            copyrightLabel.text = ""
         
            Task {
                do {
                    let photoInfo = try await photoInfoController.fetchPhotoInfo()
                    updateUI(with: photoInfo)
                } catch {
                    updateUI(with: error)
                }
            }
        }
         
        func updateUI(with photoInfo: PhotoInfo) {
            Task {
                do {
                    let image = try await photoInfoController.fetchImage(from: photoInfo.url)
                    title = photoInfo.title
                    imageView.image = image
                    descriptionLabel.text = photoInfo.description
                    copyrightLabel.text = photoInfo.copyright
                } catch {
                    updateUI(with: error)
                }
            }
        }
         
        func updateUI(with error: Error) {
            title = "Error Fetching Photo"
            imageView.image = UIImage(systemName: "exclamationmark.octagon")
            descriptionLabel.text = error.localizedDescription
            copyrightLabel.text = ""
        }
      ```

  - Now that you've made these changes, what would you expect to happen when you run the app? Check it out. If it doesn't work as expected, try to debug the issue using breakpoints and the debug console. Pay attention to any messages that may have printed to the debug console. (If you get a message about App Transport Security blocking a resource, read the next section.)

#### App Transport Security and the HTTP Protocol

- Depending on the image of the day, an error may have been printed to the console when you tried to execute a network request for the photo.
  - `App Transport Security has blocked a cleartext HTTP (http://) resource load since it is insecure. Temporary exceptions can be configured via your app's Info.plist file.`
- What's App Transport Security? App Transport Security, or ATS, improves user security and privacy by requiring apps to use secure network connections over HTTPS. Why haven't you seen this error in the last two lessons? Because you were working in playgrounds, and ATS doesn't prevent network requests in playground files. Even if that weren't the case, you accessed the Apple website and the NASA APOD API using URLs with the HTTPS protocol, so you would not have encountered this error.
- If you received this error, you'll notice that the image URL on the `PhotoInfo` object uses the HTTP protocol. So that's where ATS kicked in, blocking your request to an insecure URL. As the error indicates, you can update your app's Info file with a key that will grant you a temporary exemption.
- **Updating `Info` for HTTP Exemption**
  - Updating the `Info` file with the exemption will only work temporarily. Support for the exemption was originally scheduled to end in December 2016, but has been extended indefinitely. Because you'll need to use the exemption later, this section will briefly show how to add the exemption key. But when possible, such as in this project, use the solution in the following section to update insecure URLs to use the HTTPS protocol.
  - You've seen the `Info` file before; it's basically a dictionary of information about your app that is used when deploying your app or submitting it to the App Store. By including the ATS exemption key, you're saying that your app might access insecure URLs.
  - Open the `Info` file to view the default set of keys associated with a new iOS app. Hover over the top key, “Information Property List,” and a plus sign (+) will appear. Click it to begin adding a new key to the file.
  - To allow HTTP connections, you'll need to add the key `App Transport Security Settings`. As you start typing the key, the autocomplete menu should appear. Once it does, go ahead and press Return to add the key.
  - The value for App Transport Security Setting is another dictionary. Click the triangle next to the key name to expand the contents, then hover over the key name and click the plus sign (+).
  - “You'll need to add the key `Allow Arbitrary Loads` to the new dictionary. Start typing and let autocompletion finish for you. Press Return. The value for this key is a Boolean that you should set to `YES`.
  - Again, where possible you should always try to use a URL that supports HTTPS. But if you are working with a web API that simply does not support the more secure protocol, you can add this key to allow HTTP access.
- **Updating the URL to Use HTTPS**
  - In this project, the NASA APOD API does support fetching the image with the more secure HTTPS protocol, so you can simply update the `PhotoInfo` URL to use it.
  - How can you change the URL programmatically? Remember that the URLComponents class provides helpful tools for parsing and adding to URLs. You've used the queryItems property to safely encode query parameters on a URL.
  - In this example, you'll update the passed URL in the `fetchImage()` method on `PhotoInfoController` so that the scheme property is always "https". Here's how the beginning of the updated method should look:

    - ```swift
        func fetchImage(from url: URL) async throws -> UIImage {
            var urlComponents = URLComponents(url: url, resolvingAgainstBaseURL: true)
            urlComponents?.scheme = "https"
         
            let (data, response) = try await URLSession.shared.data(from: urlComponents!.url!)
         
            //...
        }
      ```

  - Load up your app in Simulator or on your testing device to see your code in action. At this point, your app should be working as expected. Your code will fetch the JSON data from the APOD API, decode the JSON into your PhotoInfo model object, fetch the accompanying photo, and update the user interface with all the data.
  - Take a moment to appreciate everything you've done. Working with network requests, decoding data, and loading images asynchronously are pretty advanced jobs for a new developer!
  - But wait. There's one small tweak that will make your app even better. If your code isn't working, read the next section about videos. Otherwise you can jump to the following section about using the network activity indicator.
- **A Note about Videos**
  - You may have noticed a `media_type` key in the JSON body when decoding the response body. Even though APOD is short for Astronomy Picture of the Day, NASA will occasionally include a video instead of an image. Embedding video in an app is beyond the scope of this lesson, but here's a quick approach to supporting a video response.
  - Start by updating your code to check the value of the `media_type` key. If today's post is a video, you could open the URL directly in a Safari view controller or you could use the `UIApplication.shared.open(url: URL, options: [String : Any], completionHandler: ((Bool) -> Void)?)` method.
  - If the API is returning a video today and you want to verify that your app works without making those changes, there's a workaround. You can change the query to another date that will return a photo instead of a video. To add a date query, use the format in the NASA API documentation.

#### Use the Network Activity Indicator

- As you've progressed through this course, you've learned that it's important for the user to know what's happening in your apps. You want your app to feel familiar — and that's a good reason to use the system views and controls whenever possible.
- Another thing that users expect is some sort of indication that the app is executing a network request and waiting on a response. If you're running a request that might take longer than a few seconds, you might consider adding a [UIActivityIndicatorView](https://developer.apple.com/documentation/uikit/uiactivityindicatorview) or [UIProgressView](https://developer.apple.com/documentation/uikit/uiprogressview) somewhere on your interface. For more information on how to design your app with these views, take a look at [Human Interface Guidelines: Progress Indicators](https://developer.apple.com/design/human-interface-guidelines/progress-indicators).

#### Wrap-Up (Concurrency)

- Congratulations! You've learned a lot about how to work with the web in your iOS projects. As you build more dynamic apps using network APIs, you can refer to these lessons and the accompanying labs as a resource. Browse the internet, and you'll find many open APIs to use as launching pads for experimentation with web service – enabled apps.

#### Lab - iTunes Search (Part 3)

- The objective of this lab is to integrate your iTunes Search network requests into an actual app and apply the lessons you've learned about concurrency to the project. You'll create an app that will allow the user to search for different media types and view the results in a table view. To improve the performance of the table view, you'll also learn how to update the size of the URL cache to temporarily save images.
- In your student resources folder, open the starter project called “iTunesSearch.” The app includes an initial view controller with a table view for listing search results, a search bar for entering a query, and a segmented control for narrowing the search results to a particular media type.
- The `StoreItemListTableViewController` class has placeholder properties and functions that handle the search bar and segmented control and display a list of items in the table view. Before continuing with the following steps, take a moment to review the project's code and storyboard to understand how the app is set up.

##### Step 1 Import Your Playground Code

- Open the “iTunes Search” playground you created in the previous two lessons.
- Create a `StoreItem.swift` file and copy your `StoreItem` structure definition into the file. Also copy your intermediary `SearchResponse` struct into this file, outside the declaration of StoreItem.
- Create a `StoreItemController.swift` file and define a new `StoreItemController` class. Copy your `fetchItems` function into the controller.

##### Step 2 Add the Request to the View Controller

- Remember that the `StoreItemListTableViewController` has already implemented the code for the segmented control and the search bar and for supplying data to the table view. But right now, it's set up to use an array of String instances. You'll update the view controller to use the `StoreItemController` to fetch items based on the media type selected in the segmented control and the query in the search bar. Once the items are returned, you'll reload the table view to display the results.
- When the user starts entering or editing text in the search bar, the app will display a Search button. What happens next? If you look at the bottom of the code, you'll notice that the search bar uses the delegate pattern to call the `fetchMatchingItems()` function when the user taps the Search button. The `fetchMatchingItems()` function resets `self.items` and reloads the table view to clear the old data. The function then captures the text from the search bar and the correct media type from the segmented control.
- You'll implement the rest of the `fetchMatchingItems()` function to set up a query dictionary and make the network request. Here's how to go about it:
  - Add a new `StoreItemController` property to the list view controller. You'll use this instance to run the network request to fetch the matching `StoreItem` objects.
  - Update the items array to the [StoreItem] type.
  - Update the `configure(cell: UITableViewCell, forItemAt indexPath: IndexPath)` function to set the cell's `name` to `item.name`, `artist` to `item.artist`, and `artworkImage` to `nil`.
  - In the `fetchMatchingItems()` function, within the `if !searchTerm.isEmpty` braces, set up a query dictionary, setting the `"term"` and `"media"` keys to their respective values. You might also want to use the `"limit"` key to limit the number of results or the `"lang": "en_us"` key-value pair to limit results to items in U.S. English.
  - Call the `fetchItems(matching:)` method on the `StoreItemController` instance, passing the query dictionary in a do/catch statement within a `Task`. In the case for success, set the returned [StoreItem] as the `self.items` property on the view controller and reload the table. In the case for failure, print the associated `Error` to the console.

##### Step 3 Review Your Progress

- At this stage, when the user types text into the search bar and taps the Search button, your app should trigger a network request and return an array of `StoreItem` objects. When your table view is reloaded, it should display the results.
- Run the app in Simulator to verify that it works as expected. If it doesn't, try to figure out why. Use breakpoints and the debugging console to find out whether the network request was executed and whether the completion handler is getting called.

##### Step 4 Display Images in the Cells

- In earlier projects, you used the `cellForRow` data source function to configure your table view cells. In this app, however, the configuration is more complex. You'll need to set the cell's `name`, `artist`, and `artworkImage` — but in this case, you don't have the image. You only have an image URL. So you'll need to fetch the image before you can set it on the image view.
- Because there's more work involved in configuring the cell, the code has been abstracted to a separate `configure(cell: UITableViewCell, forItemAt indexPath: IndexPath)` function. Right now, the function sets the cell's text `String` properties to display the name and artist of the `StoreItem`. You'll update it to download the media artwork and display the image.
- It's possible that your cell could be displayed before the image view has an image. The cell in the starter project has been configured to show the system image `"photo"` to act as a placeholder if the `artworkImage` is `nil` so that your cell will always have an image — if it were blank, your layout would look strange.
  - Use the shared `URLSession` to request the artwork image using the item's `artworkURL`. Then, upon success, unwrap the image data, initialize a `UIImage` from the data, and assign the image to the cell's `artworkImage` property. Consider adding a method inside `StoreItemController` to make the network request and return the result.

    - ```swift
        func fetchImage(from url: URL) async throws -> UIImage {
            var urlComponents = URLComponents(url: url, resolvingAgainstBaseURL: true)
            urlComponents?.scheme = "https"
            let (data, response) = try await URLSession.shared.data(from: urlComponents!.url!)
            
            guard let httpResponse = response as? HTTPURLResponse, httpResponse.statusCode == 200 else {
                throw StoreItemError.imageDataMissing
            }
            
            guard let image = UIImage(data: data) else {
                throw StoreItemError.imageDataMissing
            }
            
            return image
        }
      ```

- You'll notice that the starter project contains a property called `imageLoadTasks`. This property is a dictionary where the keys are `IndexPaths` and the values are of the type `Task<Void, Never>`. (This is syntax for a generic type, which you'll learn about later — for now, just consider it a kind of `Task`.) It's a good idea to cancel tasks that are no longer needed. For example, once a cell is scrolled off the screen there is no point fetching its associated artwork. The sample project implements an additional `TableViewDelegate` method that is called when a row in the table view will be moved offscreen. The method cancels any task that may still be pending to download the artwork:

  - ```swift
      override func tableView(_ tableView: UITableView, didEndDisplaying
        cell: UITableViewCell, forRowAt indexPath: IndexPath) {
          // Cancel the image fetching task if it's no longer needed
          imageLoadTasks[indexPath]?.cancel()
      }
    ```

- You can populate this dictionary when you start an image fetching task for a cell, and set the entry to `nil` once the image has been downloaded:

  - ```swift
      // Initialize a network task to fetch the item's artwork, keeping track of the task
      // in imageLoadTasks so they can be cancelled if the cell will not be shown after
      // the completes
      imageLoadTasks[indexPath] = Task {
          // Your code to fetch the image using the URL
          do {
              let image = try await storeItemController.fetchImage(from: item.artworkURL)
              cell.artworkImage = image
          } catch {
              print("Fetch image failed with error: \(error)")
          } 
          // Once the task is complete remove it from the list of pending tasks
          imageLoadTasks[indexPath] = nil
      }
    ```

- At this point, you should be able to enter a search term, execute the search, and view the results — including the correct artwork — in the table view. Run the app in Simulator. Does it work as expected?

##### Step 5 Cache URL Results

- Caching the results of your URL requests reduces the number of requests that need to be made to the server and improves your app's user experience by making certain data from the server more readily available — even when the user is offline, experiencing a slow network connection, or requesting a large piece of data that would typically take time to download, such as an image or video. The shared `URLSession` (`URLSession.shared`) will by default use `URLCache.shared`, which is preconfigured — but sometimes its a good idea to increase its capacity.
- If you cache images in this app, they will load significantly faster after the first time they've been downloaded from the server, because they won't need to be downloaded a second time. To increase the cache capacity, all you need to do is put the following code in the `application(_ :didFinishLaunchingWithOptions:)` method in your app's `AppDelegate`:

  - ```swift
      URLCache.shared.memoryCapacity = 25_000_000
      URLCache.shared.diskCapacity = 50_000_000
    ```

- This adjusts the default system - provided cache instance so that memoryCapacity and diskCapacity are 25 megabytes and 50 megabytes, respectively. Depending on your app and its data, you can decide what values provide the best user experience.
- Congratulations! You've made a more complex app that fetches a list of iTunes store items, loads their respective images, and displays them in a table view. Be sure to save it to your project folder for future reference.

### Guided Project: Restaurant

- In this unit, you learned some new Swift tools and how to send and retrieve information using a web service API. In this guided project, you'll put your learning to the test. You'll design an interactive menu for a restaurant that allows the customer to view a list of offerings, add items to an order, and submit the order to the restaurant. Using a server that you'll run on your own computer, you'll have an opportunity to modify the options, descriptions, and images that are returned from the API.

#### Part One. Server Details and Project Setup

- Before you begin thinking about the workflow of your menu app, it's important to understand how to work with the server. By learning how the API and the server work, you can begin to come up with a list of features for the project. You're not required to make any modifications to the data that accompanies the server, but you will need to run the server on your Mac.
- Take some time now to set up the server and to wrap your head around the details.
- **Run the Server**
  - The Restaurant folder included with this project contains the binary file for the server, `OpenRestaurant.app`. Right-click `OpenRestaurant` and click Open in the dropdown. Depending on your Mac's security configuration, you may see a warning dialog. Click Open to indicate that you trust the app.
  - To start and stop the server, click the button at the bottom right of the OpenRestaurant window. If you make changes to server resource files, you need to stop and restart the server.
  - You'll notice that the server app lets you view menu categories and items, edit categories and items, and open the images folder. Clicking Edit Categories or Edit Menu Items will open `categories.json` or `menu.json`, respectively. These file contain the data that the server returns. If you want to change the menu, you'll need to edit one or both of these files.
  - Clicking Open Images Folder will take you to a directory that contains dummy images for the items on the menu. If you want to put your own food images in this directory, make sure to use PNG images. Note that the image for a menu item is always named for its ID.
  - Start the server with the button at the bottom right. To verify that the server is functioning properly, open your browser and load the URL http://localhost:8080. The browser should display onscreen text that indicates the status of the server. If you receive an error, the most common cause is that you've incorrectly edited `menu.json, so the menu list wasn't created properly. You'll need to verify that the JSON data is valid, close the application window, and restart the server. You can use the Reset Data button to discard your custom data and restore the original files.
  - Depending on the configuration of your Mac, you may see a prompt asking for permission to allow incoming connections to the server. Click Allow.
- **JSON Structure**
  - Open `menu.json` from the OpenRestaurant window, then open http://localhost:8080/menu in your browser to view the menu data that will be returned from the API. At the root level of the JSON is an array of dictionaries, where each dictionary represents an item on the menu. If you look closely at the server data compared to `menu.json`, you'll notice that the JSON doesn't match up exactly. Don't be alarmed by the mismatch; the server makes some tweaks to the structure of the data when it receives a request.
  - In each dictionary of the JSON returned by the server, you'll see the following keys:
    - `id` is a unique `Int` that distinguishes one item from another. If you add or modify a dictionary, make sure each `id` value is unique.
    - `name` is a `String` that indicates the menu item's name. The server doesn't require names to be unique — but it wouldn't make sense to have two items with the same name on a menu, so keep the `names` unique.
    - `description` is a `String` that provides further detail about the menu item.
    - `price` is a `Double` that expresses the price of a menu item. In the sample data, the values are in U.S. dollars. However, the price values aren't tied to a particular currency, so you can adjust them to fit any currency you like.
    - `category` is a `String` that provides a way to divide items, just like in a real menu. In the sample data, the categories are "appetizers", "salads", "soups", "entrees", "desserts", and "sandwiches". If you want to create a new menu item, assign the appropriate category key. If you want to create a new category, such as "sides", add it to `category.json`.
    - `image_url` is automatically created by the server to have the server's base URL followed by the path `images` and then the name of the image. The name of the image is also automatically created by the server to be the id of the item followed by the file extension `.png`. If you wish to add your own image, make sure the name corresponds to the `id` of the item and that its extension is also `.png`.
  - When modifying `menu.json`, you should only use the keys `id`, `name`, `description`, `price`, `category`, and `estimated_prep_time`. The last key doesn't show in the JSON returned by the server for menu items, but it is used by the server to provide an estimate for how long it will take to fulfill an order. The server is configured to use these keys (and only these keys). If you add other keys into a dictionary, the server will ignore them.
- **Server Endpoints**
  - The API in this project has several URLs for implementing your app's features. Every URL begins with http://localhost:8080 and can be combined with the following endpoints:
    - `/categories`: A GET request to this endpoint will return an array of strings representing the categories in `menu.json`. The array will be available under the key `categories` in the JSON.
    - `/menu`: A GET request to this endpoint will return the full array of menu items. It can also be combined with the query parameter `category` to return a subset of items. The array will be available under the key items in the JSON.
    - `/images`: Combined with the name of the image, a GET request to this endpoint will return the image data associated with a menu item.
    - `/order`: A POST to this endpoint with a collection of menu item `id` values will submit the order and will return a response with the estimated time before the order will be ready. The IDs you send need to be contained in JSON data under the key `menuIds`. When you parse the JSON, the estimated time before the order is ready will be under the key `preparation_time`.

#### Part Two. Project Planning

- Think of how you view a traditional menu when you enter a restaurant. Items are sorted into categories and have a title, description, price, and maybe an accompanying image. In this project, you can customize the appearance and presentation of the menu, but the workflow should remain fairly standard.
- **Features**
  - As with any list-based app, it's pretty easy to establish the basic features for your menu app:
    - Display the list of available items
    - Add items to an order
    - Display the current order
    - Submit the order
  - In this project, the menu list comes from an API — a good idea for a restaurant. By storing the list on a server, the restaurant can update the menu without requiring customers to update the app they're using. As long as the features of the app don't change, the content that the app displays can be adjusted to reflect the restaurant's latest offerings.
- **Workflow**
  - Since a menu is a list, you've probably guessed that you'll be working with table views. Each cell can display one item from the list of data that's returned from an API. Using the `/categories` and `/menu` endpoints, the API can return a list of menu categories and the items associated with each category.
  - It might make the most sense if the first screen displays the categories and tapping a cell displays another table view listing the items in the category. Tapping a menu item could then display details on the next screen, along with a button that adds the item to the order.
  - So far, so good. But what if the customer wants to view the items they've added to their order? Whenever you need to provide easy access to a view — at any point in the user's workflow — a tab bar controller is a good way to contain the view. Tabs allow the user to switch quickly between contexts, without disrupting their experience.
- **Controllers**
  - As you learned earlier in this unit, you can place networking code in a number of different places. For this project, the best place is inside a controller object. Why structure the code this way, versus adding the networking logic to a view controller? A controller object adds a layer of separation between the view controller and the networking code. Instead of packing the necessary networking code into one or several view controllers, you can have your view controllers access a controller object that contains the network request code. And because all your networking code is in a single location, it will be easier to make future updates to network requests or reuse your code across multiple view controllers.
- **Views**
  - You already know that your app will rely heavily on table views. The standard `UITableViewCell` class has the correct number of text labels to display all the necessary data, so you're good to go. If you prefer to give your app a unique appearance, you can refer to previous lessons on creating and working with `UITableViewCell` subclasses.
  - Your app will also have two screens that don't use table views. You can customize these views to your liking, but you won't need to create any `UIView` subclasses to make them work.
- **Models**
  - Your app is primarily concerned with one model: the item on the menu. When you issue a query to the `/menu` endpoint, the `JSON` data will include a collection of dictionaries, each of which can be serialized into a structure.
  - What about using a structure for the data returned from the `/categories` endpoint? A category is a string, and when you query the endpoint, the API will return an array of strings. Typically, this form of data would be considered so simple that there's no need to create a more complex data structure. However, this app plans to use Codable, and the highest level of information returned by the API for `/categories` is a key called categories. This can't be decoded directly into a Swift type, so you'll need an intermediary object conforming to `Codable` that contains a property called categories.
  - Similarly, you'll need a `PreparationTime` intermediary object for the `/order` endpoint, since it returns an integer representing the time until an order will be completed under the key `preparation_time`.
  - The server has no GET endpoint for orders; they’re constructed and stored exclusively in the client app. But even though you won’t be decoding orders from JSON, you should still formalize their existence using an Order model object. The view controller you’ll create later to manage the order will display data from this object. The order structure itself will be very simple: a single property that stores an array of MenuItems.

#### Part Three. Set Up The Storyboard Workflow

- Now that you've thought through the project, its views, and its models, it's time to make your app happen.
- **Project Setup**
  - As you learned in a previous lesson, iOS applications need explicit permission to access APIs that use HTTP instead of HTTPS. But because it takes quite a few steps to set up the server to use HTTPS, you'll allow your menu app to run over HTTP.
  - Create a new project using the iOS App template and name it "OrderApp." When creating the project, make sure the interface option is set to "Storyboard."
  - Before you can begin developing the app, you'll need to add an entry to `Info.plist` to allow the app to make requests to your server. Click `Info` to open the `Info.plist` file to view the default set of keys associated with a new iOS app. Hover over the top key, `Information Property List`, and a plus sign (+) will appear. Click it to begin adding a new key to the file.
  - To allow HTTP connections, you'll need to add the key called `App Transport Security Settings`. As you start typing the name of the key, you can allow autocompletion to fill in the rest. Press Return to add the key.
  - The value for App Transport Security Settings is another dictionary. Click the triangle next to the key name to expand the contents, then hover over the key name and click the plus sign (+).
  - Next, you'll add the key Allows Local Networking to the new dictionary, available in the dropdown. The value for this setting is a Boolean, which you should change to YES.
  - With these new entries in the App Transport Security Settings dictionary, you're ready to begin developing an app that works with a local HTTP server.
- **Set Up the Tab Controller**
  - In the project planning phase, you established that you'll use a tab bar controller. Start by clicking Main to open `Main.storyboard` and deleting the view controller in the scene.
  - Think ahead for a second. What will your tabs do? In this design, the first view controller will display a list of the menu categories, and the second view controller will display a list of the items the user has already added to the order. Therefore, each tab should use a subclass of `UITableViewController`.
  - From the Object library, drag two `UINavigationController` objects onto the storyboard canvas. Each `UINavigationController` object has a `UITableViewController` attached to it.
  - Select the top navigation controller's navigation bar and, in the `Attributes inspector`, check `Prefers Large Titles`. Later, when you add additional view controllers to this navigation stack, be sure to set their navigation item's Large Title property in the Attributes inspector to Never, so that only the first table view controller displays a large title.
  - With both navigation controllers selected, choose Editor > Embed In > Tab Bar Controller. Then select the tab bar controller and, in the Attributes inspector, choose Is Initial View Controller. Since you won't be using the original view controller provided in the storyboard, you can delete its scene, if you haven't already. You can also delete the ViewController file from the Project navigator.
    - <img src="./resources/tab_bar_controller.png" alt="Tab Bar Controller" width="600" />
  - You've learned that a tab can display text and an icon. To update each tab, select the tab bar at the bottom of each `UINavigationController` in the scene, then use the Attributes inspector to give the bar item a title. Name the first tab “Menu” and the second tab “Your Order.” If you like, you can also add icons that represent a menu and an order — the system-provided `list.bullet` and `bag` images work well.
  - Now build and run the app. At the bottom of the screen, you should see a tab bar with your two titles (and icons, if you added them). Each should be associated with an empty table.
- **Create Table and Detail View Controllers**
  - Take a moment to think about how the user will interact with the menu. When first launched, the app will display the list of categories in the first tab, and tapping a category will display its list of items. The second tab will display a list of ordered items. That's three `UITableViewController` subclasses that you'll need to create.
  - Create a new Cocoa Touch Class called `CategoryTableViewController`, and make it a subclass of `UITableViewController`. Repeat this step two times, using `MenuTableViewController` and `OrderTableViewController` as the other class names.
  - Back in the Main storyboard, you'll need to adjust the class attribute of each table view controller in the scene. 
    - Select the table view controller in the first tab, then open the Identity inspector and set the Custom Class to `CategoryTableViewController`.
    - Next, select the table view controller in the second tab and set its Custom Class to `OrderTableViewController`.
  - When a table cell in `CategoryTableViewController` is tapped, it should push to the `MenuTableViewController`. 
    - Drag another table view controller from the Object library and position it to the right of the `CategoryTableViewController`. In the Identity inspector, set its Custom Class to `MenuTableViewController`. Now Control-drag from the prototype cell of `CategoryTableViewController` to the `MenuTableViewController`, creating a `Show` segue from one view controller to the next.
  - Now you have your three table view controllers. What about the detail screen? When the user taps a table cell in `MenuTableViewController`, it should push to a screen with information about the selected menu item. 
    - Drag a `UIViewController` from the Object library onto the canvas and position it to the right of the `MenuTableViewController`. Control-drag from the prototype cell in `MenuTableViewController` to the new view controller to create another `Show` segue. 
    - Before you forget, create another Cocoa Touch Class named `MenuItemDetailViewController` that's a subclass of `UIViewController`, then use the Identity inspector to customize the class of the new scene to `MenuItemDetailViewController`.
  - Since `CategoryTableViewController` is displayed within a navigation controller, it has a title at the top of the screen. “Categories” doesn't sound very engaging for a restaurant menu. Think of a clever name to give your restaurant and make that the title — or simply use “Restaurant.”
- **Define the Models**
  - Now you're ready to begin constructing the data for each item in the menu. Create a new Swift file called “MenuItem.swift” and define a `MenuItem` structure. Each item should have properties that correspond to the keys listed in each dictionary in the API. You can reference the earlier section on JSON structure to review how the properties work.
  - Similar to previous lessons, you'll make your structure adopt the `Codable` protocol. There are two JSON keys of concern for decoding. One, `image_url`, uses an underscore instead of the Swift-preferred camel case. The other, description, is a common property name in the API (it's declared in `NSObject` as well as in the Swift `CustomStringConvertible` protocol). So you'll need to create custom, non-matching properties for these keys. However, since you'll be using all the key/value pairs in the JSON data, you won't need a custom implementation of `init(from:)`. As long as you properly declare your `CodingKeys`, the compiler will take care of the rest.

    - ```swift
        struct MenuItem: Codable {
            var id: Int
            var name: String
            var detailText: String
            var price: Double
            var category: String
            var imageURL: URL
         
            enum CodingKeys: String, CodingKey {
                case id
                case name
                case detailText = "description"
                case price
                case category
                case imageURL = "image_url"
            }
        }
      ```

  - The API can be used to return lists of categories and menu items. These responses include an array of strings and an array of objects under the keys `categories` and `items`, respectively. A reasonable approach to this pattern is to create a Response model defining the web API's response for each endpoint. The models will be structures that `adoptCodable`. Create a new Swift file called “ResponseModels.swift” to hold all the `Response` structures.
  - The /menu endpoint returns an object with an `items` key that contains the `MenuItem` objects you're interested in. This response model can be defined as:

    - ```swift
        struct MenuResponse: Codable {
            let items: [MenuItem]
        }
      ```

  - For the /categories endpoint, the model will be `CategoriesResponse` and should have a property called `categories` of type [String]:

    - ```swift
        struct CategoriesResponse: Codable {
            let categories: [String]
        }
      ```

  - Finally, you'll need a response model for the value that comes with `preparation_time`. This is returned from the /order endpoint and represents the amount of time until an order will be ready. Since the key in JSON uses an underscore, you should create a custom key so your property can be named according to proper Swift convention. This structure can also be included in `ResponseModels`.

    - ```swift
        struct OrderResponse: Codable {
            let prepTime: Int
            
            enum CodingKeys: String, CodingKey {
              case prepTime = "preparation_time"
            }
        }
      ```

  - You'll need one last model object. The order model object will contain a simple list of the items the user has added. Put this in a new file called “Order.swift.”

    - ```swift
        struct Order: Codable {
            var menuItems: [MenuItem]
         
            init(menuItems: [MenuItem] = []) {
                self.menuItems = menuItems
            }
        }
      ```

  - With the `MenuItem`, `MenuResponse`, `CategoriesResponse`, `OrderResponse`, and `Order` structures defined, you can begin to write the networking code, as well as the code to handle the data that's returned from the API.

#### Part Four. Add Networking Code

- So far, you've set up your data to be retrieved from the server, created models to represent the server data, and defined the basic workflow for the app. But before you can begin displaying categories, you need to request the list from the API.
You already decided to pack all the networking code — creating the proper URLs, performing the request, and processing the JSON results — into a single controller object. This abstraction will reduce the amount of code in the table view controllers and will also simplify any future code updates.
- **Define the Methods**
  - Create a new Swift file called “MenuController.swift” and define a `MenuController` class within it. Since all the network calls in this class use the same protocol and host (http://localhost:8080/), you can define a constant that holds this value.

    - ```swift
        class MenuController {
            let baseURL = URL(string: "http://localhost:8080/")!
        }
      ```

  - Consider the API requests that your app is going to make. You'll have a GET for the categories, a GET for the items within a category, and a POST containing the user's order when it's time to communicate back to the restaurant's server. That means you'll need to define three methods, one for each request.
    1. Review the list of server endpoints at the beginning of this project. For the request to /categories, you know that there are no query parameters or additional data to send and that the response JSON will contain an array of strings. So the method should have no parameters and should return an array of strings: [String]. As with the examples of networking functions from previous lessons, the function should throw any errors and be marked as `async`, since it will be awaiting the results from the URLSession.

       - ```swift
           func fetchCategories() async throws -> [String] {
            
           }
         ```

       - Recall that your function should throw a custom error if the server doesn't return a status code of 200. The `Error` enum to support the `fetchCategories()` method will need at least one case:

         - ```swift
             enum MenuControllerError: Error, LocalizedError {
                 case categoriesNotFound
             }
           ```

    2. The request to /menu includes a query parameter: the category string. The JSON that's returned contains an array of dictionaries, and you'll want to deserialize each dictionary into a `MenuItem` object. So the method that will perform the request to /menu should have one parameter — the category string — and return and array of menu items.

       - ```swift
           func fetchMenuItems(forCategory categoryName: String) async throws ->[MenuItem] {
            
           }
         ```

       - It would be helpful to have a unique error case for this function, so go ahead and add an additional case to the Error enum: menuItemsNotFound.

         - ```swift
             enum MenuControllerError: Error, LocalizedError {
                 case categoriesNotFound
                 case menuItemsNotFound
             }
           ```

    3. The POST to /order will need to include the collection of menu item IDs that the user selected, and the response will be an integer specifying the number of minutes the order will take to prep. The method that will perform this network call should have a single parameter — an array of integers to hold the IDs — and return `Int`.

       - ```swift
          func submitOrder(forMenuIDs menuIDs: [Int]) async throws -> Int {
           
          }
         ```

       - You can also add another case to the Error enum to identify issues with this request: orderRequestFailed.

         - ```swift
            enum MenuControllerError: Error, LocalizedError {
                case categoriesNotFound
                case menuItemsNotFound
                case orderRequestFailed
            }
           ```

       - There's one thing you could do to make the `submitOrder(forMenuIDs:)` method easier to understand. You're writing the method, so you know what the returned `Int` value represents. But will someone else reading or working on your code know?
  - Swift has a feature called [type aliases[(https://docs.swift.org/swift-book/documentation/the-swift-programming-language/thebasics/#Type-Aliases)] that allows you to provide an additional name — an alias — for an existing type. This can be a simple way to communicate the meaning of a value to other developers without creating a whole new type. Update your `submitOrder(forMenuIDs:)` definition with the following:

    - ```swift
        typealias MinutesToPrepare = Int
         
        func submitOrder(forMenuIDs menuIDs: [Int]) async throws ->
          MinutesToPrepare {
         
        }
      ```

  - Now it is very clear what the value means, and Option-clicking `MinutesToPrepare` reveals that it's simply an alias for `Int`.
- **Make Requests with URL Session**
  - Each method will take the `baseURL` you defined earlier and adjust it in some way. For fetching the categories, it's as simple as appending the endpoint:

    - ```swift
        func fetchCategories() async throws -> [String] {
            let categoriesURL = baseURL.appendingPathComponent("categories")
        }
      ```

  - But there's an additional step for fetching the menu items in a category: adding the query parameter. You can use `URLComponents` to append a collection of `URLQueryItem` objects. In this case, there's only one query item in the collection, and the value should be equal to the supplied argument:

    - ```swift
        func fetchMenuItems(forCategory categoryName: String) async throws -> [MenuItem] {
            let baseMenuURL = baseURL.appendingPathComponent("menu")
            var components = URLComponents(url: baseMenuURL, resolvingAgainstBaseURL: true)!
            components.queryItems = [URLQueryItem(name: "category", value: categoryName)]
            let menuURL = components.url!
        }
      ```

  - There's a lot of force-unwrapping in the code that creates these URLs from strings. Do you think you should be concerned? Since you're not using server-side data to generate the URLs, it's safe to assume that if the URL can be built successfully once, it will work all the time.
  - Like all the requests you made in previous lessons, you can use the shared instance of `URLSession` to make your requests. You'll call a method on the shared `URLSession` for each request, and you'll await the results. For both `GET` requests, the code should look fairly similar:

    - ```swift
        func fetchCategories() async throws -> [String] {
            let categoriesURL = baseURL.appendingPathComponent("categories")
            let (data, response) = try await URLSession.shared.data(from: categoriesURL)
        }
         
        func fetchMenuItems(forCategory categoryName: String) async throws -> [MenuItem] {
            let initialMenuURL = baseURL.appendingPathComponent("menu")
            var components = URLComponents(url: initialMenuURL, resolvingAgainstBaseURL: true)!
            components.queryItems = [URLQueryItem(name: "category", value: categoryName)]
            let menuURL = components.url!
            let (data, response) = try await URLSession.shared.data(from: menuURL)
        }
      ```

  - Placing an order also starts by appending the API endpoint to the baseURL:

    - ```swift
        func submitOrder(forMenuIDs menuIDs: [Int]) async throws -> MinutesToPrepare {
            let orderURL = baseURL.appendingPathComponent("order")
        }
      ```

  - However, the POST request for placing the order is different. First, you'll create a `URLRequest` to specify the details of the request rather than directly submitting the request with a `URL`. You'll need to modify the request's type, which defaults to GET, to a POST. Then, you'll tell the server you'll be sending JSON data.

    - ```swift
        var request = URLRequest(url: orderURL)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
      ```

  - Next, you'll store the array of menu IDs in JSON under the key `menuIds`. To do this, create a dictionary of type [String: [Int]], a type that can be converted to or from JSON by an instance of `JSONEncoder`:

    - ```swift
        let menuIdsDict = ["menuIds": menuIDs]
        let jsonEncoder = JSONEncoder()
        let jsonData = try? jsonEncoder.encode(menuIdsDict)
      ```

  - Where does this data need to be stored in the request? Unlike a GET, which appends query parameters to the URL, the data for a POST must be stored within the body of the request. Once that's in place, you can submit the request using the shared `URLSession`'s `data(for:delegate:)` method. Just like in `data(from:delegate:)`, the `delegate` has a default value of `nil`, so you can leave it out:

    - ```swift
        request.httpBody = jsonData
        let (data, response) = try await URLSession.shared.data(for: request)
      ```

  - Here's the method in its entirety so far:

    - ```swift
        func submitOrder(forMenuIDs menuIDs: [Int]) async throws -> MinutesToPrepare {
            let orderURL = baseURL.appendingPathComponent("order")
            var request = URLRequest(url: orderURL)
            request.httpMethod = "POST"
            request.setValue("application/json", forHTTPHeaderField: "Content-Type")
         
            let menuIdsDict = ["menuIds": menuIDs]
            let jsonEncoder = JSONEncoder()
            let jsonData = try? jsonEncoder.encode(menuIdsDict)
            request.httpBody = jsonData
         
            let (data, response) = try await URLSession.shared.data(for: request)
        }
      ```

- **Parse the Responses**
  - Each of the `URLSession` requests will return the requested data and information about the response or will throw an error, which your method will propagate to its caller. You'll want to check the status code from the response to ensure that the server returned the data you requested. If the status code is not 200, you'll want to throw an error appropriate to the function. For example, the `fetchCategories()` function's check would look like:

    - ```swift
        guard let httpResponse = response as? HTTPURLResponse,
          httpResponse.statusCode == 200 else {
            throw MenuControllerError.categoriesNotFound
        }
      ```

  - Code that handles the errors — for example, when the server is down or when the device can't access the internet — is just as important as code that handles the case where everything works as expected. Since the server for this project in running on your Mac, it's unlikely anything will fail, but it's a good practice to always handle errors, even at a basic level. Ignoring an error is a surefire way to become confused and frustrated when something starts to silently fail.
  - The /categories endpoint will need to be decoded into a `CategoriesResponse` object, and for that you'll need to create a `JSONDecoder`. If the data is successfully decoded, return the `categories` from the `CategoriesResponse` object.

    - ```swift
        /* fetchCategories */
        let (data, response) = try await URLSession.shared.data(from: categoriesURL)
         
        guard let httpResponse = response as? HTTPURLResponse,
          httpResponse.statusCode == 200 else {
            throw MenuControllerError.categoriesNotFound
        }
         
        let decoder = JSONDecoder()
        let categoriesResponse = try decoder.decode(CategoriesResponse.self, from: data)
         
        return categoriesResponse.categories
      ```

  - The data retrieved from /menu will be converted into an array of `MenuItem` objects. You'll first need to create a `JSONDecoder` for decoding the JSON data returned by the API. You'll use the `JSONDecoder` to decode the data into a single `MenuResponse` object. If the data is successfully decoded, return the `items` from the `MenuResponse` object.

    - ```swift
        /* fetchMenuItems */
        let (data, response) = try await URLSession.shared.data(from: menuURL)
         
        guard let httpResponse = response as? HTTPURLResponse,
          httpResponse.statusCode == 200 else {
            throw MenuControllerError.menuItemsNotFound
        }
         
        let decoder = JSONDecoder()
        let menuResponse = try decoder.decode(MenuResponse.self, from: data)
         
        return menuResponse.items
      ```

  - Finally, the POST you make to /order will also return JSON data that can be decoded into your response model, `OrderResponse`. The value returned by the endpoint is the number of minutes it will take for the order to be prepared, which you can use to inform the user.

    - ```swift
        /* submitOrder */
        let (data, response) = try await URLSession.shared.data(for: request)
         
        guard let httpResponse = response as? HTTPURLResponse,
          httpResponse.statusCode == 200 else {
            throw MenuControllerError.orderRequestFailed
        }
         
        let decoder = JSONDecoder()
        let orderResponse = try decoder.decode(OrderResponse.self, from: data)
         
        return orderResponse.prepTime
      ```

  - You've spent some time defining all the network requests that will take place at some point in the app. Now it's time to put them to work. With the model controller methods in place, you can begin to display the results of the data in the proper tables.

#### Part Five. Categories

- The first request in your workflow is to display the menu categories. Open the `Main` storyboard and find the `CategoryTableViewController`. How will each cell look? How will you identify the cells so you can use them in code?
- **Basic Table Setup**
  - Select the prototype cell, then open the Attributes inspector. All you need to display is a single string, so set the Style to `Basic`. Use “Category” for the identifier, then set the Accessory to `Disclosure Indicator`, which will make it clear to the user that tapping the cell will take them to a new screen.
  - Open `CategoryTableViewController` and create an instance of `MenuController` so that you can make the appropriate network request in `viewDidLoad()`. You will use a do/catch statement to catch any errors. If the request succeeded, you'll pass that data into a method that will set the property and update the interface appropriately. If it failed, you'll display an error using `UIAlertController`. In this example, either the method `updateUI(with:)` or the method `displayError(_:title:)` is called — a pattern that you'll repeat for other view controllers.

    - ```swift
        @MainActor
        class CategoryTableViewController: UITableViewController {
            let menuController = MenuController()
            var categories = [String]()
         
            override func viewDidLoad() {
                super.viewDidLoad()
         
                Task.init {
                    do {
                        let categories = try await menuController.fetchCategories()
                        updateUI(with: categories)
                    } catch {
                        displayError(error, title: "Failed to Fetch Categories")
                    }
                }
            }
         
            func updateUI(with categories: [String]) {
                self.categories = categories
                self.tableView.reloadData()
            }
         
            func displayError(_ error: Error, title: String) {
                guard let _ = viewIfLoaded?.window else { return }
         
                let alert = UIAlertController(title: title, message: error.localizedDescription, preferredStyle: .alert)
                alert.addAction(UIAlertAction(title: "Dismiss", style: .default, handler: nil))
                self.present(alert, animated: true, completion: nil)
            }
        }
      ```

  - Remember that user interface code needs to run on the main actor and that the task will inherit the context from where it is called. Since a `UITableViewController` executes on the main actor, all the code in the closure will also be bound to the main actor, and it's safe to make UI calls within the closure. While not strictly necessary, the above code explicitly marks the class as being bound to the MainActor with `@MainActor` before the class definition.
  - The `displayError(error:title:)` function contains a check to make sure that the view associated with the `TableViewController` is currently in a window. This way you don't try to post an alert on a view that is not visible. Without this check, the alert would not be shown but you would see a warning in the console. Since the network request is running asynchronously, it's possible for the user to navigate away from a particular view before the network request completes. The statement checks whether the view has been loaded using a property that won't cause it to load the view if it has not been loaded yet. If the view exists and its window is currently set, the method will continue; otherwise it returns.
    - `guard let _ = viewIfLoaded?.window else { return }`
  - Next, you'll flesh out the methods that define how many rows the table view will have and create a cell for each row to display. The number of rows (and cells) is equal to the number of items in the categories property, and the text of each cell will display a string. You can clean up the appearance of a category string by capitalizing it when it's displayed in a cell.

    - ```swift
        override func numberOfSections(in tableView: UITableView) -> Int {
            return 1
        }
         
        override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
            return categories.count
        }
         
        override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
            let cell = tableView.dequeueReusableCell(withIdentifier: "Category", for: indexPath)
            configureCell(cell, forCategoryAt: indexPath)
         
            return cell
        }
         
        func configureCell(_ cell: UITableViewCell, forCategoryAt indexPath: IndexPath) {
            let category = categories[indexPath.row]
            var content = cell.defaultContentConfiguration()
            content.text = category.capitalized
            cell.contentConfiguration = content
        }
      ```

  - The `configureCell(_:forCategoryAt:)` method does not set the cell's `textLabel` property directly, but rather uses the modern approach of updating the text property of the cell's `contentConfiguration` property. Recall that to use content configurations, you first ask for the `defaultContentConfiguration` of the cell (which it inherits in this case from the prototype cell you defined in the storyboard) and then you set up all the characteristics you want the configuration to impart to the final cell when it's rendered (in this case the main text for the cell). Then you set that configuration on the cell. Using this approach, you never have to worry about the existing state of the cell, and the system can efficiently update the display as needed. You'll see other examples of using cell configurations later.
  - Build and run your app with the server running. You should see a list of the categories defined in `menu.json`. If you see the message "Failed to Fetch Categories," check that you have the server application running and that you have clicked the Start button.
- **Pass Category Data**
  - When the user taps a cell with a category name, it will push to the `MenuTableViewController`, which currently displays an empty table. The `MenuTableViewController` should be initialized with a category, which it will use to make a network request for the category's `MenuItems`. You will be using an `@IBSegueAction` when the cell is selected to initialize the new instance of `MenuTableViewController`. Create a custom initializer in `MenuTableViewController` that receives the required `NSCoder` and the `category` string.

    - ```swift
        class MenuTableViewController: UIViewController {
         
            let category: String
         
            init?(coder: NSCoder, category: String) {
                self.category = category
                super.init(coder: coder)
            }
         
            required init?(coder: NSCoder) {
                fatalError("init(coder:) has not been implemented")
            }
        }
      ```

  - In `CategoryTableViewController`, create the `@IBSegueAction` from the segue between `CategoryTableViewController` and `MenuTableViewController`. ​Name it “showMenu” and set the arguments to `Sender`.
    - <img src="./resources/segueAction_showMenu.png" alt="segueAction showMenu" width="500" />
  - The code for this method will look like the following:

    - ```swift
        @IBSegueAction func showMenu(_ coder: NSCoder, sender: Any?) -> MenuTableViewController? {
            guard let cell = sender as? UITableViewCell, 
                let indexPath = tableView.indexPath(for: cell) else {
                return nil
            }
         
            let category = categories[indexPath.row]
            return MenuTableViewController(coder: coder, category: category)
        }
      ```

  - First you cast the `sender` argument to a `UITableViewCell` (the cell that was selected). Then you look up its `IndexPath` and use that to determine the category that was selected. Next, you use your custom initializer to create and return a new instance of `MenuTableViewController`.
  - Build and run your app, then select a category from the list. The `MenuTableViewController` will be displayed, but it doesn't yet use the data it was handed.

#### Part Six. Menu Items

- So far, you've made the request for the list of categories from the server, and the user can select a category from the list. With that data available in `MenuTableViewController`, you can query for menu items listed in the category.
- The menu item screen shares many attributes with the categories screen in the previous section. They both use a `UITableViewController` subclass, they both use a network request to fetch the appropriate data, and selected cells in both screens push to a new screen. Because of those similarities, the code patterns in the following section should feel pretty familiar to you.
- **Basic Table Setup**
  - The prototype cell needs to display two strings: the name of the menu item on the left and the price of the item on the right. Later on in the project, you'll add a thumbnail image to the cell. For now, select the cell and set its Style property to `Right Detail`. Use “MenuItem” as the identifier and set the Accessory to `Disclosure Indicator`.
  - Open `MenuTableViewController` and define an array of `MenuItem` objects to be displayed in the table. Create an instance of `MenuController` so that you can make the appropriate network request in `viewDidLoad()`. Once the request is finished, use the result that was returned and pass that data into a method that will set the property and update the interface appropriately; otherwise, display any error that is caught.

    - ```swift
        @MainActor
        class MenuTableViewController: UITableViewController {
         
            let category: String
            let menuController = MenuController()
            var menuItems = [MenuItem]()
         
            init?(coder: NSCoder, category: String) {
                self.category = category
                super.init(coder: coder)
            }
         
            required init?(coder: NSCoder) {
                fatalError("init(coder:) has not been implemented")
            }
         
            override func viewDidLoad() {
                super.viewDidLoad()
                title = category.capitalized
         
                Task.init {
                    do {
                        let menuItems = try await menuController.fetchMenuItems(forCategory: category)
                        updateUI(with: menuItems)
                    } catch {
                        displayError(error, title: "Failed to Fetch Menu Items for \(self.category)")
                    }
                }
            }
            func updateUI(with menuItems: [MenuItem]) {
                self.menuItems = menuItems
                self.tableView.reloadData()
            }
         
            func displayError(_ error: Error, title: String) {
                guard let _ = viewIfLoaded?.window else { return }
                let alert = UIAlertController(title: title, message: error.localizedDescription, preferredStyle: .alert)
                alert.addAction(UIAlertAction(title: "Dismiss", style: .default, handler: nil))
                self.present(alert, animated: true, completion: nil)
            }
        }
      ```

  - Notice that the `title` property was updated as part of `viewDidLoad()`. You can set the title to anything you wish, either in code or in the storyboard. In this example, the category name has been capitalized and displayed.
  - Your table will have one section, and the number of cells in your table should be equal to the number of menu items in the array. The left label of the cell should display the name of the item, and the right label should display the price along with a currency symbol.

    - ```swift
        override func numberOfSections(in tableView: UITableView) -> Int {
            return 1
        }
         
        override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
            return menuItems.count
        }
         
        override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
            let cell = tableView.dequeueReusableCell(withIdentifier: "MenuItem", for: indexPath)
            configure(cell, forItemAt: indexPath)
            return cell
        }
         
        func configure(_ cell: UITableViewCell, forItemAt indexPath: IndexPath) {
            let menuItem = menuItems[indexPath.row]
         
            var content = cell.defaultContentConfiguration()
            content.text = menuItem.name
            content.secondaryText = "$\(menuItem.price)"
            cell.contentConfiguration = content
        }
      ```

  - Build and run your app. After you select a category, you should see the list of items assigned to the category in `menu.json`. Great work!
  - But you may have noticed a slight bug in the detail text label — it's only displaying one decimal place if the last digit is a zero. String interpolation does its best to create reasonable values for escaped expressions, but in this case its behavior doesn't match your intentions. When displaying currencies, you should use the `formatted` function on the price to make sure things look just the way you want them to.
  - Just like you saw in a previous lesson with Date objects, you can control the formatting of number types, like the `MenuItem`'s `price`, which is of type `Double`. Use the `formatted` method when setting the cell content configuration's `secondaryText` to specify that the value should be formatted as a currency, using the appropriate representation for US dollars: `content.secondaryText = menuItem.price.formatted(.currency(code: "usd"))`
  - Build and run the app again to see the properly formatted values.
- **Pass Menu Item Data**
  - Similar to the category table view, tapping a cell in the menu table view will push to the next screen — in this case, the detail screen. But right now, no data is being passed to that view controller. Once again, you'll use an `@IBSegueAction` and a custom initializer to solve this.
  - Open `MenuItemDetailViewController`, add a MenuItem property, and create a custom initializer that accepts an NSCoder and MenuItem and sets the new MenuItem property.

    - ```swift
        @MainActor
        class MenuItemDetailViewController: UIViewController {
         
            let menuItem: MenuItem
         
            init?(coder: NSCoder, menuItem: MenuItem) {
                self.menuItem = menuItem
                super.init(coder: coder)
            }
         
            required init?(coder: NSCoder) {
                fatalError("init(coder:) has not been implemented")
            }
        }
      ```

  - Open the Main storyboard with `MenuTableViewController` in the assistant editor. Select the segue between `MenuTableViewController` and `MenuItemDetailViewController` and Control-drag into `MenuTableViewController` to add a new `@IBSegueAction` named “showMenuItem” with Arguments set to Sender.
  - The implementation for `showMenuItem` should be a familiar pattern to you from the last segue action:

    - ```swift
        @IBSegueAction func showMenuItem(_ coder: NSCoder, sender: Any?) -> MenuItemDetailViewController? {
            guard let cell = sender as? UITableViewCell, let indexPath = tableView.indexPath(for: cell) else {
                return nil
            }
         
            let menuItem = menuItems[indexPath.row]
            return MenuItemDetailViewController(coder: coder, menuItem: menuItem)
        }
      ```

  - Build and run your app. You've made tremendous progress so far! You have two table view controllers that are requesting data over the network and displaying the data returned from the API. Now you're ready to add items to the order.
- **Shared Menu Controller**
  - Pause for a moment to consider the `MenuController` objects that you defined in both `CategoryTableViewController` and `MenuTableViewController`. Each `MenuController` object performs a network request. Is there any reason to have more than one instance of the object?
  - One solution could be to pass the controller property from one view controller to the next in their `@IBSegueAction` methods. That way, you create only one instance of the object. (This pattern is called “dependency injection,” and it's common in complex apps, especially when structuring code to be easily tested.) But think about the view controller that will be responsible for submitting the order to the server. It's completely separate from `CategoryTableViewController`, so it can't pass the controller object across without accessing the view controllers contained by the root tab bar controller. That kind of access would be clumsy, and it's not recommended.
  - The better solution in this case is to create an instance of `MenuController` that's shared across all the view controllers in your app — identical to using the shared instance of `URLSession` when you created the `URLSessionDataTask` subclasses. (This pattern is called a “singleton,” and it's also widely used.) To create the shared instance, define a static let property in the `MenuController` class that's the same type:

    - ```swift
        class MenuController {
          static let shared = MenuController()
         
          // The rest of the MenuController definition has been omitted
        }
      ```

  - Now you can delete the `menuController` properties in the two table view controllers and reference the shared instance instead.

    - ```swift
        let categories = try await MenuController.shared.fetchCategories()
         
        let menuItems = try await MenuController.shared.fetchMenuItems(forCategory: category)
      ```

  - Creating a shared instance may seem like a small adjustment, but consider the advantages in a much larger app. This approach reduces the number of properties you'll need to create in any view controllers that will be making a network request, while simplifying any updates in the app's future.

#### Part Seven. Menu Details

- Your user has tapped an item on the `MenuTableViewController`. Next, they'll want some information on a details screen to help them make a decision. Even though they've already viewed the item's name, price, and image (which you'll add later in the project), it's a nice idea to repeat the same information on the next screen they'll see.
- What else can you offer them? The other information available is the `detailText`, which might or might not be represented by a large chunk of text. A larger image would be nice, too.
- **Item Details Interface**
  - Here's a walkthrough of one approach to an interface for the details screen:
    - <img src="./resources/menu_item_detail.png" alt="Menu Item Detail View" width="400" />
  - Start by laying out an image view and three labels representing the item name, price, and details as you see them in the design, doing your best to match their sizes and colors. Have the image view show the `photo.on.rectangle` system image as a placeholder. The item detail label should have Lines set to 0.
  - Next, place the name and price labels in a horizontal stack view by selecting them both and choosing Embed in Stack View. When you use this feature, Interface Builder will infer that you want a horizontal stack view, based on their existing arrangement. Now select the image view, your horizontal stack view, and your item detail label and choose Embed in Stack View again. This will create a vertical stack view.
  - Your vertical stack view now needs some constraints. Create top, leading, and trailing constraints pinned to the Safe Area with 15 points of padding. Both the vertical and horizontal stack views should have Alignment and Distribution set to `Fill` and Spacing set to 8.
  - At this point, there are still some constraint issues to resolve. First, the width of your image view is defined, but not the height. Set the height of the image view equal to the height of the superview with a multiplier of 0.25.
    - Select View and Image View, click Add New Constraints, and check Equal Heights
    - On the Size inspector of the Image View, Click Edit of Equal Height, and change Mulitplier to 0.25
  - Next, select the Item Name label and, in the Size inspector, set its Horizontal Content Hugging Priority to 250. This will allow the name label to fill the available width between itself and the price label.
  - The last layout item is the Add To Order button at the bottom. Add a `Filled Button` from the object library and set the title to `Add To Order`.
  - Now pin the button's trailing, bottom, and leading edges to the Safe Area with a padding of 15 and set its height to 44. At this point, your layout should be set. You can compare your constraints and layout to the image above.
- **Create Outlets and Display Data**
  - With your interface in place, you can now create outlets for each view. Open `MenuItemDetailViewController` in the assistant editor. Control-drag from the image view, labels, and button to an available spot within the class definition. Following are examples of outlet names:

    - ```swift
        class MenuItemDetailViewController: UIViewController {
            @IBOutlet var nameLabel: UILabel!
            @IBOutlet var imageView: UIImageView!
            @IBOutlet var priceLabel: UILabel!
            @IBOutlet var detailTextLabel: UILabel!
            @IBOutlet var addToOrderButton: UIButton!
         
            // The rest of the MenuItemDetailViewController definition has been omitted.
        }
      ```

  - Since you'll want to use data to update the outlets, create a method called `updateUI()` and call it in `viewDidLoad()`. This method should update properties on each of the outlets to match the values in the `menuItem`. (You can continue to ignore the image, since you haven't yet requested the image data.)
  - Back in `MenuItemDetailViewController` create the `updateUI()` method and call it in `viewDidLoad()`.

    - ```swift
        override func viewDidLoad() {
            super.viewDidLoad()
         
            updateUI()
        }
         
        func updateUI() {
            nameLabel.text = menuItem.name
            priceLabel.text = menuItem.price.formatted(.currency(code: "usd"))
            detailTextLabel.text = menuItem.detailText
        }
      ```

  - Build and run your app. Your detail view's custom interface should display the item name, price, and description as well as the Add To Order button. Great job!

- **Customize the Button**
  - Wouldn't it be nice if the user had some feedback — maybe a quick bounce animation — after adding an item to the order? Create an action for the `Add To Order` button by Control-dragging from the button to an available space in the `MenuItemDetailViewController` class. Name the action “addToOrderButtonTapped.” Within this method, you can perform modifications to the button's transform property. First, scale the X and Y values to two times their original size; next, scale them back to the original size. Place these calls inside the `animate(withDuration:animations:)` method, and set the duration to 0.5.

    - ```swift
        @IBAction func orderButtonTapped(_ sender: UIButton) {
            UIView.animate(withDuration: 0.5, delay: 0, usingSpringWithDamping: 0.7, initialSpringVelocity: 0.1, options: [], animations: {
                self.addToOrderButton.transform = CGAffineTransform(scaleX: 2.0, y: 2.0)
                self.addToOrderButton.transform = CGAffineTransform(scaleX: 1.0, y: 1.0)
            }, completion: nil)
        }
      ```

  - Run your app again and try tapping the button to see it bounce a little. Play around with the values to customize the animation to your liking. For example, you can increase or decrease the amount of time it takes the animation to complete, adjust the damping and initial spring velocity, or add additional modifications to transform during the animation process. Have fun creating your own unique animation!

#### Part Eight. View And Edit Order

- Where are you now? You've laid out the interface for displaying the menu in its entirety, from the list of categories all the way down to the details of a single item. But the Add To Order button still doesn't do anything beyond its animation, because there's no order being tracked. The app needs to be able to keep track of the user's order and submit it back to the restaurant.
- **Display Order**
  - Open `OrderTableViewController` and create a property that will hold the order. Since it won't be received using a segue, it doesn't need to be optional. Just create an empty Order that will be filled in item by item. Unlike most of your screens, this one can be presented even if its table view contents are empty.

    - ```swift
        class OrderTableViewController: UITableViewController {
            var order = Order()
        }
      ```

  - Have you noticed that `OrderTableViewController` and `MenuTableViewController` both display a collection of `MenuItem` objects? The two controllers rely on similar logic. The number of cells is equal to the number of items in the `menuItems` property, and the cells in both controllers display identical data.

    - ```swift
        override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
            return order.menuItems.count
        }
         
        override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
            let cell = tableView.dequeueReusableCell(withIdentifier: "Order", for: indexPath)
            configure(cell, forItemAt: indexPath)
            return cell
        }
         
        func configure(_ cell: UITableViewCell, forItemAt indexPath: IndexPath) {
            let menuItem = order.menuItems[indexPath.row]
         
            var content = cell.defaultContentConfiguration()
            content.text = menuItem.name
            content.secondaryText = menuItem.price.formatted(.currency(code: "usd"))
            cell.contentConfiguration = content
        }
      ```

  - Before you can build and run your app, open the Main storyboard and find the `OrderTableViewController`. Select the prototype cell and set its Identifier to “Order” and its Style to `Right Detail`. Adjust the labels to indicate that they will show an item's name and price. 
  - You now have a properly configured table, but there are still no instances of `MenuItem` to display.
- **Create Ordering Functionality**
  - Even though you have an Add To Order button on the `MenuItemDetailViewController`, you don't have a way to pass the `MenuItem` data over to the `OrderTableViewController`. How can you hand information from one view controller to another without a segue?
  - Since you already have a shared `MenuController`, you can make the order one of its properties. This decision assumes that your app will handle only one order. If you decide later to handle multiple orders, you’ll have to change all references to the order property. But the advantage of this approach is that the controller has quick access to an order, without needing to pass it around in notifications or protocol methods. If you decide to handle multiple orders in the future, you'll already be touching all the parts of your app that access the order property.
- **Create the Order Property**
  - In `MenuController`, create and initialize a new property for the order: `var order = Order()`
  - Now go back to your `OrderTableViewController` and remove the order property. Any time you refer to it, you'll access the `MenuController`'s property instead.

    - ```swift
        override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
            return MenuController.shared.order.menuItems.count
        }
         
        func configure(_ cell: UITableViewCell, forItemAt indexPath: IndexPath) {
            let menuItem = MenuController.shared.order.menuItems[indexPath.row]
         
            var content = cell.defaultContentConfiguration()
            content.text = menuItem.name
            content.secondaryText = menuItem.price.formatted(.currency(code: "usd"))
            cell.contentConfiguration = content
        }
      ```

  - Your `MenuItemDetailViewController` can now append the ordered item to the shared object.

    - ```swift
        @IBAction func orderButtonTapped(_ sender: UIButton) {
            UIView.animate(withDuration: 0.5, delay: 0, usingSpringWithDamping: 0.7, initialSpringVelocity: 0.1, options: [], animations: {
                self.addToOrderButton.transform = CGAffineTransform(scaleX: 2.0, y: 2.0)
                self.addToOrderButton.transform = CGAffineTransform(scaleX: 1.0, y: 1.0)
            }, completion: nil)
         
            MenuController.shared.order.menuItems.append(menuItem)
        }
      ```

- **Notify of Order Changes**
  - There's one issue that you haven't addressed yet. If you add items to the shared order from your `MenuItemDetailViewController`, there's no way for your `OrderTableViewController` to know that it's changed. You can observe this by adding a few items to the order, switching tabs to see them, then adding another item and viewing your order again.
  - Why did you see the first items but not the subsequent ones? The `OrderTableViewController`'s view isn't loaded until the first time you tap its tab. So the table view sees the initial state of the order as it loads and displays its data — but there's no code to respond to subsequent changes.
  - The easiest way to fix this problem is with `NotificationCenter`. In Lesson 1.4, you learned how to respond to keyboard show and hide notifications. But you're not limited to responding to system notifications — you can create and send your own. Start by adding a static property to your `MenuController` that defines the name of the order - change notification. (The naming strategy for notifications is largely a matter of style, but keep in mind that notification names are strings under the surface, so it's possible to choose a name that collides with one that iOS is using.)
    - `static let orderUpdatedNotification = Notification.Name("MenuController.orderUpdated")`
  - When should you send this notification? The easiest solution is to observe changes to the order property. That way, whenever it's modified, all observers will have a chance to react. Sending a notification is as simple as specifying a name and an object. The object parameter is a way for observers to filter notifications if they arrive in different contexts or from different sources. But since this is a simple implementation, you can set the object parameter to `nil`.

    - ```swift
        var order = Order() {
            didSet {
                NotificationCenter.default.post(name: MenuController.orderUpdatedNotification, object: nil)
            }
        }
      ```

  - Now you'll create an observer for your new notification in `OrderTableViewController`. Add the notification observation code to your `viewDidLoad()` method. When the order is updated, you'll reload the table view, so the observer will be the view controller's tableView property. Specify the `reloadData()` method of `UITableView` as the selector, and set the last argument to `nil` again.

    - ```swift
        override func viewDidLoad() {
            super.viewDidLoad()
         
            NotificationCenter.default.addObserver(tableView!, selector: #selector(UITableView.reloadData), name: MenuController.orderUpdatedNotification, object: nil)
        }
      ```

  - Build and run your app. When you add items, you can change tabs to see them in the order list. Great job!
- **Update the Order Badge**
  - The Add To Order button animates when the order list is updated, but you can provide extra feedback to the user. A nice touch is to update the badge value of the Order tab to match the number of items in the order. Then the user can see that they've successfully added an item without needing to switch tabs.
  - Which object should be responsible for updating the badge? You might think to add the code to `OrderTableViewController`. After all, it's already handling the order update notification. But doing so would require it to know the view controller hierarchy: It's contained inside a navigation controller inside the second tab of the root tab bar controller. The view controller would also need to know that the tab bar item with the badge belongs to the navigation controller (since it's the direct child of the tab bar controller).
  - In fact, it’s better for a view controller to be ignorant of its ancestors. That way, you’ll have more flexibility to use it in different ways — without breaking your code if you should change the navigation structure of your app.
  - So which object should know how your `OrderTableViewController` is contained in parent view controllers? It should be the first object of a type that you provide that can address these view controllers as descendants. Since the parent navigation controller and its parent tab bar controller are system - provided objects, this knowledge is best contained in your scene delegate, which is specifically tasked with managing the scene's window and root view controller.
  - To start, you'll need to keep a reference to the tab bar item you want to update. Make the following changes in `SceneDelegate`. Add a new property for the tab bar item, and make it an implicitly unwrapped optional, since badge updates should always work. If it crashes, that means you've broken your assumptions about the app's UI.
    - `var orderTabBarItem: UITabBarItem!`
  - Add a method to update the item’s badge. To use `updateOrderBadge()` as a selector in your call to `addObserver(_:selector:name:object:)`, you'll need to annotate the function with `@objc`.

    - ```swift
        @objc func updateOrderBadge() {
            orderTabBarItem.badgeValue = String(MenuController.shared.order.menuItems.count)
        }
      ```

  - Now you need to set the property when the app first launches and observe the `orderUpdatedNotification` as you did in `OrderTableViewController`. Add this code to `scene(_:willConnectTo:options:)`:

    - ```swift
        NotificationCenter.default.addObserver(self, selector: #selector(updateOrderBadge), name: MenuController.orderUpdatedNotification, object: nil)
         
        orderTabBarItem = (window?.rootViewController as? UITabBarController)?.viewControllers?[1].tabBarItem
      ```

  - Build and run your app, then try adding an item to the order. The Add To Order button animates and the tab's badge updates. If you switch tabs, you can see the item displayed in the order view. Well done!
  - (Note: If you were thinking that the technique for accessing the tab bar item from the scene delegate is similar to the approach you'd take for adding a `MenuController` to all your view controllers using dependency injection — you'd be right. In that scenario, the scene delegate would create and own the `MenuController` instance.)
- **Delete Items from Order**
  - What if the user changes their mind? In iOS, it's common to reveal a red Delete button when the user swipes left. Tapping the button would remove the item from the order and update the order table.
  - To add swipe-to-delete functionality to your order table view controller's cells, you'll need to override the `tableView(_:canEditRowAt:)` method. You could use the `indexPath` property to choose which cells are editable - but since all cells in this app can be selected, simply return `true`.
  - Uncomment the following implementation in `OrderTableViewController` (or add it if it is not present):

    - ```swift
        override func tableView(_ tableView: UITableView, canEditRowAt indexPath: IndexPath) -> Bool {
            return true
        }
      ```

  - How will the deletion be executed? Uncomment or add the override of the `tableView(_:commit:forRowAt:)` method. In its implementation, verify that the Delete button triggered the method call, then delete the item from the order. The table view and badge updates will be handled automatically by your notification code.

    - ```swift
        override func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath) {
            if editingStyle == .delete {
                MenuController.shared.order.menuItems.remove(at: indexPath.row)
            }
        }
      ```

  - Since some iOS users are unfamiliar with swipe-to-delete, you'll want to also add an Edit button as a left bar button in `viewDidLoad()`:

    - ```swift
        override func viewDidLoad() {
            super.viewDidLoad()
            navigationItem.leftBarButtonItem = editButtonItem
         
            NotificationCenter.default.addObserver(tableView!, selector: #selector(UITableView.reloadData), name: MenuController.orderUpdatedNotification, object: nil)
        }
      ```

  - Build and run your app. Add some items to the order, then remove them using swipe-to-delete or the Edit button.
  - There's one small bit of finesse left. If you delete all the items from your order, the badge remains on the tab with a value of zero — which is inconsistent with the app's launch state, where the badge is absent. It’s also inelegant, since the user can infer an empty order from an absent badge.
  - To correct the issue, modify the `updateOrderBadge()` method in `SceneDelegate`. A simple fix would be to use an if-else statement, but there's a more concise way using switch. In the case where the count is zero, you can set the badge value to `nil`. Otherwise, you can use case let to assign the value of the switch expression to a local variable count. This code is cleaner than having to write `MenuController.shared.order.menuItems.count` multiple times.

    - ```swift
        @objc func updateOrderBadge() {
            switch MenuController.shared.order.menuItems.count {
            case 0:
                orderTabBarItem.badgeValue = nil
            case let count:
                orderTabBarItem.badgeValue = String(count)
            }
        }
      ```

  - Build and run your app to test the new functionality.

#### Part Nine. Submit Order

- Now you're ready to send data back to the restaurant server. It makes sense for the user to submit the order directly from the `OrderTableViewController`. Add a bar button item to the upper - right corner of the order screen, and set its title to `Submit`.
- What should happen when the user taps the button? When there's a monetary transaction involved, it's standard practice to ask the user to confirm their choice. If this were a real app, you'd begin processing payment when the user confirms the order, and you'd inform the user that the order has been placed only after you'd received notification of a successful payment. In this project, you'll confirm the user's order, then let them know how long their order is expected to take.
- **Build Order Confirmation**
  - You've reached the final screen in the workflow. The user needs confirmation that the order has been received from the server. When the server responds with a preparation time, you can display the confirmation screen. Since you have to wait for a server response before presenting the screen, you'll perform the segue programmatically.
  - Create a new file for a subclass of `UIViewController` called `OrderConfirmationViewController`. Next, open the Main storyboard; add a new view controller from the Object library and set its class to “OrderConfirmationViewController.” Position the view controller to the right of OrderTableViewController.
    - Control-drag from the table view controller in the navigation to the new view controller to create a programmatic modal segue.
  - You'll be using an `@IBSegueAction` to segue to the confirmation screen and pass the preparation time. At this point, you're probably familiar with this pattern. Create a custom initializer on `OrderConfirmationViewController` that accepts the preparation time parameter and sets it on a new property, `minutesToPrepare`.

    - ```swift
        class OrderConfirmationViewController: UIViewController {
         
            let minutesToPrepare: Int
         
            init?(coder: NSCoder, minutesToPrepare: Int) {
                self.minutesToPrepare = minutesToPrepare
                super.init(coder: coder)
            }
         
            required init?(coder: NSCoder) {
                fatalError("init(coder:) has not been implemented")
            }
        }
      ```

  - Create an `@IBSegueAction` in `OrderTableViewController` from your new segue between `OrderTableViewController` and `OrderConfirmationViewController`. Name it “confirmOrder” and set Arguments to None. When you implement the method, you'll pass in the number of minutes to prepare the order. But where will you get the preparation time? It will come from the server once the user has submitted the order. Create a property on `OrderTableViewController` that will store the value when it is returned so that you can access it in this method.
    - `var minutesToPrepareOrder = 0`
  - Now you can pass that to your @IBSegueAction.

    - ```swift
        @IBSegueAction func confirmOrder(_ coder: NSCoder) -> OrderConfirmationViewController? {
            return OrderConfirmationViewController(coder: coder, minutesToPrepare: minutesToPrepareOrder)
        }
      ```

  - But how will you call this method? You don't have an `NSCoder` to pass. Previously, your `@IBSegueActions` were called by an action like selecting a cell, but to initiate this call you will need to use the method `shouldPerformSegue(withIdentifier:sender:)`. Go back to the storyboard and set an identifier on your segue named “confirmOrder.”
  - You can design the order confirmation screen however you like. The most important piece of information is the time until the user can pick up their order. To keep things simple, you might include two objects: a label that displays the number of minutes before the food is ready and a button that unwinds back to the tabs. 
  - Drag a label and a button onto the scene. Embed them in a vertical stack view. Add the following constraints to the stack view:
    - Center vertically within the view.
    - Align the leading edges to the Safe Area, allowing for some padding.
    - Align the trailing edges to the Safe Area, allowing for some padding.
    - <img src="./resources/order_confirmation_view.png" alt="Order Confirmation View" width="500" />
  - Set the label's number of lines to 0 and center the text alignment. The label's text will change based on the response from the server, so you'll need to create an outlet to the label.

    - ```swift
        class OrderConfirmationViewController: UIViewController {
            @IBOutlet var confirmationLabel: UILabel!
         
            // The rest of the OrderConfirmationViewController definition has been omitted.
        }
      ```

  - Set the button's title to `Dismiss` and center it horizontally within its view. When the user taps the Dismiss button, it's logical to unwind back to the order screen. First, you'll need to define a method in `OrderTableViewController` that will be called when the unwind is triggered. Here's an example:
    - `@IBAction func unwindToOrderList(segue: UIStoryboardSegue) {}`
  - Next, Control-drag from the Dismiss button to the Exit button at the `OrderConfirmationViewController` in the navigation and select the name of the method you just created. This will create an unwind segue. Using the Attributes inspector, give the segue an identifier of “dismissConfirmation.”
  - The confirmation screen is now in place, but you can't present it until you've submitted the user's order and received confirmation from the server.
- **Submit and Alert**
  - Open the Main storyboard and create an `@IBAction` named “submitTapped” for the Submit button on `OrderTableViewController`. The action will alert the user that their order will be submitted if they continue.
  - The simplest approach is to present a `UIAlertController` that will display the order's total and ask the user if they wish to proceed. For the total, you can use the reduce method to add all the menu item prices, then format the price (as you've done previously). If the user chooses to continue, call a new method, `uploadOrder`. Here's how that all works together:

    - ```swift
        @IBAction func submitTapped(_ sender: Any) {
            let orderTotal = MenuController.shared.order.menuItems.reduce(0.0) { (result, menuItem) -> Double in
                return result + menuItem.price
            }
         
            let formattedTotal = orderTotal.formatted(.currency(code: "usd"))
         
            let alertController = UIAlertController(title: "Confirm Order", message: "You are about to submit your order with a total of \(formattedTotal)", preferredStyle: .actionSheet)
            alertController.addAction(UIAlertAction(title: "Submit", style: .default, handler: { _ in
                self.uploadOrder()
            }))
         
            alertController.addAction(UIAlertAction(title: "Cancel", style: .cancel, handler: nil))
         
            present(alertController, animated: true, completion: nil)
        }
      ```

  - The `uploadOrder()` method is just like the other network requests you've written in this project. Make the request using the `submitOrder` method you defined in `MenuController`, passing in a collection of menu `IDs` as the argument. Use a do/catch statement to check for errors. If the request succeeds, you'll keep the returned value in the `minutesToPrepareOrder` property and initiate the segue; otherwise, display the error.

    - ```swift
        func uploadOrder() {
            let menuIds = MenuController.shared.order.menuItems.map { $0.id }
            Task.init {
                do {
                    let minutesToPrepare = try await MenuController.shared.submitOrder(forMenuIDs: menuIds)
                    minutesToPrepareOrder = minutesToPrepare
                    performSegue(withIdentifier: "confirmOrder", sender: nil)
                } catch {
                    displayError(error, title: "Order Submission Failed")
                }
            }
        }
         
        func displayError(_ error: Error, title: String) {
            guard let _ = viewIfLoaded?.window else { return }
            let alert = UIAlertController(title: title, message: error.localizedDescription, preferredStyle: .alert)
            alert.addAction(UIAlertAction(title: "Dismiss", style: .default, handler: nil))
            self.present(alert, animated: true, completion: nil)
        }
      ```

  - Calling `self.performSegue(withIdentifier: "confirmOrder", sender: nil)` will invoke the `confirmOrder(_:) @IBSegueAction`, which will initialize a new `OrderConfirmationViewController` with the `minutesToPrepare` value returned from the server. How will you incorporate it into the label you added to the screen a few steps ago? The message can read anything you wish.

    - ```swift
        override func viewDidLoad() {
            super.viewDidLoad()
         
            confirmationLabel.text = "Thank you for your order! Your wait time is approximately \(minutesToPrepare) minutes."
        }
      ```

  - If you build and run your app, you should be able to follow the entire workflow. You can select items, view your order, submit the order, and receive a confirmation with the wait time. Excellent job!
- **Clear Out Old Order Data**
  - There's one piece of cleanup that you still need to address. After the user has submitted the order, the order data is no longer needed. The best time to remove the order list is when the confirmation screen is dismissed. Luckily, you already have an unwind segue — the perfect way to perform this work. Check the segue's identifier to verify that you're unwinding from the order confirmation screen, then remove all order items. Once again, your notification observation code will do the rest.

    - ```swift
        @IBAction func unwindToOrderList(segue: UIStoryboardSegue) {
            if segue.identifier == "dismissConfirmation" {
                MenuController.shared.order.menuItems.removeAll()
            }
        }
      ```

  - Now when you return to the list of tabs, a new order is ready to be created.

- **Fixed an issue related to Order Submit**
  - Issue:
    - Previously, I created confirmOrder segue from the submit button because I couldn't create the segue from OrderTableView to OrderConfirmView. The Control+Drag didn't work.
    - So, when the user clicked the submit button, the alert didn't show and prepare minutes didn't work.
  - Solution:
    - I deleted the confirmOrder segue and Control+Drag from OrderTableView in the Document Outline to OrderConfirmView.
    - And then, set the identifier to "ConfirmOrder," re-created @IBSegueAction confirmOrder on OrderTableViewController.

#### Part Ten. Request Images

- The final part of this project — adding images — is one of the most important pieces of the app. Why wait until the end to begin requesting image data?
- When you develop complex apps, it's important to build the workflow first and worry about the details later. If you realize during user testing that your workflow is incorrect, you'll have a lot of code to move around. In this project, you're working with data from a server, so it's crucial to verify that you're requesting and parsing the data correctly before you shift attention to the other features of your app.
- Now that every view controller contains the right data, it's easy to add images to the screens.
- **Set the URL Cache Size**
  - Your app has at least three view controllers that might display image data. In the `MenuTableViewController` and the `OrderTableViewController`, you could add a `thumbnail` image near the left edge of every cell. And you'll want to display a larger version of the image on the `MenuItemDetailViewController`, depending on how you customized the layout of the details screen.
  - The workflow allows you to move quickly and easily between these screens, but it doesn't make sense to keep fetching the same image for every single screen. To resolve this, you can increase the size of the URL cache to hold larger amounts of image data. After the image is downloaded, it will remain in the cache, and subsequent requests for the same data will check the cache first before contacting the server. For an app of this size, using 25 megabytes of memory and 50 megabytes of disk space is a reasonable set of values for the URL cache.
  - To set the cache size, create a new `URLCache` inside `AppDelegate` so that the cache has been created before any network requests are made. Store this cache in your app's temporary directory, then assign it to the shared cache that the shared `URLSession` will use when performing requests.

    - ```swift
        class AppDelegate: UIResponder, UIApplicationDelegate {
            func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
                let temporaryDirectory = NSTemporaryDirectory()
                let urlCache = URLCache(memoryCapacity: 25_000_000, diskCapacity: 50_000_000, diskPath: temporaryDirectory)
                URLCache.shared = urlCache
         
                return true
            }
        }
      ```

  - Now, whenever you make requests for images from the server, the data will be placed automatically into this larger cache.
- **Add a Placeholder Image**
  - Since you're running the server for this project on your Mac, images should appear almost instantaneously. But in most real-world scenarios, image data takes time to load. Some sort of placeholder image will reassure the user as the images are being fetched. For the purposes of this project, the system-provided `photo.on.rectangle` image that you are already using for the image view is sufficient.
  - In `MenuTableViewController`, update the `configure(_:forItemAt:)` method to add the `photo.on.rectangle` image to the content configuration for the cell:

    - ```swift
        func configure(_ cell: UITableViewCell, forItemAt indexPath: IndexPath) {
            let menuItem = menuItems[indexPath.row]
         
            var content = cell.defaultContentConfiguration()
            content.text = menuItem.name
            content.secondaryText = menuItem.price.formatted(.currency(code: "usd"))
            content.image = UIImage(systemName: "photo.on.rectangle")
            cell.contentConfiguration = content
        }
      ```

  - Repeat this for the `OrderTableViewController’s` `configure(_:forItemAt:)` method:

    - ```swift
        func configure(_ cell: UITableViewCell, forItemAt indexPath: IndexPath) {
            let menuItem = MenuController.shared.order.menuItems[indexPath.row]
         
            var content = cell.defaultContentConfiguration()
            content.text = menuItem.name
            content.secondaryText = menuItem.price.formatted(.currency(code: "usd"))
            content.image = UIImage(systemName: "photo.on.rectangle")
            cell.contentConfiguration = content
        }
      ```

  - Build and run your app. You should see the placeholder image on the menu item cells, on the order item cells, and on the item details screen.
- **Request the Right Image**
  - The placeholder is in place. How will you find the right image? In each `MenuItem` is a property, `imageURL`, that says where to retrieve its corresponding image. You'll request that image whenever the cell is about to be displayed. Behind the scenes, `URLSession` will check the URL cache to see whether it's already retrieved the image. If it has, it will skip the network request entirely and fetch the image from memory; otherwise, it'll continue with the request to retrieve the image data. Either way, the process looks the same to your code.
  - Define a new method in `MenuController` that takes in an image URL as the first parameter and returns the `UIImage` data. This method is very similar to other requests you've written — except that you don't need to parse any JSON. Instead, you'll attempt to convert the data into an image and return it, or you'll throw an error if there's a failure.
  - You can add another case to the `Error` enum to identify issues with this request: `imageDataMissing`. 

    - ```swift
        enum MenuControllerError: Error, LocalizedError {
            case categoriesNotFound
            case menuItemsNotFound
            case orderRequestFailed
            case imageDataMissing
        }
        func fetchImage(from url: URL) async throws -> UIImage {
            let (data, response) = try await URLSession.shared.data(from: url)
         
            guard let httpResponse = response as? HTTPURLResponse, 
              httpResponse.statusCode == 200 else {
                throw MenuControllerError.imageDataMissing
            }
         
            guard let image = UIImage(data: data) else {
                throw MenuControllerError.imageDataMissing
            }
         
            return image
        }
      ```

  - Important: To reference `UIImage` in your code, you'll need to import `UIKit`.
  - When you configure the cell in `MenuTableViewController`, request the image data using `fetchImage`. In the case of success, you'll have an image to use; otherwise, you can return immediately from the method.
  - While the image is nice to have, it's not worth alerting the user if it fails to load, so you'll ignore any errors that are thrown by using an optional try statement - `try?` - rather than a do/catch statement. In a real-world situation, you might choose to log the error somewhere.

    - ```swift
        func configure(_ cell: UITableViewCell, forItemAt indexPath: IndexPath) {
            let menuItem = menuItems[indexPath.row]
         
            var content = cell.defaultContentConfiguration()
            content.text = menuItem.name
            content.secondaryText = menuItem.price.formatted(.currency(code: "usd"))
            content.image = UIImage(systemName: "photo.on.rectangle")
            cell.contentConfiguration = content
            Task.init {
                if let image = try? await MenuController.shared.fetchImage(from: menuItem.imageURL) {
                    // Image was returned
                }
            }
        }
      ```

  - Now that you've verified that you have an image, you can load it into the cell configuration's image property. That's an update to the interface, but since you have inherited the context of the `MainActor` in the `Task.init` closure, you can make that update directly.
  - But wait: For table view cells, you'll need to make an additional check. Recall that, in longer lists of data, cells will be recycled and reused as you scroll up and down the table. Since you don't want to put the wrong image into a recycled cell, check the index path where the cell is now located. If it's changed, you can skip setting the image view. The final step in the process is to update the cell's `contentConfiguration`, which will cause the cell to update its layout to accommodate the new image.

    - ```swift
        func configure(_ cell: UITableViewCell, forItemAt indexPath: IndexPath) {
            let menuItem = menuItems[indexPath.row]
         
            var content = cell.defaultContentConfiguration()
            content.text = menuItem.name
            content.secondaryText = menuItem.price.formatted(.currency(code: "usd"))
            content.image = UIImage(systemName: "photo.on.rectangle")
            cell.contentConfiguration = content
            Task.init {
                if let image = try? await MenuController.shared.fetchImage(from: menuItem.imageURL) {
                    if let currentIndexPath = self.tableView.indexPath(for: cell),
                            currentIndexPath == indexPath {
                        var content = cell.defaultContentConfiguration()
                        content.text = menuItem.name
                        content.secondaryText = menuItem.price.formatted(.currency(code: "usd"))
                        content.image = image
                        cell.contentConfiguration = content
                    }
                }
            }
        }
      ```

  - You might recall from the `iTunesSearch` project from a previous lesson that it is a good idea to cancel `Tasks` if the results will never be used. Since the `TableViewCells` can be scrolled off the screen and the images for those cells will no longer be needed, you can keep track of any image loading task for an `IndexPath` and cancel that `Task` if the cell is scrolled off the screen.
  - Start by adding a property to the `MenuTableViewController` that is a dictionary with keys of type `IndexPath` and values of type `Task<Void, Never>` (the return type of `Task.init`):
    - `var imageLoadTasks: [IndexPath: Task<Void, Never>] = [:]`
  - Now add an override for the `UITableViewDelegate` method `tableView(_:didEndDisplaying:forRowAt:)` to cancel the appropriate `Tasks`:

    - ```swift
        override func tableView(_ tableView: UITableView, didEndDisplaying cell: UITableViewCell, forRowAt indexPath: IndexPath) {
            // Cancel the image fetching task if it's no longer needed
            imageLoadTasks[indexPath]?.cancel()
        }
      ```

  - The entire view can also disappear before all the tasks finish loading the images. You should cancel any outstanding image loading tasks in the `viewDidDisappear(:)` method:

    - ```swift
        override func viewDidDisappear(_ animated: Bool) {
            super.viewDidDisappear(animated)
         
            // Cancel image fetching tasks that are no longer needed
            imageLoadTasks.forEach { key, value in value.cancel() }
        }
      ```

  - You can populate the `imageLoadTasks` dictionary when you set up each cell's image fetch and remove the entry once the image fetch task has completed:

    - ```swift
        imageLoadTasks[indexPath] = Task.init {
            if let image = try? await MenuController.shared.fetchImage(from: menuItem.imageURL) {
                if let currentIndexPath = self.tableView.indexPath(for: cell),
                      currentIndexPath == indexPath {
                    var content = cell.defaultContentConfiguration()
                    content.text = menuItem.name
                    content.secondaryText = menuItem.price.formatted(.currency(code: "usd"))
                    content.image = image
                    cell.contentConfiguration = content
                }
            }
            imageLoadTasks[indexPath] = nil
        }
      ```

  - You might have noticed that you need to repeat the configuration of all the details of the cell's `contentConfiguration` when the image becomes available. The system will efficiently make these updates as it renders the new cell, but you can make the code clearer and give the system more opportunities to optimize the image update by creating a new table view cell class.
  - Start by adding a new `.swift` file to your project called `MenuItemCell`, and define your new class as follows:

    - ```swift
        import UIKit
         
        class MenuItemCell: UITableViewCell {
         
        }
      ```

  - You'll want to add three properties to `MenuItemCell` for the item name, the price, and the image. These properties will have a special characteristic where they will invoke `setNeedsUpdateConfiguration()` on the cell if they change when they are set. `setNeedsUpdateConfiguration()` will tell the cell that it needs to render the change. `MenuItemCell` should look like the following:

    - ```swift
        import UIKit
         
        class MenuItemCell: UITableViewCell {
            var itemName: String? = nil
            {
                didSet {
                    if oldValue != itemName {
                        setNeedsUpdateConfiguration()
                    }
                }
            }
            var price: Double? = nil
            {
                didSet {
                    if oldValue != price {
                        setNeedsUpdateConfiguration()
                    }
                }
            }
            var image: UIImage? = nil
            {
                didSet {
                    if oldValue != image {
                        setNeedsUpdateConfiguration()
                    }
                }
            }
        }
      ```

  - When `setNeedsUpdateConfiguration()` is called, the cell's `updateConfiguration(using:)` method will be invoked to update the cell's configuration. The contents of this method will look familiar from the `configure(_:forItemAt:)` method you wrote earlier. Add the following to your `MenuItemCell` class:

    - ```swift
        override func updateConfiguration(using state: UICellConfigurationState) {
            var content = defaultContentConfiguration().updated(for: state)
            content.text = itemName
            content.secondaryText = price?.formatted(.currency(code: "usd"))
            content.prefersSideBySideTextAndSecondaryText = true
         
            if let image = image {
                content.image = image
            } else {
                content.image = UIImage(systemName: "photo.on.rectangle")
            }
            self.contentConfiguration = content
        }
      ```

  - This code will get the `defaultContentConfiguration` for the cell and update it for the current state (you'll learn more about `UICellConfigurationState` later). It then updates the content from the cell's `itemName` and `price` and makes sure that the `text` and `secondaryText` will be displayed side by side. Finally, it checks for an image and sets a default image if there is not one; otherwise, it sets the image from the cell's `image` property. Then it updates the cell's `contentConfiguration`, resulting in the cell being updated on the display.
  - To use this new cell, you'll want to open the `Main` storyboard, locate the prototype cell for the `MenuTableViewController`, and change its class from `UITableViewCell` to `MenuItemCell`. Then, in `MenuTableViewController`, replace the existing `configure(_:forItemAt:)` with the following:

    - ```swift
        func configure(_ cell: UITableViewCell, forItemAt indexPath: IndexPath) {
            guard let cell = cell as? MenuItemCell else { return }
         
            let menuItem = menuItems[indexPath.row]
         
            cell.itemName = menuItem.name
            cell.price = menuItem.price
            cell.image = nil
         
            imageLoadTasks[indexPath] = Task.init {
                if let image = try? await MenuController.shared.fetchImage(from: menuItem.imageURL) {
                    if let currentIndexPath = self.tableView.indexPath(for: cell),
                          currentIndexPath == indexPath {
                        cell.image = image
                    }
                }
                imageLoadTasks[indexPath] = nil
            }
        }
      ```

  - Now you just update the properties of the cell to appropriate values as you get them, and allow the cell configuration machinery to update the rendering of the cell.
  - Notice that you initially set the `cell.image` to `nil`. This is so that the `MenuItemCell` will set the placeholder image if your image has not been fetched yet. Recall that you may be reusing this cell from a previous row, so you want to configure all the properties to the values that you want when you set the cell up.
  - The `MenuItemCell` is also configured appropriately to be used for the `OrderTableViewController`. Repeat this process for `OrderTableViewController`. Remember to update the prototype cell for the `OrderTableViewController` in the storyboard to be `MenuItemCell` rather than the base `UITableViewCell`.

    - ```swift
        class OrderTableViewController: UITableViewController {
            var imageLoadTasks: [IndexPath: Task<Void, Never>] = [:]
            override func viewDidDisappear(_ animated: Bood) {
              super.viewDidDisappear(animated)

                imageLoadTasks.forEach { key, value in
                    value.cancel()
                }
            }
            /* ... */
            func configure(_ cell: UITableViewCell, forItemAt indexPath: IndexPath) {
                guard let cell = cell as? MenuItemCell else { return }
                let menuItem = MenuController.shared.order.menuItems[indexPath.row]

                cell.itemName = menuItem.name
                cell.price = menuItem.price
                cell.image = nil

                imageLoadTasks[indexPath] = Task.init {
                    if let image = try? await MenuController.shared.fetchImage(from: menuItem.imageURL) {
                        if let currentIndexPath = self.tableView.indexPath(for: cell),
                            currentIndexPath == indexPath {
                                cell.image = image.resizeImageTo(size: CGSize(width: 50, height: 50))
                        }
                    }
                }
                imageLoadTasks(indexPath) = nil
            }
        }
      ```

  - While the images are appearing, the standard cell design may not be best for this layout. It would be good practice to investigate ways to update the cell content configuration for `MenuItemCell` to have a more consistent layout when loading images of different sizes — but it is not required.
  - For `MenuItemDetailViewController`, the code is much easier. Set the image view once you've received the data:

    - ```swift
        func updateUI() {
            nameLabel.text = menuItem.name
            priceLabel.text = menuItem.price.formatted(.currency(code: "usd"))
            detailTextLabel.text = menuItem.detailText
         
            Task.init {
                if let image = try? await
                  MenuController.shared.fetchImage(from: menuItem.imageURL) {
                    imageView.image = image
                }
            }
        }
      ```

  - Build and run your app, and you should see images appear in the table view cells and on the detail screen. Nice work! Now your images can help the user decide which menu items are most appealing.

- **Fix Issue**
  - Issue:
    - When the category was clicked, items on the menu items page weren't displayed because they went down.
    - The reason was that the images of items were too big.
  - Solution:
    - I added resizeImageTo method to MenuTableViewController file and resized the images in `configure(_ forItemAt:)` method. And then, the items were shown well.

    - ```swift
        extension UIImage {
            func resizeImageTo(size: CGSize) -> UIImage? {
                UIGraphicsBeginImageContextWithOptions(size, false, 0.0)
                self.draw(in: CGRect(origin: CGPoint.zero, size: size))
                let resizedImage = UIGraphicsGetImageFromCurrentImageContext()!
                UIGraphicsEndImageContext()
                return resizedImage
            }
        }

        @MainActor
        class MenuTableViewController: UITableViewController {
            /* ... */
            func configure(_ cell: UITableViewCell, forItemAt indexPath: IndexPath) {
                guard let cell = cell as? MenuItemCell else { return }
                let menuItem = menuItems[indexPath.row]

                cell.itemName = menuItem.name
                cell.price = menuItem.price
                cell.image = nil

                imageLoadTasks[indexPath] = Task.init {
                    if let image = try? await MenuController.shared.fetchImage(from: menuItem.imageURL) {
                        if let currentIndexPath = self.tableView.indexPath(for: cell),
                            currentIndexPath == indexPath {
                              cell.image = image.resizeImageTo(size: CGSize(width: 50, height: 50))
                        }
                    }
                    imageLoadTasks[indexPath] = nil
                }
            }
        }
      ```

#### Guided Project - Wrap-Up

- Congratulations! This guided project required a lot of knowledge and setup to work with the server. You also created more view controllers than in any app you've built in this course. By now, you should feel very comfortable with passing data between view controllers and working with data that's not stored locally on the device. Be sure to save the completed project to your projects folder.
- As you build larger and more complex apps, you have more things to consider, such as how data is stored and where to keep network request logic. If you need more explanation about asynchronous requests, queues, and updating the user interface with server data, you can always reread the applicable lessons in this unit. In addition, you'll find that the labs are a great way to get more practice performing requests and updating views.
- **Stretch Goals**
  - Here are some things you can try adding to your menu app that would improve the experience for users in the real world:
    - Keep the order confirmation screen updated by calculating the number of minutes remaining for the order to be prepared and add a UIProgressView that fills up as time passes.
    - Create a local notification that's displayed 10 minutes before the order will be ready. For example, if the order will take 30 minutes, fire the alert on the user's device in 20 minutes.
  - Looking for even more of a stretch? In the following section, you can learn about state restoration by adding on to the Restaurant project. It's a stretch goal that's worth exploring in detail.

### Project Extension - State Restoration

- Consider the following scenario. A user starts an order by adding a menu item or two. Then they’re interrupted by an iMessage notification that results in 15 minutes of texting, emails, and web browsing. By the time the user comes back to your app, it's been terminated. Instead of resuming where they left off, they have to create the order from scratch — not a good experience! And their menu browsing state has been reset to the category list — another annoyance if your menu hierarchy is deep and you have a lot of items. (You've probably noticed these deficiencies as you've been developing the app.)
By implementing state restoration, you can make sure the user doesn't perceive any interruption in their activity. This is also crucial for iPad apps that support multiple windows to provide a good user experience.
- Beginning with iOS 13, state restoration is handled by your `UIWindowSceneDelegate` and achieved using the `NSUserActivity` class. `NSUserActivity` is a lightweight object that enables many different features across Apple platforms, including state restoration, handoff, Spotlight search indexing, and SiriKit. You can create `NSUserActivity` instances at key moments and provide the contextual information necessary to perform these tasks.
- In this extension to the guided project, you'll add state restoration to OrderApp. For this app, you'll maintain an `NSUserActivity` instance on `MenuController` that holds the current Order as well as the items needed to re-create the view controllers in your app that your user may have left off on.
- Here’s how it works. While the user is using your app, you'll track key pieces of information in an `NSUserActivity`'s `userInfo` dictionary. When your scene is sent to the background, iOS will request an `NSUserActivity` instance to be used the next time the scene is connected. When the scene reconnects, you will be provided with the same NSUserActivity instance, which you can use to rebuild the app's state so that the user can continue what they were doing.

#### Part 1. ​Handle the Order

- The most important thing to preserve in your app is the user's order. All your other model objects are fetched directly from the web service on demand, but orders need to be stored locally. Here’s how to persist an order model object.
- **Add Data Persistence for the Order**
  - You'll be using `NSUserActivity`'s `userInfo` dictionary to track information regarding your app's state. The dictionary's values can be of the following types: `Array`, `Data`, `Date`, `Dictionary`, `NSNumber`, `Set`, `String`, or `URL`. Notice that you cannot include custom types — or even `Codable` types. To get around this limitation, you'll store the `Order` as an encoded JSON Data object, created the same way as when you send an `Order` to the server.
  - First, create a new Swift file named “Restoration.swift” that will hold details specific to your state restoration strategy. In this file, create a new extension for `NSUserActivity` with the following order property:

    - ```swift
        extension NSUserActivity {
         
            var order: Order? {
                get {
                    guard let jsonData = userInfo?["order"] as? Data else {
                        return nil
                    }
         
                    return try? JSONDecoder().decode(Order.self, from: jsonData)
                }
                set {
                    if let newValue = newValue, let jsonData = try? JSONEncoder().encode(newValue) {
                        addUserInfoEntries(from: ["order": jsonData])
                    } else {
                        userInfo?["order"] = nil
                    }
                }
            }
        }
      ```

  - When in use, this will appear to be a stored property on `NSUserActivity`, though it's more of a facade. The `get` implementation attempts to load an `Order` from the underlying `userInfo` dictionary, and set takes the provided `Order` instance and encodes it to `userInfo` as JSON data, both using the same order key. You can now easily get and set an Order on an instance of `NSUserActivity`, but where will the activity instance live? A good place would be right alongside your existing shared `order` instance on `MenuController`.
  - Create a new property on `MenuController`: `var userActivity = NSUserActivity(activityType: "com.example.OrderApp.order")`
  - The activity type parameter is a `String` in a reverse - DNS format that should be unique to your identity and app. Typically this will include your company's website domain, the app name, and any further information to distinguish what the activity is for.
- **Save and Load the Order**
  - When should you set order on your new `userActivity` instance? A good time would be whenever the model changes. You already have `didSet` implemented for the order property on `MenuController` to post a notification. Update the `MenuController`'s `didSet` implementation for the order property to assign it to the `userActivity`.

    - ```swift
        var order = Order() {
            didSet {
                NotificationCenter.default.post(name: MenuController.orderUpdatedNotification, object: nil)
                userActivity.order = order
            }
        }
      ```

  - Now it's time to get into the state restoration APIs.

#### Part 2. Opt In to State Restoration in the Scene Delegate

- Adopting state restoration for your scenes starts with two methods, `stateRestorationActivity(for:)` and `scene(_:restoreInteractionStateWith:)`. 
  - The first is called when your scene enters the background; it is invoked by UIKit, which is requesting an `NSUserActivity` to pass back to you when your scene reconnects. 
  - The second method is called after your scene connects and the storyboard and views are loaded, but before the first transition to the foreground. The `restoreInteractionStateWith` parameter will contain the `NSUserActivity` instance that you provided from the first method. You'll then use that to rebuild the app's state.
- Implement the two methods in `SceneDelegate`. In the first method, return the `userActivity` from the `MenuController`'s shared instance. In the second method, check the provided `NSUserActivity` for an order. If an order is present, assign it to the `MenuController`'s shared instance.

  - ```swift
      func stateRestorationActivity(for scene: UIScene) -> NSUserActivity? {
          return MenuController.shared.userActivity
      }
       
      func scene(_ scene: UIScene, restoreInteractionStateWith​   stateRestorationActivity: NSUserActivity) {
          if let restoredOrder = stateRestorationActivity.order {
              MenuController.shared.order = restoredOrder
          }
      }
    ```

- At this point, the Order is preserved across app terminations — an excellent help to your users! Try it out by running the app, adding items to the order, then returning to the Home screen. (In Simulator, you can return to the Home screen by pressing Command+Shift+H, using the Device > Home menu option, or tapping the Home button.) After you've returned to the Home screen, stop the app using Xcode and re-launch it.
- Do you see your restored order? Great! But what about the user interface? You were placed back on the categories list rather than the screen you were last on. You'll work on resolving this issue next.
- Note: If you do not first go to the Home screen, the `stateRestorationActivity(for:)` method will not be called, and you won't see proper state restoration. Keep this in mind while debugging your app's state restoration feature. Also, if your app crashes, the state restoration information may be thrown away; this helps ensure that the app does not continue crashing on re-launch due to a bad state.

#### Part 3. Update the Storyboard

- Next, you’ll need to think about how you're going to place the user back on the screen they left the app on. Consider the app's workflow and view controllers and the properties required to initialize them.
- The first time the user opens the app, they land on `CategoryTableViewController`, which is the first tab and requires no initial values. This controller immediately fetches categories from the server. The user can then select a category, which is passed to `MenuTableViewController`, and a list of `MenuItems` is fetched for the category. From this list, the user can select a `MenuItem`, which is passed to `MenuItemDetailViewController`, where they can then add the item to their order. At any point, the user can switch to `OrderTableViewController` using the tab bar controller and then submit their order, which presents `OrderConfirmationViewController`, or use the tab bar controller to return to whichever view controller they came from.
- From this analysis, you can identify that `CategoryTableViewController` and `OrderTableViewController` are created when the tab bar controller is initialized from the storyboard. `MenuTableViewController` and `MenuItemDetailViewController`, on the other hand, both require a parameter to be initialized and are then pushed on the navigation stack that also contains `CategoryTableViewController`. Finally, there is `OrderConfirmationViewController`, which is modally presented when an order has been successfully submitted. Normally, this all happens through the use of segues and actions defined in the storyboard.
- How will you rebuild the navigation stack based on where the user left off?
- As you may recall, you created custom initializers for `MenuTableViewController` and `MenuItemDetailViewController`. However, those initializers have an `NSCoder` parameter that was provided when an `@IBSegueAction` was called. How can you use your custom initializer outside an `@IBSegueAction`? Where will you get the `NSCoder` to call the initializer? From the storyboard, of course!
- There is an entire class, `UIStoryboard`, that allows you to interact with storyboards programmatically — most importantly, to initialize view controllers defined within the storyboard. To do that, however, you need an identifier for the view controllers you're interested in. Open the Main storyboard and select the `MenuTableViewController`. In the Identity inspector, find the Storyboard ID parameter. This is the identifier you will use to initialize the scene programmatically. You need to set IDs for both MenuTableViewController and MenuItemDetailViewController — set them to “menu” and “menuItemDetail,” respectively. 
- You may notice a Restoration ID attribute as well. While this sounds like something you'd be interested in for state restoration, it's not used in the approach you're taking here. Prior to iOS 13 and the introduction of scene - based apps, you would have used this attribute, but now state restoration is managed through `NSUserActivity`. If you find that you need to support state restoration on iOS 12 or earlier, take a look at the [developer documentation](https://developer.apple.com/documentation/uikit/view_controllers/preserving_your_app_s_ui_across_launches).
- With identifiers set for both `MenuTableViewController` and `MenuItemDetailViewController`, you are ready to move forward. These will be the only view controllers that you will initialize programmatically. `OrderConfirmationViewController` is a bit more transient and not entirely necessary to restore; decisions like this are ones you and/or your team will weigh when developing your own apps.

#### Part 4. Preserve View Controller State

- As your users interact with the app, you'll need to keep track of its state, such as which screen they're viewing and the required values to re-create it. OrderApp is relatively lightweight, with few screens and required parameters for them. The table below outlines the view controllers, their required parameters, and their storyboard identifier, if any.
- | View Controller Class | Required Parameters | Storyboard Identifier |
  |:---|:---|:---|
  | CategoryTableViewController | - | - |
  | MenuTableViewController | Category:String | Menu |
  | MenuItemDetailViewController | menuItem:MenuItem | MenuItemDetail |
  | OrderTableViewController | - | - |
- In `Restoration.swift`, create an enum with cases representing these four scenes.

  - ```swift
      enum StateRestorationController {
          case categories
          case menu(category: String)
          case menuItemDetail(MenuItem)
          case order
      }
    ```

- Each controller has a case and an associated value, if required. As the user navigates, you will need to keep track of which view controller they're on. This can be done using a `String` stored in your `NSUserActivity`'s `userInfo` dictionary. Create a nested `enum` within `StateRestorationController` named `Identifier` with a `RawValue` of type `String` that can be used to store an identifier for each controller:

  - ```swift
      enum StateRestorationController {
          enum Identifier: String {
              case categories, menu, menuItemDetail, order
          }
        ...
      }
    ```

- Each identifier should be associated with a case in `StateRestorationController`, so you'll create an `identifier` computed property that switches over `self` and returns the respective `Identifier`. This might feel redundant and strange, but it will be helpful soon. Your `StateRestorationController` enum should now look like the following:

  - ```swift
      enum StateRestorationController {
          enum Identifier: String {
              case categories, menu, menuItemDetail, order
          }
       
          case categories
          case menu(category: String)
          case menuItemDetail(MenuItem)
          case order
       
          var identifier: Identifier {
              switch self {
              case .categories: return Identifier.categories
              case .menu: return Identifier.menu
              case .menuItemDetail: return Identifier.menuItemDetail
              case .order: return Identifier.order
              }
          }
      }
    ```

- This enum now has enough state information about each screen to be stored in the `userInfo` dictionary on `MenuController`'s `userActivity` instance. You can use the same pattern you used for the `order` property on your `NSUserActivity` extension to create each of the following additional properties:
  - `var controllerIdentifier: StateRestorationController.Identifier?`
  - `var menuCategory: String?`
  - `var menuItem: MenuItem?`
- Before looking at the example code below, try creating the properties in your `NSUserActivity` extension with custom `get` and `set` implementations that add and read values from the `userInfo` dictionary.
- Here is an example of what those implementations may look like:

  - ```swift
      var controllerIdentifier: StateRestorationController.Identifier? {
          get {
              if let controllerIdentifierString = userInfo?["controllerIdentifier"] as? String {
                  return StateRestorationController.Identifier(rawValue: controllerIdentifierString)
              } else {
                  return nil
              }
          }
          set {
              userInfo?["controllerIdentifier"] = newValue?.rawValue
          }
      }
       
      var menuCategory: String? {
          get {
              return userInfo?["menuCategory"] as? String
          }
          set {
              userInfo?["menuCategory"] = newValue
          }
      }
       
      var menuItem: MenuItem? {
          get {
              guard let jsonData = userInfo?["menuItem"] as? Data else {
                  return nil
              }
              return try? JSONDecoder().decode(MenuItem.self, from: jsonData)
          }
          set {
              if let newValue = newValue, let jsonData = try? JSONEncoder().encode(newValue) {
                  addUserInfoEntries(from: ["menuItem": jsonData])
              } else {
                  userInfo?["menuItem"] = nil
              }
          }
      }
    ```

- Now, as the user navigates your app, you can easily update `userActivity` on `MenuController`. Create a method on `MenuController` to assist; it should take a `StateRestorationController` parameter:

  - ```swift
      func updateUserActivity(with controller: StateRestorationController) {
          switch controller {
          case .menu(let category):
              userActivity.menuCategory = category
          case .menuItemDetail(let menuItem):
              userActivity.menuItem = menuItem
          case .order, .categories:
              break
          }
       
          userActivity.controllerIdentifier = controller.identifier
      }
    ```

- This method switches over the passed-in `controller` parameter and extracts the available associated values, assigning them to the corresponding property on `userActivity`. Finally, it sets the `controllerIdentifier` property, which you'll use to determine which screen the user was last on.
- Your last step for preserving view controller state is to call this method within each view controller at the appropriate time. The best time is when `viewDidAppear(_:)` is called. Add the following methods to each corresponding controller.

  - ```swift
      /* In CategoryTableViewController */
      override func viewDidAppear(_ animated: Bool) {
          super.viewDidAppear(animated)
          MenuController.shared.updateUserActivity(with: .categories)
      }
       
      /* In MenuItemDetailViewController */
      override func viewDidAppear(_ animated: Bool) {
          super.viewDidAppear(animated)
          MenuController.shared.updateUserActivity(with: .menuItemDetail(menuItem))
      }
       
      /* In MenuTableViewController */
      override func viewDidAppear(_ animated: Bool) {
          super.viewDidAppear(animated)
          MenuController.shared.updateUserActivity(with: .menu(category: category))
      }
       
      /* In OrderTableViewController */
      override func viewDidAppear(_ animated: Bool) {
          super.viewDidAppear(animated)
          MenuController.shared.updateUserActivity(with: .order)
      }
    ```

- Each implementation calls `updateUserActivity(with:)`, passing information about the view controller that appeared. `MenuController.shared.userActivity` now has the information needed to restore your app's state.

#### Part 5. Restore View Controller State

- You're finally ready for the last piece of state restoration. Most of the complexity will lie within the `scene(_:restoreInteractionStateWith:)` method in `SceneDelegate`. In this method, you'll use information provided on the `NSUserActivity` instance to return the user to the state they are expecting.
- As you're aware, all that information can be neatly packaged in the `StateRestorationController` enum. It would be nice if you could initialize a value of `StateRestorationController` using a provided `NSUserActivity`. Add the following custom initializer to `StateRestorationController` in the `Restoration` file.

  - ```swift
      init?(userActivity: NSUserActivity) {
          guard let identifier = userActivity.controllerIdentifier else {
              return nil
          }
       
          switch identifier {
          case .categories:
              self = .categories
          case .menu:
              if let category = userActivity.menuCategory {
                  self = .menu(category: category)
              } else {
                  return nil
              }
          case .menuItemDetail:
              if let menuItem = userActivity.menuItem {
                  self = .menuItemDetail(menuItem)
              } else {
                  return nil
              }
          case .order:
              self = .order
          }
      }
    ```

- The initializer checks the `userActivity` for a `controllerIdentifier` — the last screen the user was on. Then, if one is present, the initializer switches over the value and extracts the relevant properties. Notice the assignment to `self` in each case; in failable enum initializers, you either assign `self` to a value or return `nil`.
- Now you can use this initializer in the `scene(_:restoreInteractionStateWith:)` method in `SceneDelegate`. While you're there, add a guard-let statement to check a few other important preconditions before you fill out the rest of the method. (Unless otherwise mentioned, each of the code segments below should be appended to your implementation of `scene(_:restoreInteractionStateWith:)`.)

  - ```swift
      func scene(_ scene: UIScene, restoreInteractionStateWith stateRestorationActivity: NSUserActivity) {
          if let restoredOrder = stateRestorationActivity.order {
              MenuController.shared.order = restoredOrder
          }
       
          guard
              let restorationController = ​StateRestorationController(userActivity:​ stateRestorationActivity),
              let tabBarController = window?.rootViewController as?​ UITabBarController,
                  tabBarController.viewControllers?.count == 2,
              let categoryTableViewController =​ (tabBarController.viewControllers?[0] as?​ UINavigationController)?.topViewController as?​ CategoryTableViewController
          else {
              return
          }
      }
    ```

- The `guard` statement is checking a number of things. 
  - First, it tries to initialize `StateRestorationController` using `stateRestorationActivity`. 
  - Next, it verifies that `rootViewController` is a `UITabBarController` and has two tabs. In state restoration, it's important to verify that things are as you expect, as you do not want to attempt to restore an invalid state. 
  - Finally, the first tab is verified to be a navigation controller whose `topViewController` is `CategoryTableViewController` — and, if so, the reference is captured.
- Now you have an instance of `StateRestorationController` and references to the `tabBarController` and `categoryTableViewController`. Next, you'll switch over `restorationController` to determine which screen should be displayed. If you recall, `CategoryTableViewController` is displayed by default, and `OrderTableViewController` is already loaded in the second tab. You will not need to initialize them programmatically. `MenuTableViewController` and `MenuItemDetailViewController`, however, both need to be initialized and presented. Using the `UIStoryboard` APIs and your storyboard IDs, you will initialize and present both when their case is matched.
- Below your guard-let statement, initialize a reference to your storyboard.
  - `let storyboard = UIStoryboard(name: "Main", bundle: nil)`
- This API simply takes the filename of your storyboard. If one does not match, your app will crash — so it's important to get it right.
- Begin the switch statement. The `.categories` and `.order` cases are straightforward: for `.categories` you don't need to do anything, and for `.order` you only need to switch the selected tab.

  - ```swift
      switch restorationController {
          case .categories:
              break
          case .order:
              tabBarController.selectedIndex = 1
      }
    ```

- For `.menu`, you will extract the associated category value and then use the `UIStoryboard` method `instantiateViewController(identifier:creator:)`. This method takes an identifier (your storyboard ID) and has a creator closure, where you'll initialize your view controller. Also notice that the closure is passed that much-needed `NSCoder` instance!

  - ```swift
      case .menu(let category):
          let menuTableViewController = storyboard.instantiateViewController(identifier: restorationController.identifier.rawValue, creator: { (coder) in
              return MenuTableViewController(coder: coder, category: category)
          })
          categoryTableViewController.navigationController?.pushViewController(menuTableViewController, animated: true)
    ```

- You pass `restorationController.identifier.rawValue` as the `identifier`, and the method uses the provided closure to create and return an instance of `MenuTableViewController` — similar to your `@IBSegueAction` implementations. That instance is then pushed onto `categoryTableViewController`'s navigation stack.
- The last case to cover is `.menuItemDetail`. It will be handled similarly to `.menu`, except that you first need to push a `MenuTableViewController`. Remember that you can't get to an item's detail without first selecting it from the menu.

  - ```swift
      case .menuItemDetail(let menuItem):
          let menuTableViewController = storyboard.instantiateViewController(identifier: StateRestorationController.Identifier.menu.rawValue, creator: { (coder) in
              return MenuTableViewController(coder: coder, category: menuItem.category)
          })
       
          let menuItemDetailViewController = storyboard.instantiateViewController(identifier: restorationController.identifier.rawValue) { (coder) in
              return MenuItemDetailViewController(coder: coder, menuItem: menuItem)
          }
       
          categoryTableViewController.navigationController?.pushViewController(menuTableViewController, animated: false)
          categoryTableViewController.navigationController?.pushViewController(menuItemDetailViewController, animated: false)
      }
    ```

- First, you initialize a `MenuTableViewController` using `StateRestorationController.Identifier.menu.rawValue` as the identifier. In the creator closure, you pass `menuItem.category` for the category. Finally, you initialize `MenuItemDetailViewController` using `menuItem` and push both of them to `categoryTableViewController`'s navigation controller.
- Whew! That was a lot of work — but consider how much time and frustration you'll save among all your users. You may even save a few orders that otherwise would have been abandoned!

#### Part 6. Test Your Code

- You're now ready to test the state restoration in your app. In real-world scenarios, state restoration happens when a long interval of time elapses between uses of an app. But it would be cumbersome and not easily repeatable to test under this condition. Instead, you might think to force-quit the app on your device or in Simulator and then relaunch it. But iOS has a protection mechanism: When a user manually terminates an app, iOS discards all state restoration data to prevent returning the app to a bad state.
- **Simulate State Restoration**
  - To properly test state restoration, launch your app and add an item to your order. Go back to the Home screen in Simulator, then stop the project from running with Xcode. (To give the app time to complete state preservation, you may need to wait a few beats before stopping the project.) Now relaunch your app from Xcode.
  - Congratulations! You've given your users a smooth experience. Every time they return to your app, they'll be able to pick up exactly where they left off.
  - Great app developers aspire to the high standards that Apple sets. And part of being a great app developer is going beyond core functionality to provide an excellent user experience. iOS is designed from the ground up to accommodate many kinds of users in many situations. For example, you can use APIs to localize your app for international audiences or to provide accommodations for users with disabilities. Not every user will notice, but many more care (and care deeply) than you might think. They’ll be pleased they can use your app with ease.
- One final note: In the real world, a restaurant might update its menu from time to time. Now that you're persisting parts of your menu data locally, what happens if the data becomes stale? Your user could view details of a discontinued item and even add the item to their order. Your restored UI might also be invalid. Of course, there are ways to handle these kinds of situations. You'll have to reconcile the stale state of your UI with the new data that’s fetched from the server. Furthermore, your server should always verify submissions from clients and respond with appropriate error messaging.
Interested in learning more about state restoration? Check out [Preserving Your App's UI Across Launches](https://developer.apple.com/documentation/uikit/view_controllers/preserving_your_app_s_ui_across_launches).

#### Summary - State Restoration

- Congratulations! You've learned some of the most important concepts behind modern apps.
- This unit covered some complex topics. First, you learned about closures and how they can be used to pass code between objects and create blocks of code that can be executed later. Then you learned how to use closures to build animations. And finally, you learned how to execute network requests to fetch information from the web and send information back.
- Now that you can work with public APIs, your apps are no longer limited to information that your users add to your app. You have the entire world wide web at your disposal. 
- In the next unit, you'll learn about collection views—iOS workhorses that allow you to display large collections of information in nearly infinite variety. You'll strengthen your existing skills and discover some common patterns that will make future exploration in UIKit feel familiar.

## Unit 3 Advanced Data Display

- You use apps that scroll through collections of information every day. When iOS was first introduced, table views were the primary interface for viewing such data. But things have changed a lot since those early days. Apps are much more complex and capable, and they require more varieties of information presentation than simple scrolling lists. Modern apps display data collections in a variety of ways to support their users with efficient and intuitive interfaces.
- In this unit, you'll learn all about collection views — the best way to create complex, adaptive layouts for displaying lots of information.
- To begin, you'll learn the basics of collection views and the similarities and differences between them and table views. Next you'll take a quick look at a Swift language feature called generics, which are a powerful way to create reusable and flexible code.
- Then you'll work through three lessons that lead you through the capabilities of collection views and compositional layouts — the standard way to define how a collection view displays data. Along the way, you'll learn about diffable data sources, which enable you to keep your collection view up to date with changing data without the headaches of the delegate callbacks you learned in the previous unit. You'll spend the final lesson learning about user notifications - a key feature of iOS that many apps use to keep the user informed when they're not running.

### Lesson 3.1 Collection Views

- You're now familiar with lists of data displayed using `UITableView`, but what about other layout arrangements? That's where `UICollectionView` comes in. Collection views allow you to arrange content in a simple scrolling grid or in complex custom layouts to suit your app's style and needs.
- You don't need to look far to find examples of collection views on iOS. The layout of the iOS Home screen, with its paging grids of app icons and folders, is an example of what can be achieved with collection views.
- In this lesson, you'll learn how to arrange data in a grid using the basic building blocks for collection views and a simple compositional layout. In an upcoming lesson, you'll learn how to create complex custom layouts for well-organized and attractive apps.
- Previously, you learned that table views are among the most-used views in iOS. Collection views are another. In fact, when `UICollectionView` was introduced, many developers speculated that it could replace `UITableView` entirely. Because of its extreme layout flexibility, you can use a collection view to present a list of data that looks and feels just like a table view. (Though it's worth noting that a table view has some unique out-of-the-box features you won't find with a collection view.)
- Collection views can be found in many of the stock iOS apps, including App Store, Files, News, Photos, Shortcuts, Watch, and Weather.
- Shortcuts, Files, and Photos all arrange their items in a vertically scrolling grid. Each of these apps also has collection view layouts in other areas. In the Face Gallery screen of the Watch app, the whole view scrolls vertically, but each section scrolls horizontally. This two-axis (or orthogonal) scrolling is also used in the App Store, News, and Watch apps.

#### 3.1.1 Anatomy of a Collection View

- Collection views and collection view controllers are very similar to table views and table view controllers, and they follow many of the same architectural principles. A collection view's main content, typically a set of items from your model data, is displayed by dequeueing reusable cells, which are subclasses of `UICollectionViewCell`. `IndexPaths` are used to identify items across sections as well as in the data source (`UICollectionViewDataSource`) and delegate (`UICollectionViewDelegate`) protocol APIs.
- To start building your first collection view, open Xcode and create a new project using the iOS App template. Name the project "BasicCollectionView." Delete the ViewController file and the view controller scene in the Main storyboard.
- You will be building a collection view with single column of items, similar to what you've done previously with table views. You can choose whatever you like for the content—this guide will be using U.S. state names.
- Add a new Cocoa Touch class to your project, subclassing `UICollectionViewController`, named `BasicCollectionViewController`. Open the Main storyboard and drag a collection view controller in from the Object library. Set its class to `BasicCollectionViewController` using the Identity inspector, then embed it in a navigation controller. Set the navigation controller as the initial view controller using the Attributes inspector.
- Interface Builder provides a prototype cell in the collection view; at the moment, it's a small empty square. Select the cell and set its Reuse Identifier to "Cell". Set the cell's background color to "System Teal Color".
- Add a label to the center of the cell and add constraints to center it horizontally and vertically. Create a new Cocoa Touch class, subclassing `UICollectionViewCell`, named `BasicCollectionViewCell`. Set the cell's class to `BasicCollectionViewCell` using the Identity inspector in the Main storyboard. You'll need an outlet for the label in `BasicCollectionViewCell`.
- It can be tricky to get the assistant editor to display `BasicCollectionViewCell.swift`. Start by selecting the Cell(Content View) in the Document Outline or on the canvas, then open the assistant editor. You'll see the code for your collection view controller. In the navigation bar above the assistant editor, click "BasicCollectionViewController.swift," then select "BasicCollectionViewCell.swift" from the dropdown menu. (Alternatively, there is a previous/next control on the right-hand side of the navigation bar that you can use to switch between the two files.) Finally, Control-drag from the label to the assistant editor to create an @IBOutlet named label.
- At this point, you have a few key items in place: Your collection view controller scene is using your custom subclass, `BasicCollectionViewController`, and your cell is designed, has a subclass, and has a reuse identifier.

#### 3.1.2 Collection View Layouts

- One of the main differences between collection views and table views is how layout is handled. A table view computes its own layout as a linear list of cells in one of several formats. A collection view relies on a separate layout instance that determines how cells are placed and sized within it. This separation of responsibility is an extra complication, but it allows for great flexibility in presenting data. When you create a new collection view in a storyboard, it comes with a flow layout instance by default. You can find the layout object in the Document Outline as Collection View Flow Layout. Its type is `UICollectionViewFlowLayout`, which is a subclass of `UICollectionViewLayout` — both of which are provided by `UIKit`.
- The preferred method, however, is to use a compositional layout. Compositional layouts, which allow for more flexibility than flow layouts, are composed in code by combining sections, groups, and items into the layout you need. You'll learn more about how to put these elements together in different ways in a future lesson. For now, you'll start with a simple implementation.
- **Simple Compositional Layout**
  - Open `BasicCollectionViewController` and, under viewDidLoad, add a new method called `generateLayout`.

    - ```swift
        private func generateLayout() -> UICollectionViewLayout {
         
        }
      ```

  - You'll compose your layout inside this method and return the layout back to the caller.
  - A compositional layout consists of three main parts. The outermost part is a section, which you can relate to the sections in a table view — it typically maps to the top-level structure of the data you're displaying. A section holds groups, which are containers for cells or other groups. Groups are most commonly used to present their elements in a linear layout, either vertically or horizontally. An item is contained within a group. It's the lowest level of a compositional layout, and, like a table view cell, typically corresponds to one model object.
    - <img src="./resources/collectionView.png" alt="Collection View" width="200" />
  - The first thing you need to do is define your item. Each item in this app will be displayed in the same way, so you'll only need one item definition. Add the following to your generateLayout method:

    - ```swift
        let itemSize = NSCollectionLayoutSize(
            widthDimension: .fractionalWidth(1.0),
            heightDimension: .fractionalHeight(1.0)
        )
         
        let item = NSCollectionLayoutItem(layoutSize: itemSize)
      ```

  - Here, the item's size is set to the full width and height of the group, and then an `NSCollectionLayoutItem` is created from the `NSCollectionLayoutSize` object. Next, add the following below the item definition to define a group:

    - ```swift
        let groupSize = NSCollectionLayoutSize(
            widthDimension: .fractionalWidth(1.0),
            heightDimension: .absolute(70.0)
        )
        
        /*
        // hotizontal(layoutSize:subitem,count:) was deprecated in iOS 16.0
        let group = NSCollectionLayoutGroup.horizontal(
            layoutSize: groupSize,
            subitem: item,
            count: 1
        )
        */
        let group = NSCollectionLayoutGroup.horizontal(
            layoutSize: groupSize,
            repeatingSubitem: item,
            count: 1
        )
      ```

  - This defines a group size, where the width is equal to the full width of the section and the height is 70 points. This group represents a row in your list of items, so the group definition creates a horizontal `NSCollectionLayoutGroup` with the size that was just defined. The group is set to contain one item per group.
  - Next, add the following after the group definition to create a section holding the group and a layout to contain the section. Finally, return the layout.

    - ```swift
        let section = NSCollectionLayoutSection(group: group)
         
        let layout = UICollectionViewCompositionalLayout(section: section)
         
        return layout
      ```

  - There's one more step required: You need to tell the collection view to use the new layout. Add the following to the bottom of viewDidLoad: `collectionView.setCollectionViewLayout(generateLayout(), animated: false)`
  - This, as you might expect, sets the layout for the collection view to the layout created by `generateLayout`. viewDidLoad should have only the call to super and the line you just added; remove all other comments and statements.

    - ```swift
        override func viewDidLoad() {
            super.viewDidLoad()
            collectionView.setCollectionViewLayout(generateLayout(), animated: false)
        }
      ```

  - Be sure to remove the call to `collectionView.register(_:forCellWithReuseIdentifier:)`, since a prototype cell has already been set up in the storyboard.
- **Collection View Data Source**
  - To see your layout in action, first add a variable of type [String] near the top of your `BasicCollectionViewController` file, under the definition for `reuseIdentifier` and above the class definition. Add some values to the array. This example uses the list of U.S. states.

    - ```swift
        private let items = [
            "Alabama", "Alaska", "Arizona", "Arkansas", "California", "Colorado", "Connecticut", "Delaware", "Florida", "Georgia", "Hawaii", "Idaho", "Illinois", "Indiana", "Iowa", "Kansas", "Kentucky", "Louisiana", "Maine", "Maryland", "Massachusetts", "Michigan", "Minnesota", "Mississippi", "Missouri", "Montana", "Nebraska", "Nevada", "New Hampshire", "New Jersey", "New Mexico", "New York", "North Carolina", "North Dakota", "Ohio", "Oklahoma", "Oregon", "Pennsylvania", "Rhode Island", "South Carolina", "South Dakota", "Tennessee", "Texas", "Utah", "Vermont", "Virginia", "Washington", "West Virginia", "Wisconsin", "Wyoming"
        ]
      ```

  - The variable is marked `private` because it doesn't need to be accessed outside the type it's defined in.
  - Now that you have some model data, it's time to set up the required methods to implement `UICollectionViewDataSource`. This process will look familiar to you from working with table views. Since you subclassed `UICollectionViewController`, your view controller already adopts the `UICollectionViewDataSource` protocol and there are base definitions of the methods you need already in the file.
  - First, locate the definition for `numberOfSections(in:)`. If you check the method definition through Quick Help (by Option-clicking the method name), you'll see that user Discussion it says, "if you do not implement this method, the collection view uses a default value of 1." Since your collection view will only have one section, remove the definition for the `numberOfSections(in:)` method.
  - Next is the `collectionView(_:numberOfItemsInSection:)` method. Remove the comment with the #warning and change the method to return the count of the items array you created above. Since there's only one section in your collection view, you'll return the total number of items that the collection will contain.

    - ```swift
        override func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
            return items.count
        }
      ```

  - The last data source method you need to implement is `collectionView(_:cellForItemAt:)`. The cells for your collection view are going to be `BasicCollectionViewCells`, not generic `UICollectionViewCells`, so add a type cast to `BasicCollectionViewCell` to the definition of cell on the first line. Next, remove the “Configure the cell” comment. Assign the cell's label text to the value of the current entry in your items array. Your subscript will use the `indexPath` passed in to the function and reference its item property. (For table views, you've used the row property of `IndexPath`; they're synonyms for the same value, so this is a style choice to make your code align to collection view terminology.)
  - Your method definition should look like this:

    - ```swift
        override func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
            let cell = collectionView.dequeueReusableCell(withReuseIdentifier: reuseIdentifier, for: indexPath) as! BasicCollectionViewCell
         
            cell.label.text = items[indexPath.item]

            return cell
        }
      ```

  - Build and run your app in Simulator and see how you did.
- **Cell Spacing**
  - The cells in the collection have expanded to fill the available space, which is what was configured above. That's not very readable, so now you'll create some space around the cells.
  - In generateLayout, first define a `CGFloat` constant above the `groupSize` definition called spacing. This will ensure that the item spacing is even and easily adjusted. Set its value to 10 points: `let spacing: CGFloat = 10`
  - Immediately after the definition for group, set the `contentInsets` property as follows:

    - ```swift
        group.contentInsets = NSDirectionalEdgeInsets(
            top: spacing,
            leading: spacing,
            bottom: 0,
            trailing: spacing
        )
      ```

  - The `contentInsets` property is of type `NSDirectionalEdgeInsets`. Each property of the `NSDirectionalEdgeInsets` struct sets the space between its edge and the next object in that direction — in this case, the leading and trailing edge of the layout itself. (The `contentInsets` property is also present on `NSCollectionLayoutItem`; it's a handy layout tool at every level of the hierarchy.)
- **Rotation**
  - Try rotating Simulator while the app is running (with the button in its menu bar or by pressing Command plus the left or right arrow key). Notice that the cells automatically adjust for the screen's new width. You'll see variations on this for different devices and different screen sizes and orientations. A flow layout provides some basic built-in adaptive behavior, but compositional layout is much more capable of automatically preserving the intent of your layout (for example, one column of items) without requiring custom code to react to environment changes such as device orientation.
  - Adjusting your layout to better handle the device size will be covered in a future lesson.

#### Lab - Emoji Dictionary

- In this lab, you will revisit a project from your previous lesson on table views: EmojiDictionary. This time, though, you will implement it using a collection view rather than a table view.

##### Step 1 Review Starter Project

- This lab has a starter project with the basic navigation and editing features already set up, so you can focus on the collection view elements themselves. Open the starter project and walk through the code so you understand what's already there and how things fit together. Once you've done that and feel comfortable, move on to step 2.

##### Step 2 Cell Setup

- Open the Main storyboard and select the collection view in the Emoji Dictionary Scene. In the Attributes inspector, at the top, change Items from 0 to 1 to add a new item to the collection view. Click the newly added cell (directly in the scene or via the Document Outline). At the top of the Attributes inspector, under Collection Reusable View, set the Identifier to `Item`.
- Drag a `label` from the Object library to the new cell and change its text to “😀” and the font to `System 24.0`. Next, drag another `label` from the Object library and place it to the right of the first label. Set its text to “Name Label” and the font to `Title 3`. Drag one more label from the Object library and place it below the second label. Change its text to “Description Label” and the font to `Subhead`.
- Select the Name Label and Description Label and, using the Embed In tool, embed them in a stack view. With the new stack view still selected, Command-click the emoji label to select it as well. Using the Embed In tool, embed the selected views in another stack view.
- With the outermost stack view selected, add top and bottom constraints equal to 11. Add a leading constraint equal to 18 and trailing constraint equal to 20. Using the Attributes inspector, make sure Alignment and Distribution are both set to Fill and that the spacing is 8.
- Select the vertical stack view inside the horizontal stack view and set the Alignment to Fill, Distribution to Fill Equally, and Spacing to 0.
- To keep the emoji label and the vertical stack view from being separated, select the emoji label and, using the Size inspector, change the Horizontal Content Hugging Priority to 252. For storyboard purposes, select the cell and drag its right border so it is 400 points wide (the actual width of the item is calculated by the layout).
With the cell still selected, open the Identity inspector and change the class to EmojiCollectionViewCell. Switch to the Connections inspector and connect the outlets for nameLabel, descriptionLabel, and symbolLabel to their appropriate UI elements in the item.
- Finally, using the Connections inspector, drag from the connector next to “selection” to the navigation controller on the storyboard. For the segue type, choose Present Modally.

##### Step 3 Complete the View Controller

- With everything wired up, open `EmojiCollectionViewController` and locate the definition for `viewDidLoad`. Under the call to super, define an item size for each cell in the collection view. Each item should be the full size of its container.

  - ```swift
      let item = NSCollectionLayoutItem(
          layoutSize: NSCollectionLayoutSize(
              widthDimension: .fractionalWidth(1),
              heightDimension: .fractionalHeight(1)
          )
      )
    ```

- As in the `BasicCollectionView` app, in `EmojiDictionary` each row in the collection is represented by a group. You'll want the group to be set to a fixed height of 70 and the full width of its container. Each group will contain one item.

  - ```swift
      @available(iOS 16.0, *)
      class EmojiCollectionViewController: UICollectionViewController {

          override func viewDidLoad() {

              let group = NSCollectionLayoutGroup.horizontal(layoutSize: NSCollectionLayoutSize(widthDimension: .fractionalWidth(1), heightDimension: .absolute(70)), repeatingSubitem: item, count: 1)
    ```

- Finally, each group is encapsulated into a section and the section is used to set the final compositional layout.

  - ```swift
      let section = NSCollectionLayoutSection(group: group)
       
      collectionView.collectionViewLayout = UICollectionViewCompositionalLayout(section: section)
    ```

- **Update/Insert Changes**
  - Find the definition for `unwindToEmojiTableView`. Depending on the state of the data from the collection view and the source view controller of the segue (which will be `AddEditEmojiTableViewController`), an emoji definition may have changed or been added.
  - If the `indexPathsForSelectedItems` property of the collection view has data, this means an item was selected and edited, so the item's definition and the displayed details both need to be updated to reflect the changes.
  - If there are no selected index paths, a new emoji was created, and its definition needs to be added to your collection of emoji. You'll also need a new row added to the collection view for it.
  - Try to add this logic yourself, adding any code beneath the guard statement. You can remove the comment. When you're done, check your work against the code below.

    - ```swift
        if let path = collectionView.indexPathsForSelectedItems?.first {
            emojis[path.row] = emoji
            collectionView.reloadItems(at: [path])
        } else {
            let newIndexPath = IndexPath(row: emojis.count, section: 0)
            emojis.append(emoji)
            collectionView.insertItems(at: [newIndexPath])
        }
      ```

- **Handle Deletes**
  - With a table view, there are built-in mechanisms to handle row deletion. A collection view doesn't have this feature, but it does allow you to attach a context menu to items, which can be used for a similar purpose. These context menus are the same as those used throughout iOS 13, so they should be familiar to your users.
  - To create a context menu for an item, you must implement the `collectionView(_ :contextMenuConfigurationForItemAt:point:)` method from `UICollectionViewDelegate` (which, as noted, is a protocol that a `UICollectionViewController` already adopts).
  - Add the following implementation at the bottom of the class definition:

    - ```swift
        override func collectionView(_ collectionView: UICollectionView, contextMenuConfigurationForItemAt indexPath: IndexPath, point: CGPoint) -> UIContextMenuConfiguration? {
            let config = UIContextMenuConfiguration(identifier: nil, previewProvider: nil) { (elements) -> UIMenu? in
                let delete = UIAction(title: "Delete") { (action) in
                    self.deleteEmoji(at: indexPath)
                }
         
                return UIMenu(title: "", image: nil, identifier: nil, options: [], children: [delete])
            }
         
            return config
        }
      ```

  - This defines a `UIContextMenuConfiguration` that contains a `UIMenu` with a single action.
  - To enable deletions, the `deleteEmoji` method must be defined. Deleting an emoji removes it from the emoji collection and then removes it from the collection view. Put this definition below the delegate method defined above.

    - ```swift
        func deleteEmoji(at indexPath: IndexPath) {
            emojis.remove(at: indexPath.row)
            collectionView.deleteItems(at: [indexPath])
        }
      ```

  - Build and run your app. Check to make sure everything displays correctly and that adding, editing, and deleting emoji all work properly. Nice work!

### Lesson 3.2 Swift Generics

- One important design principle of the Swift programming language is progressive disclosure — the practice of revealing concepts over time as the user grows in experience and confidence, rather than all at once. For example, you can use Swift as a beginner without understanding anything about protocols, even though you're using protocols every time you use even basic types such as `Int` or `String`.
- Generics are another example of progressive disclosure. They're a powerful feature of Swift, but to use them you have to be comfortable with language fundamentals. Like any new concept, generics can seem confusing at first, but you've mastered all the concepts you need to start exploring them. By the end of this unit, you'll be using generics to make your projects easier to read and maintain.
- Generic types and methods allow you to write abstract code that can be used with multiple concrete types — and that helps you avoid duplication. You may not have realized it, but you've been using generics since the very beginning of this course. Many standard types in Swift use generics behind the scenes to provide consistent APIs and functionality while avoiding duplication.

#### Generic Types

- One example of a generic type is `Array`. As you're aware, an array can contain values of any single type, such as `Int` or `String`. Swift does not define separate types for an array of `Ints`, an array of `Strings`, and so on. (If you think about it, such a design would prevent you from using arrays with your own custom types without first creating versions of Array to handle them.) But when you create, say, an array of `Int`s, you can use all the features offered by Array on it — and all the features offered by `Int` on its contents. How does that work? You're ready to find out.
- When you define an array, you either provide it with literal values or specify what type of data it will contain. When you define an array using existing or literal values, the compiler is able to infer the type of data that the array will contain (in the code below, [Int] for ints and [String] for strings):

  - ```swift
      let ints = [1, 2, 3, 4]
      let strings = ["Hello", "World!"]
       
      /* When you create an empty array, you provide the type information explicitly: */
      let ints = [Int]()
      let strings = [String]()
    ```

- This kind of code uses what's referred to as “syntactic sugar” — options for writing code that make it easier to write or read. Without the syntactic sugar, those array definitions would look like this:

  - ```swift
      let ints = Array<Int>()
      let strings = Array<String>()
    ```

- In Swift, when you see angle brackets <> with a type name between them, you're dealing with a generic. In fact, if you look at the definition of Array, you'll see the following: `struct Array<Element> { }`
- What is Element? It doesn't refer to any specific, concrete type. There's no Element in the Swift language, and it can't be initialized. Instead, it's a type parameter. When you declare an array, you're effectively replacing that identifier with a concrete type, such as `Int`.
- The benefit of Array being a generic type is that you can create arrays that hold any Swift type. The team that created Swift could implement all the functions of an array without knowing anything about the type the array contains. Additionally, once the compiler knows that an array contains a specific type, it can prevent you from making mistakes like the following:

  - ```swift
      var numbers = ["1", "2", "3"]
      numbers.append(4) // ❗️Cannot convert value of type 'Int' to expected argument type 'String'
    ```

- This differs from the approach of some other languages — notably Objective-C, whose NSArray class is defined to contain NSObjects, the base of the inheritance hierarchy (roughly equivalent to AnyObject in Swift). One direct result of this approach is that Objective-C arrays can be heterogeneous, containing items of completely unrelated types. This renders the static type checking above impossible; the equivalent Objective-C code is completely legal.
- As a human reading the definition of `numbers` above, you might see the values as being numerical, but they're actually `Strings`. The compiler ensures that you can't insert mismatched types in an array, which also guarantees that you'll be returned the proper type when accessing a value. Swift can sometimes feel overly restrictive compared to the more lax approach of Objective-C, and there may be times when heterogeneous arrays seem to be a good fit for a problem. But there are many advantages to Swift's strictness, such as safer code and behind-the-scenes performance optimizations. Xcode can also lend a hand in hinting at what operations and methods are available to the contained types through its autocompletion features — something it can't do with code that uses `NSArray`.
- Another generic type you've become familiar with is `Dictionary`. A `Dictionary` has keys associated with values, and the types of both are defined when the dictionary is initialized:

  - ```swift
      let wordsByLength = [
          1: ["a", "i"],
          2: ["hi", "by", "go"],
          3: ["the", "but", "now", "how"],
          4: ["then", "just", "when", "cool"]
      ]
    ```

- The Dictionary above has keys of type `Int` and values of type [String]; the compiler infers this based on the values provided. If you were to create this as an empty dictionary and add values to it over time, you would need to explicitly define the types:

  - ```swift
      var wordsByLength = [Int: [String]]() // Using syntactic sugar
      var wordsByLength = Dictionary<Int, Array<String>>() // Fully defined format
    ```

- The declaration of the Dictionary type looks like this: `struct Dictionary<Key, Value> where Key : Hashable { }`
- Instead of the `Element` in the `Array` definition, `Dictionary` has two type parameters: `Key` and `Value`. They're placed together in the angle brackets, separated by commas, signifying that a dictionary instance will define two types to fulfill those roles. As you'd expect, Swift can handle generics with arbitrary numbers of type parameters using a comma - delimited list — though it's rare for there to be more than a handful.
- Another difference from the `Array` definition is the where clause. `where Key : Hashable` states that the type used for `Key` must adopt the `Hashable` protocol. The compiler enforces this rule and will not allow you to declare a dictionary whose key is not `Hashable`. This constraint allows the implementation of `Dictionary` to rely on this information: Any type used for Key is guaranteed to have the capabilities defined in the `Hashable` protocol. Generics and protocols complement each other and help you write cleaner and more reusable code.

#### Generic Functions

- Just like generic types, generic functions can work with any type (unless they're constrained by a protocol). An example of this is the `max(_:_:)` function. This function takes two parameters that are the same type and returns whichever is greater: `max(1, 10) // Returns 10`
- The function works for any type that adopts the Comparable protocol. Here is its method signature: `func max<T>(_ x: T, _ y: T) -> T where T : Comparable { }`
- The `max(_:_:)` function has the identifier `T` in the angle brackets after its name. Both arguments are declared to be of type `T`, as is the return value. Like the Element identifier in `Array` and the `Key` and `Value` identifiers in `Dictionary`, `T` is not a concrete type. It's a type parameter that is resolved with an actual type at each call site. Using the same identifier for the arguments and return type means that all of them must be of the same type. The only limitation to the type represented by `T` is that it must be Comparable; this is defined in the where clause.
- In the example above, `Int` takes the place of `T`—both arguments and the return value are all `Int`s. You can imagine this method written only for `Int`, like: `func max(_ x: Int, _ y: Int) -> Int { }`
- If it were written this way, there would need to be a similar function written for `Double`, `CGFloat`, `String`, and any other type that can be compared. And if you defined custom types that were `Comparable`, you'd also need to implement your own version of `max(_:_:)` for each of those. By using generics, the `max(_:_:)` function is available to use with any `Comparable` type.
- What might this function's implementation look like?

  - ```swift
      func max<T>(_ x: T, _ y: T) -> T where T : Comparable {
          if y >= x {
              return y
          } else {
              return x
          }
      }
    ```

- Since `T` is required to be Comparable, and the `>=` operator is a requirement of the `Comparable` protocol, you can use the `>=` operator on it just like you would with `Int`.
- **Naming Type Parameters**
  - The name of the type parameter `T` doesn't mean anything in particular; “T” is just a typical shorthand notation for “Type.” When possible, it's best to name your type parameters using a significant word or phrase in upper camel case, the way type names are written. This helps convey the meaning of the type parameters and makes your code easier to understand and work with. Array<Element> and Dictionary<Key, Value> follow this rule.
  - Sometimes, though, there's no good name for the placeholder. The `max(_:_:)` function is like that. How would you describe the role of the type that is passed into it? It's just a type. In those cases, it's OK to use the placeholder name `T` and move forward through the alphabet if you need more (such as MyType<T, U, V>).
- **Protocols and Associated Types**
  - It's also possible to create a generic protocol. Much like a generic type or function, with a placeholder identifier that gets filled in with a concrete type, a generic protocol is defined with an associated type that needs to be filled in. The associated type can then be used in the properties or methods the protocol defines.
  - Suppose you wanted to create a protocol to define the role of sending an API request and decoding the response. Your protocol would need a variable to hold on to the request and a method for handling the response. That method would take in data, but what should it return? Consider the following example:

    - ```swift
        protocol APIRequest {
            associatedtype Response
         
            var urlRequest: URLRequest { get }
            func decodeResponse(data: Data) throws -> Response
        }
      ```

  - Note the new keyword, `associatedtype`. This keyword requires that any type adopting the `APIRequest` protocol define a concrete type for the `Response` placeholder, which will be the return type of the protocol's decodeResponse method.
  - In a previous lesson, you fetched information and images from NASA's Astronomy Picture of the Day API. The web service returned data in JSON — a typical format for data from web services — which you then modeled as Swift objects. Those objects adopted the `Codable` protocol, allowing you to initialize them using `JSONDecoder`'s `decode(_:from)` method.
  - The two methods you wrote for that project looked like the following:

    - ```swift
        enum PhotoInfoError: Error, LocalizedError {
            case itemNotFound
            case imageDataMissing
        }
         
        func fetchPhotoInfo() async throws -> PhotoInfo {
            var urlComponents = URLComponents(string: "https://api.nasa.gov/planetary/apod")!
            urlComponents.queryItems = [
                "api_key": "DEMO_KEY"
            ].map { URLQueryItem(name: $0.key, value: $0.value) }
         
            let (data, response) = try await URLSession.shared.data(from: urlComponents.url!)
         
            guard let httpResponse = response as? HTTPURLResponse,
                  httpResponse.statusCode == 200 else {
                throw PhotoInfoError.itemNotFound
            }
         
            let jsonDecoder = JSONDecoder()
            let photoInfo = try jsonDecoder.decode(PhotoInfo.self, from: data)
            return(photoInfo)
        }
         
        func fetchImage(from url: URL) async throws -> UIImage {
            var urlComponents = URLComponents(url: url, resolvingAgainstBaseURL: true)
            urlComponents?.scheme = "https"
            let (data, response) = try await URLSession.shared.data(from: urlComponents!.url!)
         
            guard let httpResponse = response as? HTTPURLResponse,
                  httpResponse.statusCode == 200 else {
                throw PhotoInfoError.imageDataMissing
            }
         
            guard let image = UIImage(data: data) else {
                throw PhotoInfoError.imageDataMissing
            }
         
            return image
        }

  - Notice the similar process in the two methods above:
    1. An instance of `URLComponents` is created.
    2. A data request is made for the URL constructed with `URLComponents`.
    3. If the request succeeds, the response is decoded into the expected type (if it can be) and returned. If either the data request or the decoding fails, an error is thrown.
  - When you notice patterns like this, you might find that you can write a generic solution to cover multiple cases. (You might recall that `Task` itself is generic!) The benefit of a generic solution is that you reduce lines of code — and therefore the number of potential bugs. If one piece of logic contains a bug, any copy of that logic will contain the same bug. It is very hard as a software developer to recall every place logic was copied, so if you can consolidate the logic, you can simplify your debugging.
  - When thinking about the protocol, there are a couple of unknowns:
    - The URL and any request-specific information
    - The desired result type in the completion handler
  - While the contents of the request will be different, `URLRequest` itself is a fixed type. However, you'll also need to know the type that will be returned. That's where associated types come in.

    - ```swift
        protocol APIRequest {
            associatedtype Response
         
            var urlRequest: URLRequest { get }
            func decodeResponse(data: Data) throws -> Response
        }
      ```

  - Types that adopt APIRequest would be required to provide a URLRequest, which can be used to create the data task, as well as a `decodeResponse(data:)` method that returns the desired response type. Notice that the Response type parameter is the return type for this method.
  - It's time to write some code to try it for yourself. Create a new iOS playground and add the `APIRequest` protocol definition above to it. Be sure to import `UIKit`.
  - Before creating a type that adopts this protocol, it would be best to write a method that uses it, to ensure that you have all the necessary pieces. Recall that protocols can sometimes be used as stand-ins for types. Consider a `sendRequest` method that would send a request and pass the result to an object that adopts `APIRequest`.
  - What argument should a method like that receive? An instance of `APIRequest`. And rather than providing a concrete type as the return type, you'd use the associated type for the `APIRequest` — that is, `APIRequest.Response`.
  - To do this, you'll need to write a generic function with a type parameter, using syntax similar to the definition of `max(_:_:)` you saw earlier. The type parameter will need to be constrained to the `APIRequest` protocol. Swift offers two ways to do this. You could use a where clause, like `max(_:_:)` does: `func sendRequest<Request>(_ request: Request) async throws -> Request.Response where Request: APIRequest`
  - Or you can follow the type parameter with a colon and the protocol name, similar to how you define that a type adopts a protocol: `func sendRequest<Request: APIRequest>(_ request: Request) async throws -> Request.Response`
  - Notice that the where clause is removed and <Request> is replaced with <Request: APIRequest>. Both syntax versions work the same way, and you can choose which one you prefer. But it's good to know that both exist so you'll recognize them when you encounter them.
  - Breaking it down, the method is named `sendRequest(_:)` and is generic, with a Request placeholder that must adopt `APIRequest`. The method returns `Request.Response`, meaning that the concrete type defined for the passed - in `Request`'s associated type for `Response` will be returned. This might feel like an unnecessary complication, but you'll see the benefit soon.
  - On to the method's implementation. Add this to your playground:

    - ```swift
        enum APIRequestError: Error {
            case itemNotFound
        }
        func sendRequest<Request: APIRequest>(_ request: Request) async throws -> Request.Response {
            let (data, response) = try await URLSession.shared.data(for: request.urlRequest)
         
            guard let httpResponse = response as? HTTPURLResponse,
                  httpResponse.statusCode == 200 else {
                throw APIRequestError.itemNotFound
            }
         
            let decodedResponse = try request.decodeResponse(data: data)
            return(decodedResponse)
        }
      ```

  - This method is modeled after the `fetchPhotoInfo()` method above, but `PhotoInfo` is nowhere to be seen within it. The passed-in request argument provides the necessary pieces by way of the `APIRequest` protocol. Looking at this method in isolation, you can't tell anything about the `PhotoInfo` struct (or even that there is a `PhotoInfo` struct), which makes this code reusable in a way that `fetchPhotoInfo()` is not.
  - You now have a generic method that is able to send a request and return a concrete type that adopts `APIRequest` – but you don't have any such types. Implementing a type that conforms to APIRequest for the `PhotoInfo` struct might look like the following:

    - ```swift
        struct PhotoInfo: Codable {
            var title: String
            var description: String
            var url: URL
            var copyright: String?
         
            enum CodingKeys: String, CodingKey {
                case title
                case description = "explanation"
                case url
                case copyright
            }
        }
         
        struct PhotoInfoAPIRequest: APIRequest {
            var apiKey: String
         
            var urlRequest: URLRequest {
                var urlComponents = URLComponents(string: "https://api.nasa.gov/planetary/apod")!
                urlComponents.queryItems = [
                    URLQueryItem(name: "date", value: "2021-07-15"),
                    URLQueryItem(name: "api_key", value: apiKey)
                ]

                return URLRequest(url: urlComponents.url!)
            }
         
            func decodeResponse(data: Data) throws -> PhotoInfo {
                let photoInfo = try JSONDecoder().decode(PhotoInfo.self, from: data)
                return photoInfo
            }
        }
      ```

  - This is a relatively simple, boilerplate implementation — which is great. When you find yourself writing boilerplate code, which is easier to reason about, you're often in a good position to avoid introducing new bugs. The `urlRequest` property is a simple computed property that uses the instance's apiKey property. The `decodeResponse(data:)` implementation is also straightforward (and should feel familiar — you've written this exact code already).
  - What about the protocol requirement for `associatedtype` `Response`? How is that satisfied? When you define the return type of `decodeResponse(data:)` as `PhotoInfo`, you're implicitly filling in a concrete type for the `Response` placeholder. Swift can now infer that `Response` for `PhotoInfoAPIRequest` is `PhotoInfo`. You can also explicitly satisfy the requirement by including typealias `Response` = `PhotoInfo` in the implementation.
  - Add the `PhotoInfo` and `PhotoInfoAPIRequest` definitions to your playground, and you'll be able to use all of these pieces together to make the request:

    - ```swift
        let photoInfoRequest = PhotoInfoAPIRequest(apiKey: "DEMO_KEY")
        Task {
            do {
                let photoInfo = try await sendRequest(photoInfoRequest)
                print(photoInfo)
            } catch {
                print(error)
            }
        }
      ```

  - Run the playground and look at the console for the result. You should have the information about a photo, just like you saw when you worked with the APOD API before. Great work! You've successfully created a protocol with an associated type, a generic method that utilizes the protocol, and a concrete type that implements the protocol, and you've used them to fetch data from a web service. This is complex programming that will enable you to write reusable code in your future projects.
  - Arguably the most satisfying part of this web service is fetching and viewing the actual image. Looking back at the definition of `fetchImage(from:)`, see if you can write an `ImageAPIRequest` implementation that can be used to fetch the image from the `PhotoInfoAPIRequest` response. Place the implementation right below `PhotoInfoAPIRequest`. You will also need to define an Error to throw in the event the image cannot be created using the provided Data instance; creating a nested enum to use would be a good approach.
  - Try it on your own before referencing the following solution.

    - ```swift
        struct ImageAPIRequest: APIRequest {
            enum ResponseError: Error {
                case invalidImageData
            }
         
            let url: URL
         
            var urlRequest: URLRequest {
                return URLRequest(url: url)
            }
         
            func decodeResponse(data: Data) throws -> UIImage {
                guard let image = UIImage(data: data) else {
                    throw ResponseError.invalidImageData
                }
         
                return image
            }
        }
      ```

  - Good work! Since this request conforms to APIRequest, it can also be used with the `sendRequest(_:)` method and is a great example of code reuse.
  - Calling this requires the `URL` from `PhotoInfo`, so you will need to initiate the `ImageAPIRequest` when `PhotoInfoAPIRequest` is successful. Initiate an `ImageAPIRequest` using the url property from the returned `PhotoInfo` instance when successful and send it.

    - ```swift
        let photoInfoRequest = PhotoInfoAPIRequest(apiKey: "DEMO_KEY")
        Task {
            do {
                let photoInfo = try await sendRequest(photoInfoRequest)
                print(photoInfo)
                let imageRequest = ImageAPIRequest(url: photoInfo.url)
                let image = try await sendRequest(imageRequest)
                image
            } catch {
                print(error)
            }
        }
      ```

  - Run the playground. To see the picture, you can click the Quick Look button in the results sidebar next to the line where image is called.
  - Excellent work! You've sent a request for two completely different types of data using the same method. As you can see, generics and protocols with associated types can do some pretty powerful things.

    - ```swift
        import UIKit

        protocol APIRequest {
            associatedtype Response
            
            var urlRequest: URLRequest { get }
            func decodeResponse(data: Data) throws -> Response
        }

        enum APIRequestError: Error {
            case itemNotFound
        }

        func sendRequest<Request: APIRequest>(_ request: Request) async throws -> Request.Response {
            let (data, response) = try await URLSession.shared.data(for: request.urlRequest)
            
            guard let httpResponse = response as? HTTPURLResponse,
                  httpResponse.statusCode == 200 else {
                throw APIRequestError.itemNotFound
            }
            
            let decodeResponse = try request.decodeResponse(data: data)
            return(decodeResponse)
        }

        struct PhotoInfo: Codable {
            var title: String
            var description: String
            var url: URL
            var copyright: String?
            
            enum CodingKeys: String, CodingKey {
                case title
                case description = "explanation"
                case url
                case copyright
            }
        }

        struct PhotoInfoAPIRequest: APIRequest {
            var apiKey: String
            
            var urlRequest: URLRequest {
                var urlComponents = URLComponents(string: "https://api.nasa.gov/planetary/apod")!
                urlComponents.queryItems = [
                    URLQueryItem(name: "date", value: "2021-07-15"),
                    URLQueryItem(name: "api_key", value: apiKey)
                ]
                
                return URLRequest(url: urlComponents.url!)
            }
            
            func decodeResponse(data: Data) throws -> PhotoInfo {
                let photoInfo = try JSONDecoder().decode(PhotoInfo.self, from: data)
                
                return photoInfo
            }
        }

        let photoInfoRequest = PhotoInfoAPIRequest(apiKey: "DEMO_KEY")
        Task {
            do {
                let photoInfo = try await sendRequest(photoInfoRequest)
                print(photoInfo)
            } catch {
                print(error)
            }
        }

        struct ImageAPIRequest: APIRequest {
            enum ResponseError: Error {
                case invalidImageData
            }
            
            let url: URL
            
            var urlRequest: URLRequest {
                return URLRequest(url: url)
            }
            
            func decodeResponse(data: Data) throws -> UIImage {
                guard let image = UIImage(data: data) else {
                    throw ResponseError.invalidImageData
                }
                
                return image
            }
        }

        let photoInfoReuqest = PhotoInfoAPIRequest(apiKey: "DEMO_KEY")
        Task {
            do {
                let photoInfo = try await sendRequest(photoInfoRequest)
                print(photoInfo)
                let imageRequest = ImageAPIRequest(url: photoInfo.url)
                let image = try await sendRequest(imageRequest)
                image
            } catch {
                print(error)
            }
        }
      ```

#### Lab Life-Form Search

- In this lab, you will build an app that allows users to search for life-forms and display available taxonomy information and photos. You will build on what you have learned in this lesson and previous lessons to create an app that uses the Encyclopedia of Life (www.eol.org) data services API to search its database and look up information about the life-forms found.
- As you may know, the classification of life on Earth is a complex science and changes frequently as new life-forms are discovered. This complexity is reflected in the results from the EOL data services API. Using the techniques discussed in this and previous lessons, you'll use `Codable` protocol conformance to help convert the results into simpler data structures that meet your specific needs. You'll be using various API endpoints from the EOL classic API that return different types of data. This will give you a good opportunity to use your newfound knowledge of generics to streamline your implementation.
- Create a new project using the iOS App template and name it "Life-FormSearch." When creating the project, make sure the interface option is set to "Storyboard.

##### User Interface Design

- The app will display two types of data: a list of results from a search of the EOL database and certain details associated with a search result when the user selects it. You are free to use any design that you like, but the description of the app will probably lead you to a screen showing the list of search results and a detail screen that is shown with a push transition when the user selects a returned result.
- As you've learned, the system search bar provides a consistent interface for performing a search in iOS apps, and a table view is a great way to display a list of results. The detail screen will show an image fetched from the EOL database and a specific subset of the scientific classification of the result. A view controller with a specific set of visual elements (image view and labels) to populate is a reasonable solution for this purpose. You can use stack views to help organize your views and, if you desire, embed the stack view in a scroll view to support smaller devices. Here is an example of one way to present the information to the user:

  - <img src="./resources/Life-Form Search.png" alt="Life-Form Search" width="200" />

- You'll notice that the search results display a common name (or in some cases a list of common names) and a scientific name for the items found in a search. Both of these fields are available in the EOL Search API.

  - <img src="./resources/Life-Form Search Detail.png" alt="Life-Form Search Detail" width="200" />

- The detail screen is showing an image (not all results will have an image available), the classification information - including the scientific name, kingdom, phylum, class, order, family, and genus (again, not all results will contain full classification results) - and the source of the information. You will need to use three API endpoints beyond the search API to collect this information:
  1. The `pages` API provides basic information along with a list of references to an image or images and the classification options (as mentioned, classification of life is a complex science, and different sources can report different classifications).
  2. The `hierarchy_entries` API provides the details of a specific classification scheme.
  3. The `pages` API results include a URL to fetch an item's image.
- You can look at all the details of the EOL classic API at https://eol.org/docs/what-is-eol/classic-apis. To simplify your task, the details of how you'll want to use each of these API endpoints are provided below.
- You might also find it useful to "pretty print" the JSON results using the extension to Data that you used in the JSON Serialization lesson to help visualize the results returned from each API request. Here's an extension to Data for that purpose.

  - ```swift
      extension Data {
          func prettyPrintedJSONString() {
              guard
                  let jsonObject = try? JSONSerialization.jsonObject(with:​ self, options: []),
                  let jsonData = try? JSONSerialization.data(withJSONObject:​ jsonObject, options: [.prettyPrinted]),
                  let prettyJSONString = String(data: jsonData,​               encoding: .utf8) else {
                      print("Failed to read JSON Object.")
                      return
              }
              print(prettyJSONString)
          }
      }
    ```

##### Search API

- The EOL classic API for searching that you'll use in this project results is a URL in the form: https://eol.org/api/search/1.0.json?q=Yellow%20Tang&page=1
- You can construct this URL using `URLComponents(string:)` with "https://eol.org/api/search/1.0.json" and add the query terms with `URLQueryItem(name:value:)` as you did in the `iTunesSearch` lab.
- The JSON returned from the search will have the form:

  - ```json
      {
        "itemsPerPage" : 50,
        "results" : [
          {
            "id" : 46577088,
            "title" : "Zebrasoma flavescens",
            "content" : "Yellow tang; yellow tang; Yellow Tang",
            "link" : "https:\/\/eol.org\/pages\/46577088"
          },
          {
            "id" : 46577087,
            "title" : "Zebrasoma xanthurum",
            "content" : "Yellow tang; Yellow Tang",
            "link" : "https:\/\/eol.org\/pages\/46577087"
          },
          {
            "id" : 46577097,
            "title" : "Ctenochaetus cyanocheilus",
            "content" : "Indo-pacific yellow tang",
            "link" : "https:\/\/eol.org\/pages\/46577097"
          }
        ],
        "totalResults" : 3,
        "startIndex" : 1
      }
    ```

### Lesson 3.3 Dynamic Data

- Now that you understand how generics work, you're ready to use them in practice. Many `UIKit` APIs don't use generics because they were written before Swift was the dominant language in iOS development. Some modern, Swift-only APIs take advantage of generics. They add an extra level of elegance to your code and empower features that would have be difficult or impossible before.
- In this lesson, you'll learn your first API that uses generics: `UICollectionViewDiffableDataSource`. You'll learn how it can reduce the work needed to supply data to a collection view, giving you standard features — such as animation of changes — for free.
- Because users will become frustrated scrolling through a large list or grid of items, apps that display large data sets need to provide affordances for navigating their content. Sections with headers are an example of a navigation affordance, since they give users organizational cues as they scroll.
- Apps often provide an additional affordance by allowing users to filter large collections through searching. Searching hands the reins to the user—they control what's visible by specifying the criteria they're interested in. Searching is particularly important for apps that access remote data sources, because it's often the only way to pare down huge amounts of content into a digestible form.
- A search bar is one way to filter data. By tapping in the search bar, a user can enter one or more terms, and the collection is filtered as the user types. `UIKit` provides a standard API for searching named `UISearchController`. It's easy to integrate into an existing table or collection view. In this lesson, you'll add a search controller to your existing Basic Collection Views app that displays states. You can use the starting point from your resources for this lesson or continue with the code you wrote yourself.

#### Adding a Search Controller

- The existing API for navigation stacks includes a simple hook for integrating a search controller with little effort. Because your view controller is already embedded in a navigation controller, you can quickly add the search bar interface. In `BasicCollectionViewController`, add the following instance property: `let searchController = UISearchController()`
- This initializes a new instance of a `UISearchController` that you can use, but it'll need to be configured and added to your view. You'll notice that there are no arguments passed to the initializer. If you were to look at the API documentation, you'd see that there's an initializer that accepts a `searchResultsController` argument. By passing `nil` or by using the initializer without arguments, you're telling the search controller that you'll be displaying results in the same view that you're searching.
- When the layout of your search results is the same as the original layout, it makes sense to go this route. If you'll be displaying search results with a different layout, you'll want to provide a separate view controller that displays the search results. The search controller will automatically handle showing and dismissing your custom view controller for search results.
- To configure the search controller, add the following to the beginning of `viewDidLoad()`, after the call to super.

  - ```swift
      navigationItem.searchController = searchController
      searchController.obscuresBackgroundDuringPresentation = false
      searchController.searchResultsUpdater = self
      navigationItem.hidesSearchBarWhenScrolling = false
    ```

- The first line assigns the search controller to the `navigationItem`, which lets the `UINavigationController` know to add a search bar to the navigation bar when your view controller is displayed. Next, you set the `obscuresBackgroundDuringPresentation` property to `false`. The default value of that property is `true`, which is the correct behavior if you provide a separate view controller to display the search results. When the property is `true`, your main view is dimmed or obscured as soon as the user taps the search bar to enter text — signifying that new content will be replacing the existing content. If you're not using a separate view controller for search results, you should set this property to false so that the search results are displayed in place without a view controller transition.
- The third line informs the `searchController` that `self` will be the `searchResultsUpdater` — the delegate that will handle the actual filtering logic. After you enter this code, you'll see a compiler error: “Cannot assign value of type 'BasicCollectionViewController' to type 'UISearchResultsUpdating?'.” To resolve the error, update the declaration for `BasicCollectionViewController` to adopt the protocol `UISearchResultsUpdating`.
- The fourth line ensures that the search bar will always be available at the top of the screen. The default is for the search bar to only be visible when the `collectionView` is scrolled to the top of the screen. The use of this option will depend on how search is used in any particular app: `class BasicCollectionViewController: UICollectionViewController, UISearchResultsUpdating {`
- Of course, this causes another error: “Type 'BasicCollectionViewController' does not conform to protocol 'UISearchResultsUpdating'.” Click the red circle icon and use Xcode's Fix feature to add protocol stubs. This will insert a new method at the top of the class; you can move it further down, if you prefer. `func updateSearchResults(for searchController: UISearchController) {}`
- This method is where you'll implement the logic to filter the items property and display matching results to the user. The method is called anytime the search controller deems it necessary to update the search results. If you build and run the app now, you'll see the added search bar — but typing into it doesn't change what the collection view displays. Consider for a moment what needs to happen when the user types something into the search bar: The collection view needs to refresh itself to only display results that match the user's query. Here's one strategy to achieve this:
  1. Create a copy of the original data for use as search results.
  2. Filter the duplicate data so that it only contains data that matches the search query.
  3. Refresh the collection view to show the filtered results.
- Start by adding a new instance variable named `filteredItems` that's a copy of items. `var filteredItems: [String] = items`
- `filteredItems` is the property you'll use to store filtered results. It'll also replace items as the basis for the collection view's data. Change the references to items in the collection view data source and delegate methods to `filteredItems`.

  - ```swift
      override func collectionView(_ collectionView: UICollectionView,
        numberOfItemsInSection section: Int) -> Int {
          return filteredItems.count
      }

      override func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
          let cell =
            collectionView.dequeueReusableCell(withReuseIdentifier: reuseIdentifier, for: indexPath) as! BasicCollectionViewCell
       
          cell.label.text = filteredItems[indexPath.item]
       
          return cell
      }
    ```

- Now you need to respond to the user's search by implementing `updateSearchResults(for:)`:

  - ```swift
      func updateSearchResults(for searchController: UISearchController) {
          if let searchString = searchController.searchBar.text, 
              searchString.isEmpty == false {
              filteredItems = items.filter { (item) -> Bool in
                  item.localizedCaseInsensitiveContains(searchString)
              }
          } else {
              filteredItems = items
          }

          collectionView.reloadData()
      }
    ```

- This implementation attempts to unwrap the text of the provided `searchController`'s `searchBar`. If text is present, the filter method is used to remove items that don't match the search string.
- The `localizedCaseInsensitiveContains(:)` method searches within a string, ignoring any letter case differences — so “a” is a match for both “a” and “A.” For example, the query “al” will match “Alabama,” “Alaska,” and “California.” The results of the filter function are assigned to `filteredItems`. If there's no search text, the `filteredItems` array is set back to items, so that the entire data set is displayed.
- Finally, `reloadData()` is called on the collection view to request that it re-query its data source.
- Build and run the app and try searching. You should see the collection view results update to match the search string.

#### Handling Data Changes

- You've successfully added a powerful feature for your users with very little code! While the feature is functional and complete, it could be better. When the user enters a search string, the data set changes, but it's a bit abrupt. What if you could update the data with a smooth animation?
- Both collection and table views offer APIs that allow you to smoothly animate changes to their data. `UICollectionView` offers the `performBatchUpdates(_:completion:)` method, which you supply with a closure that inserts, deletes, reloads, and moves cells using `IndexPaths`. While this API is powerful, it's often challenging to calculate the difference between one version of your data set and another.
- Consider the following update to a data set.
  - | Index | Original | Updated |
    |---|---|---|
    | 1 | Cat | Monkey |
    | 2 | Dog | Giraffe |
    | 3 | Elephant | Horse |
    | 4 | Fish | Fish |
    | 5 | Horse | Sugar Glider |
    | 6 | Giraffe | |
    | 7 | Monkey | |
- The following events have occurred:
  - Index 1 Removed
  - Index 2 Removed
  - Index 3 Removed
  - Index 4 No Change
  - Index 5 Moved to 3
  - Index 6 Moved to 2
  - Index 7 Moved to 1
  - “Sugar Glider” inserted at 5
- It's difficult to hand-code an algorithm to describe all the changes to a data set — and all too easy to fail to account for some cases. And using `performBatchUpdates(_:completion:)` also requires you to apply the updates in a particular order or risk runtime exceptions.
- Thankfully, there's a less error-prone API to assist with this task.

#### Diffable Data Sources

- When you display data in a collection or table view that doesn't change as the user is viewing it, the data source protocol methods you're already familiar with might be all that you need. However, if your data will be changing as the user is viewing it, you might find that diffable data sources are a better alternative. “Diffable” is a way of saying that this kind of data source is able to perform the complicated series of operations that describe the difference between the between two sets of data. “Performing a diff” and “diffing” are common terms of art that developers use to describe this process.
- `UICollectionViewDiffableDataSource` is a concrete class that implements `UICollectionViewDataSource` and provides a streamlined, closure-based API to use as your collection view's data source. To use a diffable data source, you encapsulate your data in an instance of `NSDiffableDataSourceSnapshot` and provide it to a diffable data source instance. (For table views, you can use the analogous `UITableViewDiffableDataSource`.)
- You'll start by refactoring `BasicCollectionViewController` to use `UICollectionViewDiffableDataSource`. The first step is to create an enum that will identify the sections in the `collectionView`; in this case, you'll just have a single section and call it main. Add the following to the top of the `BasicCollectionViewController` class: `enum Section: CaseIterable { case main }`
- Next, add an implicitly unwrapped variable on `BasicCollectionViewController` for the data source that uses the Section type you just defined.
  - `var dataSource: UICollectionViewDiffableDataSource<Section, String>!`
- Notice that `UICollectionViewDiffableDataSource` is a generic class. Option-click the type name to view its documentation inline.
- A diffable data source requires two concrete types: `SectionIdentifierType` and `ItemIdentifierType`. You can probably infer that these two types are used to represent the sections and items in your data set. This gives you a lot more flexibility than using `IndexPaths`, because you can use types that more natively express how your data is organized and provide those values to uniquely identify sections and the items within them.
- The only requirement for `SectionIdentifierType` and `ItemIdentifierType` is that they both adopt the `Hashable` protocol. Section and String are both `Hashable`, and they're already used for your sections and items, so you'll use them in this example. (As you learn more about diffable data sources, you'll use additional custom types for these identifiers.)
- One of the most important roles of the `UICollectionViewDataSource` protocol is providing cells to be displayed. Your existing code implements this by overriding the protocol method directly. When initializing `UICollectionViewDiffableDataSource`, you instead provide a closure that returns a cell instance. The diffable data source's internal implementation of the `collectionView(_:cellForItemAt:)` method uses that closure.
- Go ahead and create a new method that initializes your diffable data source.

  - ```swift
      func createDataSource() {
          dataSource = UICollectionViewDiffableDataSource<Section, String>(collectionView: collectionView, cellProvider: { (collectionView, indexPath, item) -> UICollectionViewCell? in
              let cell =
                collectionView.dequeueReusableCell(withReuseIdentifier: reuseIdentifier, for: indexPath) as! BasicCollectionViewCell
       
              cell.label.text = item
       
              return cell
          })
       
      }
    ```

- The initializer takes the collection view as an argument and a `cellProvider` closure that does almost exactly what your existing `collectionView(_:cellForItemAt:)` method does. One important difference is that in addition to an `IndexPath`, you're also passed the actual item, so you don't need to look it up.
- Look back at the closure that you passed to the initializer of the diffable data source. Where does the diffable data source get the data it's passing to you?
`NSDiffableDataSourceSnapshot` represents the collection view's contents at a moment in time. To tell the diffable data source what its contents are, you apply a snapshot to it. When the diffable data source is asked to provide cells to the collection view, the items in a snapshot are passed to your cell provider closure.
- What makes diffable data sources so powerful is that you can apply snapshots whenever you want, and the associated collection or table view refreshes its contents automatically. And the diffable data source handles the complicated series of operations that describe the difference between the previous snapshot and the current one — so you get smooth animations for free.
- You can use any kind of data to represent the contents of a collection view in a snapshot, as long as it's `Hashable`. For complex or memory-consuming data, it's common to use a separate view model that concisely describes what's displayed and then create view model instances that reflect your actual data set to store in a snapshot.
- Because your data model is simple, you'll put it directly into the snapshots you build. You'll be applying a snapshot to the data source initially with the value of `filteredItems`, and you'll continue doing this while the user changes their search query.
- Creating a snapshot is straightforward. Since you'll need to apply snapshots in multiple methods, you'll create a computed property that you can reference as needed. Add the following property on `BasicCollectionViewController` to represent an up-to-date snapshot of filtered data:

  - ```swift
      var filteredItemsSnapshot: NSDiffableDataSourceSnapshot<Section, String> {
          var snapshot = NSDiffableDataSourceSnapshot<Section, String>()
       
          snapshot.appendSections([.main])
          snapshot.appendItems(filteredItems)
       
          return snapshot
      }
    ```

- A snapshot is defined by its sections and items — the two generic parameters in its type declaration. First you initialize an empty snapshot. Next you add your sections, first appending the section, then appending the items for each one. (There's a second, optional argument for `appendItems(_:toSection:)` that you can leave blank if you just want to append the items to the most recently added section.) Of course, you are only using a single section in this `collectionView` so you just need to add the section and then the items.
- Back in the `createDataSource()` method, apply a snapshot to the data source after it's been initialized. That'll ensure that you have data to display when your view controller first appears.
  - `dataSource.apply(filteredItemsSnapshot)`
- Next, call `createDataSource()` at the end of your `viewDidLoad()` implementation. Now you can remove your `UICollectionViewDataSource` protocol method implementations, as the diffable data source will take on the responsibility. Delete the following methods in `BasicCollectionViewController`:
  - numberOfSections(in:)
  - collectionView(_:numberOfItemsInSection:)
  - collectionView(_:cellForItemAt:)
  - collectionView(_:viewForSupplementaryElementOfKind:at:)
- Build and run the app to see the diffable data source populate your collection view's contents. Unfortunately, you'll notice that the search functionality is now broken. There's one small change that you'll need to make to fix it.
- Take a look at the implementation of `updateSearchResults(for:)` and see if you can figure out what needs to be changed. Prior to using a diffable data source, you simply updated `filteredItems` and then called `reloadData()`. Now, rather than calling `reloadData()`, you'll apply a new snapshot.
- Replace the call to `reloadData()` with the following.
  - `dataSource.apply(filteredItemsSnapshot, animatingDifferences: true)`
- By passing true for `animatingDifferences`, you enable the real magic of diffable data sources. With no extra effort, you'll get the smooth animation you're after as the user changes their search query. Build and run again to see the result.
- Excellent work! Your app now uses pleasing animations as the data is filtered, and you've learned a couple powerful new APIs.

  - ```swift
      class BasicCollectionViewController: UICollectionViewController, UISearchResultsUpdating {
          enum Section: CaseIterable {
              case main
          }
          var dataSource: UICollectionViewDiffableDataSource<Section, String>!
          
          func createDataSource() {
              dataSource = UICollectionViewDiffableDataSource<Section, String>(collectionView: collectionView, cellProvider: { (collectionView, indexPath, item) -> UICollectionViewCell? in
                  let cell = collectionView.dequeueReusableCell(withReuseIdentifier: reuseIdentifier, for: indexPath) as! BasicCollectionViewCell
                  cell.label.text = item
                  
                  return cell
              })
              dataSource.apply(filteredItemsSnapshot)
          }
          
          var filteredItemsSnapshot: NSDiffableDataSourceSnapshot<Section, String> {
              var snapshot = NSDiffableDataSourceSnapshot<Section, String>()
              
              snapshot.appendSections([.main])
              snapshot.appendItems(filteredItems)
              
              return snapshot
          }
          func updateSearchResults(for searchController: UISearchController) {
              if let searchString = searchController.searchBar.text,
                searchString.isEmpty == false {
                  filteredItems = items.filter { (item) -> Bool in
                      item.localizedCaseInsensitiveContains(searchString)
                  }
              } else {
                  filteredItems = items
              }
              dataSource.apply(filteredItemsSnapshot, animatingDifferences: true)
          }
          
          let searchController = UISearchController()
          
          override func viewDidLoad() {
              super.viewDidLoad()
              
              navigationItem.searchController = searchController
              searchController.obscuresBackgroundDuringPresentation = false
              searchController.searchResultsUpdater = self
              navigationItem.hidesSearchBarWhenScrolling = false
              
              collectionView.setCollectionViewLayout(generateLayout(), animated: false)
              createDataSource()
          }

          private func generateLayout() -> UICollectionViewLayout {
              let itemSize = NSCollectionLayoutSize(
                  widthDimension: .fractionalWidth(1.0),
                  heightDimension: .fractionalHeight(1.0)
              )
              
              let item = NSCollectionLayoutItem(layoutSize: itemSize)
              
              let spacing: CGFloat = 10.0

              let groupSize = NSCollectionLayoutSize(
                  widthDimension: .fractionalWidth(1.0),
                  heightDimension: .absolute(70.0)
              )
              
              let group = NSCollectionLayoutGroup.horizontal(
                  layoutSize: groupSize,
                  subitem: item,
                  count: 1
              )

              group.contentInsets = NSDirectionalEdgeInsets(
                  top: spacing,
                  leading: spacing,
                  bottom: 0,
                  trailing: spacing
              )

              let section = NSCollectionLayoutSection(group: group)

              let layout = UICollectionViewCompositionalLayout(section: section)
              
              return layout
          }
      }
    ```

#### Lab - Itunes Search (Part 4)

- The purpose of this lab is to use your new knowledge of collection views, search controllers, and diffable data sources to update the solution from iTunes Search (Part 3). You'll start with a refactored version of the solution, which is available in the resources folder. The solution has been refactored to use a custom container view controller, enabling you to switch between two view controllers' views. (Standard `UIKit` container view controllers include `UINavigationController` and `UITabBarController`.) It has also been updated to use a search controller rather than a search bar.

##### Step 1 Review Provided Refactoring

- In the refactored solution for iTunes Search, open the Main storyboard.
- Notice the changes: Rather than a single table view controller, there's a navigation controller whose root view controller has embed segues to the table view controller and to a new collection view controller. The root view controller sitting in the middle is the container view controller.
- The container view controller has two subviews that contain the table view controller's view and the collection view controller's view. In the Document Outline, you'll see these as “Table Container View” and “Collection Container View.” The container view controller has view properties for each of the other two view controllers, which are set when the embed segues are performed.
- There's also a segmented control at the bottom, which toggles the isHidden property for both container views. Initially, the table container view is visible and the collection container view is hidden. The user can toggle the segmented control to view the results as a list or a grid.
- Open `StoreItemListTableViewController` and notice that the class implementation has mostly been removed. It has been moved to `StoreItemContainerViewController`. The container view controller is responsible for fetching the `StoreItem` results and populating both the table and collection view data sources. Because the data will be the same for both the table view and the collection view, it makes sense to maintain it in one view controller. Using view controller containment is a great way to solve this type of problem.
- In `StoreItemContainerViewController`, you'll find that `UISearchController` is now being used in place of `UISearchBar`. It's configured to always be visible and to display its scope bar — a built-in segmented control you'll use to let the user select the type of media they're searching for.
- The Task used to fetch the items using the search terms is now saved in a variable so that it can be cancelled if the result will no longer be useful. This will happen each time the search terms change before the web request has completed. The do/catch statement has also been updated to ignore the errors that are thrown when a `URLSession` request is cancelled before it completes.
- One major difference is that `UISearchController` will invoke the `updateSearchResults(for:)` method anytime the search bar content changes or becomes active. Because you don't want to send a request to the iTunes Search Service with every keystroke, you'll “debounce” the user input by only issuing network requests after the user has stopped typing for a brief period. Within `updateSearchResults(for:)` you'll find two unfamiliar methods: `cancelPreviousPerformRequests(withTarget:selector:object:)` and `perform(_:with:afterDelay:)`. With every keystroke, you queue up a new request and cancel previous ones — until the user stops typing for 0.3 seconds. Debouncing is a common technique for search controls that interact with a web service.
- Now that you're familiar with the major components of the provided refactor, you're ready to begin making changes.
