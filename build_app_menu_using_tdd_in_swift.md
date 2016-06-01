이 블로그 게시물에서 우리는 스위프트로 [테스트 주도 개발][1]을 사용하여 간단한 iOS 앱 메뉴 (아래 그림 참조)를 작성하는 방법을 배울 것이다.
[![app\_menu.png][image-1]][2]
이 게시물에 제시된 개념을 완전히 이해하기 위해 알아야 할 몇가지 사항들이 있다:

* Xcode 6
* 스위프트 [기본 개념][3]에 대한 숙지
* UIKit 과 Foundation 에서 일반적으로 사용되는 클래스에 대한 숙지 (예를 들어, UITableView 과 NSNotificationCenter)
* XCTest에 대한 숙지. 이전에 XCTest를 사용해 본 적 없는 경우, [Matt Thompson의][4] [블로그 게시물][5]에서 *XCTestCase* 섹션을 참조.

Xcode 6에서 새로운 iOS 프로젝트를 만들자. *Single View Application* 템플릿을 선택한다. Product Name은 *AppMenu*, language는 *Swift* 를 선택하고, Devices는 *iPhone*을 선택한다. *Use Core Data* 옵션을 선택하지 않도록 한다. 이 실습에서 우리는 [Storyboards][6]를 사용하지 않는다. 그러므로 `Main.storyboard` 파일을 삭제한다. *AppMenu* 타깃의 *General* 탭 아래 *Deployment Info* 섹션에 위치한 *Main Interface* 드롭다운에 있는 스토리보드 이름 (Main)을 없애는 걸 잊지 말자. 하는 김에, `ViewController.swift`와 `AppMenuTests.swift` 파일도 삭제하자. 그것들은 Xcode에 의해 만들어지고 우리는 그것들이 필요가 없다.

우리가 테스트 주도 개발 (이하 TDD)의 아름다운 여행을 시작하기 전에, 한 걸음 뒤로 물러나 앱의 설계에 대해 조금 생각해보자. TDD는 테스트 활동보다 설계 연습에 더 가깝다고 들었을 지도 모른다. 그렇다면, 당신은 올바르게 들은 것이다. TDD는 테스트를 통해 존재하기도 전에 코드를 사용하도록 강제함으로써 우리가 테스트 중인 클래스가 응용 프로그램 코드의 나머지 부분과 상호 작용하는 방법에 대해 생각하도록 한다. 이러한 코드를 사용하는 행위는 우리가 재사용 가능한 클래스와 사용하기 쉬운 애플리케이션 프로그래밍 인터페이스(API)의 생성으로 이어지는 (좋은) 디자인 결정을 내릴 수 있게 한다. 그러나, 테스트를 먼저 쓰는 것으로 응용 프로그램을 만드는 것이 항상 잘 설계된다고 보장할 수는 없다. 우리는 여전히 테스트를 먼저 작성하는 것 외에도 좋은 디자인 [원칙][7]과 [패턴][8]을 적용해야 한다.

아래 그림은 우리가 목표로 하는 초기 디자인을 보여준다. 우리가 [Big Up Front Design][9] (BUFD) 영역을 침범한다고 걱정할 수도 있지만, 두려워 하지 마라. 아래의 설계는 우리에게 시작점을 준다. 우리는 아직 앱의 모든 면을 생각하지 않았다. 예를 들어, 우리는 각 클래스의 공개 API가 어떤 모습으로 보여질지 모른다. 우리는 앱을 만들어 가며, 아래의 디자인을 완전히 변경해야 한다고 생각할 수도, 그것이 완벽하게 괜찮았다고 생각할 수도 있다.

[![initial\_app\_design.png][image-2]][10]

<a name="identifying_domain_objects"></a>

## Identifying Domain Objects
==========================

새로운 프로젝트를 시작할 때, 나는 종종 작성할 *첫 번째 좋은* 테스트를 찾으려고 노력한다. 그 결과, 나는 보통 테스트하기 쉬운 도메인 객체를 찾는데 의존한다. 우리의 앱 메뉴는 각 메뉴 항목에 대해 예를 들어, 제목, 부제목 및 아이콘 정보를 표시한다. 우리는 메뉴 항목에 대한 정보를 저장하는 인스턴스가 필요하다. 이것을 `MenuItem`이라고 하자. 우리는 테스트를 통해 `MenuItem` 인스턴스가 포함 할 정보를 정의할 것이다.

`MenuItemTests.swift`라는 이름의 새로운 파일을 만들고 `AppMenuTests` 그룹 아래에 둔다. `AppMenuTests` 그룹에서 오른쪽 클릭을 하고 *New File \> iOS \> Source \> Test Case Class*를 선택해서 `MenuItemTests`라는 이름의 새로운 테스트 클래스를 만든다. `XCTestCase`의 서브클래스로 만들고 언어는 스위프트를 선택한다. 클래스 정의와 import 문을 제외하고 `MenuItemTests.swift` 파일의 모든 내용을 삭제한다.

\~\~\~swift
import UIKit
import XCTest

class MenuItemTests: XCTestCase {
}
\~\~\~

우리의 첫 번째 테스트는 메뉴 아이템이 제목을 갖고 있는지 확인하는 것이다. `MenuItemTests` 클래스에 다음의 테스트를 추가하자.

\~\~\~swift
func testThatMenuItemHasATitle() {
	let menuItem = MenuItem(title: "Contributions")
	XCTAssertEqual(menuItem.title, "Contributions", 
	    "A title should always be present")
}
\~\~\~

위의 테스트에서, 우리는 `MenuItem`의 인스턴스를 만들고; 제목을 넣고; Xcode에 있는 XCTest 프레임워크에서 제공하는 `XCTAssertEqual` 검증을 사용하여 제목을 가졌는지 확인한다. 우리는 또한 `MenuItem`이 `title`를 매개변수로 가지는 초기화 메서드를 제공해야 한다는 것을 (암묵적으로) 명시한다. 테스트를 먼저 작성할 때, 이렇게 우리의 API에 관한 미묘한 세부사항을 발견하는 경향이 있다.

> XCTest는 [다수의 검증들][11]을 제공한다. 각각의 검증은 테스트에 관한 것을 설명하는 테스트 명세를 명시할 수 있다. 나는 항상 이 명세를 제공하는 것을 권한다.

현재 상태로는, 테스트를 실행할 수 없다. 심지어 컴파일도 되지 않는다. 우리는 `MenuItem`를 만들어야 한다. `MenuItem`을 구조체로 만들지 클래스로 만들지 결정하기 전에, 우선 *The Swift Programming Language Guide*에서 [Choosing Between Classes and Structures][12] 섹션을 읽어보길 권장한다.

언뜻 보기에, 여기서는 `struct`가 충분해 보일 수 있다. 그러나, 아래의 [Handling Menu Item Tap Event][13] 섹션에서 우리는 `NSNotification` 객체에 메뉴 항목을 저장해야 한다. `NSNotification`는 저장하기 위해 `AnyObject` 프로토콜을 따르는 객체를 요구한다. `struct` 타입은 `AnyObject`을 따르지 않는다. 따라서, 우리는 `MenuItem`을 class로 만들어야 한다. `MenuItem.swift`라는 이름의 새로운 파일을 만든다 (*File \> New \> File \> iOS \> Source \> Swift File*). *AppMenu* 와 *AppMenuTests* 타깃 모두에 추가하고 다음과 같이 내용을 바꾼다.

\~\~\~swift
import Foundation

class MenuItem {
	let title: String
	
	init(title: String) {
	    self.title = title
	}
}
\~\~\~

> 애플리케이션 클래스를 테스트 타깃으로 추가할 필요가 없지만, 그렇게 하도록 만드는 버그가 Xcode 6에 있는 것 같다. 그렇지 않으면, *해결되지 않은 식별자의 사용* 에러를 얻을 것이다. 앞으로 테스트의 어떤 지점에서 에러가 발생하면, 애플리케이션 클래스 파일을 테스트 타깃에도 추가하여 해결할 수 있다. *Build Phases* 탭의 *Compile Sources* 섹션에서 *+* 버튼을 눌러서 타깃에 파일을 추가할 수 있다.

테스트를 실행한다 (*Product \> Test* 또는 ⌘U). 통과할 것이다. 다음으로 부제목 속성에 대한 테스트를 작성하자.

\~\~\~swift
func testThatMenuItemCanBeAssignedASubTitle() {
	let menuItem = MenuItem(title: "Contributions")
	menuItem.subTitle = "Repos contributed to"
	
	XCTAssertEqual(menuItem.subTitle!, "Repos contributed to",
	    "Subtitle should be what we assigned")
}
\~\~\~

이것은 `MenuItem` 클래스에 `subTitle` 속성을 추가한 뒤에 통과할 것이다.

