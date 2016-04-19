이 블로그 게시물에서 우리는 스위프트로 [테스트 주도 개발][1]을 사용하여 간단한 iOS 앱 메뉴 (아래 그림 참조)를 작성하는 방법을 학습한다.
[![app\_menu.png][image-1]][2]
이 게시물에 제시된 개념을 완전히 이해하기 위해 알아야 할 사항은 다음과 같다:

* Xcode 6
* 스위프트 [기본 개념][3]에 익숙
* UIKit 과 Foundation 에서 일반적으로 사용되는 클래스에 익숙 (예를 들어, UITableView 과 NSNotificationCenter)
* XCTest에 익숙. 이전에 XCTest를 사용하지 않은 경우, [Matt Thompson’의][4] [블로그 게시물][5]에서 *XCTestCase* 섹션을 참조.

Xcode 6에서 새로운 iOS 프로젝트를 만든다. *Single View Application* 템플릿을 선택한다. Product Name은 *AppMenu*, language는 *Swift*, 그리고 Devices는 *iPhone*으로 한다. *Use Core Data* 옵션을 선택하지 않도록 한다. 이 실습에서 우리는 [Storyboards][6]를 사용하지 않는다. 그러므로 `Main.storyboard` 파일을 삭제한다. *AppMenu* 타깃의 *General* 탭 아래 *Deployment Info* 섹션에 위치한 *Main Interface* 드롭다운에 있는 스토리보드 이름 (Main)을 없애는 걸 잊지 않는다. 하는 김에, `ViewController.swift`와 `AppMenuTests.swift` 파일도 삭제하자. 그것들은 Xcode에 의해 만들어지고 우리는 그것들이 필요가 없다.

우리가 테스트 주도 개발 (TDD)의 아름다운 여행을 시작하기 전에, 한 걸음 뒤로 물러나 앱의 설계에 대해 조금 생각해보자. TDD는 테스트 활동보다 디자인 연습이라고 더 많이 들었을 수도 있다. 그렇다면, 당신은 올바르게 들었다. TDD는 테스트를 통해 존재하기도 전에 코드를 사용하도록 강요함으로써 우리가 테스트 중인 클래스가 응용 프로그램 코드의 나머지 부분과 상호 작용하는 방법에 대해 생각하도록 한다. 이러한 코드를 사용하는 행위는 우리가 재사용 가능한 클래스와 사용하기 쉬운 응용 프로그램 프로그래밍 인터페이스(API)의 생성으로 이어지는 (좋은) 디자인 결정을 내릴 수 있게 한다. 그러나, 테스트를 먼저 쓰는 것으로 응용 프로그램을 만드는 것이 항상 잘 설계된다고 보장할 수는 없다. 우리는 여전히 테스트를 먼저 작성하는 것 외에도 좋은 디자인 [원칙][7]과 [패턴][8]을 적용해야 한다.

아래 그림은 우리가 목표로 하는 초기 디자인을 보여준다. 우리가 [Big Up Front Design][9] (BUFD) 영역에 들어간다고 걱정할 수도 있지만, 두려워 마라. 아래의 설계는 우리에게 시작점을 준다. 우리는 아직 앱의 모든 면을 생각하지 않았다. 예를 들어, 우리는 각 클래스의 공개 API가 어떤 모습으로 보여질지 모른다. 우리는 응용 프로그램을 만드는 절차를 통해, 아래의 디자인을 완전히 변경해야 한다는 것과 그것이 완벽하게 괜찮은 것을 깨닫게 될 수 있다.

[![initial\_app\_design.png][image-2]][10]

<a name="identifying_domain_objects"></a>
Identifying Domain Objects
==========================

새로운 프로젝트를 시작할 때, 나는 종종 작성할 *첫 번째 좋은* 테스트를 찾기 위해 노력한다. 그 결과, 나는 보통 테스트하기 쉬운 도메인 객체를 찾는데 의존한다. 우리의 앱 메뉴는 각 메뉴 항목에 대해 예를 들어, 제목, 부제목 및 아이콘 정보를 표시한다. 우리는 메뉴 항목에 대한 정보를 저장하는 인스턴스가 필요하다. 이것을 `MenuItem`이라고 하자. 우리는 테스트를 통해 `MenuItem` 인스턴스가 포함하는 정보를 정의한다.

`MenuItemTests.swift`라는 이름의 새로운 파일을 만들고 `AppMenuTests` 그룹 아래에 둔다. `AppMenuTests` 그룹에서 오른쪽 클릭을 하고 *New File \> iOS \> Source \> Test Case Class*를 선택해서 `MenuItemTests`라는 이름의 새로운 테스트 클래스를 만든다. `XCTestCase`의 서브클래스로 만들고 언어는 스위프트를 선택한다. 클래스 정의와 import 문을 제외하고 `MenuItemTests.swift` 파일의 모든 내용을 삭제한다.

\~\~\~swift
import UIKit
import XCTest

class MenuItemTests: XCTestCase {
}
\~\~\~

우리의 첫 번째 테스트는 메뉴 항목의 제목이 있는지 확인하는 것이다. `MenuItemTests` 클래스에 다음의 테스트를 추가한다.

\~\~\~swift
func testThatMenuItemHasATitle() {
	let menuItem = MenuItem(title: "Contributions")
	XCTAssertEqual(menuItem.title, "Contributions", 
	    "A title should always be present")
}
\~\~\~

위의 테스트에서, 우리는 `MenuItem`의 인스턴스를 만들고; 제목을 넣고; Xcode에 있는 XCTest 프레임워크에서 제공하는 `XCTAssertEqual` 검증을 사용하여 제목을 가졌는지 확인한다. 우리는 또한 `MenuItem`이 `title`를 매개변수로 가지는 초기화 메서드를 제공해야 한다는 것을 (암묵적으로) 명시한다. 테스트를 먼저 작성할 때, 이렇게 우리의 API에 관한 미묘한 세부사항을 발견하는 경향이 있다.

> XCTest는 [다수의 검증들][11]을 제공한다. 각각의 검증은 테스트에 관한 것을 설명하는 테스트 기술을 전달할 수 있다. 나는 항상 이 설명을 제공하는 것을 권한다.

