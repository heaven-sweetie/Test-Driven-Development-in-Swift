In this blog post, we will learn how to build a simple iOS app menu (shown below) using [Test-Driven Development](http://martinfowler.com/bliki/TestDrivenDevelopment.html) in Swift.
[![app_menu.png](https://d23f6h5jpj26xu.cloudfront.net/aacqorrxf8wpiw.png)](http://img.svbtle.com/aacqorrxf8wpiw.png)
Here are things you need to know to fully understand the concepts presented in this post:

* Xcode 6
* Familiarity with [basic concepts](http://goo.gl/PNWoSw) in Swift
* Familiarity with commonly used classes in UIKit and Foundation (e.g., UITableView and NSNotificationCenter)
* Familiarity with XCTest. If you have never used XCTest before, please read the *XCTestCase* section from [Matt Thompson's](https://twitter.com/mattt) [blog post](http://nshipster.com/xctestcase/) on the topic.

Create a new iOS project in Xcode 6. Select the *Single View Application* template. Use *AppMenu* for Product Name, *Swift* as the language, and *iPhone* for Devices. Make sure the *Use Core Data* check box is not selected. For this exercise we won't be using [Storyboards](http://goo.gl/YOK2kR). So delete the `Main.storyboard` file. Don't forget to remove the storyboard name (Main) from the *Main Interface* drop-down located in *Deployment Info* section under *General* tab for *AppMenu* target. While we are at it, let's delete `ViewController.swift` and `AppMenuTests.swift` files as well. They were created by Xcode and we won't need them.

Before we embark on this beautiful journey of Test-Driven Development (TDD), let's step back and think about the app design a little bit. You might have heard that TDD is more of a design exercise than a testing activity. If so, you have heard it correct. TDD forces us to use the code even before it exists through tests which in turn forces us to think about how the class under test interacts with the rest of the code in an application. This act of using code we wish we had helps us make (good) design decisions that lead to the creation of reusable classes and easy-to-use application programming interfaces (API). That being said, it is not always guaranteed that an application built by writing tests first is well designed. We still need to apply good design [principles](http://goo.gl/bbzSpz) and [patterns](http://goo.gl/U23sC8) in addition to writing tests first.

The figure below shows the initial design we are aiming for. If you are worried that we might have just stepped into the [Big Up Front Design](http://en.wikipedia.org/wiki/Big_Design_Up_Front) (BUFD) territory, fear not. The design below gives us a starting point. We haven't figured out every single aspect of the app yet. For example, we don't know what the public API for each class is going to look like. As we go through the process of building the app, we might realize that we need to change the design below completely and that is perfectly fine.

[![initial_app_design.png](https://d23f6h5jpj26xu.cloudfront.net/ttuxgmqb3ia.png)](http://img.svbtle.com/ttuxgmqb3ia.png)

<a name="identifying_domain_objects"></a>
Identifying Domain Objects
==========================

When I am starting on a new project, I often struggle to find the *first good* test to write. As a result, I resort to looking for domain objects which are usually easy to test. Our app menu will display information about each menu item for example, title, subtitle, and icon. We need an instance that stores information about a menu item. Let's call it `MenuItem`. We'll define what information a `MenuItem` instance contains through tests.

Create a new file named `MenuItemTests.swift` and place it under `AppMenuTests` group. Right click on `AppMenuTests` group and select *New File > iOS > Source > Test Case Class* to create a new test class named `MenuItemTests`. Make it a subclass of `XCTestCase` and select Swift as the language. Delete everything in `MenuItemTests.swift` file except the class definition and import statements.

~~~swift
import UIKit
import XCTest

class MenuItemTests: XCTestCase {
}
~~~

Our first test will be to make sure that a menu item has a title. Add following test to the `MenuItemTests` class.

~~~swift
func testThatMenuItemHasATitle() {
    let menuItem = MenuItem(title: "Contributions")
    XCTAssertEqual(menuItem.title, "Contributions", 
        "A title should always be present")
}
~~~

In above test, we create an instance of `MenuItem`; give it a title; and verify that it holds onto that title by using the `XCTAssertEqual` assertion provided by XCTest framework that comes with Xcode. We are also (implicitly) specifying that `MenuItem` should provide an initializer that takes `title` as a parameter. When we write tests first, we tend to discover subtle details like this about our APIs.

> XCTest provides [a number of assertions](http://goo.gl/PU24UU). Each assertion allows you to pass a test description that explains what the test is about. I recommend you always provide this description.

As things stand right now, we are not able to run the test. It doesn't even compile. We need to create `MenuItem`. Before we decide whether we should make `MenuItem` a struct or a class, I encourage you to read the [Choosing Between Classes and Structures](http://goo.gl/ptBqYR) section from *The Swift Programming Language Guide* first. 

At a first glance, `struct` might seem sufficient here. However, in [Handling Menu Item Tap Event](#handling_menu_item_tap_event) section below we will need to store a menu item in a `NSNotification` object. `NSNotification` expects an object that needs to be stored in it to conform to `AnyObject` protocol. A `struct` type doesn't conform to `AnyObject`. Therefore, we need to make `MenuItem` a class. Create a new file named `MenuItem.swift` (*File > New > File > iOS > Source > Swift File*). Add it to both *AppMenu* and *AppMenuTests* targets and replace its content with following.

~~~swift
import Foundation

class MenuItem {
    let title: String
    
    init(title: String) {
        self.title = title
    }
}
~~~

> You shouldn't have to add application classes into test targets, but there seems to be a bug in Xcode 6 that forces you to do so. Otherwise, you will get *use of unresolved identifier* error. If you receive that error in tests at any point in the future, you can fix it by adding the application class file to the test target as well. You can add a file to a target by clicking the *+* button from *Compile Sources* section in *Build Phases* tab.

Run the test (*Product > Test* or ⌘U). It should pass. Let's write a test for the subtitle property next.

~~~swift
func testThatMenuItemCanBeAssignedASubTitle() {
    let menuItem = MenuItem(title: "Contributions")
    menuItem.subTitle = "Repos contributed to"

    XCTAssertEqual(menuItem.subTitle!, "Repos contributed to",
        "Subtitle should be what we assigned")
}
~~~

It should pass after adding the `subTitle` property to `MenuItem` class.

~~~swift
class MenuItem {
    let title: String
    var subTitle: String?
    
    init(title: String) {
        self.title = title
    }
}
~~~

Since a menu item must have a title, it's defined as a constant property. Whereas a `subTitle` is not required. Therefore, we define it as a variable property. Finally, here is a test for the `iconName` property:

~~~swift
func testThatMenuItemCanBeAssignedAnIconName() {
    let menuItem = MenuItem(title: "Contributions")
    menuItem.iconName = "iconContributions"

    XCTAssertEqual(menuItem.iconName!, "iconContributions",
        "Icon name should be what we assigned")
}
~~~

It should pass by adding the `iconName` property to `MenuItem`.

~~~swift
class MenuItem {
    let title: String
    var subTitle: String?
    var iconName: String?
    
    init(title: String) {
        self.title = title
    }
}
~~~

Before we move on, let's refactor our tests by moving the `MenuItem` creation code into `setup` method.

> It is a general practice in TDD to [refactor](http://refactoring.com/) once all tests are passing. The process of refactoring helps us improve design in small increments by organizing the code and tests better. It also helps us remove any duplication. This iterative process of writing a failing test first, making it pass with just enough code and improving the design before writing the next failing test is known as *red-green-refactor* cycle.

[![red_green_refactor.png](https://d23f6h5jpj26xu.cloudfront.net/fqgfy5r3w7nkq.png)](http://img.svbtle.com/fqgfy5r3w7nkq.png)

~~~swift
class MenuItemTests: XCTestCase {
    var menuItem: MenuItem?
    
    override func setUp() {
        super.setUp()
        menuItem = MenuItem(title: "Contributions")
    }
    
    func testThatMenuItemHasATitle() {
        XCTAssertEqual(menuItem!.title, "Contributions",
            "A title should always be present")
    }
    
    func testThatMenuItemCanBeAssignedASubTitle() {
        menuItem!.subTitle = "Repos contributed to"
        XCTAssertEqual(menuItem!.subTitle!, "Repos contributed to",
            "Subtitle should be what we assigned")
    }
    
    func testThatMenuItemCanBeAssignedAnIconName() {
        menuItem!.iconName = "iconContributions"
        XCTAssertEqual(menuItem!.iconName!, "iconContributions",
            "Icon name should be what we assigned")
    }
}
~~~

XCTest calls the `setup` method before running each test. When a test is finished running, the variables assigned in `setup` method are set to `nil`. After that it creates brand new instances of objects and assigns them to those variables in `setup` method again. XCTest does this to isolate each test. We don't want any residual data created by previous tests affect the next ones. As XCTest automatically sets variables to `nil` when a test is finished running, we don't need to explicitly set them to `nil` in `tearDown` method (also provided by XCTest). That being said, if you need to do any cleanup other than setting variables to `nil`, you should do that in `tearDown` method.

<a name="reading_metadata_from_plist"></a>
Reading Metadata from Plist
===========================

Next up we will read the metadata required to create `MenuItem` instances from a plist. As our initial design suggests, we will be storing the metadata for each menu item in a [plist](http://goo.gl/naf71B) file. That way if we need to populate the menu dynamically by fetching the metadata from a remote server, we won't need to make too many changes as long as the metadata format remains the same. Before building `MenuItemsPlistReader` class, we need to know how the `MenuItemsReader` protocol looks like. Here is my initial pass at it:

~~~swift
import Foundation

protocol MenuItemsReader {
  func readMenuItems() -> ([[String : String]]?, NSError?)
}
~~~

`readMenuItems` method doesn't take any parameters and returns a [tuple](http://goo.gl/9k9O5u). The first item in the tuple contains an array of [dictionaries](http://goo.gl/ucLgzF) if the file was read successfully. The second item contains an [NSError](http://goo.gl/VYcIen) object if the file couldn't be read. `readMenuItems` is a required method. So any class that wants to conform to `MenuItemsReader` protocol, must implement it. Create a new file named `MenuItemsReader.swift`. Add it to both targets and then replace its content with the protocol definition code listed above.

Let's read the metadata from a plist file next. We will write tests first. Create a new file named `MenuItemsPlistReaderTests.swift` in `AppMenuTests` target. Now that you know how to create a Swift test file, I will skip those instructions moving forward. Delete everything in `MenuItemsPlistReaderTests.swift` file except the class definition and import statements. Our first test will be to make sure that `MenuItemsPlistReader` returns an error if it can't read the specified plist file. Add following test to the `MenuItemsPlistReaderTests` class.

~~~swift
func testErrorIsReturnedWhenPlistFileDoesNotExist() {
    let plistReader = MenuItemsPlistReader()
    plistReader.plistToReadFrom = "notFound"
    
    let (metadata, error) = plistReader.readMenuItems()
    XCTAssertNotNil(error, "Error is returned when plist doesn't exist")
}
~~~

We create an instance of `MenuItemsPlistReader`; give it a non-existent plist file name to read from; and call `readMenuItems` method. Then we verify that it returns an error. To make the test pass, we need to create `MenuItemsPlistReader` class and add it to both targets. Replace its content with following.

~~~swift
import Foundation

class MenuItemsPlistReader: MenuItemsReader {
    var plistToReadFrom: String? = nil
    
    func readMenuItems() -> ([[String : String]]?, NSError?) {
        let error = NSError(domain: "Some domain", 
                            code: 0, 
                            userInfo: nil)
        return ([], error)
    }
}
~~~

Now run the test. It should pass. Although the test passes, something doesn't look right. `readMenuItems` doesn't even attempt to read the file. It always returns a tuple containing an empty array and not-so-useful error. This brings us to an important aspect of TDD: *write minimum code to pass the test*. Being disciplined about not writing anymore code than necessary to pass the tests is key to TDD. Therefore, we won't be fixing the simingly broken `readMenuItems` method unless our tests require us to do so.

The only requirement we have defined for `MenuItemsPlistReader` class so far is that *it returns an error if the file doesn't exist*. We haven't specified what should be in that error object. Let's add a couple more tests to make sure the error contains the domain, code and description we are expecting.

> Apple [recommends](http://goo.gl/akPPGh) that we use NSError objects to capture information about runtime errors. These objects should contain the error *domain*, a domain-specific error *code*, and a *user info* dictionary containing the error *description*. You can add other details about the error in *user info* dictionary, for example what steps to take to resolve the error.

~~~swift
func testCorrectErrorDomainIsReturnedWhenPlistDoesNotExist() {
    let plistReader = MenuItemsPlistReader()
    plistReader.plistToReadFrom = "notFound"
    
    let (metadata, error) = plistReader.readMenuItems()
    let errorDomain = error?.domain
    
    XCTAssertEqual(errorDomain!, MenuItemsPlistReaderErrorDomain,
        "Correct error domain is returned")
}

func testFileNotFoundErrorCodeIsReturnedWhenPlistDoesNotExist() {
    let plistReader = MenuItemsPlistReader()
    plistReader.plistToReadFrom = "notFound"
    
    let (metadata, error) = plistReader.readMenuItems()
    let errorCode = error?.code
    
    XCTAssertEqual(errorCode!, 
        MenuItemsPlistReaderErrorCode.FileNotFound.toRaw(),
        "Correct error code is returned")
}

func testCorrectErrorDescriptionIsReturnedWhenPlistDoesNotExist() {
    let plistReader = MenuItemsPlistReader()
    plistReader.plistToReadFrom = "notFound"
    
    let (metadata, error) = plistReader.readMenuItems()
    let userInfo = error?.userInfo
    let description: String = 
        userInfo![NSLocalizedDescriptionKey]! as String
    
    XCTAssertEqual(description,
        "notFound.plist file doesn't exist in app bundle",
        "Correct error description is returned")
}
~~~

Following changes to `MenuItemsPlistReader` should make the above tests pass.

~~~swift
import Foundation

let MenuItemsPlistReaderErrorDomain = "MenuItemsPlistReaderErrorDomain"

enum MenuItemsPlistReaderErrorCode : Int {
    case FileNotFound
}

class MenuItemsPlistReader: MenuItemsReader {
    var plistToReadFrom: String? = nil
    
    func readMenuItems() -> ([[String : String]]?, NSError?) {
        let errorMessage = 
            "\(plistToReadFrom!).plist file doesn't exist in app bundle"

        let userInfo = [NSLocalizedDescriptionKey: errorMessage]

        let error = NSError(domain: MenuItemsPlistReaderErrorDomain,
            code: MenuItemsPlistReaderErrorCode.FileNotFound.toRaw(),
            userInfo: userInfo)

        return ([], error)
    }
}
~~~

`readMenuItems` method still doesn't look right. Next tests we are going to write will force us not to cheat. Before we move forward, delete the test named `testErrorIsReturnedWhenPlistFileDoesNotExist`. It is made redundant by previous three tests.

~~~swift
func testPlistIsDeserializedCorrectly() {
    let plistReader = MenuItemsPlistReader()
    plistReader.plistToReadFrom = "menuItems"
    
    let (metadata, error) = plistReader.readMenuItems()
    XCTAssertTrue(metadata?.count == 3, 
        "There should only be three dictionaries in plist")
    
    let firstRow = metadata?[0]
    XCTAssertEqual(firstRow!["title"]!, "Contributions",
        "First row's title should be what's in plist")
    XCTAssertEqual(firstRow!["subTitle"]!, "Repos contributed to",
        "First row's subtitle should be what's in plist")
    XCTAssertEqual(firstRow!["iconName"]!, "iconContributions",
        "First row's icon name should be what's in plist")
    
    let secondRow = metadata?[1]
    XCTAssertEqual(secondRow!["title"]!, "Repositories",
        "Second row's title should be what's in plist")
    XCTAssertEqual(secondRow!["subTitle"]!, "Repos collaborating",
        "Second row's subtitle should be what's in plist")
    XCTAssertEqual(secondRow!["iconName"]!, "iconRepositories",
        "Second row's icon name should be what's in plist")
    
    let thirdRow = metadata?[2]
    XCTAssertEqual(thirdRow!["title"]!, "Public Activity",
        "Third row's title should be what's in plist")
    XCTAssertEqual(thirdRow!["subTitle"]!, "Activity viewable by anyone",
        "Third row's subtitle should be what's in plist")
    XCTAssertEqual(thirdRow!["iconName"]!, "iconPublicActivity",
        "Third row's icon name should be what's in plist")
}
~~~

Here we are making sure that `readMenuItems` method actually reads data from the specified plist file and creates proper objects from that data.

> A rule of thumb while writing unit tests is not to include more than one assertion in a test method. I am violating that rule here, because it makes sense to verify that the data read from the file is correct in one place.

To pass above test, create a file named "menuItems.plist" *(Right click AppMenu group > New File > iOS > Resource > Property List)*. Add it to both targets. Open that file in source code mode *(Right click the file in Xcode > Open As > Source Code)* and replace its content with following:

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<array>
    <dict>
        <key>title</key>
        <string>Contributions</string>
        <key>subTitle</key>
        <string>Repos contributed to</string>
        <key>iconName</key>
        <string>iconContributions</string>
        <key>featureName</key>
        <string>contributions</string>
    </dict>
    <dict>
        <key>title</key>
        <string>Repositories</string>
        <key>subTitle</key>
        <string>Repos collaborating</string>
        <key>iconName</key>
        <string>iconRepositories</string>
        <key>featureName</key>
        <string>repositories</string>
    </dict>
    <dict>
        <key>title</key>
        <string>Public Activity</string>
        <key>subTitle</key>
        <string>Activity viewable by anyone</string>
        <key>iconName</key>
        <string>iconPublicActivity</string>
        <key>featureName</key>
        <string>publicActivity</string>
    </dict>
</array>
</plist>
~~~

Next add following images to the asset catalog (Images.xcassets). They are included in the [finished project](http://goo.gl/WvUBDo).

* iconContributions@2x.png
* iconRepositories@2x.png
* iconPublicActivity@2x.png

Now modify `readMenuItems` method as shown below:

~~~swift
func readMenuItems() -> ([[String : String]]?, NSError?) {
    var error: NSError? = nil
    var fileContents: [[String : String]]? = nil    
    let bundle = NSBundle(forClass: object_getClass(self))
    
    if let filePath = 
        bundle.pathForResource(plistToReadFrom, ofType: "plist")
    {
        fileContents = 
            NSArray(contentsOfFile: filePath) as? [[String : String]]
    }
    else {
        let errorMessage = 
            "\(plistToReadFrom!).plist file doesn't exist in app bundle"

        let userInfo = [NSLocalizedDescriptionKey: errorMessage]
        
        error = NSError(domain: MenuItemsPlistReaderErrorDomain,
            code: MenuItemsPlistReaderErrorCode.FileNotFound.toRaw(),
            userInfo: userInfo)
    }

    return (fileContents, error)
}
~~~

Now that the tests are passing, let's refactor by extracting the error building code into a separate method. Here is how `readMenuItems` method looks like after refactoring:

~~~swift
func readMenuItems() -> ([[String : String]]?, NSError?) {
    var error: NSError? = nil
    var fileContents: [[String : String]]? = nil
    let bundle = NSBundle(forClass: object_getClass(self))
    
    if let filePath = 
        bundle.pathForResource(plistToReadFrom, ofType: "plist") 
    {
        fileContents = 
            NSArray(contentsOfFile: filePath) as? [[String : String]]
    }
    else {
        error = fileNotFoundError()
    }

    return (fileContents, error)
}

func fileNotFoundError() -> NSError {
    let errorMessage = 
        "\(plistToReadFrom!).plist file doesn't exist in app bundle"

    let userInfo = [NSLocalizedDescriptionKey: errorMessage]
    
    return NSError(domain: MenuItemsPlistReaderErrorDomain,
        code: MenuItemsPlistReaderErrorCode.FileNotFound.toRaw(),
        userInfo: userInfo)
}
~~~

Run the tests again (*⌘U*) to make sure that we didn't break anything. I see some refactoring opportunity with our tests as well. Let's move the common code from all tests into `setup` method.

~~~swift
class MenuItemsPlistReaderTests: XCTestCase {
    var plistReader: MenuItemsPlistReader?
    var metadata: [[String : String]]?
    var error: NSError?
    
    override func setUp() {
        super.setUp()
        plistReader = MenuItemsPlistReader()
        plistReader?.plistToReadFrom = "notFound"
        (metadata, error) = plistReader!.readMenuItems()
    }
    
    func testCorrectErrorDomainIsReturnedWhenPlistDoesNotExist() {
        let errorDomain = error?.domain
        XCTAssertEqual(errorDomain!, MenuItemsPlistReaderErrorDomain,
            "Correct error domain is returned")
    }
    
    func testFileNotFoundErrorCodeIsReturnedWhenPlistDoesNotExist() {
        let errorCode = error?.code
        XCTAssertEqual(errorCode!, 
            MenuItemsPlistReaderErrorCode.FileNotFound.toRaw(),
            "Correct error code is returned")
    }
    
    func testCorrectErrorDescriptionIsReturnedWhenPlistDoesNotExist() {
        let userInfo = error?.userInfo

        let description: String = 
            userInfo![NSLocalizedDescriptionKey]! as String

        XCTAssertEqual(description, 
            "notFound.plist file doesn't exist in app bundle",
            "Correct error description is returned")
    }
    
    func testPlistIsDeserializedCorrectly() {
        plistReader!.plistToReadFrom = "menuItems"
        (metadata, error) = plistReader!.readMenuItems()
        
        XCTAssertTrue(metadata?.count == 3, 
            "There should only be three dictionaries in plist")
        
        let firstRow = metadata?[0]
        XCTAssertEqual(firstRow!["title"]!, "Contributions",
            "First row's title should be what's in plist")
        XCTAssertEqual(firstRow!["subTitle"]!, "Repos contributed to",
            "First row's subtitle should be what's in plist")
        XCTAssertEqual(firstRow!["iconName"]!, "iconContributions",
            "First row's icon name should be what's in plist")
        
        let secondRow = metadata?[1]
        XCTAssertEqual(secondRow!["title"]!, "Repositories",
            "Second row's title should be what's in plist")
        XCTAssertEqual(secondRow!["subTitle"]!, "Repos collaborating",
            "Second row's subtitle should be what's in plist")
        XCTAssertEqual(secondRow!["iconName"]!, "iconRepositories",
            "Second row's icon name should be what's in plist")
        
        let thirdRow = metadata?[2]
        XCTAssertEqual(thirdRow!["title"]!, "Public Activity",
            "Third row's title should be what's in plist")
        XCTAssertEqual(thirdRow!["subTitle"]!, 
            "Activity viewable by anyone",
            "Third row's subtitle should be what's in plist")
        XCTAssertEqual(thirdRow!["iconName"]!, "iconPublicActivity",
            "Third row's icon name should be what's in plist")
    }
}
~~~

I realized that we forgot to add a test for a scenario when the plist exists, but `readMenuItems` is unable to read it perhaps due to bad data. I will leave that as an exercise for you my dear readers.

<a name="building_menu_items"></a>
Building Menu Items
===================

We are now ready to build `MenuItem` instances from the metadata we just read. Create a new test class named `MenuItemBuilderTests` and replace its content with following:

~~~swift
import UIKit
import XCTest

class MenuItemBuilderTests: XCTestCase {
    var menuItemBuilder: MenuItemBuilder?
    var fakeMenuItemsReader: FakeMenuItemsReader?
    var menuItems: [MenuItem]?
    var error: NSError?
    
    override func setUp() {
        fakeMenuItemsReader = FakeMenuItemsReader()
        fakeMenuItemsReader!.missingTitle = true
        let (metadata, _) = fakeMenuItemsReader!.readMenuItems()
        
        menuItemBuilder = MenuItemBuilder()

        (menuItems, error) = 
            menuItemBuilder!.buildMenuItemsFromMetadata(metadata!)
    }
    
    func testCorrectErrorDomainIsReturnedWhenTitleIsMissing() {
        let errorDomain = error?.domain
        XCTAssertEqual(errorDomain!, MenuItemBuilderErrorDomain,
            "Correct error domain is returned")
    }
    
    func testMissingTitleErrorCodeIsReturnedWhenTitleIsMissing() {
        let errorCode = error?.code
        XCTAssertEqual(errorCode!, 
            MenuItemBuilderErrorCode.MissingTitle.toRaw(),
            "Correct error code is returned")
    }
    
    func testCorrectErrorDescriptionIsReturnedWhenTitleIsMissing() {
        let userInfo = error?.userInfo
        let description: String = 
            userInfo![NSLocalizedDescriptionKey]! as String

        XCTAssertEqual(description, 
            "All menu items must have a title",
            "Correct error description is returned")
    }

    func testEmptyArrayIsReturnedWhenErrorIsPresent() {
        XCTAssertTrue(menuItems?.count == 0,
            "No menu item instances are returned when error is present")
    }
}
~~~

A menu item must have a title. Therefore, we need to make sure that `MenuItemBuilder` returns an error if the title is missing. We also need to make sure that an empty list of menu items is returned when an error occurs.

In above test, instead of using the real menu items metadata reader (`MenuItemsPlistReader`), we are using a fake one called `FakeMenuItemsReader`. The reason for that is we need to isolate the class under test from all other components in the app. By doing so when a test fails, we can be reasonably certain that the issue is in the class under test and not someother class that it depends on. Furthermore, if we used the real metadata reader in our tests and if that class decides to download the plist from a remote server in the future, the tests for `MenuItemBuilder` will suffer unnecessarily if the download takes a while. We should always aim towards *speedy* and *non-brittle* tests that are easy to maintain.

For `FakeMenuItemsReader` to be able to stand-in for other menu items readers out there, it must conform to the `MenuItemsReader` protocol. Instead of reading the metadata from a file or remote server, it always returns a hard-coded array of dictionaries. Create `FakeMenuItemsReader` class and add it only to `AppMenuTests` target. We won't be using this class in any of the application code. Replace the content of `FakeMenuItemsReader.swift` file with following:

~~~swift
import Foundation

class FakeMenuItemsReader : MenuItemsReader {
    var missingTitle: Bool = false
    
    func readMenuItems() -> ([[String : String]]?, NSError?) {
        let menuItem1 = 
            missingTitle ? menuItem1WithMissingTitle() 
                         : menuItem1WithNoMissingTitle()
        
        let menuItem2 = ["title": "Menu Item 2",
                         "subTitle": "Menu Item 2 subtitle",
                         "iconName": "iconName2"]
        
        return ([menuItem1, menuItem2], nil)
    }
    
    func menuItem1WithMissingTitle() -> [String : String] {
        return ["subTitle": "Menu Item 1 subtitle",
                "iconName": "iconName1"]
    }
    
    func menuItem1WithNoMissingTitle() -> [String : String] {
        var menuItem = menuItem1WithMissingTitle()
        menuItem["title"] = "Menu Item 2"
        return menuItem
    }
}
~~~

One concern that often arises with these *fake* classes is that they might go out of date if the public API of the original classes they are standing in for change. It's a valid concern. However, since Swift throws a compile time error if a class that claims to conform to a protocol doesn't actually implement the required methods, we don't need to worry about it here. For example, if we decided to return a non-optional array of menu items from the `readMenuItems` method in `MenuItemsReader` protocol we will be forced to apply that change to both `MenuItemsPlistReader` and `FakeMenuItemsReader` classes. Go ahead, try it. Isn't Swift great that way?

> If you would like to learn more about an "alternate universe" these fake objects might create in your tests with specific examples, please read chapter 9 (*Creating Test Doubles* section) from [Practical Object Oriented Design in Ruby](http://goo.gl/bbzSpz) book.

The other concern with *fake* classes is that they might be giving us a false sense of security. How do we really know that `MenuItemsPlistReader` and `MenuItemBuilder` work well together? The answer is we don't, at least through unit tests. The job of making sure that different units of an app work well together is usually given to [integration tests](http://goo.gl/bRjDIS), which won't be covered in this blog post.

Now create a new Swift class named `MenuItemBuilder` in *AppMenu* group. Add it to both targets and replace its content with following:

~~~swift
import Foundation

let MenuItemBuilderErrorDomain = "MenuItemBuilderErrorDomain"

enum MenuItemBuilderErrorCode : Int {
    case MissingTitle
}

class MenuItemBuilder {
    func buildMenuItemsFromMetadata(metadata: [[String : String]]) 
         -> ([MenuItem]?, NSError?) 
    {
        let userInfo = 
            [NSLocalizedDescriptionKey: "All menu items must have a title"]

        let error = NSError(domain: MenuItemBuilderErrorDomain,
            code: MenuItemBuilderErrorCode.MissingTitle.toRaw(),
            userInfo: userInfo)

        return ([], error)
    }
}
~~~

We have written just enough code to make the error tests pass. Next we need to make sure that the builder creates correct number of menu items. Add following test to `MenuItemBuilderTests` class.

~~~swift
func testOneMenuItemInstanceIsReturnedForEachDictionary() {
    fakeMenuItemsReader!.missingTitle = false
    let (metadata, _) = fakeMenuItemsReader!.readMenuItems()

    (menuItems, _) =
        menuItemBuilder!.buildMenuItemsFromMetadata(metadata!)
    
    XCTAssertTrue(menuItems?.count == 2,
        "Number of menu items should be equal to number of dictionaries")
}
~~~

One feature I particularly like about Swift is that I can easily ignore a return value that I am not interested in by using `_`. Following modification to `MenuItemBuilder` class should make all tests pass.

~~~swift
class MenuItemBuilder {
    func buildMenuItemsFromMetadata(metadata: [[String : String]]) 
         -> ([MenuItem]?, NSError?) 
    {
        var menuItems = [MenuItem]()
        var error: NSError?
        
        for dictionary in metadata {
            if let title = dictionary["title"] {
                let menuItem = MenuItem(title: title)
                menuItem.subTitle = dictionary["subTitle"]
                menuItem.iconName = dictionary["iconName"]     
                menuItems.append(menuItem)
            }
            else {
                error = missingTitleError()
                menuItems.removeAll(keepCapacity: false)
                break
            }
        }
        
        return (menuItems, error)
    }
    
    private func missingTitleError() -> NSError {
        let userInfo = 
            [NSLocalizedDescriptionKey: "All menu items must have a title"]

        return NSError(domain: MenuItemBuilderErrorDomain,
            code: MenuItemBuilderErrorCode.MissingTitle.toRaw(),
            userInfo: userInfo)
    }
}
~~~

> You might have noticed that I didn't strictly follow the *red-green-refactor* cycle with above changes. I should be extracting the error building code into a separate method only after all tests are passing. Although, I don't encourage ignoring the *write minimum amount of code to pass tests first* rule from TDD, I will be doing exactly  so every now and then so as not to make this blog post too long.

The last test for `MenuItemBuilder` is to verify that it populates the menu item instances' properties with values present in metadata dictionaries.

~~~swift
func testMenuItemPropertiesContainValuesPresentInDictionary() {
    fakeMenuItemsReader!.missingTitle = false
    let (metadata, _) = fakeMenuItemsReader!.readMenuItems()

    (menuItems, _) = 
        menuItemBuilder!.buildMenuItemsFromMetadata(metadata!)
    
    let rawDictionary1 = metadata![0]
    let menuItem1 = menuItems![0]

    XCTAssertEqual(menuItem1.title, 
        rawDictionary1["title"]!,
        "1st menu item's title should be what's in the 1st dictionary")

    XCTAssertEqual(menuItem1.subTitle!, 
        rawDictionary1["subTitle"]!,
        "1st menu item's subTitle should be what's in the 1st dictionary")

    XCTAssertEqual(menuItem1.iconName!, 
        rawDictionary1["iconName"]!,
        "1st menu item's icon name should be what's in the 1st dictionary")
    
    let rawDictionary2 = metadata![1]
    let menuItem2 = menuItems![1]

    XCTAssertEqual(menuItem2.title, 
        rawDictionary2["title"]!,
        "2nd menu item's title should be what's in the 2nd dictionary")

    XCTAssertEqual(menuItem2.subTitle!, 
        rawDictionary2["subTitle"]!,
        "2nd menu item's subTitle should be what's in the 2nd dictionary")

    XCTAssertEqual(menuItem2.iconName!, 
        rawDictionary2["iconName"]!,
        "2nd menu item's icon name should be what's in the 2nd dictionary")
}
~~~

Once again, we are using multiple assertions within a test here. The test above should pass without any code changes.

<a name="displaying_menu_items"></a>
## 메뉴 아이템 표시
=====================

이제 우리는 `MenuItem` 인스턴스를 빌드하는 방법과 plist의 정보들로 부터 채울 수 있는 방법을 알고있다. 그럼 컨텐츠를 보여주는 것으로 초점을 옮겨보자. 우리는 테이블뷰를 이용해 메뉴 아이템들을 보여줄 것이다. 우리의 초기 설계 알 수 있듯이, `MenuTableDefaultDataSource` 는 철저히 설정된 각각의 `UITableViewCell` 메뉴 아이템에서 제공되는 정보를 응답할 것이다. 테이블뷰 자체는 `MenuViewController`에 의해 관리된다. 

<a name="providing_data_to_table_view"></a>
### 테이블뷰에 데이터 제공

우리는 테이블뷰의 데이터소스를 `MenuViewController`에 직접 구현을 하기 보다  분리된 객체를 사용할 것이다. `MenuViewController` 는 이미 뷰들을 관리하는 역할을 하고 있습니다. 나는 테이블뷰의 데이터를 미리 준비해서 [단일 책임 원칙](http://www.objectmentor.com/resources/articles/srp.pdf)을 위반하는 것을 피할 것이다. 그러나 첫번 째로 우리는 `MenuTableDefaultDataSource`에 일치하는 프로토콜을 만들것 이다. `MenuTableDataSource.swift`라는 스위프트 파일 파일을 *AppMenu* 그룹에 새로만들고. 파일의 타겟을 추가한 뒤 아래의 코드로 변경한다.

~~~swift
import UIKit

protocol MenuTableDataSource : UITableViewDataSource {
    func setMenuItems(menuItems: [MenuItem])
}
~~~

`MenuTableDataSource`는 `UITableViewDataSource` 로 부터 상속된 프로토콜이다. 또한 `setMenuItems` 메소드를 필수로 요구하고 있다. 이제 우리는 `MenuTableDefaultDataSource`의 테스트 작성이 준비됐다. `AppMenuTests` 타겟 안에서 `MenuTableDefaultDataSourceTests.swift` 라는 새로운 테스트 파일을 생성하고 다음의 코드를 추가한다.

~~~swift
import UIKit
import XCTest

class MenuTableDefaultDataSourceTests: XCTestCase {
    func testReturnsOneRowForOneMenuItem() {
        let testMenuItem = MenuItem(title: "Test menu item")
        let menuItemsList = [testMenuItem]
        
        let dataSource = MenuTableDefaultDataSource()
        dataSource.setMenuItems(menuItemsList)

        let numberOfRows = 
            dataSource.tableView(nil, numberOfRowsInSection:0)
        
        XCTAssertEqual(numberOfRows, 
            menuItemsList.count,
            "Only 1 row is returned since we're passing 1 menu item")
    }
}
~~~

여기서 우리는 각각의 메뉴 아이템 데이터 소스가 하나의 데이블뷰 셀 인스턴스를 만들고 있다는 것을 확인했다. 이제 `MenuTableDefaultDataSource.swift` 라는 새로운 스위프트 파일을 만들고 아래의 코드를 입력한다.

~~~swift
import Foundation
import UIKit

class MenuTableDefaultDataSource : NSObject, MenuTableDataSource {
    var menuItems: [MenuItem]?
    
    func setMenuItems(menuItems: [MenuItem]) {
        self.menuItems = menuItems
    }
    
    func tableView(tableView: UITableView!,
                   numberOfRowsInSection section: Int)
                   -> Int
    {
        return 1
    }
    
    func tableView(tableView: UITableView!,
                   cellForRowAtIndexPath indexPath: NSIndexPath!)
                   -> UITableViewCell!
    {
        return nil;
    }
}
~~~

아직 우리는 테스트를 위한 `tableView:cellForRowAtIndexPath:` 메소드를 아직 작성하지 않았다. 우리는 요청에 대해 동작할 수 있는 구현이 테스트 이전에 필요하다. 이 작업은 `UITableViewDataSource` 프로토콜에서 요구하는 메소드가 있어야 하고 스위프트는 `MenuTableDefaultDataSource` 없이 컴파일을 할 수 없기 때문이다.

혹시 `MenuTableDefaultDataSource` 에 대하여 알아 차렸을 수도 있겠지만 이것은 `NSObject` 를 상속받고 있다. 그 이유는 `UITableViewDataSource` 프로토콜과 일치하기 위해, 또한 `NSObject` 프로토콜을 준수할 필요가 있기 때문이다. 이것을 가장 쉽게 따르는 방법으로는 `NSObject` 프로토콜을 준수 하는 `NSObject` 의 서브클래스를 만드는 것이다.

위의 테스트에서, 우리는 `tableView:numberOfRowsInSection:` 메서드에서 무조건 `1` 을 반환하도록 만들었다. 데이터 소스가 얼마나 많은 메뉴 아이템이 있든 항상 정확한 수를 반환하는지 증명하는 테스트를 추가한다.

~~~swift
func testReturnsTwoRowsForTwoMenuItems() {
    let testMenuItem1 = MenuItem(title: "Test menu item 1")
    let testMenuItem2 = MenuItem(title: "Test menu item 2")
    let menuItemsList = [testMenuItem1, testMenuItem2]
    
    let dataSource = MenuTableDefaultDataSource()
    dataSource.setMenuItems(menuItemsList)

    let numberOfRows = 
        dataSource.tableView(nil, numberOfRowsInSection:0)
    
    XCTAssertEqual(numberOfRows, 
        menuItemsList.count,
        "Returns two rows as we're passing two menu items")
}
~~~

`menuItems` 는 테스트를 통과하기 위해 하드코딩된 `1` 대신 실제 카운트 값을 반환한다.

~~~swift
func tableView(tableView: UITableView!,
               numberOfRowsInSection section: Int)
               -> Int
{
    return menuItems!.count
}
~~~

또한 우리는 데이터소스가 정확한 섹션의 갯수를 반환하는지 확인이 필요하다. 여기 그것에 대한 테스트이다:

~~~swift
func testReturnsOnlyOneSection() {
    let dataSource = MenuTableDefaultDataSource()
    let numberOfSections = dataSource.numberOfSectionsInTableView(nil)
    XCTAssertEqual(numberOfSections, 1, 
        "There should only be one section")
}
~~~

`numberOfSectionsInTableView` 메서드로부터 리턴되는 `1` 은 이전 테스트의 통과를 시키기 위해 만들었다. 또한 이 메서드는 `UITableViewDataSource` 프로토콜에서 필수로 요구되는 메서드는 아니다. 그리고 기본적으로 이미 `1` 이 리턴되고 있다. 우리는 테스트로 부터 명령이 가능한 호출에 대해 구현이 필요하다.

~~~swift
func numberOfSectionsInTableView(tableView: UITableView!) -> Int {
    return 1
}
~~~

> 또 다른 테스트로 검증을 위해 작성하기 좋은것으로 데이터소스로 부터 예외처리를 발생시키는 것이다. 섹션안에 행의 숫자 인덱스를 0이 아닌 다른숫자로 입력하라고 묻는것 이라면, 하지만 우리의 오래된 친구 `XCTAssertThrows` 를 Xcode 6의 XCTest에서는 찾을 수 없다. 그래서 예외상황이 발생된 상황을 다르게 증명하는 방법을 모르겠다.

iOS에서 모든 뷰의 테스트를 하는건 상당히 지루할 수 있다. 나는 최소의 뷰만 테스트하는 경향이 있다. 이말은 대표적인 뷰들만 테스트를 하는것인데, 이 경우에는 각각의 메뉴를 대표하는 테이블뷰 셀만 테스트를 하는것이다. 그러므로, 나는 각각의 메뉴 아이템으로 부터의 대표적인 셀만 증명하는 것을 선호한다. 아래의 코드를 참고하자.

~~~swift
func testEachCellContainsTitleForRespectiveMenuItem() {
    let testMenuItem = MenuItem(title: "Test menu item")
    let dataSource = MenuTableDefaultDataSource()
    dataSource.setMenuItems([testMenuItem])
    
    let firstMenuItem = NSIndexPath(forRow: 0, inSection: 0)
    let cell = 
        dataSource.tableView(nil, cellForRowAtIndexPath: firstMenuItem)
    
    XCTAssertEqual(cell.textLabel.text!, 
        "Test menu item",
        "A cell contains the title of a menu item it's representing")
}
~~~

`tableView:cellForRowAtIndexPath:` 메서드는 이전에 테스트가 통과되도록 변경해야 한다.

~~~swift
func tableView(tableView: UITableView!,
    cellForRowAtIndexPath indexPath: NSIndexPath!)
    -> UITableViewCell!
{
    // Ideally we should be reusing table view cells here
    let cell = UITableViewCell(style: .Subtitle, reuseIdentifier: nil)
    let menuItem = menuItems?[indexPath.row]
    
    cell.textLabel.text = menuItem?.title
    cell.detailTextLabel.text = menuItem?.subTitle
    cell.imageView.image = UIImage(named: menuItem?.iconName)
    cell.accessoryType = .DisclosureIndicator
    
    return cell
}
~~~

`MenuTableDefaultDataSource` 테스트를 위한 리팩토링을 해보자. 우리는 예전에 `setup` 메서드에서 공통 코드를 추출했었다.

~~~swift
import UIKit
import XCTest

class MenuTableDefaultDataSourceTests: XCTestCase {
    var dataSource: MenuTableDefaultDataSource?
    var menuItemsList: [MenuItem]?
    
    override func setUp() {
        super.setUp()
        
        let testMenuItem = MenuItem(title: "Test menu item")
        menuItemsList = [testMenuItem]
        
        dataSource = MenuTableDefaultDataSource()
        dataSource!.setMenuItems(menuItemsList!)
    }
    
    func testReturnsOneRowForOneMenuItem() {
        let numberOfRows = 
            dataSource!.tableView(nil, numberOfRowsInSection:0)

        XCTAssertEqual(numberOfRows, 
            menuItemsList!.count,
            "Only one row is returned since we're passing one menu item")
    }
    
    func testReturnsTwoRowsForTwoMenuItems() {
        let testMenuItem1 = MenuItem(title: "Test menu item 1")
        let testMenuItem2 = MenuItem(title: "Test menu item 2")
        let menuItemsList = [testMenuItem1, testMenuItem2]
        
        dataSource!.setMenuItems(menuItemsList)
        let numberOfRows = 
            dataSource!.tableView(nil, numberOfRowsInSection:0)
        
        XCTAssertEqual(numberOfRows, 
            menuItemsList.count,
            "Returns two rows as we're passing two menu items")
    }
    
    func testReturnsOnlyOneSection() {
        let numberOfSections = 
            dataSource!.numberOfSectionsInTableView(nil)

        XCTAssertEqual(numberOfSections, 1, 
            "There should only be one section")
    }
    
    func testEachCellContainsTitleForRespectiveMenuItem() {
        let firstMenuItem = NSIndexPath(forRow: 0, inSection: 0)
        let cell = 
            dataSource!.tableView(nil, 
                cellForRowAtIndexPath: firstMenuItem)
        
        XCTAssertEqual(cell.textLabel.text!, 
            "Test menu item",
            "A cell contains the title of a menu item it's representing")
    }
}
~~~

<a name="handling_menu_item_tap_event"></a>
### Handling Menu Item Tap Event (메뉴 아이템 탭 이벤트 핸들링)

테이블뷰 설정은 보는것과 같이 굉장히 간단하다. 그러므로, 데이터소스나 델리게이트 같은 오브젝트의 사용도 이해가 될것이다. 테이블 뷰의 셀이 탭될때 데이터소스는 알림을 보낼 것이다. `MenuViewController` (또는 관심이 있는 다른 클래스) 는 어떤 셀이 탭이 되었고 어떤 액션을 받을 것인지 찾기위해 신호를 보낼 수 있다.

[![table_view_architecture.png](https://d23f6h5jpj26xu.cloudfront.net/9pmnjzuddxpv3g.png)](http://img.svbtle.com/9pmnjzuddxpv3g.png)

이 디자인은 [Test-Driven iOS Development](http://goo.gl/iiKpC1) 책의 챕터 9로 부터 약간의 영감을 받았다. 그럼 `MenuTableDataSource` 프로토콜에 델리게이트와 관련된 세부항목을 추가하자.

~~~swift
import UIKit

let MenuTableDataSourceDidSelectItemNotification = 
    "MenuTableDataSourceDidSelectItemNotification"

protocol MenuTableDataSource : UITableViewDataSource, UITableViewDelegate {
    func setMenuItems(menuItems: [MenuItem])
}
~~~

이제 우리는 데이터소스가 정말로 테이블뷰의 아이템이 탭되었을 때 알림을 보내는지 알수 있는 증명이 필요하다. 다음과 같이 테스트해볼 것이다.

~~~swift
class MenuTableDefaultDataSourceTests: XCTestCase {
    var dataSource: MenuTableDefaultDataSource?
    var testMenuItem: MenuItem?
    var menuItemsList: [MenuItem]?
    var postedNotification: NSNotification?
    var selectedIndexPath: NSIndexPath?
    
    override func setUp() {
        super.setUp()
        
        testMenuItem = MenuItem(title: "Test menu item")
        menuItemsList = [testMenuItem!]
        selectedIndexPath = NSIndexPath(forRow: 0, inSection: 0)
        
        dataSource = MenuTableDefaultDataSource()
        dataSource!.setMenuItems(menuItemsList!)
        
        let notificationCenter = NSNotificationCenter.defaultCenter()
        notificationCenter.addObserver(self,
            selector: "didReceiveNotification:",
            name: MenuTableDataSourceDidSelectItemNotification,
            object: nil)
    }
    
    override func tearDown() {
        super.tearDown()
        postedNotification = nil
        NSNotificationCenter.defaultCenter().removeObserver(self)
    }
    
    func didReceiveNotification(notification: NSNotification) {
        postedNotification = notification
    }
    
    func testANotificationIsPostedWhenACellIsTapped() {
        dataSource!.tableView(nil, 
            didSelectRowAtIndexPath:selectedIndexPath)

        XCTAssertEqual(postedNotification!.name, 
            MenuTableDataSourceDidSelectItemNotification,
            "Data source posts a notification when a cell is tapped")
    }
    
    func testPostedNotificationContainsMenuItemInfo() {
        dataSource!.tableView(nil, 
            didSelectRowAtIndexPath:selectedIndexPath)

        XCTAssertTrue(postedNotification!.object.isEqual(testMenuItem!),
            "Notification contains menu item object")
    }

    // Previous tests for data source are excluded here...
}
~~~
 
`setup` 메서드 안에서, 우리는 `MenuTableDataSourceDidSelectItemNotification` 라는 이름의 노티피케이션 옵져버를 테스트 클래스에 추가했었다. 알림이 도착할 때, `didReceiveNotification:` 메서드는 호출될 것이다. 노피티케이션 오브젝트는 `postedNotification` 변수에 저장된다. 그때 우리는 이것이 정확한 이름과 메뉴 아이템 인스턴스라는 것이 증명된다. 여기서 중요하게 생각해야 할 점은 `tearDown` 메서드 안에 옵져버가 테스트 클래스를 지운다는 것이다. 우리는 [NSNotificationCenter](http://goo.gl/TfnJ3T) 에서 노티피케이션이 보내졌을때 확인 할 수 있는 API를 제공하지 않기 때문에 이 복잡한 프로세스를 통해서 노티피케이션이 정말로 보내지는지 증명할 수 있었다. 

> [Building Menu Items](#building_menu_items) 섹션 안에서 나는 가짜 오브젝트를 사용하는 것을 추천한다. 그러나, 나는 위에 테스트에서 `NSNotificationCenter` 클래스를 사용했다. 일반적으로 나는 애플 프레임워크에서 제공되는 오브젝트의 대체되는 다른것을 사용하지 않는다. 그들은 신뢰할 수 있는 용어와 속도를 공평하게 한다. 그 들이 말하기를 만약 당신의 테스트에서 애플 프레임워크로 부터 제공하는 객체의 사용이 줄어든다면 그들을 위해서 테스트를 대체하기 위한 다른걸 만드는 것을 주저하지 않을 것이다.

`MenuTableDefaultDataSource` 클래스 안에서 `UITableViewDataSource` 프로토콜로 부터 `tableView:didSelectRowAtIndexPath:` 메서드가 테스트를 통과 하도록 구현한다.

~~~swift
func tableView(tableView: UITableView!,
    didSelectRowAtIndexPath indexPath: NSIndexPath!)
{
    let menuItem = menuItems?[indexPath.row]

    let notification = 
        NSNotification(name: MenuTableDataSourceDidSelectItemNotification,
                       object:menuItem)

    NSNotificationCenter.defaultCenter().postNotification(notification)
}
~~~

<a name="managing_menu_table_view"></a>
### Managing Menu Table View (메뉴 테이블 뷰 관리)

`MenuViewController` 는 테이블뷰와 메뉴에서 보여줄 모든 필요한 뷰들을 관리하는 역할을 할 것이다. 첫번째로 우리는 필요한 데이터소스를 만들야 한다. 또한 테이블뷰와 타이틀이 확실히 필요하다. 새로운 테스트 파일 `MenuViewControllerTests.swift` 을 만들고 아래의 내용을 추가한다.

~~~swift
import UIKit
import XCTest

class MenuViewControllerTests: XCTestCase {
    var menuViewController: MenuViewController?
    var dataSource: MenuTableDataSource?
    var tableView: UITableView?
    
    override func setUp() {
        super.setUp()
        dataSource = MenuTableFakeDataSource()
        tableView = UITableView()
        
        menuViewController = MenuViewController()
        menuViewController?.dataSource = dataSource
        menuViewController?.tableView = tableView
    }

    func testHasATitle() {
        menuViewController?.viewDidLoad()
        XCTAssertEqual(menuViewController!.title!, "App Menu",
            "Menu view should show a proper title")
    }
    
    func testCanBeAssignedADataSource() {
        XCTAssertTrue(dataSource!.isEqual(menuViewController?.dataSource),
            "A data source can be assigned to a menu view controller")
    }
    
    func testHasATableView() {
        XCTAssertTrue(tableView!.isEqual(menuViewController?.tableView),
            "Menu view controller has a table view")
    }
}
~~~

여기에 실제 데이터 소스를 사용하는 것 대신에, 우리는 `MenuTableFakeDataSource` 라는 이름의 가짜 데이터를 사용할 것이다. `AppMenuTests` 안에 `MenuTableFakeDataSource.swift` 라는 이름의 스위프트 파일을 생성하고 타겟을 정하고 다음의 코드로 대체한다.

~~~swift
import Foundation
import UIKit

class MenuTableFakeDataSource : NSObject, MenuTableDataSource {
    func setMenuItems(menuItems: [MenuItem]) {
    }
    
    // MARK: - UITableView data source methods
    
    func tableView(tableView: UITableView!,
                   numberOfRowsInSection section: Int)
                   -> Int
    {
        return 1
    }
    
    func tableView(tableView: UITableView!,
                   cellForRowAtIndexPath indexPath: NSIndexPath!)
                   -> UITableViewCell!
    {
        return nil
    }
}
~~~

모든 `MenuTableFakeDataSource` 은 `MenuTableDataSource` 프로토콜 안의 필요 메서들 구현을 요구한다. 그리고 `MenuTableDataSource` 에 일치하는 모든 객체들을 대신한다. 지금 `MenuViewController` 클래스를 생성한다. (*AppMenu 그룹을 오른쪽으로 클릭 > New File > iOS > Source > Cocoa Touch Class*). `UIViewController` 의 서브클래스로 생성한다 그리고 *Also create XIB file* 체크박스도 선택한다. 그리고 절대 타겟을 두개다 추가하는 것을 잊지말자. 이 작업은 위의 선언된 두개의 구역과 타이틀을 선정하기 위한 테스트를 통과 하기 위함이다.

~~~swift
import UIKit

class MenuViewController: UIViewController {
    @IBOutlet weak var tableView: UITableView!
    var dataSource: MenuTableDataSource?

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "App Menu"
    }
}
~~~

`MenuViewCOntroller.xib` 의 메인뷰 사이즈는 *Attricbutes Inspector* 섹션 안에서 *Simulated Metrics* 을 *iPhont 4-inch* 로 변경한다. 뷰의 오리엔테이션은 *Portrait* 로 설정한다. 메인 뷰의 서브뷰, 테이블뷰 같은 뷰들을 추가한 후에. `MenuViewController` 클래스의 `tableView` 아울렛을 XIB의 테이블뷰와 연결한다.

다음으로 우리는 `MenuViewController` 의 델리게이트와 데이터소스 영역들에 대한 설정들을 우리가 정한 데이터 소스 객체로 지정해야 한다. [viewDidLoad](http://goo.gl/OeT0hV) 메서드는 우리가 원하는 연결을 정할 수 있는 곳이다. 다음 테스트를 코드를 확인하자.

~~~swift
func testTableViewIsGivenADataSourceInViewDidLoad() {
    menuViewController?.viewDidLoad()
    XCTAssertTrue(tableView!.dataSource.isEqual(dataSource),
        "Data source for the menu table view is set in viewDidLoad method")
}

func testTableViewIsGivenADelegateInViewDidLoad() {
    menuViewController?.viewDidLoad()
    XCTAssertTrue(tableView!.delegate.isEqual(dataSource),
        "Delegate for the menu table view is set in viewDidLoad method")
}
~~~

위의 테스트를 통과 시키기 위해 `viewDidLoad` 안에서 테이블뷰의 데이터소스와 델리게이트를 연결한다.

Set table view's data source and delegate properties in `viewDidLoad` to make above tests pass.

~~~swift
override func viewDidLoad() {
    super.viewDidLoad()
    title = "App Menu"
    tableView.dataSource = dataSource
    tableView.delegate = dataSource
}
~~~

[Handling Menu Item Tap Event](#handling_menu_item_tap_event) 에서 우리는 메뉴 아이템을 탭했을 때  `MenuTableDefaultDataSource` 가 노티피케이션을 보내는 것을 만들었었다. `MenuViewController` 는 노티피케이션을 받아서 정확한 메뉴 아이템의 뷰인지 확인할 수 있는 것이 필요하다. 만약 그 노티피케이션이 도착했는데 `MenuViewController` 의 뷰가 감춰져 있다면, 그건 무시될 것이다. 그러므로, `viewDidAppear:` 메서드에서 노티피케이션을 등록해야한다. 또한 `viewDidDisaapear:` 메서드 안에서 해체를 해주어야 한다. 다음의 테스트를 통해서 요구하는 것을 확인하자.

~~~swift
let postedNotification = "MenuViewControllerTestsPostedNotification"

class MenuViewControllerTests: XCTestCase {
    var menuViewController: MenuViewController?
    var dataSource: MenuTableDataSource?
    var tableView: UITableView?
    
    override func setUp() {
        super.setUp()
        dataSource = MenuTableFakeDataSource()
        tableView = UITableView()
        
        menuViewController = MenuViewController()
        menuViewController?.dataSource = dataSource
        menuViewController?.tableView = tableView
    }

    override func tearDown() {
        super.tearDown()        
        objc_removeAssociatedObjects(menuViewController)
    }
    
    // ...

    func testRegistrationForNotificationHappensInViewDidAppear() {
        swizzleNotificationHandler()
        menuViewController?.viewDidAppear(false)
        
        let notification =
            NSNotification(
                name: MenuTableDataSourceDidSelectItemNotification,
                object: nil)
        
        NSNotificationCenter.defaultCenter().postNotification(notification)
        
        XCTAssertNotNil(
        objc_getAssociatedObject(menuViewController, postedNotification),
        "Listens to notification only when it's view is visible")
    }
    
    func testRemovesItselfAsListenerForNotificationInViewDidDisappear() {
        swizzleNotificationHandler()
        menuViewController?.viewDidAppear(false)
        menuViewController?.viewDidDisappear(false)
        
        let notification =
            NSNotification(
                name: MenuTableDataSourceDidSelectItemNotification,
                object: nil)
        
        NSNotificationCenter.defaultCenter().postNotification(notification)
        
        XCTAssertNil(
        objc_getAssociatedObject(menuViewController, postedNotification),
        "Stops listening for notfication when view is not visible anymore")
    }
    
    // Mark: - Method swizzling
    
    func swizzleNotificationHandler() {
        var realMethod: Method = 
            class_getInstanceMethod(
                object_getClass(menuViewController),
            Selector.convertFromStringLiteral(
                "didSelectMenuItemNotification:"))
        
        var testMethod: Method = 
            class_getInstanceMethod(
                object_getClass(menuViewController),
            Selector.convertFromStringLiteral(
                "testImpl_didSelectMenuItemNotification:"))
        
        method_exchangeImplementations(realMethod, testMethod)
    }
}

extension MenuViewController {
    func testImpl_didSelectMenuItemNotification(
        notification: NSNotification) 
    {
        objc_setAssociatedObject(self,
            postedNotification,
            notification,
            UInt(OBJC_ASSOCIATION_RETAIN))
    }
}
~~~

굉장히 많은 코드가 있는데, 설명을 하자면. `MenuViewController` `MenuTableDataSourceDidSelectItemNotification` 의 호출을 받기 위해 자기 자신을 등록했다. 우리는 노티피케이션이 도착했을 때 우리가 원하는 메서드를 어떻게 호출할 것인지 알아야 한다. 이것은 한번만 받을 수 있어서, 노티피케이션이 메서드를 통과할 때 실체를 알수 있는 검증이 필요하다. 간단하게 비 개인화 속성으로 `MenuViewController` 안에 노티피케이션을 만들 것이다. 그러나 개인적으로 이 접근 방법을 좋아하진 않는다. `MenuViewController` 는 단지 테스트를 위해서 강제로 노출되지 않아야 한다. 여기에 더 좋은 방법이 있다. 우리는 어떻게 [swizzle](http://nshipster.com/method-swizzling/) 노티피케이션 핸들러를 런타임에서 각자 다르게 요구되는 구현을 테스트의 목적에 맞게 사용할 수 있을까? 다음 코드를 보자.

~~~swift
func swizzleNotificationHandler() {
    var realMethod: Method = 
        class_getInstanceMethod(
            object_getClass(menuViewController),
        Selector.convertFromStringLiteral(
            "didSelectMenuItemNotification:"))
    
    var testMethod: Method = 
        class_getInstanceMethod(
            object_getClass(menuViewController),
        Selector.convertFromStringLiteral(
            "testImpl_didSelectMenuItemNotification:"))
    
    method_exchangeImplementations(realMethod, testMethod)
}
~~~

그리고 여기에 `MenuViewController` 클래스를 테스트 구현에 요구되는 대로 [extension](http://goo.gl/lL1Cwy) 하였다.

~~~swift
extension MenuViewController {
    func testImpl_didSelectMenuItemNotification(
        notification: NSNotification) 
    {
        objc_setAssociatedObject(self,
                                 postedNotification,
                                 notification,
                                 UInt(OBJC_ASSOCIATION_RETAIN))
    }
}
~~~

우리는 여기에 테스트안에서 측정할 수 있는 `postedNotification` 을 지속적으로 받기 위한 노티피케이션 등록을 모두했다. `testRegistrationForMenuItemTappedNotificationHappensInViewDidAppear` 안에서 swizzle 노티피케이션 핸들러를 등록한 후에 `viewDidAppear` 를 호출하고, nil 이 아닌 `postedNotificiation` 을 노티피케이션과 검증을 위해 보냈다. 반면 `testRemovesItselfAsListenerForMenuItemTappedNotificationInViewDidDisappear` 에서는 먼저 `viewDidAppear` 를 호출하고 `MenuViewController` 에 노티피케이션을 등록했다. 이 노티피케이션은 `viewDidDisappear` 안에서 자신을 옵져버를 제거한다. 이는 노티피케이션이 `MenuViewController` 에서 `viewDidDisappear` 호출된 후에도 노티피케이션을 받을 수 있기 때문이다.

테스트들을 통과하기 위해선 우리는 모두 노티피케이션의 등록과 해제 적절한 곳에서 해야한다.

~~~
override func viewDidAppear(animated: Bool) {
    super.viewDidAppear(animated)
    NSNotificationCenter.defaultCenter().addObserver(self,
        selector: "didSelectMenuItemNotification:",
        name: MenuTableDataSourceDidSelectItemNotification,
        object: nil)
}

override func viewDidDisappear(animated: Bool) {
    super.viewDidDisappear(animated)
    NSNotificationCenter.defaultCenter().removeObserver(self,
        name: MenuTableDataSourceDidSelectItemNotification,
        object: nil)
}

func didSelectMenuItemNotification(notification: NSNotification?) {
    // Handle notification
}
~~~

<a name="sliding_views_in"></a>
Sliding Views In
================

menu item이 눌렸을 때 view가 보여져야 한다. 하지만 어떤것? menu item에 직접 물어보는 건 어떨까? 단순함을 유지하기 위해 view controller의 이름을 `MenuItem`에 `tapHandlerName` property 로 저장하자.

~~~swift
class MenuItemTests: XCTestCase {
    // ...

    func testThatMenuItemCanBeAssignedATapHandlerName() {
        menuItem!.tapHandlerName = "someViewController"
        XCTAssertEqual(menuItem!.tapHandlerName!, 
            "someViewController",
            "Tap handler name should be what we assigned")
    }
}
~~~

~~~swift
class MenuItem {
    // ...

    var tapHandlerName: String?
}
~~~

이것은 menu item이 tap handler를 갖지 않을 좋을 방법이다. Therefore, `tapHandlerName` property를 optional로 만들어야 한다.  이제 `MenuItem`에 property를 추가하고 `menuItems.plist`, `FakeMenuItemsReader`, `MenuItemsPlistReaderTests`, `MenuItemBuilderTests`, 그리고 `MenuItemBuilder`를 맞추자. 조정된 코드는 아래에 나열되어 있다.

~~~xml
<!--menuItems.plist-->

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<array>
    <dict>
        //...
        <key>tapHandlerName</key>
        <string>ContributionsViewController</string>
    </dict>
    <dict>
        //...
        <key>tapHandlerName</key>
        <string>RepositoriesViewController</string>
    </dict>
    <dict>
        //...
        <key>tapHandlerName</key>
        <string>PublicActivityViewController</string>
    </dict>
</array>
</plist>
~~~

~~~swift
class FakeMenuItemsReader : MenuItemsReader {
    // ...
    
    func readMenuItems() -> ([[String : String]]?, NSError?) {
        let menuItem1 = 
            missingTitle ? menuItem1WithMissingTitle() 
                         : menuItem1WithNoMissingTitle()
        
        let menuItem2 = ["title": "Menu Item 2",
                         "subTitle": "Menu Item 2 subtitle",
                         "iconName": "iconName2",
                         "tapHandlerName": "someViewController1"]
        
        return ([menuItem1, menuItem2], nil)
    }
    
    func menuItem1WithMissingTitle() -> [String : String] {
        return ["subTitle": "Menu Item 1 subtitle",
                "iconName": "iconName1",
                "tapHandlerName": "someViewController2"]
    }

    // ...
}
~~~

~~~swift
class MenuItemsPlistReaderTests: XCTestCase {
    // ...

    func testPlistIsDeserializedCorrectly() {        
        // ...
        XCTAssertEqual(firstRow!["tapHandlerName"]!, 
            "ContributionsViewController",
            "1st row's tap handler should be what's in plist")
        
        // ...
        XCTAssertEqual(secondRow!["tapHandlerName"]!, 
            "RepositoriesViewController",
            "2nd row's tap handler should be what's in plist")
        
        // ...
        XCTAssertEqual(thirdRow!["tapHandlerName"]!, 
            "PublicActivityViewController",
            "3rd row's tap handler should be what's in plist")
    }
}
~~~

~~~swift
class MenuItemBuilderTests: XCTestCase {
    // ...

    func testMenuItemPropertiesContainValuesPresentInDictionary() {
        // ...
        
        let rawDictionary1 = metadata![0]
        let menuItem1 = menuItems![0]

        // ...
        
        XCTAssertEqual(menuItem1.tapHandlerName!, 
            rawDictionary1["tapHandlerName"]!,
            "1st menu item's tap handler should be what's in the 1st dict")
        
        let rawDictionary2 = metadata![1]
        let menuItem2 = menuItems![1]
        
        // ...

        XCTAssertEqual(menuItem2.tapHandlerName!, 
            rawDictionary2["tapHandlerName"]!,
            "2nd menu item's tap handler should be what's in the 2nd dict")
    }
}
~~~

~~~swift
class MenuItemBuilder {
    func buildMenuItemsFromMetadata(metadata: [[String : String]]) 
        -> ([MenuItem]?, NSError?) 
    {
        // ...
        
        for dictionary in metadata {
            if let title = dictionary["title"] {
                let menuItem = MenuItem(title: title)
                menuItem.subTitle = dictionary["subTitle"]
                menuItem.iconName = dictionary["iconName"]
                menuItem.tapHandlerName = dictionary["tapHandlerName"]
                menuItems.append(menuItem)
            }
            else {
                error = missingTitleError()
                menuItems.removeAll(keepCapacity: false)
                break
            }
        }
        
        return (menuItems, error)
    }

    // ...
}
~~~

다음으로 menu item이 눌렸을 때 `MenuViewController`가 맞는 view를 보여주도록 해야한다. 다음 test가 그렇게 할 것이다.

~~~swift
class MenuViewControllerTests: XCTestCase {
    // ...
    var navController: UINavigationController?

    override func setUp() {
        // ...                
        navController = 
            UINavigationController(rootViewController: menuViewController)
    }

    // ...

    func testCorrectViewIsDisplayedWhenContributionsMenuItemIsTapped() {
        let menuItem = MenuItem(title: "Contributions")
        menuItem.tapHandlerName = "ContributionsViewController"

        let notification = 
            NSNotification(
                name: MenuTableDataSourceDidSelectItemNotification,
                object: menuItem)
        
        menuViewController?.didSelectMenuItemNotification(notification)
        let topViewController = navController?.topViewController
        
        XCTAssertTrue(topViewController is ContributionsViewController,
            "Contributions view is displayed for Contributions menu item")
    }
    
    func testCorrectViewIsDisplayedWhenRepositoriesMenuItemIsTapped() {
        let menuItem = MenuItem(title: "Repositories")
        menuItem.tapHandlerName = "RepositoriesViewController"

        let notification = 
            NSNotification(
                name: MenuTableDataSourceDidSelectItemNotification,
                object: menuItem)
        
        menuViewController?.didSelectMenuItemNotification(notification)
        let topViewController = navController?.topViewController
        
        XCTAssertTrue(topViewController is RepositoriesViewController,
            "Repositories view is displayed for Contributions menu item")
    }
    
    func testCorrectViewIsDisplayedWhenPublicActivityMenuItemIsTapped() {
        let menuItem = MenuItem(title: "PublicActivity")
        menuItem.tapHandlerName = "PublicActivityViewController"

        let notification = 
            NSNotification(
                name: MenuTableDataSourceDidSelectItemNotification,
                object: menuItem)
        
        menuViewController?.didSelectMenuItemNotification(notification)
        let topViewController = navController?.topViewController
        
        XCTAssertTrue(topViewController is PublicActivityViewController,
            "Public activity view is displayed for Contributions menu item")
    }

    // ...
}
~~~

`MenuViewController`를 app navigation stack의 오른쪽 view controller로 push하도록 만들자

~~~swift
class MenuViewController: UIViewController {
    // ...
    func didSelectMenuItemNotification(notification: NSNotification?) {
        var menuItem: MenuItem? = notification!.object as? MenuItem
        
        if menuItem != nil {
            var tapHandler: UIViewController?
            
            switch menuItem!.tapHandlerName! {
                case "ContributionsViewController":
                tapHandler = 
                    ContributionsViewController(
                        nibName: "ContributionsViewController",
                        bundle: nil)

                case "RepositoriesViewController":
                tapHandler = 
                    RepositoriesViewController(
                        nibName: "RepositoriesViewController",
                        bundle: nil)

                case "PublicActivityViewController":
                tapHandler = 
                    PublicActivityViewController(
                        nibName: "PublicActivityViewController",
                        bundle: nil)

                default:
                tapHandler = nil
            }
            
            if tapHandler != nil {
                self.navigationController.pushViewController(tapHandler,
                                                             animated: true)
            }
        }
    }
}
~~~

또한 tap handler class들을 만들어야 한다. 다음의 view controller를 만들고 각각에 *AppMenu* and *AppMenuTests* target을 추가한다. 각각을 위한 *XIB* 파일을 만드는 것 또한 잊지 말자.

* `ContributionsViewController`
* `RepositoriesViewController`
* `PublicActivityViewController`

switch case 문을 사용하는 대신에 runtime에 view controller를 만들지 않는지 궁금할 것이다.

* 나는 Swift에서 가장 좋은 방법이 무엇인지 확신할 수 없다. Objective-C에서는 다음의 code에서와 같이 쉽게 할 수 있다.

  ~~~Obj-C
    UIViewController *tapHandler = nil;
    Class tapHandlerClass =
         NSClassFromString(menuItem.tapHandlerName)

    if (tapHandlerClass) {
        tapHandler = [[tapHandlerClass alloc] init];
    }
  ~~~

* Objective-C와는 다르게, Swift는 XIB의 이름과 view controller의 이름이 같더라도 view controller의 instance를 만들 때, XIB를 지정해야 한다. 게다가, 간단히 `alloc`, `init`으로 부르는 기능은 Swift에서는 필요하지 않다.

불행히도, test들을 통과하게 하진 못한다. 모든 class에 드러내다. It turns out that so far we have built every class we initially set out to build except `AppMenuManager`. 그 class가 만들어지면 위의 test를 통과할 수 있어야 한다. 해보자.

<a name="managing_app_menu"></a>
### Managing App Menu

Create a new test file named `AppMenuManagerTests.swift` in *AppMenuTests* target. Add following tests to it.

~~~swift
import UIKit
import XCTest

class AppMenuManagerTests: XCTestCase {
    var menuManager: AppMenuManager?
    var fakeMenuItemsReader: FakeMenuItemsReader?
    var fakeMenuItemBuilder: FakeMenuItemBuilder?
    var menuViewController: MenuViewController?
    
    override func setUp() {
        super.setUp()
        menuManager = AppMenuManager()
        fakeMenuItemsReader = FakeMenuItemsReader()
        fakeMenuItemBuilder = FakeMenuItemBuilder()
        menuManager?.menuItemsReader = fakeMenuItemsReader
        menuManager?.menuItemBuilder = fakeMenuItemBuilder
    }

    func testReturnsNilIfMetadataCouldNotBeRead() {
        fakeMenuItemsReader?.errorToReturn = fakeError()
        menuViewController = menuManager?.menuViewController()
        
        XCTAssertNil(menuViewController,
        "Doesn't create menu VC if metadata couldn't be read")
    }
    
    func testMetadataIsPassedToMenuItemBuilder() {
        menuViewController = menuManager?.menuViewController()

        var (metadataReturnedByReader, _) = 
            fakeMenuItemsReader!.readMenuItems()

        let metadataReceivedByBuilder = 
            fakeMenuItemBuilder!.metadata
        
        XCTAssertTrue(metadataReceivedByBuilder?.count == 
            metadataReturnedByReader?.count,
            "Number of dictionaries in metadata should match")
    }
    
    func testReturnsNilIfMenuItemsCouldNotBeBuilt() {
        fakeMenuItemBuilder?.errorToReturn = fakeError()
        menuViewController = menuManager?.menuViewController()
        
        XCTAssertNil(menuViewController,
        "Doesn't create menu VC if menu items couldn't be built")
    }
    
    func testCreatesMenuViewControllerIfMenuItemsAvailable() {
        fakeMenuItemBuilder?.menuItemsToReturn = fakeMenuItems()
        menuViewController = menuManager?.menuViewController()
        
        XCTAssertNotNil(menuViewController,
        "Creates menu view controller if menu items are available")
        
        XCTAssertNotNil(menuViewController?.dataSource,
            "Menu view controller is given a data source")
    }
    
    func fakeError() -> NSError {
        let errorMessage = "Fake error description"
        let userInfo = [NSLocalizedDescriptionKey: errorMessage]
        
        return NSError(domain: "Fake Error domain",
                       code: 0,
                       userInfo: userInfo)
    }
    
    func fakeMenuItems() -> [MenuItem] {
        let menuItem = MenuItem(title: "Fake menu item")
        return [menuItem]
    }
}
~~~

`AppMenuManager` is responsible for creating `MenuViewController` if `MenuItem` objects were created successfully from the metadata. If not, it just returns nil. Since `AppMenuManager` mostly coordinates the interaction between various objects rather than doing the work itself, we also need to make sure that it passes the metadata (if read successfully) to the builder. You might have noticed that we are using fake menu items reader and builder objects here so that we can control what gets returned to app menu manager in tests. We built a fake menu items reader in [*Building Menu Items*](#building_menu_items), but it doesn't provide a way for us to set the error. Let's take care of that.

~~~swift
class FakeMenuItemsReader : MenuItemsReader {
    var missingTitle: Bool = false
    var errorToReturn: NSError? = nil
    
    func readMenuItems() -> ([[String : String]]?, NSError?) {
        if errorToReturn != nil {
            return (nil, errorToReturn)
        }
        else {
            let menuItem1 = 
                missingTitle ? menuItem1WithMissingTitle() 
                             : menuItem1WithNoMissingTitle()
            
            let menuItem2 = ["title": "Menu Item 2",
                "subTitle": "Menu Item 2 subtitle",
                "iconName": "iconName2",
                "tapHandlerName": "someViewController1"]
            
            return ([menuItem1, menuItem2], nil)
        }
    }

    // ...
~~~

Next we need to create `FakeMenuItemBuilder` class. Now that there is going to be more than one class playing the role of a menu item builder, we should create a protocol to make it clear what it means for a class to become a menu item builder. For now, playing that role means implementing `buildMenuItemsFromMetadata` method correctly. Listed below is the new protocol.

~~~swift
import Foundation

protocol MenuItemBuilder {
    func buildMenuItemsFromMetadata(metadata: [[String : String]]) -> ([MenuItem]?, NSError?)
}
~~~

Wait a minute. Didn't we already name our real builder class `MenuItemBuilder`? Yes we did. `MenuItemBuilder` name is better suited for a protocol. Let's rename the original builder class to `MenuItemDefaultBuilder`.

~~~swift
import Foundation

let MenuItemDefaultBuilderErrorDomain = "MenuItemDefaultBuilderErrorDomain"

enum MenuItemDefaultBuilderErrorCode : Int {
    case MissingTitle
}

class MenuItemDefaultBuilder : MenuItemBuilder {
    func buildMenuItemsFromMetadata(metadata: [[String : String]]) 
        -> ([MenuItem]?, NSError?) 
    {
        var menuItems = [MenuItem]()
        var error: NSError?
        
        for dictionary in metadata {
            if let title = dictionary["title"] {
                let menuItem = MenuItem(title: title)
                menuItem.subTitle = dictionary["subTitle"]
                menuItem.iconName = dictionary["iconName"]
                menuItem.tapHandlerName = dictionary["tapHandlerName"]
                menuItems.append(menuItem)
            }
            else {
                error = missingTitleError()
                menuItems.removeAll(keepCapacity: false)
                break
            }
        }
        
        return (menuItems, error)
    }
    
    private func missingTitleError() -> NSError {
        let userInfo = 
            [NSLocalizedDescriptionKey: "All menu items must have a title"]

        return NSError(domain: MenuItemDefaultBuilderErrorDomain,
            code: MenuItemDefaultBuilderErrorCode.MissingTitle.toRaw(),
            userInfo: userInfo)
    }
}
~~~

We also need to adjust tests to use the new name.

~~~swift
class MenuItemDefaultBuilderTests: XCTestCase {
    var menuItemBuilder: MenuItemDefaultBuilder?
    var fakeMenuItemsReader: FakeMenuItemsReader?
    var menuItems: [MenuItem]?
    var error: NSError?
    
    override func setUp() {
        fakeMenuItemsReader = FakeMenuItemsReader()
        fakeMenuItemsReader!.missingTitle = true

        let (metadata, _) = 
            fakeMenuItemsReader!.readMenuItems()
        
        menuItemBuilder = MenuItemDefaultBuilder()
        (menuItems, error) = 
            menuItemBuilder!.buildMenuItemsFromMetadata(metadata!)
    }
    
    func testCorrectErrorDomainIsReturnedWhenTitleIsMissing() {
        let errorDomain = error?.domain
        XCTAssertEqual(errorDomain!, 
            MenuItemDefaultBuilderErrorDomain,
            "Correct error domain is returned")
    }
    
    func testMissingTitleErrorCodeIsReturnedWhenTitleIsMissing() {
        let errorCode = error?.code
        XCTAssertEqual(errorCode!, 
            MenuItemDefaultBuilderErrorCode.MissingTitle.toRaw(),
            "Correct error code is returned")
    }
    
    // ...
}
~~~

Finally, here is what the `FakeMenuItemReader` class looks like. You don't need to add this class to the *AppMenu* target since it's only used in tests.

~~~swift
import Foundation

class FakeMenuItemBuilder : MenuItemBuilder {
    var errorToReturn: NSError? = nil
    var menuItemsToReturn: [MenuItem]? = nil
    var metadata: [[String : String]]? = nil
    
    func buildMenuItemsFromMetadata(metadata: [[String : String]]) 
        -> ([MenuItem]?, NSError?) 
    {
        self.metadata = metadata
        return (menuItemsToReturn, errorToReturn)
    }
}
~~~

It makes the metadata passed to it available for inspection. It also allows us to set the error and menu items we want it to return which is very convenient. Now we are ready to build the `AppMenuManager` class. Here is what it looks like.

~~~swift
import Foundation
import UIKit

class AppMenuManager {
    var menuItemsReader: MenuItemsReader? = nil
    var menuItemBuilder: MenuItemBuilder? = nil
    
    func menuViewController() -> MenuViewController? {
        let (metadata, metadataError) = 
            menuItemsReader!.readMenuItems()
        
        if metadataError != nil {
            tellUserAboutError(metadataError!)
        }
        else if let menuItems = menuItemsFromMetadata(metadata!) {
            return menuViewControllerFromMenuItems(menuItems)
        }
        
        return nil
    }
    
    private func tellUserAboutError(error: NSError) {
        println("Error domain: \(error.domain)")
        println("Error code: \(error.code)")
        
        let alert = UIAlertView(title: "Error",
                                message: error.localizedDescription,
                                delegate: nil,
                                cancelButtonTitle: nil,
                                otherButtonTitles: "OK")
        alert.show()
    }
    
    private func menuItemsFromMetadata(metadata: [[String : String]]) 
        -> [MenuItem]? 
    {
        let (menuItems, builderError) = 
            menuItemBuilder!.buildMenuItemsFromMetadata(metadata)
        
        if builderError != nil {
            tellUserAboutError(builderError!)
            return nil
        }

        return menuItems
    }
    
    private func menuViewControllerFromMenuItems(menuItems: [MenuItem]) 
        -> MenuViewController 
    {
        let dataSource = MenuTableDefaultDataSource()
        dataSource.menuItems = menuItems
        
        let menuViewController = 
            MenuViewController(nibName: "MenuViewController", bundle: nil)

        menuViewController.dataSource = dataSource        
        return menuViewController
    }
}
~~~

I apologize for not staying true to the *read-green-refactor* cycle here. I wanted to focus more on important techniques that make writting tests a bit easier rather than showing you every single step in the process. One of those techniques is creating fake (or test double) objects that play the same role as the real objects so that we can easily swap them to make our tests more maintainable. Speaking of fake objects, [Martin Fowler](http://martinfowler.com/) has written a [great post](http://martinfowler.com/articles/mocksArentStubs.html) on the topic.

Before we move on, I would like to emphasize the importance of [Dependency Injection](http://www.martinfowler.com/articles/injection.html) in writing testable and reusable classes. Our `AppMenuManager` class needs to work with two other classes that conform to `MenuItemsReader` and `MenuItemBuilder` protocols to successfully create `MenuItem` objects. Had we not exposed these two dependencies via public properties, we would not have been able to pass in fake objects. Those fake objects came very handy while setting up the desired test scenarios in order to verify that `AppMenuManager` behaved as expected. Therefore, I recommend exposing every single dependency your classes have unless those dependencies are classes provided by Apple frameworks.

<a name="putting_it_all_together"></a>
Putting It All Together
=======================

We are almost there. Now that we have built every class, let's put them together in `AppDelegate`. But first we will write some tests to verify that  `AppDeleate` behaves as expected. Create a new test file named `AppDelegateTests.swift` in *AppMenuTests* target. Add following tests to it.

~~~swift
import UIKit
import XCTest

class AppDelegateTests: XCTestCase {
    var window: UIWindow?
    var navController: UINavigationController?
    var appDelegate: AppDelegate?
    var appMenuManager: AppMenuManager?
    var didFinishLaunchingWithOptionsReturnValue: Bool?
    
    override func setUp() {
        super.setUp()
        
        window = UIWindow()
        navController = UINavigationController()
        appMenuManager = AppMenuManager()
        appDelegate = AppDelegate()
        appDelegate?.window = window
        appDelegate?.navController = navController
    }
    
    func testRootVCForWindowIsNotSetIfMenuViewControllerCannotBeCreated() {
        class FakeAppMenuManager: AppMenuManager {
            override func menuViewController() -> MenuViewController? {
                return nil
            }
        }

        appDelegate?.appMenuManager = FakeAppMenuManager()
        appDelegate?.application(nil, didFinishLaunchingWithOptions: nil)

        XCTAssertNil(window!.rootViewController,
        "Window's root VC shouldn't be set if menu VC can't be created")
    }
    
    func testWindowHasRootViewControllerIfMenuViewControllerIsCreated() {
        class FakeAppMenuManager: AppMenuManager {
            override func menuViewController() -> MenuViewController? {
                return MenuViewController()
            }
        }
        
        appDelegate?.appMenuManager = FakeAppMenuManager()
        appDelegate?.application(nil, didFinishLaunchingWithOptions: nil)
        XCTAssertEqual(window!.rootViewController, navController!,
        "App delegate's nav controller should be the root view controller")
    }
    
    func testMenuViewControllerIsRootVCForNavigationController() {
        class FakeAppMenuManager: AppMenuManager {
            override func menuViewController() -> MenuViewController? {
                return MenuViewController()
            }
        }
        
        appDelegate?.appMenuManager = FakeAppMenuManager()        
        appDelegate?.application(nil, didFinishLaunchingWithOptions: nil)
        
        let topViewController = 
            appDelegate?.navController?.topViewController

        XCTAssertTrue(topViewController is MenuViewController,
            "Menu view controlelr is root VC for nav controller")
    }
    
    func testWindowIsKeyAfterAppIsLaunched() {
        appDelegate?.application(nil, didFinishLaunchingWithOptions: nil)
        XCTAssertTrue(window!.keyWindow,
            "App delegate's window should be the key window for the app")
    }
    
    func testAppDidFinishLaunchingDelegateMethodAlwaysReturnsTrue() {
        didFinishLaunchingWithOptionsReturnValue =
            appDelegate?.application(nil, 
                                     didFinishLaunchingWithOptions: nil)
        
        XCTAssertTrue(didFinishLaunchingWithOptionsReturnValue!,
            "Did finish launching delegate method should return true")
    }
}
~~~

> In [Managing App Menu](#managing_app_menu), we created `MenuItemBuilder` protocol when we realized that we needed a fake object that could stand-in for the real menu builder. But, here we are creating fake app menu manager objects inside the tests themselves. It's perfectly fine to do so. If we decide to rename `menuViewController` method in real app menu manager class, Swift will force us to modify all our fake objects to use the new method name. Because of that, all these fake objects will always be in sync with the real app menu manager. This approach comes very handy if you need to create quick fake objects inside the tests.

When we created a new Xcode project, `AppDelegate` was added only to the *AppMenu* target. We need to add it to `AppMenuTests* target as well. After that replace its content with following:

~~~swift
import UIKit

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?
    var navController: UINavigationController?
    var appMenuManager: AppMenuManager?

    func application(application: UIApplication!,
        didFinishLaunchingWithOptions launchOptions: NSDictionary!)
        -> Bool
    {
        if window == nil {
            window = UIWindow(frame: UIScreen.mainScreen().bounds)
        }
        
        let menuItemsPlistReader = MenuItemsPlistReader()
        menuItemsPlistReader.plistToReadFrom = "menuItems"
        
        if appMenuManager == nil {
            appMenuManager = AppMenuManager()
        }

        appMenuManager!.menuItemsReader = menuItemsPlistReader
        appMenuManager!.menuItemBuilder = MenuItemDefaultBuilder()
        
        if let menuViewController = appMenuManager!.menuViewController() {
            if navController == nil {
                navController = UINavigationController()
            }
            
            navController?.viewControllers = [menuViewController]
            window!.rootViewController = navController!
        }
        
        window!.makeKeyAndVisible()
        return true
    }
}
~~~

It would be nice to extract the code that configures `AppMenuManager` out from `AppDelegate`. We are going to apply what [Graham Lee](https://twitter.com/secboffin) taught us in [Test-Driven iOS Development](http://goo.gl/iiKpC1) here and create our own dependency injection class instead of using a full blown [depdendency injection framework](http://www.typhoonframework.org/). AppMenu is a simple app, at least for now. So we shouldn't be adding dependencies to it unless we need them. Create a new test file named `ObjectConfiguratorTests.swift` in *AppMenuTests* target and replace its content with following.

~~~swift
import UIKit
import XCTest

class ObjectConfiguratorTests: XCTestCase {
    var objectConfigurator: ObjectConfigurator?
    
    override func setUp() {
        super.setUp()
        objectConfigurator = ObjectConfigurator()
    }
    
    func testConfiguresAppMenuManagerCorrectly() {
        let appMenuManager = objectConfigurator?.appMenuManager()
        XCTAssertNotNil(appMenuManager, "App menu manager is not nil")
        
        XCTAssertTrue(appMenuManager?.menuItemsReader != nil,
            "App menu manager has a menu items reader")
        XCTAssertTrue(appMenuManager?.menuItemBuilder != nil,
            "App menu manager has a menu item builder")
    }
}
~~~

Create the `ObjectConfigurator` class and add it to both targets. Replace its content with following.

~~~swift
import UIKit

class ObjectConfigurator {
    func appMenuManager() -> AppMenuManager {
        let appMenuManager = AppMenuManager()
        let menuItemsPlistReader = MenuItemsPlistReader()
        
        menuItemsPlistReader.plistToReadFrom = "menuItems"
        appMenuManager.menuItemsReader = menuItemsPlistReader
        appMenuManager.menuItemBuilder = MenuItemDefaultBuilder()
        
        return appMenuManager
    }
}
~~~

Instead of creating an `AppMenuManager` object itself, app delegate will tell the object configurator to do so. Let's make changes to `AppDelegate` and its tests to include the new approach.

~~~swift
class FakeAppMenuManager: AppMenuManager {
    override func menuViewController() -> MenuViewController? {
        return MenuViewController()
    }
}

class FakeObjectConfigurator : ObjectConfigurator {
    override func appMenuManager() -> AppMenuManager {
        return FakeAppMenuManager()
    }
}

class AppDelegateTests: XCTestCase {
    var window: UIWindow?
    var navController: UINavigationController?
    var appDelegate: AppDelegate?
    var objectConfigurator: ObjectConfigurator?
    var didFinishLaunchingWithOptionsReturnValue: Bool?
    
    override func setUp() {
        super.setUp()
        window = UIWindow()
        navController = UINavigationController()
        appDelegate = AppDelegate()
        appDelegate?.window = window
        appDelegate?.navController = navController
    }
    
    func testRootVCForWindowIsNotSetIfMenuViewControllerCannotBeCreated() {
        class FakeAppMenuManager: AppMenuManager {
            override func menuViewController() -> MenuViewController? {
                return nil
            }
        }
        
        class FakeObjectConfigurator : ObjectConfigurator {
            override func appMenuManager() -> AppMenuManager {
                return FakeAppMenuManager()
            }
        }
        
        appDelegate?.objectConfigurator = FakeObjectConfigurator()
        appDelegate?.application(nil, didFinishLaunchingWithOptions: nil)

        XCTAssertNil(window!.rootViewController,
        "Window's root VC shouldn't be set if menu VC can't be created")
    }
    
    func testWindowHasRootViewControllerIfMenuViewControllerIsCreated() {
        appDelegate?.objectConfigurator = FakeObjectConfigurator()
        appDelegate?.application(nil, didFinishLaunchingWithOptions: nil)

        XCTAssertEqual(window!.rootViewController, navController!,
        "App delegate's nav controller should be the root VC")
    }
    
    func testMenuViewControllerIsRootVCForNavigationController() {
        appDelegate?.objectConfigurator = FakeObjectConfigurator()
        appDelegate?.application(nil, didFinishLaunchingWithOptions: nil)
        
        let topViewController = 
            appDelegate?.navController?.topViewController

        XCTAssertTrue(topViewController is MenuViewController,
            "Menu view controlelr is root VC for nav controller")
    }

    // ...
}
~~~

~~~swift
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?
    var navController: UINavigationController?
    var objectConfigurator: ObjectConfigurator?

    func application(application: UIApplication!,
        didFinishLaunchingWithOptions launchOptions: NSDictionary!)
        -> Bool
    {
        if window == nil {
            window = UIWindow(frame: UIScreen.mainScreen().bounds)
        }
        
        if objectConfigurator == nil {
            objectConfigurator = ObjectConfigurator()
        }
        
        let appMenuManager = objectConfigurator?.appMenuManager()
        if let menuViewController = appMenuManager!.menuViewController() {
            if navController == nil {
                navController = UINavigationController()
            }
            
            navController?.viewControllers = [menuViewController]
            window!.rootViewController = navController!
        }
        
        window!.makeKeyAndVisible()
        return true
    }
}
~~~

Let's turn our attention to `MenuViewController`. Although, all tests for it should be passing now we need to do a little refactoring. Let's extract the code that decides which view controller should be the tap handler into a separate class. Create a new test class named `MenuItemTapHandlerBuilderTests` in *AppMenuTests* target and replace its content with following.

~~~swift
import UIKit
import XCTest

class MenuItemTapHandlerBuilderTests: XCTestCase {
    var tapHandlerBuilder: MenuItemTapHandlerBuilder?
    var menuItem: MenuItem?

    override func setUp() {
        super.setUp()
        tapHandlerBuilder = MenuItemTapHandlerBuilder()
        menuItem = MenuItem(title: "Test menu item")
    }
    
    func testReturnsContributionsVCForContributionsMenuItem() {
        menuItem?.tapHandlerName = "ContributionsViewController"
        let tapHandler = tapHandlerBuilder?.tapHandlerForMenuItem(menuItem)
        
        XCTAssertTrue(tapHandler is ContributionsViewController,
            "Contributions VC should handle contributions menu item tap")
    }
    
    func testReturnsRepositoriesVCForRepositoriesMenuItem() {
        menuItem?.tapHandlerName = "RepositoriesViewController"
        let tapHandler = tapHandlerBuilder?.tapHandlerForMenuItem(menuItem)
        
        XCTAssertTrue(tapHandler is RepositoriesViewController,
            "Repositories VC should handle repositories menu item tap")
    }
    
    func testReturnsPublicActivityVCForPublicActivityMenuItem() {
        menuItem?.tapHandlerName = "PublicActivityViewController"
        let tapHandler = tapHandlerBuilder?.tapHandlerForMenuItem(menuItem)
        
        XCTAssertTrue(tapHandler is PublicActivityViewController,
            "PublicActivity VC should handle public activity menu item tap")
    }
    
    func testReturnsNilForAnyOtherMenuItem() {
        menuItem?.tapHandlerName = "UnknownViewController"
        let tapHandler = tapHandlerBuilder?.tapHandlerForMenuItem(menuItem)

        XCTAssertNil(tapHandler, 
            "Tap handler is not built for an unkown menu item")
    }
}
~~~

Let's make the test pass by creating a new class named `MenuItemTapHandlerBuilder`. Add it to both targets and replace its content with following.

~~~swift
import UIKit

class MenuItemTapHandlerBuilder {
    func tapHandlerForMenuItem(menuItem: MenuItem?) -> UIViewController? {
        var tapHandler: UIViewController?
        
        if menuItem != nil {
            switch menuItem!.tapHandlerName! {
            case "ContributionsViewController":
                tapHandler =
                    ContributionsViewController(
                        nibName: "ContributionsViewController",
                        bundle: nil)
                
            case "RepositoriesViewController":
                tapHandler =
                    RepositoriesViewController(
                        nibName: "RepositoriesViewController",
                        bundle: nil)
                
            case "PublicActivityViewController":
                tapHandler = PublicActivityViewController(
                    nibName: "PublicActivityViewController",
                    bundle: nil)
                
            default:
                tapHandler = nil
            }
        }
        
        return tapHandler
    }
}
~~~

Now that we have extracted the tap handler building code, we should inject `MenuItemTapHandlerBuilder` as a dependency to `MenuViewController`. In addition, let's leverage the depdency injection facility we have built to configure an instance of `MenuViewController` as well.

~~~swift
class MenuViewController: UIViewController {
    // ...
    var tapHandlerBuilder: MenuItemTapHandlerBuilder?

    //...
    func didSelectMenuItemNotification(notification: NSNotification?) {
        var menuItem: MenuItem? = notification!.object as? MenuItem
        
        if let tapHandler = 
            tapHandlerBuilder?.tapHandlerForMenuItem(menuItem) {
            self.navigationController.pushViewController(tapHandler, 
                                                         animated: true)
        }
    }
~~~

~~~swift
class ObjectConfiguratorTests: XCTestCase {
    // ...

    func testConfiguresAppMenuManagerCorrectly() {
        // ...
        XCTAssertNotNil(appMenuManager?.objectConfigurator,
            "App menu manager has an object configurator")
    }

    func testConfiguresMenuViewControllerCorrectly() {
        let menuViewController = objectConfigurator?.menuViewController()

        XCTAssertNotNil(menuViewController, 
            "Menu view controller is not nil")

        XCTAssertNotNil(menuViewController?.dataSource,
            "Menu view controller has a data source")

        XCTAssertNotNil(menuViewController?.tapHandlerBuilder,
            "Menu view controller has a tap handler builder")
    }
}
~~~

~~~swift
class ObjectConfigurator {
    func appMenuManager() -> AppMenuManager {
        // ...
        appMenuManager.objectConfigurator = self
        return appMenuManager
    }

    func menuViewController() -> MenuViewController {
        let menuViewController 
            = MenuViewController(nibName: "MenuViewController",
                                 bundle: nil)

        menuViewController.dataSource = MenuTableDefaultDataSource()
        menuViewController.tapHandlerBuilder = MenuItemTapHandlerBuilder()
        
        return menuViewController
    }
}
~~~

~~~swift
class AppMenuManager {
    // ...
    var objectConfigurator: ObjectConfigurator? = nil

    //...
    private func menuViewControllerFromMenuItems(menuItems: [MenuItem]) 
        -> MenuViewController 
    {
        let menuViewController = objectConfigurator?.menuViewController()
        let dataSource = menuViewController!.dataSource
        dataSource?.setMenuItems(menuItems)
        
        return menuViewController!
    }
~~~

~~~swift
class AppMenuManagerTests: XCTestCase {
    // ...
    override func setUp() {
        //...
        menuManager?.objectConfigurator = ObjectConfigurator()        
    }
}
~~~

Let's run the app (*Product > Run* or ⌘R). When each menu item is tapped, the correct view controller should be pushed into the app navigation stack. Our final app design (listed below) ended up not deviating too much from the initial design. However, it is quite possible for the final design to evolve into something completely different.

[![final_app_design.png](https://d23f6h5jpj26xu.cloudfront.net/3fmqoko8psjlrw.png)](http://img.svbtle.com/3fmqoko8psjlrw.png)

<a name="conclusion"></a>
Conclusion
===========

In this post we learned how to build a simple iOS app using TDD. Although Xcode 6 beta is a bit unstable as of this writing, XCTest itself seems to be quite stable. Despite the lack of mocking libraries such as [OCMock](http://ocmock.org/) and [Kiwi](https://github.com/kiwi-bdd/Kiwi), we were able to create fake objects easily and use them in our tests. Swift's ability to create classes inside a method came very handy while creating specialized fake objects quickly.

Although Swift is a completely new language, the techniques you might have learned for testing features in Objective-C (or any other language for that matter) in the past are still applicable to Swift. We only scratched the surface of Test-Driven Development in this post. I encourage you to read the reference material listed in [Further Reading](#further_reading) section below for an in-depth examination of TDD. Hopefully, you will give TDD a try with your next iOS app. The only way to get better at designing (and testing) is by doing more of it.

The finished project is available on [Github](https://github.com/pawanpoudel/AppMenu).

<a name="further_reading"></a>
Further Reading
===============

* [XCTest​Case / XCTest​Expectation / measure​Block()](http://nshipster.com/xctestcase/)
* [Test-Driven iOS Development](http://goo.gl/iiKpC1)
* [xUnit Test Patterns: Refactoring Test Code](http://goo.gl/HD4b3X)
* [Practical Object Oriented Design in Ruby](http://goo.gl/bbzSpz)