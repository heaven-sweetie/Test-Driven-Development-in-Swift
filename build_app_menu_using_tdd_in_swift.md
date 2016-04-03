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
### Building Menu Items
===================

우리는 우리가 지금 읽은 메타데이타에서 `MenuItem` 개체를 만들 준비가 되었다. 새로운 test 클래스의 이름을 `MenuItemBuilderTests`로 만들고 아래 내용을 작성하자:

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

Menu item은 반드시 title을 가져야 한다. 그러므로 우리는 `MenuItemBuilder`가 title이 아닌 에러를 리턴하는지 확인해야된다. 또한 menu item이 오류가 발생할때 비어있는 리스트를 리턴하는지 확인해야된다.

위의 테스트에선 진짜 menu items 메타데이타 reader (`MenuItemsPlistReader`) 대신에, 우리는 `FakeMenuItemsReader`라고 부르는 가짜를 사용하였다. 그 이유는 앱안의 모든 다른 구성요소로부터 test 클래스를 분리할 필요가 있기 때문이다. 이렇게함으로써 test가 실패 했을때 다른 클래스가 아닌 test 중인 클래스에서 문제가 발생했다는 것을 합리적으로 확신할 수 있습니다. 게다가, 만약 우리의 테스트에서 진짜 메타데이타 reader를 사용한다면 그리고 만약 미래에 클래스를 원격서버로 부터 plist를 다운로드 하기로 결정한다면, `MenuItemBuilder`를 test할때 다운로드 하는동안 쓸데없는 고통을 받을 것입니다. 우리는 항상 깨지지 않고 빠른 쉬운 유지보수를 목표해 왔다.

`FakeMenuItemsReader`는 다른 menu items에서 독자적으로 있을 경우 `MenuItemsReader`의 프로토콜을 준수 해야한다. 메타데이터를 파일이나 원격 서버로 부터 읽을때 이것은 항상 hard-coded 된 배열 dictionaries로 리턴한다. `FakeMenuItemsReader` 클래스를 만들고 더한하는건 오직 `AppMenuTests` 대상으로 한다. 우리는 이 클래스를 어떠한 어플리케이션 코드에서도 사용하지 않을 것이다. `FakeMenuItemsReader.swift` 파일의 내용을 대신 작성한다:

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

*가짜* 클래스들에 종종 생기는 한가지 관심사는 만약 원래 클래스들의 퍼블릭 API들의 바뀌는것이며 그것은 유효한 관심사다. 그렇지만 Swift가 컴파일 할때 에러가 발생할때 우리는 그것에 대해 걱정할 필요가 없다. 
예를들어 만약 우리가 `MenuItemsReader` 프로토콜안의 `readMenuItems` 로부터 menu items의 non-optional 배열을 반환하기로 결정한 경우 우리는 모두 `MenuItemsPlistReader` 및 `FakeMenuItemsReader` 클래스에 그 변경 사항을 적용하도록 강요한다. 어서 시도해보자. Swift는 좋은방법인가?