현재 상태로는, 테스트를 실행할 수 없다. 심지어 컴파일되지 않는다. 우리는 `MenuItem`를 만들어야 한다. `MenuItem`를 구조체로 만들지 클래스로 만들지 결정하기 전에, 우선 *The Swift Programming Language Guide*에서 [Choosing Between Classes and Structures][12] 섹션을 읽어보길 권장한다.

언뜻 보기에, `struct`가 여기에 충분해 보일 수 있다. 그러나, 아래의 [Handling Menu Item Tap Event][13] 섹션에서 우리는 `NSNotification` 객체에 메뉴 항목을 저장해야 한다. `NSNotification`는 저장하기 위해 `AnyObject` 프로토콜을 따르는 객체를 요구한다. `struct` 타입은 `AnyObject`을 따르지 않는다. 따라서, 우리는 class로 `MenuItem`을 만들어야 한다. `MenuItem.swift`라는 이름의 새로운 파일을 만든다 (*File \> New \> File \> iOS \> Source \> Swift File*). *AppMenu* 와 *AppMenuTests* 타깃 모두에 추가하고 다음과 같이 내용을 바꾼다.

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

테스트를 실행한다 (*Product \> Test* 또는 ⌘U). 그것은 통과할 것이다. 다음으로 부제목 속성에 대한 테스트를 작성하자.

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

메뉴 항목은 제목을 가지고 있어야 하므로, 상수 속성으로 정의되어 있다. 반면 `subTitle`는 필수적이지 않다. 그러므로, 변수 속성으로 정의한다. 마지막으로, `iconName` 속성에 대한 테스트는 다음과 같다:

\~\~\~swift
func testThatMenuItemCanBeAssignedAnIconName() {
	let menuItem = MenuItem(title: "Contributions")
	menuItem.iconName = "iconContributions"
	
	XCTAssertEqual(menuItem.iconName!, "iconContributions",
	    "Icon name should be what we assigned")
}
\~\~\~

이것은 `MenuItem`에 `iconName` 속성을 추가한 뒤에 통과할 것이다.

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

> 모든 테스트가 통과했을 때 [리팩토링][14] 하는 것은 TDD에서 일반적인 방법이다. 리팩토링 과정은 더 나은 코드와 테스트를 구성하여 작은 단위로 디자인을 개선하는 데 도움이 된다. 그것은 또한 중복을 없앨 수 있도록 도와준다. 실패하는 테스트를 먼저 작성하고 그것을 통과하기에 충분한 코드를 만들고 다음 실패하는 테스트를 작성하기 전에 설계를 개선하는 이 반복되는 과정은 *red-green-refactor* 순환으로 알려져 있다.

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

<a name="reading_metadata_from_plist"></a>
Reading Metadata from Plist
===========================

다음은 plist로 부터 `MenuItem` 인스턴스를 만들기 위해 필요한 메타데이터를 읽을 것이다. 우리의 초기 설계에서 알 수 있듯이, 우리는 [plist][16]파일에서 각 메뉴 항목에 대한 메타데이터를 저장할 것이다. 그렇게 하면 만약 원격 서버로부터 메타데이터를 가져와 동적으로 메뉴를 만들 필요가 있을 경우에도, 메타데이터 형식이 동일하게 유지되는 한 너무 많은 변경을 할 필요가 없다. `MenuItemsPlistReader` 클래스를 만들기 전에, 우리는 `MenuItemsReader` 프로토콜이 어떻게 생겼는지 알아야 한다. 초기 단계는 다음과 같다:

\~\~\~swift
import Foundation

protocol MenuItemsReader {
  func readMenuItems() -\> ([[String : String]]?, NSError?)
}
\~\~\~

`readMenuItems` 메서드는 어떤 매개변수도 가지지 않고 [튜플][17]을 반환한다. 파일을 성공적으로 읽은 경우 튜플의 첫 번째 항목은 [딕셔너리][18] 배열을 포함하고 있다. 파일을 읽을 수 없는 경우 두 번째 항목은 [NSError][19] 객체가 포함되어 있다. `readMenuItems`는 필수 메서드다. 그래서 `MenuItemsReader` 프로토콜을 따르고자 하는 모든 클래스는 그것을 구현해야 한다. `MenuItemsReader.swift`라는 이름의 새로운 파일을 만든다. 두 타깃에 추가한 다음 위와 같이 프로토콜 정의 코드로 내용을 바꾼다.

다음에는 plist 파일에서 메타데이터를 읽어오자. 우리는 먼저 테스트를 작성할 것이다. `AppMenuTests` 타깃에 `MenuItemsPlistReaderTests.swift`라는 이름의 새로운 파일을 만든다. Now that you know how to create a 이제 Swift 테스트 파일을 어떻게 만드는지 알기 때문에, 앞으로 이러한 설명은 생략할 것이다. 클래스 정의와 import 문을 제외하고 `MenuItemsPlistReaderTests.swift` 파일의 모든 내용을 삭제한다. 우리의 첫 번째 테스트는 지정된 plist 파일을 읽을 수 없는 경우 `MenuItemsPlistReader`가 에러를 반환하는지 확인하는 것이다. `MenuItemsPlistReaderTests` 클래스에 다음 테스트를 추가한다.

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
	
	func readMenuItems() -> ([[String : String]]?, NSError?) {
	    let error = NSError(domain: "Some domain", 
	                        code: 0, 
	                        userInfo: nil)
	    return ([], error)
	}
}
\~\~\~

이제 테스트를 실행한다. 그것은 통과할 것이다. 테스트를 통과했지만, 뭔가 좋아 보이지 않는다. `readMenuItems`은 심지어 파일을 읽을 시도도 하지 않는다. 항상 빈 배열과 그다지 유용하지 않은 에러를 포함하는 튜플을 반환한다. 이것은 TDD의 중요한 측면을 우리에게 제시한다: *테스트를 통과하기 위해 최소한의 코드를 작성한다*. 테스트를 통과하기 위해 필요한 것보다 더 많은 코드를 작성하지 않도록 훈련하는 것이 TDD의 핵심이다. 그러므로, 우리는 테스트에 필요하지 않는 한 겉보기에는 불완전한 `readMenuItems` 메서드를 수정하지 않을 것이다.