\~\~\~swift
class MenuItem {
	let title: String
	var subTitle: String?
	
	init(title: String) {
	    self.title = title
	}
}
\~\~\~

메뉴 항목은 제목을 반드시 가지고 있어야 하므로, 상수 속성으로 정의되어 있다. 반면 `subTitle`는 필수가 아니다. 그러므로, 변수 속성으로 정의한다. 마지막으로, `iconName` 속성에 대한 테스트는 다음과 같다:

\~\~\~swift
func testThatMenuItemCanBeAssignedAnIconName() {
	let menuItem = MenuItem(title: "Contributions")
	menuItem.iconName = "iconContributions"
	
	XCTAssertEqual(menuItem.iconName!, "iconContributions",
	    "Icon name should be what we assigned")
}
\~\~\~

이것은 `MenuItem`에 `iconName` 속성을 추가한 뒤에 통과할 것이다.
testThatMenuItemCanBeAssignedAnIconName
\~\~\~swift
class MenuItem {
	let title: String
	var subTitle: String?
	var iconName: String?
	
	init(title: String) {
	    self.title = title
	}
}
\~\~\~

넘어가기 전에, `MenuItem` 생성 코드를 `setup` 메서드로 이동하여 테스트를 리팩토링 하자.

> 모든 테스트가 통과했을 때 [리팩토링][14] 하는 것은 TDD에서 일반적인 방법이다. 리팩토링 과정은 더 나은 코드와 테스트를 구성하여 작은 단위로 설계를 개선하는 데 도움이 된다. 또한, 리팩토링은 중복을 없앨 수 있도록 도와준다. 실패하는 테스트를 먼저 작성하고 그것을 통과하기에 충분한 코드를 만들고 다음 실패하는 테스트를 작성하기 전에 설계를 개선하는 이 반복되는 과정은 *red-green-refactor* 순환으로 알려져 있다.

[![red\_green\_refactor.png][image-3]][15]

\~\~\~swift
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
\~\~\~

XCTest는 각 테스트를 실행하기 전에 `setup` 메서드를 호출한다. 테스트 실행이 완료되면, `setup` 메서드에서 할당된 변수는 `nil`로 설정된다. 객체의 새로운 인스턴스를 만들고 난 후에 다시 `setup` 메서드에서 해당 변수에 그것을 할당한다. XCTest는 각 테스트를 분리하기 위해 이렇게 한다. 우리는 이전 테스트에 의해 남은 데이터가 다음 테스트에 영향을 주는 것을 원하지 않는다. 테스트 실행이 완료될 때 XCTest가 자동적으로 변수에 `nil`로 설정하기 때문에, (마찬가지로 XCTest에서 제공하는) `tearDown` 메서드에서 명시적으로 그것들에 `nil`로 설정할 필요가 없다. 그러나, 변수에 `nil`을 설정하는 것 이외의 다른 정리 작업을 수행해야 할 경우, `tearDown` 메서드에서 해야 한다.

다음은 plist로 부터 `MenuItem` 인스턴스를 만들기 위해 필요한 메타데이터를 읽을 것이다. 우리의 초기 설계에서 제안한 것처럼, 우리는 각 메뉴 항목에 필요한 메타데이터를 [plist]()()파일에 저장할 것이다. 그렇게 하면, 원격 서버로부터 메타데이터를 가져와 동적으로 메뉴를 만들 필요가 있을 경우에도, 메타데이터 포맷을 동일하게 유지하는 한, 너무 많은 변경을 할 필요가 없다. `MenuItemsPlistReader` 클래스를 만들기 전에, 우리는 `MenuItemsReader` 프로토콜이 어떻게 생겼는지 알아야 한다. 초기 단계는 다음과 같다:

\~\~\~swift
import Foundation

protocol MenuItemsReader {
  func readMenuItems() -\> ([[String : String]()]?, NSError?)
}
\~\~\~

`readMenuItems` 메서드는 어떤 매개변수도 가지지 않고 [튜플]()[17]()을 반환한다. 파일을 성공적으로 읽은 경우 튜플의 첫 번째 항목은 [딕셔너리]()[18]() 배열을 포함하고 있다. 파일을 읽을 수 없는 경우 두 번째 항목은 [NSError]()[19]() 객체가 포함되어 있다. `readMenuItems`는 필수 메서드다. 그래서 `MenuItemsReader` 프로토콜을 따르고자 하는 모든 클래스는 그것을 구현해야 한다. `MenuItemsReader.swift`라는 이름의 새로운 파일을 만들자. 그리고, 두가지 타깃에 추가한 다음, 위와 같이 프로토콜 정의 코드로 내용을 채운다.

다음에는 plist 파일에서 메타데이터를 읽어오자. 우리는 먼저 테스트를 작성할 것이다. `AppMenuTests` 타깃에 `MenuItemsPlistReaderTests.swift`라는 이름의 새로운 파일을 만든다. 이제 Swift 테스트 파일을 어떻게 만드는지 알기 때문에, 앞으로 이러한 설명은 생략할 것이다. 클래스 정의와 import 문을 제외하고 `MenuItemsPlistReaderTests.swift` 파일의 모든 내용을 삭제한다. 우리의 첫 번째 테스트는, 지정된 plist 파일을 읽을 수 없는 경우, `MenuItemsPlistReader`가 에러를 반환하는지 확인하는 것이다. `MenuItemsPlistReaderTests` 클래스에 다음 테스트를 추가한다.

\~\~\~swift
func testErrorIsReturnedWhenPlistFileDoesNotExist() {
let plistReader = MenuItemsPlistReader()
plistReader.plistToReadFrom = "notFound"

let (metadata, error) = plistReader.readMenuItems()
XCTAssertNotNil(error, "Error is returned when plist doesn't exist")
}
\~\~\~


우리는 `MenuItemsPlistReader`의 인스턴스를 만들고; 읽어올 plist 파일이름을 존재하지 않는 것으로 지정하고; `readMenuItems` 메서드를 호출한다. 그런 다음 에러를 반환하는지 확인한다. 테스트를 통과하려면, 우리는 `MenuItemsPlistReader` 클래스를 만들고 두 타깃에 추가해야 한다. 다음과 같이 내용을 바꾼다.

\~\~\~swift
import Foundation

class MenuItemsPlistReader: MenuItemsReader {
var plistToReadFrom: String? = nil

func readMenuItems() -\> ([[String : String]()]?, NSError?) {
let error = NSError(domain: "Some domain", 
code: 0, 
userInfo: nil)
return ([](), error)
}
}
\~\~\~

이제 테스트를 실행하자. 통과할 것이다. 테스트를 통과했지만, 뭔가 좋아 보이지 않는다. `readMenuItems`은 심지어 파일을 읽을 시도도 하지 않는다. 항상 빈 배열과 그다지 유용하지 않은 에러를 포함하는 튜플을 반환한다. 이것은 TDD의 중요한 측면을 우리에게 제시한다: *테스트를 통과하기 위해 최소한의 코드를 작성한다*. 테스트를 통과하기 위해 필요한 것보다 더 많은 코드를 작성하지 않도록 훈련하는 것이 TDD의 핵심이다. 그러므로, 우리는 테스트에 필요하지 않는 한 겉보기에는 불완전한 `readMenuItems` 메서드를 수정하지 않을 것이다.

아직까지 우리가 `MenuItemsPlistReader`클래스에서 필요한 최소한 명세는, *파일이 존재하지 않는다면, 에러를 반환해야한다*는 것이다. 우리는 에러 객체에 무엇이 있어야 하는지 지정하지 않았다. 우리가 기대하는 도메인, 코드 그리고 설명이 에러에 포함되어 있는지 확인하는 몇 가지 테스트를 더 추가하자.

> 애플은 런타임 에러에 대한 정보를 담기 위해 NSError 객체를 사용하는 것을 [권고한다]()[20](). 이러한 객체는 에러 *도메인*, 도메인 특정 에러 *코드*, 그리고 에러 *설명*을 포함하는 *user info* 딕셔너리가 들어있어야 한다. *user info* 딕셔너리 안에는 에러에 관한 다른 세부사항, 예를 들어 에러를 해결하기 위해 취할 수 있는 단계 같은 것을 추가할 수 있다.

\~\~\~swift
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
userInfo![NSLocalizedDescriptionKey]()! as String

XCTAssertEqual(description,
"notFound.plist file doesn't exist in app bundle",
"Correct error description is returned")
}
\~\~\~

 위의 테스트를 통과하기 위해, `MenuItemsPlistReader`를 다음과 같이 변경한다.

\~\~\~swift
import Foundation

let MenuItemsPlistReaderErrorDomain = "MenuItemsPlistReaderErrorDomain"

