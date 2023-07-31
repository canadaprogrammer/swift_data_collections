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