The only requirement we have defined for `MenuItemsPlistReader` class so far is that *it returns an error if the file doesn't exist*. 우리는 에러 객체에 무엇이 있어야 하는지 지정하지 않았다. 우리가 기대하는 도메인, 코드 그리고 설명이 에러에 포함되어 있는지 확인하는 몇 가지 테스트를 더 추가하자.

> 애플은 런타임 에러에 대한 정보를 담기 위해 NSError 객체를 사용하는 것을 [추천한다][20]. 이러한 객체는 에러 *도메인*, 도메인 특정 에러 *코드*, 그리고 에러 *설명*을 포함하는 *user info* 딕셔너리가 들어있어야 한다. *user info* 딕셔너리 안에는 에러에 관한 다른 세부사항, 예를 들어 에러를 해결하기 위해 취할 수 있는 단계 같은 것을 추가할 수 있다.

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
	    userInfo![NSLocalizedDescriptionKey]! as String
	
	XCTAssertEqual(description,
	    "notFound.plist file doesn't exist in app bundle",
	    "Correct error description is returned")
}
\~\~\~

`MenuItemsPlistReader`가 위의 테스트를 통과하도록 만들기 위해 다음과 같이 변경한다.

\~\~\~swift
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
\~\~\~

`readMenuItems` 메서드는 여전히 좋아 보이지 않는다. Next tests we are going to write will force us not to cheat. 넘어가기 전에, `testErrorIsReturnedWhenPlistFileDoesNotExist`라는 이름의 테스트를 삭제한다. 이것은 이전의 세 테스트와 중복된다.

\~\~\~swift
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
\~\~\~

Here we are making sure that `readMenuItems` method actually reads data from the specified plist file and creates proper objects from that data.

> 유닛 테스트를 작성하는 동안 경험적인 규칙은 테스트 메서드에서 하나 이상의 검증을 포함하지 않는 것이다. 파일에서 읽은 데이터를 한 곳에서 올바른지 확인하는 것은 타당하기 때문에, 여기서 규칙을 위반하고 있다.

위의 테스트를 통과하기 위해, "menuItems.plist”이라는 이름의 파일을 만든다 *(AppMenu 그룹을 오른쪽으로 클릭 \> New File \> iOS \> Resource \> Property List)*. 두 타깃에 추가한다. 소스 코드 모드에서 파일을 열고 *(Xcode에서 파일을 오른쪽으로 클릭 \> Open As \> Source Code)* 다음과 같이 내용을 바꾼다:

\~\~\~xml
\<?xml version="1.0" encoding="UTF-8"?\>
\<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"\>
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
\~\~\~

어셋 카탈로그 (Images.xcassets)에 다음의 이미지를 추가한다. 그것들은 [완성된 프로젝트][21]에 포함되어 있다.

* iconContributions@2x.png
* iconRepositories@2x.png
* iconPublicActivity@2x.png

이제 아래 보이는 것과 같이 `readMenuItems` 메서드를 수정한다:

\~\~\~swift
func readMenuItems() -\> ([[String : String]]?, NSError?) {
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
\~\~\~

이제 테스트가 통과했으니, 별도의 메서드로 에러 생성 코드를 추출하여 리팩토링할 수 있다. `readMenuItems` 메서드를 리팩토링한 후의 모습이 여기 있다:

\~\~\~swift
func readMenuItems() -\> ([[String : String]]?, NSError?) {
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

func fileNotFoundError() -\> NSError {
	let errorMessage = 
	    "\(plistToReadFrom!).plist file doesn't exist in app bundle"
	
	let userInfo = [NSLocalizedDescriptionKey: errorMessage]
	
	return NSError(domain: MenuItemsPlistReaderErrorDomain,
	    code: MenuItemsPlistReaderErrorCode.FileNotFound.toRaw(),
	    userInfo: userInfo)
}
\~\~\~

우리가 아무것도 부수지 않았음을 확인하기 위해 테스트를 다시 실행한다 (*⌘U*). 나는 우리의 테스트와 함께 약간의 리팩토링 기회도 확인했다. 모든 테스트에서 공통된 코드를 `setup` 메서드로 옮기자.

\~\~\~swift
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
\~\~\~

나는 우리가 plist가 존재하는 시나리오에 대한 테스트를 추가하는 것을 깜빡했다는 것을 깨달았지만, 그러나 `readMenuItems`는 잘못된 데이터 때문에 아마 그것을 읽을 수 없다. 나는 나의 소중한 독자들을 위한 연습으로 남겨둘 것이다.

<a name="building_menu_items"></a>
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

Menu item은 반드시 title을 가져야 한다. 그러므로 우리는 `MenuItemBuilder`가 title이 아닌 에러를 리턴하는지 확인해야된다. 또한 menu item이 오류가 발생할때 비어있는 리스트를 리턴하는지 확인해야된다.

위의 테스트에선 진짜 menu items 메타데이타 reader (`MenuItemsPlistReader`) 대신에, 우리는 `FakeMenuItemsReader`라고 부르는 가짜를 사용하였다. 그 이유는 앱안의 모든 다른 구성요소로부터 test 클래스를 분리할 필요가 있기 때문이다. 이렇게함으로써 test가 실패 했을때 다른 클래스가 아닌 test 중인 클래스에서 문제가 발생했다는 것을 합리적으로 확신할 수 있습니다. 게다가, 만약 우리의 테스트에서 진짜 메타데이타 reader를 사용한다면 그리고 만약 미래에 클래스를 원격서버로 부터 plist를 다운로드 하기로 결정한다면, `MenuItemBuilder`를 test할때 다운로드 하는동안 쓸데없는 고통을 받을 것입니다. 우리는 항상 깨지지 않고 빠른 쉬운 유지보수를 목표해 왔다.

`FakeMenuItemsReader`는 다른 menu items에서 독자적으로 있을 경우 `MenuItemsReader`의 프로토콜을 준수 해야한다. 메타데이터를 파일이나 원격 서버로 부터 읽을때 이것은 항상 hard-coded 된 배열 dictionaries로 리턴한다. `FakeMenuItemsReader` 클래스를 만들고 더한하는건 오직 `AppMenuTests` 대상으로 한다. 우리는 이 클래스를 어떠한 어플리케이션 코드에서도 사용하지 않을 것이다. `FakeMenuItemsReader.swift` 파일의 내용을 대신 작성한다:

\~\~\~swift
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
\~\~\~

*가짜* 클래스들에 종종 생기는 한가지 관심사는 만약 원래 클래스들의 퍼블릭 API들의 바뀌는것이며 그것은 유효한 관심사다. 그렇지만 Swift가 컴파일 할때 에러가 발생할때 우리는 그것에 대해 걱정할 필요가 없다. 
예를들어 만약 우리가 `MenuItemsReader` 프로토콜안의 `readMenuItems` 로부터 menu items의 non-optional 배열을 반환하기로 결정한 경우 우리는 모두 `MenuItemsPlistReader` 및 `FakeMenuItemsReader` 클래스에 그 변경 사항을 적용하도록 강요한다. 어서 시도해보자. Swift는 좋은방법인가?

> 만약 너가 "alternate universe"를 더 배우길 원한다면 이러한 가짜 객체는 테스트에서 만들수 있습니다. 구체적인 예로 [Practical Object Oriented Design in Ruby][22] 책의 챕터 9(*Creating Test Double* section)를 읽어 주십시요.

*가짜* 객체의 다른 관심사는 그들이 잘못된 보안감각을 제공할 수 있다는 점이다. 우리는 어떻게 'MenuItemsPlistReader'와 'MenuItemBuilder'가 함께 작동하는지 잘 알수있을까요? 그 대답은 유닛테스트를 통해 하진 않을 것이다. 앱의 다른 유닛이 서로 잘 작동하는지 확인하는 작업은 통합테스트에서 주어지며 이 블로그에선 포함되어 있지 않다.

*AppMenu* 그룹안에 'MenuItemBuilder'라는 이름의 새로운 Swift 클래스를 만들자. 모두 대상에 추가하고 다음과 같이 내용을 바꿉니다:

\~\~\~swift
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
\~\~\~

우리는 에러 tests통과할 충분한 코드를 썻다. 다음으로 우리는 빌더가 정확한 수의 menu items를 만드는지 확인해야한다. `MenuItemBuilderTests` 클래스에 다음 테스트를 추가하자.

\~\~\~swift
func testOneMenuItemInstanceIsReturnedForEachDictionary() {
	fakeMenuItemsReader!.missingTitle = false
	let (metadata, _) = fakeMenuItemsReader!.readMenuItems()
	
	(menuItems, _) =
	    menuItemBuilder!.buildMenuItemsFromMetadata(metadata!)
	
	XCTAssertTrue(menuItems?.count == 2,
	    "Number of menu items should be equal to number of dictionaries")
}
\~\~\~

Swift는 내가 관심없는 것은 `-`를 사용하여 리턴값을 무시하기 쉽고 이것은 내가 특별히 좋아하는 기능이다. `MenuItemBuilder` 클래스의 변경에 따라 모든 테스트를 통과 해야한다.

\~\~\~swift
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
\~\~\~

> 너는 내가 *red-green-refactor* 주기와 위의 변경을 엄격하게 수행하지 않은 것을 알수 있다. 나는 모든 테스트가 통과한 후 별도의 방법으로 error building code에서 추출해야합니다. 비록 나는 최소한의 코드를 써서 테스트를 통과하는 TDD의 첫번째 룰을 무시하는건 권장하지 않지만, 나는 이제 모든걸 정확하게 그리고 이 블로그 게시물이 너무 길지 않도록 할 것이다.
`MenuItemBuilder`의 마지막 테스트는 metadata dictionaries에 있는 값으로 menu item의 인스턴스의 속성 값을 채웠는지 확인하는 것입니다.

\~\~\~swift
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
\~\~\~

다시한번, 우리는 이 테스트 내부에서 여러가지 주장을 사용하고 있다. 위의 테스트는 어떠힌 코드를 변경하지 않고 통과 해야한다.

<a name="displaying_menu_items"></a>
## 메뉴 아이템 표시
=====================

이제 우리는 `MenuItem` 인스턴스를 빌드하는 방법과 plist의 정보들로 부터 채울 수 있는 방법을 알고있다. 그럼 컨텐츠를 보여주는 것으로 초점을 옮겨보자. 우리는 테이블뷰를 이용해 메뉴 아이템들을 보여줄 것이다. 우리의 초기 설계 알 수 있듯이, `MenuTableDefaultDataSource` 는 철저히 설정된 각각의 `UITableViewCell` 메뉴 아이템에서 제공되는 정보를 응답할 것이다. 테이블뷰 자체는 `MenuViewController`에 의해 관리된다. 

<a name="providing_data_to_table_view"></a>
### 테이블뷰에 데이터 제공

우리는 테이블뷰의 데이터소스를 `MenuViewController`에 직접 구현을 하기 보다  분리된 객체를 사용할 것이다. `MenuViewController` 는 이미 뷰들을 관리하는 역할을 하고 있습니다. 나는 테이블뷰의 데이터를 미리 준비해서 [단일 책임 원칙][23]을 위반하는 것을 피할 것이다. 그러나 첫번 째로 우리는 `MenuTableDefaultDataSource`에 일치하는 프로토콜을 만들것 이다. `MenuTableDataSource.swift`라는 스위프트 파일 파일을 *AppMenu* 그룹에 새로만들고. 파일의 타겟을 추가한 뒤 아래의 코드로 변경한다.

\~\~\~swift
import UIKit

protocol MenuTableDataSource : UITableViewDataSource {
	func setMenuItems(menuItems: [MenuItem])
}
\~\~\~

`MenuTableDataSource`는 `UITableViewDataSource` 로 부터 상속된 프로토콜이다. 또한 `setMenuItems` 메소드를 필수로 요구하고 있다. 이제 우리는 `MenuTableDefaultDataSource`의 테스트 작성이 준비됐다. `AppMenuTests` 타겟 안에서 `MenuTableDefaultDataSourceTests.swift` 라는 새로운 테스트 파일을 생성하고 다음의 코드를 추가한다.

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

여기서 우리는 각각의 메뉴 아이템 데이터 소스가 하나의 데이블뷰 셀 인스턴스를 만들고 있다는 것을 확인했다. 이제 `MenuTableDefaultDataSource.swift` 라는 새로운 스위프트 파일을 만들고 아래의 코드를 입력한다.

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

아직 우리는 테스트를 위한 `tableView:cellForRowAtIndexPath:` 메소드를 아직 작성하지 않았다. 우리는 요청에 대해 동작할 수 있는 구현이 테스트 이전에 필요하다. 이 작업은 `UITableViewDataSource` 프로토콜에서 요구하는 메소드가 있어야 하고 스위프트는 `MenuTableDefaultDataSource` 없이 컴파일을 할 수 없기 때문이다.

혹시 `MenuTableDefaultDataSource` 에 대하여 알아 차렸을 수도 있겠지만 이것은 `NSObject` 를 상속받고 있다. 그 이유는 `UITableViewDataSource` 프로토콜과 일치하기 위해, 또한 `NSObject` 프로토콜을 준수할 필요가 있기 때문이다. 이것을 가장 쉽게 따르는 방법으로는 `NSObject` 프로토콜을 준수 하는 `NSObject` 의 서브클래스를 만드는 것이다.

위의 테스트에서, 우리는 `tableView:numberOfRowsInSection:` 메서드에서 무조건 `1` 을 반환하도록 만들었다. 데이터 소스가 얼마나 많은 메뉴 아이템이 있든 항상 정확한 수를 반환하는지 증명하는 테스트를 추가한다.

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

또한 우리는 데이터소스가 정확한 섹션의 갯수를 반환하는지 확인이 필요하다. 여기 그것에 대한 테스트이다:

\~\~\~swift
func testReturnsOnlyOneSection() {
	let dataSource = MenuTableDefaultDataSource()
	let numberOfSections = dataSource.numberOfSectionsInTableView(nil)
	XCTAssertEqual(numberOfSections, 1, 
	    "There should only be one section")
}
\~\~\~

`numberOfSectionsInTableView` 메서드로부터 리턴되는 `1` 은 이전 테스트의 통과를 시키기 위해 만들었다. 또한 이 메서드는 `UITableViewDataSource` 프로토콜에서 필수로 요구되는 메서드는 아니다. 그리고 기본적으로 이미 `1` 이 리턴되고 있다. 우리는 테스트로 부터 명령이 가능한 호출에 대해 구현이 필요하다.

\~\~\~swift
func numberOfSectionsInTableView(tableView: UITableView!) -\> Int {
	return 1
}
\~\~\~

> 또 다른 테스트로 검증을 위해 작성하기 좋은것으로 데이터소스로 부터 예외처리를 발생시키는 것이다. 섹션안에 행의 숫자 인덱스를 0이 아닌 다른숫자로 입력하라고 묻는것 이라면, 하지만 우리의 오래된 친구 `XCTAssertThrows` 를 Xcode 6의 XCTest에서는 찾을 수 없다. 그래서 예외상황이 발생된 상황을 다르게 증명하는 방법을 모르겠다.

iOS에서 모든 뷰의 테스트를 하는건 상당히 지루할 수 있다. 나는 최소의 뷰만 테스트하는 경향이 있다. 이말은 대표적인 뷰들만 테스트를 하는것인데, 이 경우에는 각각의 메뉴를 대표하는 테이블뷰 셀만 테스트를 하는것이다. 그러므로, 나는 각각의 메뉴 아이템으로 부터의 대표적인 셀만 증명하는 것을 선호한다. 아래의 코드를 참고하자.

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

`MenuTableDefaultDataSource` 테스트를 위한 리팩토링을 해보자. 우리는 예전에 `setup` 메서드에서 공통 코드를 추출했었다.

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

<a name="handling_menu_item_tap_event"></a>
### Handling Menu Item Tap Event (메뉴 아이템 탭 이벤트 핸들링)

테이블뷰 설정은 보는것과 같이 굉장히 간단하다. 그러므로, 데이터소스나 델리게이트 같은 오브젝트의 사용도 이해가 될것이다. 테이블 뷰의 셀이 탭될때 데이터소스는 알림을 보낼 것이다. `MenuViewController` (또는 관심이 있는 다른 클래스) 는 어떤 셀이 탭이 되었고 어떤 액션을 받을 것인지 찾기위해 신호를 보낼 수 있다.

[![table\_view\_architecture.png][image-4]][24]

이 디자인은 [Test-Driven iOS Development][25] 책의 챕터 9로 부터 약간의 영감을 받았다. 그럼 `MenuTableDataSource` 프로토콜에 델리게이트와 관련된 세부항목을 추가하자.

\~\~\~swift
import UIKit

let MenuTableDataSourceDidSelectItemNotification = 
	"MenuTableDataSourceDidSelectItemNotification"

protocol MenuTableDataSource : UITableViewDataSource, UITableViewDelegate {
	func setMenuItems(menuItems: [MenuItem])
}
\~\~\~

이제 우리는 데이터소스가 정말로 테이블뷰의 아이템이 탭되었을 때 알림을 보내는지 알수 있는 증명이 필요하다. 다음과 같이 테스트해볼 것이다.

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
 
`setup` 메서드 안에서, 우리는 `MenuTableDataSourceDidSelectItemNotification` 라는 이름의 노티피케이션 옵져버를 테스트 클래스에 추가했었다. 알림이 도착할 때, `didReceiveNotification:` 메서드는 호출될 것이다. 노피티케이션 오브젝트는 `postedNotification` 변수에 저장된다. 그때 우리는 이것이 정확한 이름과 메뉴 아이템 인스턴스라는 것이 증명된다. 여기서 중요하게 생각해야 할 점은 `tearDown` 메서드 안에 옵져버가 테스트 클래스를 지운다는 것이다. 우리는 [NSNotificationCenter][26] 에서 노티피케이션이 보내졌을때 확인 할 수 있는 API를 제공하지 않기 때문에 이 복잡한 프로세스를 통해서 노티피케이션이 정말로 보내지는지 증명할 수 있었다. 

> [Building Menu Items][27] 섹션 안에서 나는 가짜 오브젝트를 사용하는 것을 추천한다. 그러나, 나는 위에 테스트에서 `NSNotificationCenter` 클래스를 사용했다. 일반적으로 나는 애플 프레임워크에서 제공되는 오브젝트의 대체되는 다른것을 사용하지 않는다. 그들은 신뢰할 수 있는 용어와 속도를 공평하게 한다. 그 들이 말하기를 만약 당신의 테스트에서 애플 프레임워크로 부터 제공하는 객체의 사용이 줄어든다면 그들을 위해서 테스트를 대체하기 위한 다른걸 만드는 것을 주저하지 않을 것이다.

`MenuTableDefaultDataSource` 클래스 안에서 `UITableViewDataSource` 프로토콜로 부터 `tableView:didSelectRowAtIndexPath:` 메서드가 테스트를 통과 하도록 구현한다.

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

<a name="managing_menu_table_view"></a>
### Managing Menu Table View (메뉴 테이블 뷰 관리)

`MenuViewController` 는 테이블뷰와 메뉴에서 보여줄 모든 필요한 뷰들을 관리하는 역할을 할 것이다. 첫번째로 우리는 필요한 데이터소스를 만들야 한다. 또한 테이블뷰와 타이틀이 확실히 필요하다. 새로운 테스트 파일 `MenuViewControllerTests.swift` 을 만들고 아래의 내용을 추가한다.

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

여기에 실제 데이터 소스를 사용하는 것 대신에, 우리는 `MenuTableFakeDataSource` 라는 이름의 가짜 데이터를 사용할 것이다. `AppMenuTests` 안에 `MenuTableFakeDataSource.swift` 라는 이름의 스위프트 파일을 생성하고 타겟을 정하고 다음의 코드로 대체한다.

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

모든 `MenuTableFakeDataSource` 은 `MenuTableDataSource` 프로토콜 안의 필요 메서들 구현을 요구한다. 그리고 `MenuTableDataSource` 에 일치하는 모든 객체들을 대신한다. 지금 `MenuViewController` 클래스를 생성한다. (*AppMenu 그룹을 오른쪽으로 클릭 \> New File \> iOS \> Source \> Cocoa Touch Class*). `UIViewController` 의 서브클래스로 생성한다 그리고 *Also create XIB file* 체크박스도 선택한다. 그리고 절대 타겟을 두개다 추가하는 것을 잊지말자. 이 작업은 위의 선언된 두개의 구역과 타이틀을 선정하기 위한 테스트를 통과 하기 위함이다.

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

`MenuViewCOntroller.xib` 의 메인뷰 사이즈는 *Attricbutes Inspector* 섹션 안에서 *Simulated Metrics* 을 *iPhont 4-inch* 로 변경한다. 뷰의 오리엔테이션은 *Portrait* 로 설정한다. 메인 뷰의 서브뷰, 테이블뷰 같은 뷰들을 추가한 후에. `MenuViewController` 클래스의 `tableView` 아울렛을 XIB의 테이블뷰와 연결한다.

다음으로 우리는 `MenuViewController` 의 델리게이트와 데이터소스 영역들에 대한 설정들을 우리가 정한 데이터 소스 객체로 지정해야 한다. [viewDidLoad][28] 메서드는 우리가 원하는 연결을 정할 수 있는 곳이다. 다음 테스트를 코드를 확인하자.

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

위의 테스트를 통과 시키기 위해 `viewDidLoad` 안에서 테이블뷰의 데이터소스와 델리게이트를 연결한다.

Set table view's data source and delegate properties in `viewDidLoad` to make above tests pass.

\~\~\~swift
override func viewDidLoad() {
	super.viewDidLoad()
	title = "App Menu"
	tableView.dataSource = dataSource
	tableView.delegate = dataSource
}
\~\~\~

[Handling Menu Item Tap Event][29] 에서 우리는 메뉴 아이템을 탭했을 때  `MenuTableDefaultDataSource` 가 노티피케이션을 보내는 것을 만들었었다. `MenuViewController` 는 노티피케이션을 받아서 정확한 메뉴 아이템의 뷰인지 확인할 수 있는 것이 필요하다. 만약 그 노티피케이션이 도착했는데 `MenuViewController` 의 뷰가 감춰져 있다면, 그건 무시될 것이다. 그러므로, `viewDidAppear:` 메서드에서 노티피케이션을 등록해야한다. 또한 `viewDidDisaapear:` 메서드 안에서 해체를 해주어야 한다. 다음의 테스트를 통해서 요구하는 것을 확인하자.

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

굉장히 많은 코드가 있는데, 설명을 하자면. `MenuViewController` `MenuTableDataSourceDidSelectItemNotification` 의 호출을 받기 위해 자기 자신을 등록했다. 우리는 노티피케이션이 도착했을 때 우리가 원하는 메서드를 어떻게 호출할 것인지 알아야 한다. 이것은 한번만 받을 수 있어서, 노티피케이션이 메서드를 통과할 때 실체를 알수 있는 검증이 필요하다. 간단하게 비 개인화 속성으로 `MenuViewController` 안에 노티피케이션을 만들 것이다. 그러나 개인적으로 이 접근 방법을 좋아하진 않는다. `MenuViewController` 는 단지 테스트를 위해서 강제로 노출되지 않아야 한다. 여기에 더 좋은 방법이 있다. 우리는 어떻게 [swizzle][30] 노티피케이션 핸들러를 런타임에서 각자 다르게 요구되는 구현을 테스트의 목적에 맞게 사용할 수 있을까? 다음 코드를 보자.

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

그리고 여기에 `MenuViewController` 클래스를 테스트 구현에 요구되는 대로 [extension][31] 하였다.

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

우리는 여기에 테스트안에서 측정할 수 있는 `postedNotification` 을 지속적으로 받기 위한 노티피케이션 등록을 모두했다. `testRegistrationForMenuItemTappedNotificationHappensInViewDidAppear` 안에서 swizzle 노티피케이션 핸들러를 등록한 후에 `viewDidAppear` 를 호출하고, nil 이 아닌 `postedNotificiation` 을 노티피케이션과 검증을 위해 보냈다. 반면 `testRemovesItselfAsListenerForMenuItemTappedNotificationInViewDidDisappear` 에서는 먼저 `viewDidAppear` 를 호출하고 `MenuViewController` 에 노티피케이션을 등록했다. 이 노티피케이션은 `viewDidDisappear` 안에서 자신을 옵져버를 제거한다. 이는 노티피케이션이 `MenuViewController` 에서 `viewDidDisappear` 호출된 후에도 노티피케이션을 받을 수 있기 때문이다.

테스트들을 통과하기 위해선 우리는 모두 노티피케이션의 등록과 해제 적절한 곳에서 해야한다.

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

<a name="sliding_views_in"></a>
Sliding Views In
================

menu item이 눌렸을 때 view가 보여져야 한다. 하지만 어떤것? menu item에 직접 물어보는 건 어떨까? 단순함을 유지하기 위해 view controller의 이름을 `MenuItem`에 `tapHandlerName` property 로 저장하자.

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

이것은 menu item이 tap handler를 갖지 않을 좋을 방법이다. Therefore, `tapHandlerName` property를 optional로 만들어야 한다.  이제 `MenuItem`에 property를 추가하고 `menuItems.plist`, `FakeMenuItemsReader`, `MenuItemsPlistReaderTests`, `MenuItemBuilderTests`, 그리고 `MenuItemBuilder`를 맞추자. 조정된 코드는 아래에 나열되어 있다.

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

다음으로 menu item이 눌렸을 때 `MenuViewController`가 맞는 view를 보여주도록 해야한다. 다음 test가 그렇게 할 것이다.

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

`MenuViewController`를 app navigation stack의 오른쪽 view controller로 push하도록 만들자

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

또한 tap handler class들을 만들어야 한다. 다음의 view controller를 만들고 각각에 *AppMenu* and *AppMenuTests* target을 추가한다. 각각을 위한 *XIB* 파일을 만드는 것 또한 잊지 말자.

* `ContributionsViewController`
* `RepositoriesViewController`
* `PublicActivityViewController`

switch case 문을 사용하는 대신에 runtime에 view controller를 만들지 않는지 궁금할 것이다.

* 나는 Swift에서 가장 좋은 방법이 무엇인지 확신할 수 없다. Objective-C에서는 다음의 code에서와 같이 쉽게 할 수 있다.

  \~\~\~Obj-C
	UIViewController *tapHandler = nil;
	Class tapHandlerClass =
	     NSClassFromString(menuItem.tapHandlerName)
	
	if (tapHandlerClass) {
	    tapHandler = [[tapHandlerClass alloc] init];
	}
  \~\~\~

* Objective-C와는 다르게, Swift는 XIB의 이름과 view controller의 이름이 같더라도 view controller의 instance를 만들 때, XIB를 지정해야 한다. 게다가, 간단히 `alloc`, `init`으로 부르는 기능은 Swift에서는 필요하지 않다.

불행히도, test들을 통과하게 하진 못한다. 모든 class에 드러내다. It turns out that so far we have built every class we initially set out to build except `AppMenuManager`. 그 class가 만들어지면 위의 test를 통과할 수 있어야 한다. 해보자.

<a name="managing_app_menu"></a>
### App Memu 관리하기

*AppMenuTests* target안에 file name이 `AppMenuManagerTests.swift`인 새로운 test 파일을 만든다. 다음의 test를 추가한다.

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

metadata로부터 성공적으로 `MenuItem` object가 만들어진 경우 `AppMenuManager`는 `MenuViewController`를 만들 책임이 있다. 그렇지 못할 경우 nil을 반환한다. Since `AppMenuManager` mostly coordinates the interaction between various objects rather than doing the work itself, we also need to make sure that it passes the metadata (if read successfully) to the builder. You might have noticed that we are using fake menu items reader and builder objects here so that we can control what gets returned to app menu manager in tests. We built a fake menu items reader in [*Building Menu Items*][32], but it doesn't provide a way for us to set the error. Let's take care of that.

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

다음으로 `FakeMenuItemBuilder`class를 만들어야 한다. Now that there is going to be more than one class playing the role of a menu item builder, we should create a protocol to make it clear what it means for a class to become a menu item builder. For now, playing that role means implementing `buildMenuItemsFromMetadata` method correctly. 아래는 새로운 protocol이다.

\~\~\~swift
import Foundation

protocol MenuItemBuilder {
	func buildMenuItemsFromMetadata(metadata: [[String : String]]) -> ([MenuItem]?, NSError?)
}
\~\~\~

잠깐만. Didn't we already name our real builder class `MenuItemBuilder`? 그렀다, 했다. `MenuItemBuilder` 이름은 protocol에 더 적합하다. 원래의 builder class의 이름을 `MenuItemDefaultBuilder`로 바꾸자.

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

또한 새로운 이름을 사용하도록 test를 조정해야 한다.

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

마지막으로, here is what the `FakeMenuItemReader` class looks like. 오직 test에서만 사용하기 때문에 *AppMenu* target에 추가할 필요는 없다.

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

It makes the metadata passed to it available for inspection. It also allows us to set the error and menu items we want it to return which is very convenient. `AppMenuManager` class를 build할 준비가 되었다. Here is what it looks like.

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

여기에서 *read-green-refactor* cycle을 따르지 않는 것을 사과한다. I매 단계 진행하는 절차를 보여주는 것보다 test를 보다 쉽게 작성하는 중요한 테크닉에 집중하길 원했다. 이러한 테크닉 중 하나는 test를 유지하면서 쉽게 그것들을 바꿀 수 있도록 실제 object와 동일한 역할을 하는 가짜(또는 테스트를 위한) object를 만든 것이다. 가짜 object에 대해서 [Martin Fowler][33]가 쓴 [좋은 게시물][34]이 있다.

넘어가기 전에, 테스트할 수 있고 재사용할 수 있는 class들을 작성하는데 [Dependency Injection][35]의 중요성에 대해 강조하고 싶다. 우리의 `AppMenuManager` class는 `MenuItem` object를 만들기 위해 `MenuItemsReader`과 `MenuItemBuilder` protocol을 따르는 두 개의 다른 class들의 함께 동작해야 한다. Had we not exposed these two dependencies via public properties, we would not have been able to pass in fake objects. Those fake objects came very handy while setting up the desired test scenarios in order to verify that `AppMenuManager` behaved as expected. Therefore, I recommend exposing every single dependency your classes have unless those dependencies are classes provided by Apple frameworks.

<a name="putting_it_all_together"></a>
Putting It All Together
=======================

거의 다 왔다. Now that we have built every class, let's put them together in `AppDelegate`. But first we will write some tests to verify that  `AppDeleate` behaves as expected. *AppMenuTests* target에 이름이 `AppDelegateTests.swift`인 새로운 테스트 파일을 만든다. 다음의 test를 추가한다.

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


> [App Menu 관리하기][36]에서, 진짜 menu builder를 위한 가짜 object가 필요하다고 알아차렸을 때 `MenuItemBuilder` protocol을 만들었다. 하지만, 여기에서는 내부적으로 test 자체의 가짜 app menu manager object들을 만든다. 이건 완벽하게 괜찮다. 진짜 app menu manager class에 있는 `menuViewController` method의 이름을 변경하기로 결정한 경우, Swift는 모든 우리의 가짜 object들의 이름도 새로운 method 이름을 사용하도록 강제할 것이다. 그 덕분에 모든 가짜 object들은 진짜 app menu manager와 동기화된다. test 내부에서 빠르게 가짜 object를 만들 때, 이 방법은 매우 유용하다.

새로운 Xcode project를 만들 때, `AppDelegate`는 *AppMenu* target에만 추가되어 있다. `AppMenuTests` target에도 잘 추가해야 한다. 그 후에 다음과 같이 내용을 변경한다:

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

`AppDelegate`에서 `AppMenuManager`를 설정하는 code를 추출하는 것이 좋다. We are going to apply what [Graham Lee][37] taught us in [Test-Driven iOS Development][38] here and create our own dependency injection class instead of using a full blown [depdendency injection framework][39]. 적어도 지금은 App Menu는 간단한 app이다. 그래서 필요한 것이 아니면 dependency를 추가해서는 안된다. *AppMenuTests* target에 이름이 `ObjectConfiguratorTests.swift`인 새로운 test 파일을 만들고 다음의 내용으로 바꾸자.

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

`ObjectConfigurator` class를 만들고 각 target에 추가한다. 다음의 내용으로 바꾸자.

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

Instead of creating an `AppMenuManager` object itself, app delegate will tell the object configurator to do so. 새로운 접근법이 포함되도록 `AppDelegate`와 그의 test를 변경하자.

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

`MenuViewController`로 관심을 돌려보자. 지금 모든 test를 통과해야 하지만 조금의 refactoring을 해야한다. Let's extract the code that decides which view controller should be the tap handler into a separate class. *AppMenuTests* target에 이름이 `MenuItemTapHandlerBuilderTests`인 새로운 test class를 만들고 다음의 내용으로 바꾸자.

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

[![final\_app\_design.png][image-5]][40]

<a name="conclusion"></a>
결말
===========

이 post에서 TDD를 활용한 간단한 iOS app을 만드는 법에 대해서 배웠다. Xcode 6 beta는 이 글을 쓰는동안 약간 불안정했지만, XCTest 자체는 상당히 안정적으로 보였다. [OCMock][41]과 [Kiwi][42] 같은 mocking library들의 부족에도 불구하고, fake object를 쉽게 만들고 그것을 test에 사용하는 것이 가능했다. Swift의 method 내부에서 class를 만드는 능력은 전문적인 가짜 object를 빠르게 만드는데 편리했다.

Swift는 완전히 새로운 언어임에도 불구하고, 이미 배웠던 Objective-C(또는 그 문제를 위한 어떤 다른 언어)에서의 test 기능을 위해 배웠을 기술들도 여전히 Swift에서 적용할 수 있다. 이 post에서 Test-Driven Development를 겉핥기만 했다.나는 TDD의 깊이있는 이해를 위해 [더 읽을거리][43] section의 참고자료를 읽을 것을 권장한다. 당신의 다음 iOS app에서 TDD를 시도하기를 바란다. design(그리고 test)에서 더 좋게 하는 유일한 방법은 그것을 더욱 많이 하는 것이다.

완성된 project는 [Github][44]에서 있다.

<a name="further_reading"></a>
더 읽을거리
===============

* [XCTest​Case / XCTest​Expectation / measure​Block()][45]
* [Test-Driven iOS Development][46]
* [xUnit Test Patterns: Refactoring Test Code][47]
* [Practical Object Oriented Design in Ruby][48]

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
[16]:	http://goo.gl/naf71B
[17]:	http://goo.gl/9k9O5u
[18]:	http://goo.gl/ucLgzF
[19]:	http://goo.gl/VYcIen
[20]:	http://goo.gl/akPPGh
[21]:	http://goo.gl/WvUBDo
[22]:	http://goo.gl/bbzSpz
[23]:	http://www.objectmentor.com/resources/articles/srp.pdf
[24]:	http://img.svbtle.com/9pmnjzuddxpv3g.png
[25]:	http://goo.gl/iiKpC1
[26]:	http://goo.gl/TfnJ3T
[27]:	#building_menu_items
[28]:	http://goo.gl/OeT0hV
[29]:	#handling_menu_item_tap_event
[30]:	http://nshipster.com/method-swizzling/
[31]:	http://goo.gl/lL1Cwy
[32]:	#building_menu_items
[33]:	http://martinfowler.com/
[34]:	http://martinfowler.com/articles/mocksArentStubs.html
[35]:	http://www.martinfowler.com/articles/injection.html
[36]:	#managing_app_menu
[37]:	https://twitter.com/secboffin
[38]:	http://goo.gl/iiKpC1
[39]:	http://www.typhoonframework.org/
[40]:	http://img.svbtle.com/3fmqoko8psjlrw.png
[41]:	http://ocmock.org/
[42]:	https://github.com/kiwi-bdd/Kiwi
[43]:	#further_reading
[44]:	https://github.com/pawanpoudel/AppMenu
[45]:	http://nshipster.com/xctestcase/
[46]:	http://goo.gl/iiKpC1
[47]:	http://goo.gl/HD4b3X
[48]:	http://goo.gl/bbzSpz

[image-1]:	https://d23f6h5jpj26xu.cloudfront.net/aacqorrxf8wpiw.png
[image-2]:	https://d23f6h5jpj26xu.cloudfront.net/ttuxgmqb3ia.png
[image-3]:	https://d23f6h5jpj26xu.cloudfront.net/fqgfy5r3w7nkq.png
[image-4]:	https://d23f6h5jpj26xu.cloudfront.net/9pmnjzuddxpv3g.png
[image-5]:	https://d23f6h5jpj26xu.cloudfront.net/3fmqoko8psjlrw.png