enum MenuItemsPlistReaderErrorCode : Int {
case FileNotFound
}

class MenuItemsPlistReader: MenuItemsReader {
var plistToReadFrom: String? = nil

func readMenuItems() -\> ([[String : String]()]?, NSError?) {
let errorMessage = 
"\(plistToReadFrom!).plist file doesn't exist in app bundle"

let userInfo = [NSLocalizedDescriptionKey: errorMessage]()

let error = NSError(domain: MenuItemsPlistReaderErrorDomain,
code: MenuItemsPlistReaderErrorCode.FileNotFound.toRaw(),
userInfo: userInfo)

return ([](), error)
}
}
\~\~\~

`readMenuItems` 메서드는 여전히 좋아 보이지 않는다. 다음으로 우리를 속이지 못하도록 강제할 테스트들을 작성할 것이다.  넘어가기 전에, `testErrorIsReturnedWhenPlistFileDoesNotExist`라는 이름의 테스트를 삭제한다. 이것은 이전의 세 테스트와 중복된다.

\~\~\~swift
func testPlistIsDeserializedCorrectly() {
let plistReader = MenuItemsPlistReader()
plistReader.plistToReadFrom = "menuItems"

let (metadata, error) = plistReader.readMenuItems()
XCTAssertTrue(metadata?.count == 3, 
"There should only be three dictionaries in plist")

let firstRow = metadata?[0]()
XCTAssertEqual(firstRow!["title"]()!, "Contributions",
"First row's title should be what's in plist")
XCTAssertEqual(firstRow!["subTitle"]()!, "Repos contributed to",
"First row's subtitle should be what's in plist")
XCTAssertEqual(firstRow!["iconName"]()!, "iconContributions",
"First row's icon name should be what's in plist")

let secondRow = metadata?[1]()
XCTAssertEqual(secondRow!["title"]()!, "Repositories",
"Second row's title should be what's in plist")
XCTAssertEqual(secondRow!["subTitle"]()!, "Repos collaborating",
"Second row's subtitle should be what's in plist")
XCTAssertEqual(secondRow!["iconName"]()!, "iconRepositories",
"Second row's icon name should be what's in plist")

let thirdRow = metadata?[2]()
XCTAssertEqual(thirdRow!["title"]()!, "Public Activity",
"Third row's title should be what's in plist")
XCTAssertEqual(thirdRow!["subTitle"]()!, "Activity viewable by anyone",
"Third row's subtitle should be what's in plist")
XCTAssertEqual(thirdRow!["iconName"]()!, "iconPublicActivity",
"Third row's icon name should be what's in plist")
}
\~\~\~

여기서 우리는 `readMenuItems` 메서드가 실제로 명시된 plist 파일로부터 데이터를 읽고 그 데이터로부터 올바른 객체를 생성하는 것을 확인할 수 있다.

> 유닛 테스트를 작성하는 동안 경험적인 규칙은 테스트 메서드에서 하나 이상의 검증을 포함하지 않는 것이다. 파일에서 읽은 데이터를 한 곳에서 올바른지 확인하는 것은 타당하기 때문에, 여기서 규칙을 위반하고 있다.

위의 테스트를 통과하기 위해, "menuItems.plist”이라는 이름의 파일을 만들자.*(AppMenu 그룹을 오른쪽으로 클릭 \> New File \> iOS \> Resource \> Property List)*. 두 타깃에 추가한다. 소스 코드 모드에서 파일을 열고 *(Xcode에서 파일을 오른쪽으로 클릭 \> Open As \> Source Code)* 다음과 같이 내용을 바꾼다:

\~\~\~xml
\<?xml version="1.0" encoding="UTF-8"?\>
\<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"\>
\<plist version="1.0"\>
\<array\>
\<dict\>
\<key\>title\</key\>
\<string\>Contributions\</string\>
\<key\>subTitle\</key\>
\<string\>Repos contributed to\</string\>
\<key\>iconName\</key\>
\<string\>iconContributions\</string\>
\<key\>featureName\</key\>
\<string\>contributions\</string\>
\</dict\>
\<dict\>
\<key\>title\</key\>
\<string\>Repositories\</string\>
\<key\>subTitle\</key\>
\<string\>Repos collaborating\</string\>
\<key\>iconName\</key\>
\<string\>iconRepositories\</string\>
\<key\>featureName\</key\>
\<string\>repositories\</string\>
\</dict\>
\<dict\>
\<key\>title\</key\>
\<string\>Public Activity\</string\>
\<key\>subTitle\</key\>
\<string\>Activity viewable by anyone\</string\>
\<key\>iconName\</key\>
\<string\>iconPublicActivity\</string\>
\<key\>featureName\</key\>
\<string\>publicActivity\</string\>
\</dict\>
\</array\>
\</plist\>
\~\~\~

에셋 카탈로그 (Images.xcassets)에 다음의 이미지를 추가한다. 이 이미지들은 [완성된 프로젝트]()[21]()에 포함되어 있다.

* iconContributions@2x.png
* iconRepositories@2x.png
* iconPublicActivity@2x.png

이제 다음과 같이 `readMenuItems` 메서드를 수정한다:

\~\~\~swift
func readMenuItems() -\> ([[String : String]()]?, NSError?) {
var error: NSError? = nil
var fileContents: [[String : String]()]? = nil  
let bundle = NSBundle(forClass: object\_getClass(self))

if let filePath = 
bundle.pathForResource(plistToReadFrom, ofType: "plist")
{
fileContents = 
NSArray(contentsOfFile: filePath) as? [[String : String]()]
}
else {
let errorMessage = 
"\(plistToReadFrom!).plist file doesn't exist in app bundle"

let userInfo = [NSLocalizedDescriptionKey: errorMessage]()

error = NSError(domain: MenuItemsPlistReaderErrorDomain,
code: MenuItemsPlistReaderErrorCode.FileNotFound.toRaw(),
userInfo: userInfo)
}

return (fileContents, error)
}
\~\~\~

이제 테스트가 통과했으니, 별도의 메서드로 에러 생성 코드를 추출하여 리팩토링 하자.`readMenuItems` 메서드를 리팩토링한 후의 모습이 여기 있다:

\~\~\~swift
func readMenuItems() -\> ([[String : String]()]?, NSError?) {
var error: NSError? = nil
var fileContents: [[String : String]()]? = nil
let bundle = NSBundle(forClass: object\_getClass(self))

if let filePath = 
bundle.pathForResource(plistToReadFrom, ofType: "plist") 
{
fileContents = 
NSArray(contentsOfFile: filePath) as? [[String : String]()]
}
else {
error = fileNotFoundError()
}

return (fileContents, error)
}

func fileNotFoundError() -\> NSError {
let errorMessage = 
"\(plistToReadFrom!).plist file doesn't exist in app bundle"

let userInfo = [NSLocalizedDescriptionKey: errorMessage]()

return NSError(domain: MenuItemsPlistReaderErrorDomain,
code: MenuItemsPlistReaderErrorCode.FileNotFound.toRaw(),
userInfo: userInfo)
}
\~\~\~

우리가 아무 것도 깨트리지 않았다는 것을 확인하기 위해 다시 테스트를 실행해보자 (*⌘U*). 우리의 테스트들에서도 리팩토링 할 수 있는 곳도 보인다. 모든 테스트에서 공통된 코드를 `setup` 메서드로 옮기자.

\~\~\~swift
class MenuItemsPlistReaderTests: XCTestCase {
var plistReader: MenuItemsPlistReader?
var metadata: [[String : String]()]?
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
userInfo![NSLocalizedDescriptionKey]()! as String

XCTAssertEqual(description, 
"notFound.plist file doesn't exist in app bundle",
"Correct error description is returned")
}

func testPlistIsDeserializedCorrectly() {
plistReader!.plistToReadFrom = "menuItems"
(metadata, error) = plistReader!.readMenuItems()

XCTAssertTrue(metadata?.count == 3, 
"There should only be three dictionaries in plist")

let firstRow = metadata?[0]()
XCTAssertEqual(firstRow!["title"]()!, "Contributions",
"First row's title should be what's in plist")
XCTAssertEqual(firstRow!["subTitle"]()!, "Repos contributed to",
"First row's subtitle should be what's in plist")
XCTAssertEqual(firstRow!["iconName"]()!, "iconContributions",
"First row's icon name should be what's in plist")

let secondRow = metadata?[1]()
XCTAssertEqual(secondRow!["title"]()!, "Repositories",
"Second row's title should be what's in plist")
XCTAssertEqual(secondRow!["subTitle"]()!, "Repos collaborating",
"Second row's subtitle should be what's in plist")
XCTAssertEqual(secondRow!["iconName"]()!, "iconRepositories",
"Second row's icon name should be what's in plist")

let thirdRow = metadata?[2]()
XCTAssertEqual(thirdRow!["title"]()!, "Public Activity",
"Third row's title should be what's in plist")
XCTAssertEqual(thirdRow!["subTitle"]()!, 
"Activity viewable by anyone",
"Third row's subtitle should be what's in plist")
XCTAssertEqual(thirdRow!["iconName"]()!, "iconPublicActivity",
"Third row's icon name should be what's in plist")
}
}
\~\~\~