> 만약 너가 "alternate universe"를 더 배우길 원한다면 이러한 가짜 객체는 테스트에서 만들수 있습니다. 구체적인 예로 [Practical Object Oriented Design in Ruby](http://goo.gl/bbzSpz) 책의 챕터 9(*Creating Test Double* section)를 읽어 주십시요.

*가짜* 객체의 다른 관심사는 그들이 잘못된 보안감각을 제공할 수 있다는 점이다. 우리는 어떻게 'MenuItemsPlistReader'와 'MenuItemBuilder'가 함께 작동하는지 잘 알수있을까요? 그 대답은 유닛테스트를 통해 하진 않을 것이다. 앱의 다른 유닛이 서로 잘 작동하는지 확인하는 작업은 통합테스트에서 주어지며 이 블로그에선 포함되어 있지 않다.

*AppMenu* 그룹안에 'MenuItemBuilder'라는 이름의 새로운 Swift 클래스를 만들자. 모두 대상에 추가하고 다음과 같이 내용을 바꿉니다:

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

우리는 에러 tests통과할 충분한 코드를 썻다. 다음으로 우리는 빌더가 정확한 수의 menu items를 만드는지 확인해야한다. `MenuItemBuilderTests` 클래스에 다음 테스트를 추가하자.

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

Swift는 내가 관심없는 것은 `-`를 사용하여 리턴값을 무시하기 쉽고 이것은 내가 특별히 좋아하는 기능이다. `MenuItemBuilder` 클래스의 변경에 따라 모든 테스트를 통과 해야한다.

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

> 너는 내가 *red-green-refactor* 주기와 위의 변경을 엄격하게 수행하지 않은 것을 알수 있다. 나는 모든 테스트가 통과한 후 별도의 방법으로 error building code에서 추출해야합니다. 비록 나는 최소한의 코드를 써서 테스트를 통과하는 TDD의 첫번째 룰을 무시하는건 권장하지 않지만, 나는 이제 모든걸 정확하게 그리고 이 블로그 게시물이 너무 길지 않도록 할 것이다.
`MenuItemBuilder`의 마지막 테스트는 metadata dictionaries에 있는 값으로 menu item의 인스턴스의 속성 값을 채웠는지 확인하는 것입니다.

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

다시한번, 우리는 이 테스트 내부에서 여러가지 주장을 사용하고 있다. 위의 테스트는 어떠힌 코드를 변경하지 않고 통과 해야한다.

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

Return `1` from `numberOfSectionsInTableView` method to make previous test pass. Although this method is not required by `UITableViewDataSource` protocol and the default implementation already returns `1`, we need to implement it in order to be able to call it from the test.

~~~swift
func numberOfSectionsInTableView(tableView: UITableView!) -> Int {
    return 1
}
~~~

> One other test I would have liked to write is to verify that the data source throws an exception if asked for number of rows in a section whose index is anything other than 0. However, I couldn't find our old friend `XCTAssertThrows` in Xcode 6 version of XCTest. I don't know how else to verify that an exception was thrown.

Testing every aspect of a view could turn out to be tedius on iOS. I tend to test at least the semantics behind a view. By that I mean test what a view should represent. In this case, each table view cell represents a menu item. Therefore, I would at least like to verify that a cell displays the title from a respective menu item. Here is what that test looks like:

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

Following changes to `tableView:cellForRowAtIndexPath:` method should make the previous test pass.

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

Let's refactor the tests for `MenuTableDefaultDataSource` we have written so far by extracting the common code into `setup` method.

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
### Handling Menu Item Tap Event

As it turns out our table view setup is fairly simple. Therefore, it makes sense to use the same object as a data source and a delegate. When a table view cell is tapped, the data source will post a notification. `MenuViewController` (or any other class that is interested in that notification) can then query the notification to find out which cell was tapped and take appropriate action.

[![table_view_architecture.png](https://d23f6h5jpj26xu.cloudfront.net/9pmnjzuddxpv3g.png)](http://img.svbtle.com/9pmnjzuddxpv3g.png)

This design is somewhat inspired by chapter 9 from [Test-Driven iOS Development](http://goo.gl/iiKpC1) book. Let's add the delegate related details to `MenuTableDataSource` protocol.

~~~swift
import UIKit

let MenuTableDataSourceDidSelectItemNotification = 
    "MenuTableDataSourceDidSelectItemNotification"

protocol MenuTableDataSource : UITableViewDataSource, UITableViewDelegate {
    func setMenuItems(menuItems: [MenuItem])
}
~~~

Now we need to verify that the data source indeed posts a notification when a menu item is tapped. Following tests will do that.

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

In `setup` method, we added the test class to be an observer for a notification with name `MenuTableDataSourceDidSelectItemNotification`. When that notification arrives, `didReceiveNotification:` method should get called. The notification object passed to that method is stored in `postedNotification` variable. Then we verify that it has the correct name and menu item instance. It is important that the test class is removed as an observer in `tearDown` method. We go through this complex process to verify that a notification is indeed posted because [NSNotificationCenter](http://goo.gl/TfnJ3T) doesn't provide an API to query if a notification has been posted to it.

> In [Building Menu Items](#building_menu_items) section I recommended that we use fake objects in tests. However, I am using `NSNotificationCenter` class straight up in above tests. Generally, I don't use stand-ins for objects provided by Apple frameworks. They are fairly reliable in terms of stability and speed. That being said, if it turns out that the reliability of your tests are going down due to the use of real objects provided by Apple frameworks, don't hesitate to create test replacements for them.

Implement `tableView:didSelectRowAtIndexPath:` method from `UITableViewDataSource` protocol in `MenuTableDefaultDataSource` class to make above tests pass.

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
### Managing Menu Table View

`MenuViewController` will be responsible for managing the table view and any other views that might need to be createed in order to present the menu. The first thing we need to make sure is that we can give it a data source. We also need to ensure that it has a title and a table view. Create a new test file named `MenuViewControllerTests.swift` and replace its content with following tests.

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

Instead of using a real data source object here, we are using a fake one named `MenuTableFakeDataSource`. Create a new Swift file named `MenuTableFakeDataSource.swift` in `AppMenuTests` target and replace its content with following.

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

All `MenuTableFakeDataSource` does is provide stub implementation of all the required methods in `MenuTableDataSource` protocol so that it can stand in for any object that conforms to `MenuTableDataSource`. Now create `MenuViewController` class (*Right click AppMenu group > New File > iOS > Source > Cocoa Touch Class*). Make it a subclass of `UIViewController` and select the *Also create XIB file* checkbox. Don't forget to add it to both targets. The tests above should pass just by declaring two properties and assigning a title.

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

Change the size of main view in `MenuViewController.xib` to *iPhone 4-inch* from *Simulated Metrics* section in *Attributes Inspector*. Set the view's orientation to *Portrait*. After that add a table view as the main view's subview. Connect the table view in XIB to the `tableView` outlet in `MenuViewController` class.

Next we need to make sure that `MenuViewController` sets the table view's delegate and dataSource properties to the data source object we assigned to it. [viewDidLoad](http://goo.gl/OeT0hV) method is where we want it to make that connection. Following tests should ensure that.

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

Set table view's data source and delegate properties in `viewDidLoad` to make above tests pass.

~~~swift
override func viewDidLoad() {
    super.viewDidLoad()
    title = "App Menu"
    tableView.dataSource = dataSource
    tableView.delegate = dataSource
}
~~~

In [Handling Menu Item Tap Event](#handling_menu_item_tap_event) we made `MenuTableDefaultDataSource` post a notification when a menu item is tapped. `MenuViewController` needs to listen to that notification in order to show a correct view for that menu item. If that notification arrives when `MenuViewController`'s view is hidden, it should ignore it. Therefore, it should register for that notification in `viewDidAppear:` method. It should also stop listening for that notification in `viewDidDisappear:` method. Let's capture that requirement through tests.

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

That is a lot of code. Let me explain. In order to verify that `MenuViewController` registers itself to listen for `MenuTableDataSourceDidSelectItemNotification`, we need to somehow get hold of the method that gets called when that notification arrives. Once we get hold of it, we need to capture the notification passed to that method and verify its existence. We could simply make a non-private property for that notification in `MenuViewController`, but I don't like that approach. `MenuViewController` shouldn't be forced to expose something just because tests need it. There has to be a better way. How about we [swizzle](http://nshipster.com/method-swizzling/) the notification handler during runtime by providing a different implementation that is suited for our testing purpose? Following code will do just that.

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

And here is the [extension](http://goo.gl/lL1Cwy) for `MenuViewController` class that provides the test implementation:

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

All we are doing here is assign the notification to `postedNotification` constant so that we can evaluate it in tests. In `testRegistrationForMenuItemTappedNotificationHappensInViewDidAppear` after we swizzle the notification handler, we call `viewDidAppear`, post a notification and verify that `postedNotification` is not nil. Whereas in `testRemovesItselfAsListenerForMenuItemTappedNotificationInViewDidDisappear`, we first call `viewDidAppear` so that `MenuViewController` registers for the notification. After that we call `viewDidDisappear` and post a notification. This notification shouldn't reach to `MenuViewController` as we expect it to remove itself as an observer from `NSNotificationCenter` in `viewDidDisapper` method.

To make the tests pass, all we need to do is register and unregister for the notification in proper places.

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

When a menu item is tapped, we need to display a view. But which one? How about we ask the menu item itself? To keep things simple, let's store the view controller's name in `tapHandlerName` property in `MenuItem`.

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

It's perfectly fine for a menu item to not have a tap handler. Therefore, we should make the `tapHandlerName` property an optional. Now that we have added an additional property to `MenuItem`, we need to adjust `menuItems.plist`, `FakeMenuItemsReader`, `MenuItemsPlistReaderTests`, `MenuItemBuilderTests`, and `MenuItemBuilder`. The adjusted code is listed below.

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

Next up we need to make sure that `MenuViewController` displays the correct view when a menu item is tapped. Following tests will do just that.

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

Let's make `MenuViewController` push the right view controller into app navigation stack.

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

We also need to create tap handler classes. Create following view controllers and add them to both *AppMenu* and *AppMenuTests* targets. Don't forget to also create a *XIB* file for each one of them.

* `ContributionsViewController`
* `RepositoriesViewController`
* `PublicActivityViewController`

You might be wondering why we didn't create view controllers listed above during runtime instead of using the switch case statement. Two reasons:

* I am not quite sure what's the best way to do that in Swift. In Objective-C you could easily accomplish that with following code.

  ~~~Obj-C
    UIViewController *tapHandler = nil;
    Class tapHandlerClass =
         NSClassFromString(menuItem.tapHandlerName)

    if (tapHandlerClass) {
        tapHandler = [[tapHandlerClass alloc] init];
    }
  ~~~

* Unlike Objective-C, Swift requires you to specify which XIB file to use when you create an instance of a view controller even though the XIB name is the same as the view controller's class name. Therefore, simply calling the `alloc`, `init` equivalent in Swift isn't sufficient.

Unfortunately, that still doesn't make the tests pass. It turns out that so far we have built every class we initially set out to build except `AppMenuManager`. Once that class is built, we should be in a position to make above tests pass. Let's get to it.

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