나는 우리가 plist가 존재하는 시나리오에 대한 테스트를 추가해야 한다는 것을 깜빡 했다는 것을 깨닳았지만, 그러나 `readMenuItems`는 잘못된 데이터 때문에 아마 그것을 읽을 수 없을 것이다. 나의 소중한 독자들을 위한 연습으로 남겨두겠다.

### Building Menu Items
===================

우리는 우리가 지금 읽은 메타데이타에서 `MenuItem` 개체를 만들 준비가 되었다. 새로운 test 클래스의 이름을 `MenuItemBuilderTests`로 만들고 아래 내용을 작성하자:

\~\~\~swift
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
\~\~\~

Menu item은 반드시 타이틀을 가져야 한다. 그러므로 우리는 `MenuItemBuilder`가 타이틀이 없을 때, 에러를 리턴하는지 확인해야 한다. 또한, 에러가 발생했을 때, menu item이 비어있는 리스트를 리턴하는지도 확인해야 한다.

위의 테스트에선 진짜 menu items 메타데이터 리더 (`MenuItemsPlistReader`) 대신에, 우리는 `FakeMenuItemsReader`라고 부르는 가짜를 사용하였다. 그 이유는 앱안의 모든 다른 구성요소로부터 test 클래스를 분리할 필요가 있기 때문이다. 이렇게 함으로써 test가 실패 했을때 다른 클래스가 아닌 test 중인 클래스에서 문제가 발생했다는 것을 합리적으로 확신할 수 있다. 게다가, 만약 우리의 테스트에서 진짜 메타데이타 reader를 사용한다면 그리고 만약 미래에 클래스를 원격서버로 부터 plist를 다운로드 하기로 결정한다면, `MenuItemBuilder`를 test할때 다운로드 하는동안 쓸데없는 고통을 받을 것이다. 우리는 항상 견고하고, 빠르고 쉬운 유지보수를 목표해 왔다.

`FakeMenuItemsReader`이 다른 menuItemsReader의 대역을 하려면 반드시`MenuItemsReader`의 프로토콜을 준수해야 한다. 메타데이터를 파일이나 원격 서버로 부터 직접 읽지 않고, 이것은 항상 하드코딩 된 딕셔너리의 배열을 리턴한다. `FakeMenuItemsReader` 클래스를 만들고, 오직 `AppMenuTests` 를 타겟으로만 추가하자. 우리는 이 클래스를 어떠한 어플리케이션 코드에서도 사용하지 않을 것이다. `FakeMenuItemsReader.swift` 파일의 내용을 다음과 같이 변경한다.:

\~\~\~swift
import Foundation

class FakeMenuItemsReader : MenuItemsReader {
var missingTitle: Bool = false

func readMenuItems() -\> ([[String : String]()]?, NSError?) {
let menuItem1 = 
missingTitle ? menuItem1WithMissingTitle() 
 : menuItem1WithNoMissingTitle()

let menuItem2 = ["title": "Menu Item 2",
]() "subTitle": "Menu Item 2 subtitle",
 "iconName": "iconName2"]

return ([menuItem1, menuItem2](), nil)
}

func menuItem1WithMissingTitle() -\> [String : String]() {
return ["subTitle": "Menu Item 1 subtitle",
]()"iconName": "iconName1"]
}

func menuItem1WithNoMissingTitle() -\> [String : String]() {
var menuItem = menuItem1WithMissingTitle()
menuItem["title"]() = "Menu Item 2"
return menuItem
}
}
\~\~\~

*가짜* 클래스들에 종종 생기는 한가지 관심사는 만약 원래 클래스들의 퍼블릭 API들의 바뀌는것이며 그것은 유효한 관심사다. 그렇지만 만약 프로토콜을 만족해야하는 클래스에서 필수 메서드들을 실제로 구현하지 않고 있다면, Swift가 컴파일 시점에 에러를 던져주기 때문에(throw error), *우리는 그것에 대해 걱정할 필요가 없다.*
예를 들어 만약 우리가 `MenuItemsReader` 프로토콜안의 `readMenuItems` 로부터 menu items의 옵셔널이 아닌 배열을 반환하기로 결정한 경우, 우리는 모두 `MenuItemsPlistReader` 및 `FakeMenuItemsReader` 클래스에 그 변경 사항을 적용하도록 강제된다. 바로, 시도해보자. 이게 Swift의 위대한 점 아닌가?

> 만약  "alternate universe"를 더 배우길 원한다면 이러한 가짜 객체는 테스트에서 만들수 있습니다. 구체적인 예로 [Practical Object Oriented Design in Ruby]()[22]() 책의 챕터 9(*Creating Test Double* section)를 읽어 보라.

*가짜* 객체의 우려되는 점은, 그것이 잘못된 보안감각을 줄 수 있다는 점이다. 우리는 어떻게 'MenuItemsPlistReader'와 'MenuItemBuilder'가 함께 작동하는지 잘 알 수 있을까? 그 대답은 유닛테스트를 통해 하진 않겠다. 앱의 다른 유닛이 서로 잘 작동하는지 확인하는 작업은 통합테스트에서 하게 되며, 이 포스트에서는 다루지 않는다.

*AppMenu* 그룹안에 'MenuItemBuilder'라는 이름의 새로운 Swift 클래스를 만들자. 두 타겟에 추가하고 다음과 같이 내용을 변경하자:

\~\~\~swift
import Foundation

let MenuItemBuilderErrorDomain = "MenuItemBuilderErrorDomain"

enum MenuItemBuilderErrorCode : Int {
case MissingTitle
}

class MenuItemBuilder {
func buildMenuItemsFromMetadata(metadata: [[String : String]()]) 
 -\> ([MenuItem]()?, NSError?) 
{
let userInfo = 
[NSLocalizedDescriptionKey: "All menu items must have a title"]()

let error = NSError(domain: MenuItemBuilderErrorDomain,
code: MenuItemBuilderErrorCode.MissingTitle.toRaw(),
userInfo: userInfo)

return ([](), error)
}
}
\~\~\~

우리는 테스트를 통과할 충분한 코드를 썼다. 다음으로 우리는 빌더가 정확한 수의 menu items를 만드는지 확인해야 한다. `MenuItemBuilderTests` 클래스에 다음 테스트를 추가하자.

\~\~\~swift
func testOneMenuItemInstanceIsReturnedForEachDictionary() {
fakeMenuItemsReader!.missingTitle = false
let (metadata, \_) = fakeMenuItemsReader!.readMenuItems()

(menuItems, \_) =
menuItemBuilder!.buildMenuItemsFromMetadata(metadata!)

XCTAssertTrue(menuItems?.count == 2,
"Number of menu items should be equal to number of dictionaries")
}
\~\~\~

Swift는 내가 관심없는 것은 `-`를 사용하여 리턴값을 무시하기 쉽고 이것은 내가 특별히 좋아하는 기능이다. `MenuItemBuilder` 클래스의 변경에 따라 모든 테스트를 통과 해야한다.

\~\~\~swift
class MenuItemBuilder {
func buildMenuItemsFromMetadata(metadata: [[String : String]()]) 
 -\> ([MenuItem]()?, NSError?) 
{
var menuItems = [MenuItem]()()
var error: NSError?

for dictionary in metadata {
if let title = dictionary["title"]() {
let menuItem = MenuItem(title: title)
menuItem.subTitle = dictionary["subTitle"]()
menuItem.iconName = dictionary["iconName"]()  
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

private func missingTitleError() -\> NSError {
let userInfo = 
[NSLocalizedDescriptionKey: "All menu items must have a title"]()

return NSError(domain: MenuItemBuilderErrorDomain,
code: MenuItemBuilderErrorCode.MissingTitle.toRaw(),
userInfo: userInfo)
}
}
\~\~\~

> 너는 내가 *red-green-refactor* 주기와 위의 변경을 엄격하게 수행하지 않은 것을 알수 있다. 나는 모든 테스트가 통과한 후 별도의 방법으로 error building code에서 추출해야합니다. 비록 나는 최소한의 코드를 써서 테스트를 통과하는 TDD의 첫번째 룰을 무시하는건 권장하지 않지만, 나는 이제 모든걸 정확하게 그리고 이 블로그 게시물이 너무 길지 않도록 할 것이다.
`MenuItemBuilder`의 마지막 테스트는 metadata dictionaries에 있는 값으로 menu item의 인스턴스의 속성 값을 채웠는지 확인하는 것입니다.

\~\~\~swift
func testMenuItemPropertiesContainValuesPresentInDictionary() {
fakeMenuItemsReader!.missingTitle = false
let (metadata, \_) = fakeMenuItemsReader!.readMenuItems()

(menuItems, \_) = 
menuItemBuilder!.buildMenuItemsFromMetadata(metadata!)

let rawDictionary1 = metadata![0]()
let menuItem1 = menuItems![0]()

XCTAssertEqual(menuItem1.title, 
rawDictionary1["title"]()!,
"1st menu item's title should be what's in the 1st dictionary")

XCTAssertEqual(menuItem1.subTitle!, 
rawDictionary1["subTitle"]()!,
"1st menu item's subTitle should be what's in the 1st dictionary")

XCTAssertEqual(menuItem1.iconName!, 
rawDictionary1["iconName"]()!,
"1st menu item's icon name should be what's in the 1st dictionary")

let rawDictionary2 = metadata![1]()
let menuItem2 = menuItems![1]()

XCTAssertEqual(menuItem2.title, 
rawDictionary2["title"]()!,
"2nd menu item's title should be what's in the 2nd dictionary")

XCTAssertEqual(menuItem2.subTitle!, 
rawDictionary2["subTitle"]()!,
"2nd menu item's subTitle should be what's in the 2nd dictionary")

XCTAssertEqual(menuItem2.iconName!, 
rawDictionary2["iconName"]()!,
"2nd menu item's icon name should be what's in the 2nd dictionary")
}
\~\~\~

다시한번, 우리는 이 테스트 내부에서 여러가지 주장을 사용하고 있다. 위의 테스트는 어떠힌 코드를 변경하지 않고 통과 해야한다.

## 메뉴 아이템 표시
=====================

이제 우리는 `MenuItem` 인스턴스를 만드는 방법과 plist로부터 정보를 채우는 방법을 알고있다. 이제 컨텐츠를 보여주는 것으로 초점을 옮겨보자. 우리는  메뉴 아이템들을 보여주기 위해 테이블뷰를 이용할 것이다. 우리가 초기 설계에서 제안한 것처럼, 
`MenuTableDefaultDataSource` 는 각 메뉴 아이템을 `UITableViewCell`에 완벽히 맞추어 정보를 제공하는 역할을 할 것이다. 테이블뷰 자체는 `MenuViewController`가 관리한다. 

### 테이블뷰에 데이터 제공

우리는 테이블뷰의 데이터소스를 `MenuViewController`에 직접 구현을 하기 보다  분리된 객체를 사용할 것이다. `MenuViewController` 는 이미 뷰들을 관리하는 역할을 하고 있습니다. 나는 테이블뷰의 데이터를 제공하는 역할까지 해서 [단일 책임 원칙][72]을 위반하는 것을 싫어한다. 그러나 첫번 째로 우리는 `MenuTableDefaultDataSource`을 따르는 프로토콜을 만들 것이다. *AppMenu* 그룹에 `MenuTableDataSource.swift`라는 스위프트 파일 파일을 새로 만들고, 두 타겟에 추가한 뒤 아래의 코드로 변경한다.

\~\~\~swift
import UIKit

protocol MenuTableDataSource : UITableViewDataSource {
	func setMenuItems(menuItems: [MenuItem])
}
\~\~\~

`MenuTableDataSource`는 `UITableViewDataSource` 을 상속한 프로토콜이다. 또한 `setMenuItems` 메소드를 필수로 요구한다. 이제 우리는 `MenuTableDefaultDataSource`의 테스트를 작성할 준비가 됐다. `AppMenuTests` 타겟 안에 `MenuTableDefaultDataSourceTests.swift` 라는 새로운 테스트 파일을 만들고, 다음의 코드를 추가한다.

\~\~\~swift
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
\~\~\~

여기서 우리는 데이터 소스가 각각의 메뉴 아이템에 대해 하나의 테이블뷰 셀 인스턴스를 만든다는 것을 확인한다. 이제 `MenuTableDefaultDataSource.swift` 라는 새로운 스위프트 파일을 만들고 아래의 코드를 입력한다.

\~\~\~swift
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
\~\~\~

아직 우리는 테스트를 위한 `tableView:cellForRowAtIndexPath:` 메소드를 아직 작성하지 않았지만, 우리는 테스트에 앞서, 요청에 대해 실행 가능한 구현이 필요하다. 이 작업은 `UITableViewDataSource` 프로토콜에서 요구하는 메소드가 있어야 하며, 스위프트는 `MenuTableDefaultDataSource` 없이 컴파일을 할 수 없다.

 하나 더, 혹시 `MenuTableDefaultDataSource` 에 대하여 알아 차렸을 수도 있겠지만 이것은 `NSObject` 를 상속받고 있다. 그 이유는 `UITableViewDataSource` 프로토콜을 따르기 위해, 또한 `NSObject` 프로토콜을 준수할 필요가 있기 때문이다. 이것을 가장 쉽게 만족시킬 방법으로는 이미 `NSObject` 프로토콜을 준수 하는 `NSObject` 를 상속받게 만드는 것이다.

앞선 테스트에서, 우리는 `tableView:numberOfRowsInSection:` 메서드에서 무조건 `1` 을 반환하도록 만들었다. 데이터 소스가  메뉴 아이템이 얼마나 있든 항상 정확한 수를 반환하는지 증명하는 테스트를 추가하자.

\~\~\~swift
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
\~\~\~

`menuItems` 는 테스트를 통과하기 위해 하드코딩된 `1` 대신 실제 카운트 값을 반환한다.

\~\~\~swift
func tableView(tableView: UITableView!,
	           numberOfRowsInSection section: Int)
	           -> Int
{
	return menuItems!.count
}
\~\~\~

또한 우리는 데이터소스가 정확한 섹션의 개수를 반환 하는지 확인이 필요하다. 여기 그것에 대한 테스트이다:

\~\~\~swift
func testReturnsOnlyOneSection() {
	let dataSource = MenuTableDefaultDataSource()
	let numberOfSections = dataSource.numberOfSectionsInTableView(nil)
	XCTAssertEqual(numberOfSections, 1, 
	    "There should only be one section")
}
\~\~\~

`numberOfSectionsInTableView` 메서드는 앞선 테스트를 통과 시키기 위해 `1` 을 리턴한다. 또한 이 메서드는 `UITableViewDataSource` 프로토콜에서 필수로 요구되는 메서드도 아니고, 이미 기본적으로 `1` 을 리턴함에도 불구하고, 우리는 테스트를 위해 구현이 필요하다.

\~\~\~swift
func numberOfSectionsInTableView(tableView: UITableView!) -\> Int {
	return 1
}
\~\~\~

> 내가 작성하고 싶은 또 다른 테스트는 데이터 소스가 섹션의 행의 수가 0이 아닌 다른 수를 요청했을 때, 예외를 던지는지 확인하는 테스트다. 하지만 우리의 오래된 친구 `XCTAssertThrows` 를 Xcode 6의 XCTest에서는 찾을 수 없다. 그래서 예외상황이 발생된 상황을 다르게 증명하는 방법을 모르겠다.

iOS에서 뷰의 모든점을 테스트 하는건 상당히 지루할 수 있다. 나는 최소의 뷰만 테스트하는 경향이 있다. 뷰가 표시해야 하는 것이 무엇인지 테스트하는 것이다. 따라서, 나는 각각의 메뉴 아이템으로 부터 최소한, 타이틀을 표시하는지 확인할 것이다. 아래의 코드를 참고하자.

\~\~\~swift
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
\~\~\~

`tableView:cellForRowAtIndexPath:` 메서드는 이전에 테스트가 통과되도록 변경해야 한다.

\~\~\~swift
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
\~\~\~

이제 `MenuTableDefaultDataSource` 에 대한 테스트를 리팩토링 해보자. 우리가 여지껏 작성한 코드에서 공통적인 코드를 `setup` 매서드로 추출한다.

\~\~\~swift
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
\~\~\~

### Handling Menu Item Tap Event (메뉴 아이템 탭 이벤트 핸들링)

테이블뷰 설정은 보는것과 같이 굉장히 간단하다. 그러므로,  데이터소스나 델리게이트로서, 동일한 객체를 사용해도 이해할 수 있다. 테이블 뷰의 셀을 탭할 때, 데이터소스는 알림을 보낼 것이다. `MenuViewController` (또는 알림에 관심이 있는 다른 어떠한 클래스)는 어떤 셀이 탭이 되었는지 그 알림에 물어보고, 적절한 액션을 취할 수 있다.

[![table\_view\_architecture.png][image-28]][73]

이 설계는 [Test-Driven iOS Development][74] 책의 챕터 9로 부터 약간의 영감을 받았다. 그럼 `MenuTableDataSource` 프로토콜에 델리게이트와 관련된 세부항목을 추가하자.

\~\~\~swift
import UIKit

let MenuTableDataSourceDidSelectItemNotification = 
	"MenuTableDataSourceDidSelectItemNotification"

protocol MenuTableDataSource : UITableViewDataSource, UITableViewDelegate {
	func setMenuItems(menuItems: [MenuItem])
}
\~\~\~

이제 우리는 메뉴 아이템이 탭 되었을 때, 데이터소스가 진짜로 알림을 보내는지 확인이 필요하다. 다음 테스트가 그 것을 증명할 것이다.

\~\~\~swift
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
\~\~\~
 
`setup` 메서드에서, 우리는 `MenuTableDataSourceDidSelectItemNotification` 라는 이름으로 알림에 대해 테스트 클래스를 옵저버로서 추가했다. 알림이 통보될 때, `didReceiveNotification:` 메서드가 호출될 것이다. 그 매서드에 전달된 알림 객체는 `postedNotification` 변수에 저장된다. 그러면, 우리는 이것이 정확한 이름과 메뉴 아이템 인스턴스를 갖고 있는지 확인한다. `tearDown`매서드에서 테스트클래스가 옵저버를 제거한다는 사실이 중요하다.  우리는 이 복잡한 프로세스를 통해서 알림이 정말로 보내지는지 증명해야 한다. 왜냐하면, [NSNotificationCenter][75] 에서 노티피케이션이 보내졌을 때 확인 할 수 있는 API를 제공하지 않기 때문이다. 

> [Building Menu Items][76] 섹션에서 나는 테스트에서 가짜 오브젝트를 사용하는 것을 추천했다. 그러나, 나는 위에 테스트에서 `NSNotificationCenter` 클래스를 그대로 사용하고 있다. 일반적으로 나는 애플 프레임워크에서 제공되는 객체에 대해서는 대체제를 사용하지 않는다. 그것들은 꽤 믿을만큼 안정적이며 빠르다.  그런 까닭에, 애플 프레임워크가 제공한 실제 객체의 사용으로 인해 당신의 테스트의 신뢰도가 떨어진다면, 그들을 대체할 테스트를 만드는 것을 주저하지 마라.

앞선 테스트들을 통과 시키기 위해 `MenuTableDefaultDataSource` 클래스 안의 `UITableViewDataSource` 프로토콜로부터 `tableView:didSelectRowAtIndexPath:` 메서드를 구현하자.

\~\~\~swift
func tableView(tableView: UITableView!,
	didSelectRowAtIndexPath indexPath: NSIndexPath!)
{
	let menuItem = menuItems?[indexPath.row]
	
	let notification = 
	    NSNotification(name: MenuTableDataSourceDidSelectItemNotification,
	                   object:menuItem)
	
	NSNotificationCenter.defaultCenter().postNotification(notification)
}
\~\~\~

### Managing Menu Table View (메뉴 테이블 뷰 관리)

`MenuViewController` 는 테이블뷰와 메뉴를 표시하는데 필요한 모든 뷰들을 관리하는 역할을 할 것이다. 먼저, `MenuViewController`에 데이터소스를 줄 수 있는지 첫번째로 확인해야한다. 또한, 타이틀과 테이블뷰를 갖고 있는지 확인이 필요하다. 새로운 테스트 파일 `MenuViewControllerTests.swift` 을 만들고 아래의 내용을 추가하자.

\~\~\~swift
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
\~\~\~

여기에 실제 데이터 소스를 사용하는 것 대신에, 우리는 `MenuTableFakeDataSource` 라는 이름의 가짜 데이터를 사용할 것이다. `AppMenuTests` 타겟jc `MenuTableFakeDataSource.swift` 라는 이름의 스위프트 파일을 생성하고 내용을 다음의 코드로 대체한다.

\~\~\~swift
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
\~\~\~

모든 `MenuTableFakeDataSource` 은 `MenuTableDataSource` 프로토콜 안의 필요 메서들 구현을 요구한다. 그리고 `MenuTableDataSource` 에 일치하는 모든 객체들을 대신한다. 지금 `MenuViewController` 클래스를 생성한다. (*AppMenu 그룹을 우클릭 \> New File \> iOS \> Source \> Cocoa Touch Class*). `UIViewController` 를 상속받도록 하고, *Also create XIB file* 체크박스도 선택한다. 그리고 두 타겟 모두  추가하는 것을 잊지말자. 두개의 속성 선언과 타이틀을 할당하는 것만으로 앞의 테스트들은  통과해야 한다. 

\~\~\~swift
import UIKit

class MenuViewController: UIViewController {
	@IBOutlet weak var tableView: UITableView!
	var dataSource: MenuTableDataSource?
	
	override func viewDidLoad() {
	    super.viewDidLoad()
	    title = "App Menu"
	}
}
\~\~\~

`MenuViewCOntroller.xib` 의 메인뷰 사이즈를 *Attricbutes Inspector* 섹션 안에서 *Simulated Metrics* 을 *iPhont 4-inch* 로 변경하자. 뷰의 오리엔테이션은 *Portrait* 로 셋팅한다. 그 후에, 테이블뷰를 메인 뷰의 서브뷰로 추가한다. XIB의 테이블뷰를 `MenuViewController` 클래스의 `tableView`아울렛과 연결한다.

다음으로 우리는 `MenuViewController` 의 델리게이트와 데이터소스 속성이 우리가 정한 데이터 소스 객체로 지정하는지 확인해야 할 필요가 있다. [viewDidLoad][77] 메서드는 그 연결을 하기 적절한 곳이다. 다음 테스트를 코드를 확인하자.

\~\~\~swift
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
\~\~\~

위의 테스트를 통과시키기 위해 `viewDidLoad` 안에서 테이블뷰의 데이터소스와 델리게이트를 셋팅한다.

\~\~\~swift
override func viewDidLoad() {
	super.viewDidLoad()
	title = "App Menu"
	tableView.dataSource = dataSource
	tableView.delegate = dataSource
}
\~\~\~

[Handling Menu Item Tap Event][78] 에서 우리는 메뉴 아이템을 탭했을 때  `MenuTableDefaultDataSource` 가 알림을 보내도록 만들었다. `MenuViewController` 는 알림을 받았을 때, 메뉴 아이템에 대한 정확한 뷰를 보여주기 위해 알림을 듣고(listen) 있어야 한다. 만약 그 알림이 `MenuViewController` 의 뷰가 감춰져 있을 때 도착한다면, 무시되어야 한다. 그러므로, `viewDidAppear:` 메서드에서 알림을 등록해야 한다. 또한, `viewDidDisaapear:` 메서드 안에서 알림을 듣기 해체(stop listening)를 해주어야 한다. 다음의 테스트를 통해서 요구하는 것을 확인하자.

\~\~\~swift
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
\~\~\~

굉장히 많은 코드를 작성했다. 설명을 하자면, `MenuViewController` 가`MenuTableDataSourceDidSelectItemNotification` 을 듣기(listen) 위해 자기 자신을 등록했는지 확인하기 위해, 우리는 노티피케이션이 도착했을 때 호출되는 매서드를 어떻게든 알아야 할 필요가 있다. 일단 우리가 그 매서드를 알아내기만 하면, 그 곳으로 넘겨지는 노티피케이션을 잡아서 그 존재를 확인해야 한다. 간단하게 `MenuViewController` 안에 노티피케이션을 비-내부적(non-private) 속성으로 만들 수 있지만, 개인적으로 이 접근 방법을 좋아하진 않는다. `MenuViewController` 는 단지 테스트에서 필요하다고 해서 강제로 뭔가를 노출하지 말아야 한다. 여기에 더 좋은 방법이 있다. 우리가 테스트 목적에 맞춰서 다른 구현을 런타임동안에 알림 핸들러와 바꿔치기(swizzle)하는 것은 어떤가? 다음 코드가 그것을 보여준다.

\~\~\~swift
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
\~\~\~

그리고 다음은 테스트 구현에 제공하는 `MenuViewController` 클래스에 대한  [확장(extension)][79]이다.

\~\~\~swift
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
\~\~\~

우리가 여기서 할 것은 `postedNotification` 상수를 알림에 할당해서 테스트에서 확인할 수 있도록 하는게 전부다. `testRegistrationForMenuItemTappedNotificationHappensInViewDidAppear` 내부이기때문에 우리는 먼저 `viewDidAppear`을 호출하고 `MenuViewController`에 그 알림을 등록한다. 우리가 `viewDidAppear`을 호출한 뒤에, 알림을 보내자(post). 이 알림은  우리가 기대하는 것 처럼 `viewDidDisapper`에서 `NSNotificationCenter`로부터 옵저버로서 제거되기 때문에  `MenuViewController`에 도달하지 못해야 한다.

이 테스트를 통과하기 위해서, 우리가 필요한 것은 그 알림을 적절한 곳에서 등록하고 해지하는것이다.

\~\~\~
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
\~\~\~


## Sliding Views In
================

메뉴 아이템을 탭하면 우린 뷰를 보여줘야 한다. 하지만 어떤 뷰를 보여줘야 할까? 메뉴아이템에 직접 물어보는 건 어떨까? 단순함을 유지하기 위해 뷰컨트롤러의 이름을 `MenuItem`에 `tapHandlerName` property 로 저장하자.

\~\~\~swift
class MenuItemTests: XCTestCase {
	// ...
	
	func testThatMenuItemCanBeAssignedATapHandlerName() {
	    menuItem!.tapHandlerName = "someViewController"
	    XCTAssertEqual(menuItem!.tapHandlerName!, 
	        "someViewController",
	        "Tap handler name should be what we assigned")
	}
}
\~\~\~

\~\~\~swift
class MenuItem {
	// ...
	
	var tapHandlerName: String?
}
\~\~\~

메뉴 아이템이 tap handler를 갖지 않게 할 좋은 방법이다. 그러므로, `tapHandlerName` property를 optional로 만들어야 한다.  이제 `MenuItem`에 property를 추가했고 `menuItems.plist`, `FakeMenuItemsReader`, `MenuItemsPlistReaderTests`, `MenuItemBuilderTests`, 그리고 `MenuItemBuilder`에 적용해야 한다. 그 적용된 코드는 아래에 나열되어 있다.

\~\~\~xml
<!--menuItems.plist-->

\<?xml version="1.0" encoding="UTF-8"?\>
\<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"\>
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
\~\~\~

\~\~\~swift
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
\~\~\~

\~\~\~swift
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
\~\~\~

\~\~\~swift
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
\~\~\~

\~\~\~swift
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
\~\~\~

다음으로 menu item을 탭 했을 때, `MenuViewController`가 맞는 view를 보여주는지  확인해야 한다. 다음 test가 그렇게 할 것이다.

\~\~\~swift
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
\~\~\~

`MenuViewController`가 올바른 뷰 컨트롤러를 앱 내비게이션 스택에 푸시(push) 하도록 만들자
\~\~\~swift
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
\~\~\~

또한 tap handler class들을 만들어야 한다. 다음의 뷰컨트롤러들을 만들고 각각을 *AppMenu* and *AppMenuTests* 타겟에 추가한다. 각각에 대한 *XIB* 파일을 만드는 것 또한 잊지 말자.

* `ContributionsViewController`
* `RepositoriesViewController`
* `PublicActivityViewController`

왜 런타임에 뷰컨트롤러를 만들지 않고, switch case 문을 사용했는지 궁금할 것이다: 두 가지 이유가 있다.

* 나는 Swift에서 가장 좋은 방법이 무엇인지 확신할 수 없다. Objective-C에서는 다음의 코드로 쉽게 구현할 수 있었다.

  \~\~\~Obj-C
	UIViewController *tapHandler = nil;
	Class tapHandlerClass =
	     NSClassFromString(menuItem.tapHandlerName)
	
	if (tapHandlerClass) {
	    tapHandler = [[tapHandlerClass alloc] init];
	}
  \~\~\~

* Objective-C와는 다르게, Swift는 뷰컨트롤러의 인스턴스를 만들 때, XIB의 이름과 뷰컨트롤러의 이름이 같더라도 XIB를 지정해야 한다. 그래서, 간단히 `alloc`, `init`을 호출하는 것은 Swift에서 필요하지 않다.

불행히도, 여전이 test들을 통과하진 못한다. 그것은 `AppMenuManager` 를 제외하고 초기에 우리가 만들기 시작한 모든 클래스가 지금까지 만들어졌는지 확인한다. 그 클래스가 만들어지면 앞의 테스트들을 통과해야 한다. 해보자.

### App Menu 관리하기

*AppMenuTests* target에 파일명이 `AppMenuManagerTests.swift`인 새로운 테스트 파일을 만들자. 다음의 test를 추가한다.

\~\~\~swift
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
\~\~\~

메타데이터로부터 성공적으로 `MenuItem` 객체를 만든 경우, `AppMenuManager`는 `MenuViewController`를 만들 책임이 있다. 그렇지 못할 경우, nil을 반환한다. `AppMenuManager` 가 그 자체로 동작하는 것 보다는 여러 객체 사이에 상호작용을  주로 하기 때문에, 우리는 `AppMenuManager`가 메타데이터를(성공적으로 읽었다면) 빌더로 전달하는지 확인할 필요가 있다. 우리가 테스트에서 앱메뉴매니져에 무엇이 반환되는지 제어 할 수 있도록 가짜 메뉴아이템 리더와 빌더 객체들을 사용하고 있다는 것을 알아차렸을 것이다. [*Building Menu Items*][80]에서 우리는 가짜 메뉴 아이템 리더를 만들었지만, 이것은 에러를 셋팅하는 방법을 제공하지 않는다. 다음을 살펴보자.

\~\~\~swift
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
\~\~\~

다음으로 `FakeMenuItemBuilder` 클래스를 만들어야 한다. 하나 이상의 클래스가 메뉴 아이템 빌더의 역할을 할 것이기 때문에, 우리는 어떤 클래스가 메뉴 아이템 빌더가 될 수 있는지 분명히 알기 위해 프로토콜을 만들어야 한다. 지금으로선, 그 메뉴 아이템 빌더의 역할을 한다는 것은 `buildMenuItemsFromMetadata`메서드를 제대로 구현하고 있다는 것을 의미한다. 아래는 그 새로운 프로토콜이다.

\~\~\~swift
import Foundation

protocol MenuItemBuilder {
	func buildMenuItemsFromMetadata(metadata: [[String : String]]) -> ([MenuItem]?, NSError?)
}
\~\~\~

잠깐만. 우리가 이미 실제 빌더 클래스를 `MenuItemBuilder` 라고 이름 붙이지 않았던가? 그렇다. `MenuItemBuilder`라는 이름은 프로토콜에 더 적합하다. 원래의 빌더 클래스의 이름을 `MenuItemDefaultBuilder`로 바꾸자.

\~\~\~swift
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
\~\~\~

또한, 그 새로운 이름을 사용하도록 테스트들에게도 적용해야 한다.

\~\~\~swift
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
\~\~\~
마침내, `FakeMenuItemReader`클래스를 완성했다. 오직 테스트에서만 사용하기 때문에 이 클래스를 *AppMenu* target에 추가할 필요는 없다.

\~\~\~swift
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
\~\~\~
그것은 메타데이터를 검사할수 있도록 전달한다. 그것은 또한 우리가 에러와 반환하길 원하는 메뉴 아이템들을 아주 편리하게 설정 하도록 해 준다. 이제 `AppMenuManager` 클래스를 만들 준비가 되었다. 그 코드는 다음과 같다.

\~\~\~swift
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
\~\~\~

여기에서 *read-green-refactor* 주기를 지키지 않은 것을 사과한다. 진행할때마다 모든 과정을 보여주는 것보다는 테스트를 보다 쉽게 작성하는 중요한 테크닉에 집중하길 원했다. 그러한 테크닉 중 하나는 실제 객체와 동일한 역할을 하는 가짜(또는 테스트를 위한) 객체를 만든 것이고, 그것은 테스트를 유지하면서 쉽게 그것들을 바꿀 수 있다 . [Martin Fowler][81]는 가짜 객체에 대해서 [좋은 게시물][82]을 썼다.

넘어가기 전에, 테스트 가능하고, 재사용할 수 있는 클래스들을 작성함에 있어, [Dependency Injection][83]의 중요성에 대해 강조하고 싶다. 우리의 `AppMenuManager` 클래스는 성공적으로 `MenuItem` 객체를 만들기 위해 `MenuItemsReader`과 `MenuItemBuilder` 프로토콜을 따르는 두 개의 다른 class들과 함께 동작해야 한다. 우리가 퍼블릭 속성을 통해 이 두개의 의존성을 노출시키지 않았다면, 가짜 객체들에게 넘겨줄 수 없었을 것이다. 그러한 가짜 객체들은 `AppMenuManager`가 기대한대로 행동하는지 증명하기 위해 원하는 테스트 시나리오를 작성함에 있어서 매우 유용하다. 따라서, 당신의 클래스가 가진 모든 단일 의존성이 애플 프레임워크가 제공하는 클래스들에 대한게 아닌 이상, 노출시키는 것을 추천한다.

## Putting It All Together
=======================
우리가 모든 클래스를 만들었으니, 그것들을 `AppDelegate`에 함께 두어 보도록 하자.
거의 다 왔다. 그러나 먼저 우리는 `AppDelegate`가 기대한대로 동작하는지 확인하는 몇가지 테스트를 작성할 것이다. *AppMenuTests* 타겟에 `AppDelegateTests.swift`라는 이름의 새로운 테스트 파일을 만들자. 다음의 테스트를 추가한다.

\~\~\~swift
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
\~\~\~


> [App Menu 관리하기][84]에서, 진짜 메뉴 빌더에 대한 대역을 할 가짜 객체가 필요하다는 것을 알아차렸을 때 `MenuItemBuilder` 프로토콜을 만들었다. 하지만, 여기에서는 test 자체 내부에서 가짜 app menu manager 객체들을 만든다. 이렇게 하는 것은 완벽하다. 진짜 app menu manager 클래스에 있는 `menuViewController` 매서드의 이름을 변경하기로 결정한 경우, Swift는 모든 우리의 가짜 객체들도 새로운 method 이름을 사용하도록 강제할 것이다. 그 이유로 이러한 모든 가짜 객체들은 진짜 app menu manager와 동기화 될 것이다. 테스트 내부에서 빠르게 가짜 object를 만들어야 할 때, 이 접근은 매우 유용하다.

새로운 Xcode 프로젝트를 만들었을 때, `AppDelegate`는 *AppMenu* 타겟에만 추가된다. `AppMenuTests` 타겟에도 추가해야 한다. 그 후에 다음과 같이 내용을 변경한다:

\~\~\~swift
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
\~\~\~

`AppDelegate`에서 `AppMenuManager`를 설정하는 코드를 추출하는 것이 좋을 것이다. [Graham Lee]()가 [Test-Driven iOS Development]()에서 가르쳐준 것을 적용하고, 완전한 [depdendency injection framework][87]를 사용하는 대신, 우리 스스로의  [depdendency injection]()클래스를 만들 것이다. 최소한 지금으로서는, AppMenu는 심플한 앱이다. 그래서 의존성이 필요하지 않다면, 그것을 추가할 필요가 없다. *AppMenuTests* 타겟에 `ObjectConfiguratorTests.swift`라는 이름의 새로운 테스트 파일을 만들고 다음의 내용으로 바꾸자.

\~\~\~swift
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
\~\~\~

`ObjectConfigurator` 클래스를 만들고 두 타겟에 추가하자. 다음의 내용으로 바꾸자.

\~\~\~swift
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
\~\~\~
`AppMenuManager` 객체 자체를 만드는 대신, 앱델리게이트가 ObjectConfigurator에게 그렇게 하도록 시킬 것이다. 새로운 접근법이 포함되도록 `AppDelegate`와 그의 테스트를 수정하자.

\~\~\~swift
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
\~\~\~

\~\~\~swift
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
\~\~\~

`MenuViewController`로 관심을 돌려보자. 지금 모든 테스트가 통과되고 있지만, 약간 리팩토링을 해야 한다. Let's extract the code that decides which view controller should be the tap handler into a separate class. *AppMenuTests* 타겟에  `MenuItemTapHandlerBuilderTests`라는 이름의 새로운 테스트 클래스를 만들고 다음의 내용으로 바꾸자.

\~\~\~swift
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
\~\~\~

이름이 `MenuItemTapHandlerBuilder`인 새로운 class를 만들어 test를 통과시키자. 각 target에 추가하고 다음의 내용으로 바꾸자.

\~\~\~swift
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
\~\~\~

Now that we have extracted the tap handler building code, we should inject `MenuItemTapHandlerBuilder` as a dependency to `MenuViewController`. In addition, let's leverage the depdency injection facility we have built to configure an instance of `MenuViewController` as well.

\~\~\~swift
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
\~\~\~

\~\~\~swift
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
\~\~\~

\~\~\~swift
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
\~\~\~

\~\~\~swift
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
\~\~\~

\~\~\~swift
class AppMenuManagerTests: XCTestCase {
	// ...
	override func setUp() {
	    //...
	    menuManager?.objectConfigurator = ObjectConfigurator()        
	}
}
\~\~\~

app을 실행시켜보자(*Product \> Run* or ⌘R). 각각의 menu item을 선택했을 때, app navigation stack에 적절한 view controller가 push 될 것이다. 최종 app design(아래에 나열된)은 초기의 design에서 크게 벗어나지 않았다. 그러나, 최종 design은 완전히 다른 것으로 진화하는 것이 충분히 가능하다.

[![final\_app\_design.png][image-29]][89]

<a name="conclusion"></a>
## 결말
===========

이 post에서 TDD를 활용한 간단한 iOS app을 만드는 법에 대해서 배웠다. Xcode 6 beta는 이 글을 쓰는동안 약간 불안정했지만, XCTest 자체는 상당히 안정적으로 보였다. [OCMock][90]과 [Kiwi][91] 같은 mocking library들의 부족에도 불구하고, fake object를 쉽게 만들고 그것을 test에 사용하는 것이 가능했다. Swift의 method 내부에서 class를 만드는 능력은 전문적인 가짜 object를 빠르게 만드는데 편리했다.

Swift는 완전히 새로운 언어임에도 불구하고, 이미 배웠던 Objective-C(또는 그 문제를 위한 어떤 다른 언어)에서의 test 기능을 위해 배웠을 기술들도 여전히 Swift에서 적용할 수 있다. 이 post에서 Test-Driven Development를 겉핥기만 했다.나는 TDD의 깊이있는 이해를 위해 [더 읽을거리][92] section의 참고자료를 읽을 것을 권장한다. 당신의 다음 iOS app에서 TDD를 시도하기를 바란다. design(그리고 test)에서 더 좋게 하는 유일한 방법은 그것을 더욱 많이 하는 것이다.

완성된 project는 [Github][93]에서 있다.

<a name="further_reading"></a>
더 읽을거리
===============

* [XCTest​Case / XCTest​Expectation / measure​Block()][94]
* [Test-Driven iOS Development][95]
* [xUnit Test Patterns: Refactoring Test Code][96]
* [Practical Object Oriented Design in Ruby][97]

[1]:	http://martinfowler.com/bliki/TestDrivenDevelopment.html
[2]:	http://img.svbtle.com/aacqorrxf8wpiw.png
[3]:	http://goo.gl/PNWoSw
[4]:	https://twitter.com/mattt
[5]:	http://nshipster.com/xctestcase/
[6]:	http://goo.gl/YOK2kR
[7]:	http://goo.gl/bbzSpz
[8]:	http://goo.gl/U23sC8
[9]:	http://en.wikipedia.org/wiki/Big_Design_Up_Front
[10]:	http://img.svbtle.com/ttuxgmqb3ia.png
[11]:	http://goo.gl/PU24UU
[12]:	http://goo.gl/ptBqYR
[13]:	#handling_menu_item_tap_event
[14]:	http://refactoring.com/
[15]:	http://img.svbtle.com/fqgfy5r3w7nkq.png
[72]:	http://www.objectmentor.com/resources/articles/srp.pdf
[73]:	http://img.svbtle.com/9pmnjzuddxpv3g.png
[74]:	http://goo.gl/iiKpC1
[75]:	http://goo.gl/TfnJ3T
[76]:	#building_menu_items
[77]:	http://goo.gl/OeT0hV
[78]:	#handling_menu_item_tap_event
[79]:	http://goo.gl/lL1Cwy
[80]:	#building_menu_items
[81]:	http://martinfowler.com/
[82]:	http://martinfowler.com/articles/mocksArentStubs.html
[83]:	http://www.martinfowler.com/articles/injection.html
[84]:	#managing_app_menu
[87]:	http://www.typhoonframework.org/
[89]:	http://img.svbtle.com/3fmqoko8psjlrw.png
[90]:	http://ocmock.org/
[91]:	https://github.com/kiwi-bdd/Kiwi
[92]:	#further_reading
[93]:	https://github.com/pawanpoudel/AppMenu
[94]:	http://nshipster.com/xctestcase/
[95]:	http://goo.gl/iiKpC1
[96]:	http://goo.gl/HD4b3X
[97]:	http://goo.gl/bbzSpz

[image-1]:	https://d23f6h5jpj26xu.cloudfront.net/aacqorrxf8wpiw.png
[image-2]:	https://d23f6h5jpj26xu.cloudfront.net/ttuxgmqb3ia.png
[image-3]:	https://d23f6h5jpj26xu.cloudfront.net/fqgfy5r3w7nkq.png
[image-28]:	https://d23f6h5jpj26xu.cloudfront.net/9pmnjzuddxpv3g.png
[image-29]:	https://d23f6h5jpj26xu.cloudfront.net/3fmqoko8psjlrw.png