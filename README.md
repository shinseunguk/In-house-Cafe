# ☕️ 사내 카페 앱
## 프로젝트 소개
`이 앱은 비공개 앱 배포된 앱입니다.`

- 사내에서 APP으로 간편하게 주문하고 픽업 하는 앱
- Push Notification으로 주문 알림, 주문 완료 알림 수신
- 주문내역 확인
- 이 외 사내 관련 정보들을 확인 가능




<br>

## 팀원 구성

<div align="center">

| **신승욱** |
| :------: |
| [<img src="https://avatars.githubusercontent.com/u/69791286?v=4" height=150 width=150> <br/> @shinseunguk](https://github.com/shinseunguk) |

</div>

<br>

## 1. 개발 환경

- App(Front) : iOS(RxSwift + ReactorKit)
- Back-end : node.js
- 버전 및 이슈관리 : Gitlab, Gitlab Issue, GitFlow
- 협업 툴 : Slack, Figma
- Library : ReactorKit, Snapkit, Then, Alamofire, RxSwift, Firebase, KingFisher, Toast-Swift
- 서비스 배포 환경 : AppStore 비공개 배포
<br>

## 2. 채택한 개발 기술과 Trouble Shooting

## 개발 기술

#### UIKit
- UIKit을 이용하여 프로젝트의 기본적인 UI와 화면 구성을 구현하였습니다. UIKit의 강력한 뷰 계층과 커스터마이징 옵션을 활용하여 직관적이고 사용자 친화적인 UI를 만들었습니다. 높은 제어와 커스터마이징이 필요한 경우에 적합하여 복잡한 레이아웃과 애니메이션을 구현하는 데 큰 도움을 주었습니다.

#### SnapKit
- 레이아웃 설정을 간편하게 하기 위해 `SnapKit`을 도입하여 UI 구성 요소의 오토레이아웃을 관리했습니다. SnapKit을 사용함으로써 코드의 가독성이 향상되고 오토레이아웃 설정을 더욱 직관적이고 효율적으로 처리할 수 있었습니다. 특히 UIKit과 결합하여 제약 조건을 간편하게 설정할 수 있어, 레이아웃 유지보수가 용이해졌습니다.

#### ReactorKit
- `ReactorKit`을 채택하여 애플리케이션의 상태 관리와 로직을 구조화하였습니다. 이를 통해 복잡한 상태 관리와 비동기 작업을 효과적으로 처리할 수 있었으며, 코드의 가독성과 유지보수성이 향상되었습니다.

#### MVVM (Model-View-ViewModel)
- MVVM 패턴을 채택하여 코드의 모듈화와 테스트 가능성을 높였습니다. MVVM 패턴을 사용함으로써 View와 Model의 종속성을 줄이고 ViewModel을 통해 데이터를 바인딩하여 보다 클린하고 유지보수가 용이한 코드를 작성할 수 있었습니다. 특히 데이터와 UI가 분리되어, View와 비즈니스 로직 간의 의존성을 낮추는 데 큰 도움이 되었습니다.

### GitFlow
- **GitFlow**를 채택하여 프로젝트의 버전 관리를 체계적으로 진행했습니다. GitFlow는 주요 브랜치를 역할에 따라 분리함으로써 안정성과 협업 효율을 높일 수 있는 브랜치 전략입니다.
- **master** 브랜치는 안정적인 릴리스 버전만을 포함하도록 유지하고, **develop** 브랜치를 통해 새로운 기능을 개발하며, **feature** 브랜치를 각 기능별로 생성하여 기능 단위로 작업을 분리하였습니다.
- 릴리스가 준비되면 **release** 브랜치를 통해 안정화 작업을 진행하였고, 버그 수정이나 긴급한 업데이트는 **hotfix** 브랜치를 통해 master 브랜치에 직접 반영하였습니다. 이를 통해 협업 과정에서 충돌을 최소화하고, 체계적인 개발과 배포 프로세스를 유지할 수 있었습니다.



## Trouble Shooting

#### 1. 주문이 한꺼번에 들어와 서버에 과부하가 생겨 딜레이 되는 케이스
- Server Timeout을 늘리고 서버 요청후 5초 안쪽으로 Response를 받지 못 하면 Toast로 알림 노출

```swift
// 서버 POST 요청
    /// - Parameters:
    ///   - url: Server URL
    ///   - parameters?: 파라미터
    ///   - headers?: http헤더
    /// - Returns: 서버에서 넘어온 Observable Data
func postRequest(url: String, parameters: [String: Any]?, headers: HTTPHeaders?) -> Observable<Data> {
    traceLog(url)

    // 타이머를 설정하여 5초 후에 특정 함수를 호출합니다.
    if url.contains("/complete") || url.contains("/delay") {
        self.timer = Timer.scheduledTimer(withTimeInterval: 5.0, repeats: false) { _ in
            makeToast(message: "현재 주문량이 많아서 지연되고 있습니다. 잠시만 기다려주세요")
        }
    }
    
    return Observable.create { observer in
        let request = AlamofireManager.shared.request(url,
                                    method: RequestType.post.method(),
                                    parameters: parameters,
                                    encoding: JSONEncoding.default,
                                    headers: headers)
            .validate()
            .responseData { response in
                if let headerFields = response.response?.allHeaderFields as? [String: String],
                    let _ = response.response?.url {
                    if let member = headerFields["member"] {
                        UserDefaults.standard.set(member, forKey: "member")
                    }
                }
                
                switch response.result {
                case .success(let data):
                    self.timer?.invalidate()
                    
                    observer.onNext(data)
                    observer.onCompleted()
                case .failure(let error):
                    makeToast(message: "네트워크 오류, 잠시후 다시 시도해주세요\n동일 현상이 지속된다면 인크로스 서비스개발팀에 문의해주세요")
                    self.timer?.invalidate()
                    traceLog(error)
                    observer.onError(error)
                }
            }
        
        return Disposables.create {
            request.cancel()
        }
    }
}
```

### 2. 카페 오픈 상황과 음료 제조 가능 여부 확인
- 해당 Case가 테스트하기가 상당히 까다로웠던 Case, 주문을 하려면 카페가 열려있는지 음료 제조가 가능한지를 체크해야하는데 서버를 호출하는 생명주기는 화면에 진입할때마다 호출 해주었고 웹훅을 통해 상태가 변경되면 앱에서도 대응을 진행하였음.
  
<br>


## 3. 프로젝트 구조

```
.
├── IncrossOfficeCafe
│   ├── API
│   │   └── APIService.swift
│   ├── App
│   │   ├── AppDelegate.swift
│   │   └── SceneDelegate.swift
│   ├── Base.lproj
│   ├── Enum
│   │   └── Enum.swift
│   ├── Extension
│   │   ├── Extension+Double.swift
│   │   ├── Extension+Int.swift
│   │   ├── Extension+ObservableType.swift
│   │   ├── Extension+String.swift
│   │   ├── Extension+UIApplication.swift
│   │   ├── Extension+UIColor.swift
│   │   ├── Extension+UIImage.swift
│   │   ├── Extension+UIImageView.swift
│   │   ├── Extension+UILabel.swift
│   │   ├── Extension+UINavigationItem.swift
│   │   ├── Extension+UITextField.swift
│   │   └── Extension+UIViewController.swift
│   ├── GoogleService-Info.plist
│   ├── IncrossOfficeCafe.entitlements
│   ├── Info.plist
│   ├── Model
│   │   ├── Banner.swift
│   │   ├── BusinessHours.swift
│   │   ├── Cart.swift
│   │   ├── Category.swift
│   │   ├── CellData.swift
│   │   ├── ConferenceRoom.swift
│   │   ├── DeleteCartId.swift
│   │   ├── Menu.swift
│   │   ├── MyOrder.swift
│   │   ├── Notice.swift
│   │   ├── NotificationCollection.swift
│   │   ├── Oauth.swift
│   │   ├── OauthVerify.swift
│   │   ├── OrderStatus.swift
│   │   ├── ServerResponse.swift
│   │   ├── UserInfo.swift
│   │   └── WaitingTime.swift
│   ├── Presentation
│   │   ├── Common
│   │   │   └── Utility
│   │   └── View
│   │       ├── Cart
│   │       │   ├── CartViewController.swift
│   │       │   ├── CartViewModel.swift
│   │       │   └── TableViewCell
│   │       │       └── CartTableViewCell.swift
│   │       ├── CustomView
│   │       │   ├── CountButtonView.swift
│   │       │   ├── HalfModalPresentationController.swift
│   │       │   ├── ModalViewController.swift
│   │       │   └── PeriodAlertViewController.swift
│   │       ├── Developer
│   │       │   └── DeveloperViewController.swift
│   │       ├── ExamViewController.swift
│   │       ├── Home
│   │       │   ├── CollectionViewCell
│   │       │   │   └── HomeCollectionViewCell.swift
│   │       │   ├── HomeViewController.swift
│   │       │   └── HomeViewModel.swift
│   │       ├── ItemDetail
│   │       │   ├── ItemDetailViewController.swift
│   │       │   ├── ItemDetailViewModel.swift
│   │       │   └── TableViewCell
│   │       │       ├── ItemDetailTableViewCell.swift
│   │       │       └── ItemDetailTitleTableViewCell.swift
│   │       ├── LaunchScreen
│   │       │   ├── LaunchScreenViewController.swift
│   │       │   └── LaunchScreenViewModel.swift
│   │       ├── Login
│   │       │   ├── LoginViewController.swift
│   │       │   └── LoginViewModel.swift
│   │       ├── Main
│   │       │   └── MainViewController.swift
│   │       ├── MyPage
│   │       │   ├── MyPageViewController.swift
│   │       │   ├── MyPageViewModel.swift
│   │       │   └── TableViewCell
│   │       │       └── PreferencesTableViewCell.swift
│   │       ├── Notice
│   │       │   ├── NoticeViewController.swift
│   │       │   ├── NoticeViewModel.swift
│   │       │   └── TableViewCell
│   │       │       ├── NoticeContentTableViewCell.swift
│   │       │       └── NoticeTitleTableViewCell.swift
│   │       ├── NotificationCenter
│   │       │   ├── NotificationCenterViewController.swift
│   │       │   ├── NotificationCenterViewModel.swift
│   │       │   └── TableViewCell
│   │       │       └── NotificationCenterTableViewCell.swift
│   │       ├── Order
│   │       │   ├── OrderViewController.swift
│   │       │   ├── OrderViewModel.swift
│   │       │   └── TableViewCell
│   │       │       └── OrderTableViewCell.swift
│   │       ├── OrderHistory
│   │       │   ├── OrderHistoryViewController.swift
│   │       │   ├── OrderHistoryViewModel.swift
│   │       │   └── TableViewCell
│   │       │       └── OrderHistoryTableViewCell.swift
│   │       ├── OrderHistoryDetail
│   │       │   ├── OrderHistoryDetailViewController.swift
│   │       │   └── TableViewCell
│   │       │       └── OrderHistoryDetailTableViewCell.swift
│   │       ├── OrderProcess
│   │       │   ├── OrderProcessingViewController.swift
│   │       │   └── OrderProcessingViewModel.swift
│   │       └── OrderResult
│   │           ├── OrderResultViewController.swift
│   │           └── OrderResultViewModel.swift
│   ├── Protocol
│   │   └── Protocol.swift
│   ├── Resource
│   │   └── Assets.xcassets
│   │       ├── AppIcon
│   │       │   ├── AppIcon Dev.appiconset
│   │       │   │   ├── 100.png
│   │       │   │   ├── 102.png
│   │       │   │   ├── 1024.png
│   │       │   │   ├── 114.png
│   │       │   │   ├── 120.png
│   │       │   │   ├── 128.png
│   │       │   │   ├── 144.png
│   │       │   │   ├── 152.png
│   │       │   │   ├── 16.png
│   │       │   │   ├── 167.png
│   │       │   │   ├── 172.png
│   │       │   │   ├── 180.png
│   │       │   │   ├── 196.png
│   │       │   │   ├── 20.png
│   │       │   │   ├── 216.png
│   │       │   │   ├── 256.png
│   │       │   │   ├── 29.png
│   │       │   │   ├── 32.png
│   │       │   │   ├── 40.png
│   │       │   │   ├── 48.png
│   │       │   │   ├── 50.png
│   │       │   │   ├── 512.png
│   │       │   │   ├── 55.png
│   │       │   │   ├── 57.png
│   │       │   │   ├── 58.png
│   │       │   │   ├── 60.png
│   │       │   │   ├── 64.png
│   │       │   │   ├── 66.png
│   │       │   │   ├── 72.png
│   │       │   │   ├── 76.png
│   │       │   │   ├── 80.png
│   │       │   │   ├── 87.png
│   │       │   │   ├── 88.png
│   │       │   │   ├── 92.png
│   │       │   │   └── Contents.json
│   │       │   ├── AppIcon Prod.appiconset
│   │       │   │   ├── 100.png
│   │       │   │   ├── 1024.png
│   │       │   │   ├── 114.png
│   │       │   │   ├── 120.png
│   │       │   │   ├── 128.png
│   │       │   │   ├── 144.png
│   │       │   │   ├── 152.png
│   │       │   │   ├── 16.png
│   │       │   │   ├── 167.png
│   │       │   │   ├── 172.png
│   │       │   │   ├── 180.png
│   │       │   │   ├── 196.png
│   │       │   │   ├── 20.png
│   │       │   │   ├── 216.png
│   │       │   │   ├── 256.png
│   │       │   │   ├── 29.png
│   │       │   │   ├── 32.png
│   │       │   │   ├── 40.png
│   │       │   │   ├── 48.png
│   │       │   │   ├── 50.png
│   │       │   │   ├── 512.png
│   │       │   │   ├── 55.png
│   │       │   │   ├── 57.png
│   │       │   │   ├── 58.png
│   │       │   │   ├── 60.png
│   │       │   │   ├── 64.png
│   │       │   │   ├── 66.png
│   │       │   │   ├── 72.png
│   │       │   │   ├── 76.png
│   │       │   │   ├── 80.png
│   │       │   │   ├── 87.png
│   │       │   │   ├── 88.png
│   │       │   │   ├── 92.png
│   │       │   │   ├── Contents.json
│   │       │   │   ├── Group 8781-2.png
│   │       │   │   ├── Group 8781-3.png
│   │       │   │   ├── Group 8781-4.png
│   │       │   │   └── Group 8781-5.png
│   │       │   ├── Contents.json
│   │       │   └── Monitoring Push.appiconset
│   │       │       ├── 100.png
│   │       │       ├── 102.png
│   │       │       ├── 1024.png
│   │       │       ├── 114.png
│   │       │       ├── 120.png
│   │       │       ├── 128.png
│   │       │       ├── 144.png
│   │       │       ├── 152.png
│   │       │       ├── 16.png
│   │       │       ├── 167.png
│   │       │       ├── 172.png
│   │       │       ├── 180.png
│   │       │       ├── 196.png
│   │       │       ├── 20.png
│   │       │       ├── 216.png
│   │       │       ├── 256.png
│   │       │       ├── 29.png
│   │       │       ├── 32.png
│   │       │       ├── 40.png
│   │       │       ├── 48.png
│   │       │       ├── 50.png
│   │       │       ├── 512.png
│   │       │       ├── 55.png
│   │       │       ├── 57.png
│   │       │       ├── 58.png
│   │       │       ├── 60.png
│   │       │       ├── 64.png
│   │       │       ├── 66.png
│   │       │       ├── 72.png
│   │       │       ├── 76.png
│   │       │       ├── 80.png
│   │       │       ├── 87.png
│   │       │       ├── 88.png
│   │       │       ├── 92.png
│   │       │       └── Contents.json
│   │       ├── Client
│   │       │   ├── Cart
│   │       │   │   ├── Cart0.imageset
│   │       │   │   │   ├── Basket_alt_duotone.png
│   │       │   │   │   ├── Basket_alt_duotone@2x.png
│   │       │   │   │   ├── Basket_alt_duotone@3x.png
│   │       │   │   │   └── Contents.json
│   │       │   │   ├── Cart1.imageset
│   │       │   │   │   ├── Contents.json
│   │       │   │   │   ├── Group 8734.png
│   │       │   │   │   ├── Group 8734@2x.png
│   │       │   │   │   └── Group 8734@3x.png
│   │       │   │   ├── Cart2.imageset
│   │       │   │   │   ├── Contents.json
│   │       │   │   │   ├── Group 8735.png
│   │       │   │   │   ├── Group 8735@2x.png
│   │       │   │   │   └── Group 8735@3x.png
│   │       │   │   ├── Cart3.imageset
│   │       │   │   │   ├── Contents.json
│   │       │   │   │   ├── Group 8736.png
│   │       │   │   │   ├── Group 8736@2x.png
│   │       │   │   │   └── Group 8736@3x.png
│   │       │   │   ├── Cart4.imageset
│   │       │   │   │   ├── Contents.json
│   │       │   │   │   ├── Group 8737.png
│   │       │   │   │   ├── Group 8737@2x.png
│   │       │   │   │   └── Group 8737@3x.png
│   │       │   │   ├── Cart5.imageset
│   │       │   │   │   ├── Contents.json
│   │       │   │   │   ├── Group 8738.png
│   │       │   │   │   ├── Group 8738@2x.png
│   │       │   │   │   └── Group 8738@3x.png
│   │       │   │   ├── Cart6.imageset
│   │       │   │   │   ├── Contents.json
│   │       │   │   │   ├── Group 8740.png
│   │       │   │   │   ├── Group 8740@2x.png
│   │       │   │   │   └── Group 8740@3x.png
│   │       │   │   ├── Cart7.imageset
│   │       │   │   │   ├── Contents.json
│   │       │   │   │   ├── Group 8741.png
│   │       │   │   │   ├── Group 8741@2x.png
│   │       │   │   │   └── Group 8741@3x.png
│   │       │   │   ├── Cart8.imageset
│   │       │   │   │   ├── Contents.json
│   │       │   │   │   ├── Group 8742.png
│   │       │   │   │   ├── Group 8742@2x.png
│   │       │   │   │   └── Group 8742@3x.png
│   │       │   │   ├── Cart9+.imageset
│   │       │   │   │   ├── Contents.json
│   │       │   │   │   ├── Group 8739.png
│   │       │   │   │   ├── Group 8739@2x.png
│   │       │   │   │   └── Group 8739@3x.png
│   │       │   │   ├── Cart9.imageset
│   │       │   │   │   ├── Contents.json
│   │       │   │   │   ├── Group 8743.png
│   │       │   │   │   ├── Group 8743@2x.png
│   │       │   │   │   └── Group 8743@3x.png
│   │       │   │   └── Contents.json
│   │       │   ├── Contents.json
│   │       │   ├── Empty Basket
│   │       │   │   ├── CartIcon.imageset
│   │       │   │   │   ├── CartIcon 1.png
│   │       │   │   │   ├── CartIcon 1@2x.png
│   │       │   │   │   ├── CartIcon 1@3x.png
│   │       │   │   │   └── Contents.json
│   │       │   │   ├── Contents.json
│   │       │   │   └── Empty_Basket.imageset
│   │       │   │       ├── Contents.json
│   │       │   │       ├── Empty Basket 1.png
│   │       │   │       ├── Empty Basket 1@2x.png
│   │       │   │       └── Empty Basket 1@3x.png
│   │       │   ├── NavigationBar Icon
│   │       │   │   ├── Bell
│   │       │   │   │   ├── Bell_fill.imageset
│   │       │   │   │   │   ├── Bell_fill.png
│   │       │   │   │   │   ├── Bell_fill@2x.png
│   │       │   │   │   │   ├── Bell_fill@3x.png
│   │       │   │   │   │   └── Contents.json
│   │       │   │   │   ├── Bell_pin_fill.imageset
│   │       │   │   │   │   ├── Bell_pin_fill.png
│   │       │   │   │   │   ├── Bell_pin_fill@2x.png
│   │       │   │   │   │   ├── Bell_pin_fill@3x.png
│   │       │   │   │   │   └── Contents.json
│   │       │   │   │   └── Contents.json
│   │       │   │   ├── Bell_duotone
│   │       │   │   │   ├── Bell_duotone_line.imageset
│   │       │   │   │   │   ├── Bell_duotone_line.png
│   │       │   │   │   │   ├── Bell_duotone_line@2x.png
│   │       │   │   │   │   ├── Bell_duotone_line@3x.png
│   │       │   │   │   │   └── Contents.json
│   │       │   │   │   ├── Bell_pin_duotone_line.imageset
│   │       │   │   │   │   ├── Bell_pin_duotone_line.png
│   │       │   │   │   │   ├── Bell_pin_duotone_line@2x.png
│   │       │   │   │   │   ├── Bell_pin_duotone_line@3x.png
│   │       │   │   │   │   └── Contents.json
│   │       │   │   │   └── Contents.json
│   │       │   │   ├── CoffeeIcon
│   │       │   │   │   ├── Coffee.imageset
│   │       │   │   │   │   ├── Contents.json
│   │       │   │   │   │   ├── coffee.png
│   │       │   │   │   │   ├── coffee@2x.png
│   │       │   │   │   │   └── coffee@3x.png
│   │       │   │   │   ├── Coffee_fill.imageset
│   │       │   │   │   │   ├── Contents.json
│   │       │   │   │   │   ├── coffee_fill.png
│   │       │   │   │   │   ├── coffee_fill@2x.png
│   │       │   │   │   │   └── coffee_fill@3x.png
│   │       │   │   │   └── Contents.json
│   │       │   │   ├── Contents.json
│   │       │   │   ├── HomeIcon
│   │       │   │   │   ├── Contents.json
│   │       │   │   │   ├── Home.imageset
│   │       │   │   │   │   ├── Contents.json
│   │       │   │   │   │   ├── solid-1.png
│   │       │   │   │   │   ├── solid@2x-1.png
│   │       │   │   │   │   └── solid@3x-1.png
│   │       │   │   │   └── Home_fill.imageset
│   │       │   │   │       ├── Contents.json
│   │       │   │   │       ├── solid.png
│   │       │   │   │       ├── solid@2x.png
│   │       │   │   │       └── solid@3x.png
│   │       │   │   └── MyPageIcon
│   │       │   │       ├── Contents.json
│   │       │   │       ├── MyPage
│   │       │   │       │   ├── Alarm.imageset
│   │       │   │       │   │   ├── Contents.json
│   │       │   │       │   │   ├── alarm 1.png
│   │       │   │       │   │   ├── alarm 2.png
│   │       │   │       │   │   └── alarm.png
│   │       │   │       │   ├── Contents.json
│   │       │   │       │   ├── Down.imageset
│   │       │   │       │   │   ├── Contents.json
│   │       │   │       │   │   ├── caret_2 1.png
│   │       │   │       │   │   ├── caret_2 2.png
│   │       │   │       │   │   └── caret_2.png
│   │       │   │       │   ├── FAQ.imageset
│   │       │   │       │   │   ├── Contents.json
│   │       │   │       │   │   ├── FAQ 1.png
│   │       │   │       │   │   ├── FAQ 2.png
│   │       │   │       │   │   └── FAQ.png
│   │       │   │       │   ├── Info.imageset
│   │       │   │       │   │   ├── Contents.json
│   │       │   │       │   │   ├── Info_fill.png
│   │       │   │       │   │   ├── Info_fill@2x.png
│   │       │   │       │   │   └── Info_fill@3x.png
│   │       │   │       │   ├── Notice.imageset
│   │       │   │       │   │   ├── Contents.json
│   │       │   │       │   │   ├── Notice.png
│   │       │   │       │   │   ├── Notice@2x.png
│   │       │   │       │   │   └── Notice@3x.png
│   │       │   │       │   ├── Paper.imageset
│   │       │   │       │   │   ├── Contents.json
│   │       │   │       │   │   ├── Paper_fill.png
│   │       │   │       │   │   ├── Paper_fill@2x.png
│   │       │   │       │   │   └── Paper_fill@3x.png
│   │       │   │       │   ├── RectangleStroke.imageset
│   │       │   │       │   │   ├── Contents.json
│   │       │   │       │   │   ├── RectangleStroke 1.png
│   │       │   │       │   │   ├── RectangleStroke 2.png
│   │       │   │       │   │   └── RectangleStroke.png
│   │       │   │       │   ├── Solid.imageset
│   │       │   │       │   │   ├── Contents.json
│   │       │   │       │   │   ├── solid 1.png
│   │       │   │       │   │   ├── solid 2.png
│   │       │   │       │   │   └── solid.png
│   │       │   │       │   ├── Up.imageset
│   │       │   │       │   │   ├── Contents.json
│   │       │   │       │   │   ├── caret_2-1 1.png
│   │       │   │       │   │   ├── caret_2-1 2.png
│   │       │   │       │   │   └── caret_2-1.png
│   │       │   │       │   ├── icon_atom.imageset
│   │       │   │       │   │   ├── Atom_light.png
│   │       │   │       │   │   ├── Atom_light@2x.png
│   │       │   │       │   │   ├── Atom_light@3x.png
│   │       │   │       │   │   └── Contents.json
│   │       │   │       │   ├── icon_cart.imageset
│   │       │   │       │   │   ├── Contents.json
│   │       │   │       │   │   ├── icon_cart.png
│   │       │   │       │   │   ├── icon_cart@2x.png
│   │       │   │       │   │   └── icon_cart@3x.png
│   │       │   │       │   ├── icon_notice.imageset
│   │       │   │       │   │   ├── Contents.json
│   │       │   │       │   │   ├── icon_notice.png
│   │       │   │       │   │   ├── icon_notice@2x.png
│   │       │   │       │   │   └── icon_notice@3x.png
│   │       │   │       │   ├── icon_order.imageset
│   │       │   │       │   │   ├── Contents.json
│   │       │   │       │   │   ├── icon_order.png
│   │       │   │       │   │   ├── icon_order@2x.png
│   │       │   │       │   │   └── icon_order@3x.png
│   │       │   │       │   └── icon_setting.imageset
│   │       │   │       │       ├── Contents.json
│   │       │   │       │       ├── icon_setting.png
│   │       │   │       │       ├── icon_setting@2x.png
│   │       │   │       │       └── icon_setting@3x.png
│   │       │   │       ├── Notice
│   │       │   │       │   ├── Contents.json
│   │       │   │       │   ├── Stackframe_duotone.imageset
│   │       │   │       │   │   ├── Contents.json
│   │       │   │       │   │   ├── Stackframe_duotone.png
│   │       │   │       │   │   ├── Stackframe_duotone@2x.png
│   │       │   │       │   │   └── Stackframe_duotone@3x.png
│   │       │   │       │   └── Stackframe_light.imageset
│   │       │   │       │       ├── Contents.json
│   │       │   │       │       ├── Stackframe_light.png
│   │       │   │       │       ├── Stackframe_light@2x.png
│   │       │   │       │       └── Stackframe_light@3x.png
│   │       │   │       ├── User_alt_fill
│   │       │   │       │   ├── Contents.json
│   │       │   │       │   └── User_alt_fill
│   │       │   │       │       ├── Contents.json
│   │       │   │       │       └── User_alt_fill.imageset
│   │       │   │       │           ├── Contents.json
│   │       │   │       │           ├── User_alt_fill.png
│   │       │   │       │           ├── User_alt_fill@2x.png
│   │       │   │       │           └── User_alt_fill@3x.png
│   │       │   │       ├── User_box_duotone_line.imageset
│   │       │   │       │   ├── Contents.json
│   │       │   │       │   ├── User_box_duotone_line.png
│   │       │   │       │   ├── User_box_duotone_line@2x.png
│   │       │   │       │   └── User_box_duotone_line@3x.png
│   │       │   │       ├── User_box_light.imageset
│   │       │   │       │   ├── Contents.json
│   │       │   │       │   ├── User_box_light.png
│   │       │   │       │   ├── User_box_light@2x.png
│   │       │   │       │   └── User_box_light@3x.png
│   │       │   │       └── phone_fill.imageset
│   │       │   │           ├── Contents.json
│   │       │   │           ├── phone_fill.png
│   │       │   │           ├── phone_fill@2x.png
│   │       │   │           └── phone_fill@3x.png
│   │       │   ├── Notification Center
│   │       │   │   ├── Cancel_duotone.imageset
│   │       │   │   │   ├── Cancel_duotone.png
│   │       │   │   │   ├── Cancel_duotone@2x.png
│   │       │   │   │   ├── Cancel_duotone@3x.png
│   │       │   │   │   └── Contents.json
│   │       │   │   ├── Contents.json
│   │       │   │   ├── Done_ring_round_duotone_line.imageset
│   │       │   │   │   ├── Contents.json
│   │       │   │   │   ├── Done_ring_round_duotone_line.png
│   │       │   │   │   ├── Done_ring_round_duotone_line@2x.png
│   │       │   │   │   └── Done_ring_round_duotone_line@3x.png
│   │       │   │   └── coffee_fill.imageset
│   │       │   │       ├── Contents.json
│   │       │   │       ├── coffee_fill.png
│   │       │   │       ├── coffee_fill@2x.png
│   │       │   │       └── coffee_fill@3x.png
│   │       │   ├── Order
│   │       │   │   ├── Contents.json
│   │       │   │   ├── Polygon Down.imageset
│   │       │   │   │   ├── Contents.json
│   │       │   │   │   ├── Polygon 1.png
│   │       │   │   │   ├── Polygon 1@2x.png
│   │       │   │   │   └── Polygon 1@3x.png
│   │       │   │   └── Polygon Up.imageset
│   │       │   │       ├── Contents.json
│   │       │   │       ├── Polygon 2.png
│   │       │   │       ├── Polygon 2@2x.png
│   │       │   │       └── Polygon 2@3x.png
│   │       │   └── Paper
│   │       │       ├── Contents.json
│   │       │       └── Paper_duotone_line.imageset
│   │       │           ├── Contents.json
│   │       │           ├── Paper_duotone_line.png
│   │       │           ├── Paper_duotone_line@2x.png
│   │       │           └── Paper_duotone_line@3x.png
│   │       ├── Contents.json
│   │       └── Logo
│   │           ├── AppLogo.imageset
│   │           │   ├── Contents.json
│   │           │   ├── incross_SK-square_CI_EN 1.png
│   │           │   ├── incross_SK-square_CI_EN 1@2x.png
│   │           │   └── incross_SK-square_CI_EN 1@3x.png
│   │           ├── Contents.json
│   │           ├── IncrossIcon
│   │           │   ├── Contents.json
│   │           │   └── IncrossIcon.imageset
│   │           │       ├── 1024 1.png
│   │           │       ├── 1024 1@2x.png
│   │           │       ├── 1024 1@3x.png
│   │           │       └── Contents.json
│   │           ├── LogoIcon.imageset
│   │           │   ├── Contents.json
│   │           │   └── image (1).png
│   │           ├── MindKnockAppLogo.imageset
│   │           │   ├── Contents.json
│   │           │   ├── Group 8780.png
│   │           │   ├── Group 8780@2x.png
│   │           │   └── Group 8780@3x.png
│   │           └── incrossAppLogo.imageset
│   │               ├── Contents.json
│   │               ├── Group 8779.png
│   │               ├── Group 8779@2x.png
│   │               └── Group 8779@3x.png
│   ├── Service
│   │   └── Model
│   ├── Storyboard.storyboard
│   └── Utility
│       ├── Analytics
│       │   └── AnalyticsHelper.swift
│       ├── DataObservable
│       │   └── DataObservable.swift
│       ├── Date
│       │   └── DateHelper.swift
│       ├── Device
│       │   └── DeviceInfo.swift
│       ├── Font
│       │   ├── FontManager.swift
│       │   ├── Pretendard-Black.otf
│       │   ├── Pretendard-Bold.otf
│       │   ├── Pretendard-ExtraBold.otf
│       │   ├── Pretendard-ExtraLight.otf
│       │   ├── Pretendard-Light.otf
│       │   ├── Pretendard-Medium.otf
│       │   ├── Pretendard-Regular.otf
│       │   ├── Pretendard-SemiBold.otf
│       │   └── Pretendard-Thin.otf
│       ├── Log
│       │   └── LogHelper.swift
│       ├── Toast
│       │   └── ToastHelper.swift
│       ├── Version
│       │   └── Version.swift
│       └── View
│           └── ViewHelper.swift
├── IncrossOfficeCafe.app.dSYM.zip
├── IncrossOfficeCafe.ipa
├── IncrossOfficeCafe.xcodeproj
│   ├── project.pbxproj
│   ├── project.xcworkspace
│   │   ├── contents.xcworkspacedata
│   │   ├── xcshareddata
│   │   │   ├── IDEWorkspaceChecks.plist
│   │   │   └── swiftpm
│   │   │       └── configuration
│   │   └── xcuserdata
│   │       └── incross0915.xcuserdatad
│   │           └── UserInterfaceState.xcuserstate
│   ├── xcshareddata
│   │   └── xcschemes
│   │       ├── IncrossOfficeCafe.xcscheme
│   │       └── NotificationServiceExtension.xcscheme
│   └── xcuserdata
│       └── incross0915.xcuserdatad
│           └── xcschemes
│               └── xcschememanagement.plist
├── IncrossOfficeCafe.xcworkspace
│   ├── contents.xcworkspacedata
│   ├── xcshareddata
│   │   ├── IDEWorkspaceChecks.plist
│   │   └── swiftpm
│   │       └── configuration
│   └── xcuserdata
│       └── incross0915.xcuserdatad
│           ├── IDEFindNavigatorScopes.plist
│           ├── UserInterfaceState.xcuserstate
│           └── xcdebugger
│               └── Breakpoints_v2.xcbkptlist
├── IncrossOfficeCafeTests
│   └── IncrossOfficeCafeTests.swift
├── IncrossOfficeCafeUITests
│   ├── IncrossOfficeCafeUITests.swift
│   └── IncrossOfficeCafeUITestsLaunchTests.swift
├── NotificationService
│   ├── Info.plist
│   └── NotificationService.swift
├── NotificationServiceExtension
│   ├── Info.plist
│   ├── Media.xcassets
│   │   ├── Contents.json
│   │   └── Siren.imageset
│   │       ├── Contents.json
│   │       ├── Group 8811.png
│   │       ├── Group 8811@2x.png
│   │       └── Group 8811@3x.png
│   └── NotificationService.swift
├── Podfile
├── Podfile.lock
├── Pods
│   ├── Alamofire
│   │   ├── LICENSE
│   │   ├── README.md
│   │   └── Source
│   │       ├── Alamofire.swift
│   │       ├── Core
│   │       │   ├── AFError.swift
│   │       │   ├── DataRequest.swift
│   │       │   ├── DataStreamRequest.swift
│   │       │   ├── DownloadRequest.swift
│   │       │   ├── HTTPHeaders.swift
│   │       │   ├── HTTPMethod.swift
│   │       │   ├── Notifications.swift
│   │       │   ├── ParameterEncoder.swift
│   │       │   ├── ParameterEncoding.swift
│   │       │   ├── Protected.swift
│   │       │   ├── Request.swift
│   │       │   ├── RequestTaskMap.swift
│   │       │   ├── Response.swift
│   │       │   ├── Session.swift
│   │       │   ├── SessionDelegate.swift
│   │       │   ├── URLConvertible+URLRequestConvertible.swift
│   │       │   ├── UploadRequest.swift
│   │       │   └── WebSocketRequest.swift
│   │       ├── Extensions
│   │       │   ├── DispatchQueue+Alamofire.swift
│   │       │   ├── OperationQueue+Alamofire.swift
│   │       │   ├── Result+Alamofire.swift
│   │       │   ├── StringEncoding+Alamofire.swift
│   │       │   ├── URLRequest+Alamofire.swift
│   │       │   └── URLSessionConfiguration+Alamofire.swift
│   │       ├── Features
│   │       │   ├── AlamofireExtended.swift
│   │       │   ├── AuthenticationInterceptor.swift
│   │       │   ├── CachedResponseHandler.swift
│   │       │   ├── Combine.swift
│   │       │   ├── Concurrency.swift
│   │       │   ├── EventMonitor.swift
│   │       │   ├── MultipartFormData.swift
│   │       │   ├── MultipartUpload.swift
│   │       │   ├── NetworkReachabilityManager.swift
│   │       │   ├── RedirectHandler.swift
│   │       │   ├── RequestCompression.swift
│   │       │   ├── RequestInterceptor.swift
│   │       │   ├── ResponseSerialization.swift
│   │       │   ├── RetryPolicy.swift
│   │       │   ├── ServerTrustEvaluation.swift
│   │       │   ├── URLEncodedFormEncoder.swift
│   │       │   └── Validation.swift
│   │       └── PrivacyInfo.xcprivacy
│   ├── Differentiator
│   │   ├── LICENSE.md
│   │   ├── README.md
│   │   └── Sources
│   │       └── Differentiator
│   │           ├── AnimatableSectionModel.swift
│   │           ├── AnimatableSectionModelType+ItemPath.swift
│   │           ├── AnimatableSectionModelType.swift
│   │           ├── Changeset.swift
│   │           ├── Diff.swift
│   │           ├── IdentifiableType.swift
│   │           ├── IdentifiableValue.swift
│   │           ├── ItemPath.swift
│   │           ├── Optional+Extensions.swift
│   │           ├── SectionModel.swift
│   │           ├── SectionModelType.swift
│   │           └── Utilities.swift
│   ├── Firebase
│   │   ├── CoreOnly
│   │   │   └── Sources
│   │   │       ├── Firebase.h
│   │   │       └── module.modulemap
│   │   ├── LICENSE
│   │   └── README.md
│   ├── FirebaseAnalytics
│   │   └── Frameworks
│   │       └── FirebaseAnalytics.xcframework
│   │           ├── Info.plist
│   │           ├── ios-arm64
│   │           │   └── FirebaseAnalytics.framework
│   │           │       ├── FirebaseAnalytics
│   │           │       ├── Headers
│   │           │       │   ├── FIRAnalytics+AppDelegate.h
│   │           │       │   ├── FIRAnalytics+Consent.h
│   │           │       │   ├── FIRAnalytics+OnDevice.h
│   │           │       │   ├── FIRAnalytics.h
│   │           │       │   ├── FIREventNames.h
│   │           │       │   ├── FIRParameterNames.h
│   │           │       │   ├── FIRUserPropertyNames.h
│   │           │       │   ├── FirebaseAnalytics-Swift.h
│   │           │       │   ├── FirebaseAnalytics-umbrella.h
│   │           │       │   └── FirebaseAnalytics.h
│   │           │       ├── Info.plist
│   │           │       └── Modules
│   │           │           ├── FirebaseAnalytics.swiftmodule
│   │           │           │   ├── Project
│   │           │           │   │   └── arm64-apple-ios.swiftsourceinfo
│   │           │           │   ├── arm64-apple-ios.abi.json
│   │           │           │   ├── arm64-apple-ios.private.swiftinterface
│   │           │           │   ├── arm64-apple-ios.swiftdoc
│   │           │           │   └── arm64-apple-ios.swiftinterface
│   │           │           └── module.modulemap
│   │           ├── ios-arm64_x86_64-maccatalyst
│   │           │   └── FirebaseAnalytics.framework
│   │           │       ├── FirebaseAnalytics
│   │           │       ├── Headers
│   │           │       │   ├── FIRAnalytics+AppDelegate.h
│   │           │       │   ├── FIRAnalytics+Consent.h
│   │           │       │   ├── FIRAnalytics+OnDevice.h
│   │           │       │   ├── FIRAnalytics.h
│   │           │       │   ├── FIREventNames.h
│   │           │       │   ├── FIRParameterNames.h
│   │           │       │   ├── FIRUserPropertyNames.h
│   │           │       │   ├── FirebaseAnalytics-Swift.h
│   │           │       │   ├── FirebaseAnalytics-umbrella.h
│   │           │       │   └── FirebaseAnalytics.h
│   │           │       ├── Info.plist
│   │           │       └── Modules
│   │           │           ├── FirebaseAnalytics.swiftmodule
│   │           │           │   ├── Project
│   │           │           │   │   ├── arm64-apple-ios-macabi.swiftsourceinfo
│   │           │           │   │   └── x86_64-apple-ios-macabi.swiftsourceinfo
│   │           │           │   ├── arm64-apple-ios-macabi.abi.json
│   │           │           │   ├── arm64-apple-ios-macabi.private.swiftinterface
│   │           │           │   ├── arm64-apple-ios-macabi.swiftdoc
│   │           │           │   ├── arm64-apple-ios-macabi.swiftinterface
│   │           │           │   ├── x86_64-apple-ios-macabi.abi.json
│   │           │           │   ├── x86_64-apple-ios-macabi.private.swiftinterface
│   │           │           │   ├── x86_64-apple-ios-macabi.swiftdoc
│   │           │           │   └── x86_64-apple-ios-macabi.swiftinterface
│   │           │           └── module.modulemap
│   │           ├── ios-arm64_x86_64-simulator
│   │           │   └── FirebaseAnalytics.framework
│   │           │       ├── FirebaseAnalytics
│   │           │       ├── Headers
│   │           │       │   ├── FIRAnalytics+AppDelegate.h
│   │           │       │   ├── FIRAnalytics+Consent.h
│   │           │       │   ├── FIRAnalytics+OnDevice.h
│   │           │       │   ├── FIRAnalytics.h
│   │           │       │   ├── FIREventNames.h
│   │           │       │   ├── FIRParameterNames.h
│   │           │       │   ├── FIRUserPropertyNames.h
│   │           │       │   ├── FirebaseAnalytics-Swift.h
│   │           │       │   ├── FirebaseAnalytics-umbrella.h
│   │           │       │   └── FirebaseAnalytics.h
│   │           │       ├── Info.plist
│   │           │       └── Modules
│   │           │           ├── FirebaseAnalytics.swiftmodule
│   │           │           │   ├── Project
│   │           │           │   │   ├── arm64-apple-ios-simulator.swiftsourceinfo
│   │           │           │   │   └── x86_64-apple-ios-simulator.swiftsourceinfo
│   │           │           │   ├── arm64-apple-ios-simulator.abi.json
│   │           │           │   ├── arm64-apple-ios-simulator.private.swiftinterface
│   │           │           │   ├── arm64-apple-ios-simulator.swiftdoc
│   │           │           │   ├── arm64-apple-ios-simulator.swiftinterface
│   │           │           │   ├── x86_64-apple-ios-simulator.abi.json
│   │           │           │   ├── x86_64-apple-ios-simulator.private.swiftinterface
│   │           │           │   ├── x86_64-apple-ios-simulator.swiftdoc
│   │           │           │   └── x86_64-apple-ios-simulator.swiftinterface
│   │           │           └── module.modulemap
│   │           ├── macos-arm64_x86_64
│   │           │   └── FirebaseAnalytics.framework
│   │           │       ├── FirebaseAnalytics
│   │           │       ├── Headers
│   │           │       │   ├── FIRAnalytics+AppDelegate.h
│   │           │       │   ├── FIRAnalytics+Consent.h
│   │           │       │   ├── FIRAnalytics+OnDevice.h
│   │           │       │   ├── FIRAnalytics.h
│   │           │       │   ├── FIREventNames.h
│   │           │       │   ├── FIRParameterNames.h
│   │           │       │   ├── FIRUserPropertyNames.h
│   │           │       │   ├── FirebaseAnalytics-Swift.h
│   │           │       │   ├── FirebaseAnalytics-umbrella.h
│   │           │       │   └── FirebaseAnalytics.h
│   │           │       ├── Info.plist
│   │           │       └── Modules
│   │           │           ├── FirebaseAnalytics.swiftmodule
│   │           │           │   ├── Project
│   │           │           │   │   ├── arm64-apple-macos.swiftsourceinfo
│   │           │           │   │   └── x86_64-apple-macos.swiftsourceinfo
│   │           │           │   ├── arm64-apple-macos.abi.json
│   │           │           │   ├── arm64-apple-macos.private.swiftinterface
│   │           │           │   ├── arm64-apple-macos.swiftdoc
│   │           │           │   ├── arm64-apple-macos.swiftinterface
│   │           │           │   ├── x86_64-apple-macos.abi.json
│   │           │           │   ├── x86_64-apple-macos.private.swiftinterface
│   │           │           │   ├── x86_64-apple-macos.swiftdoc
│   │           │           │   └── x86_64-apple-macos.swiftinterface
│   │           │           └── module.modulemap
│   │           ├── tvos-arm64
│   │           │   └── FirebaseAnalytics.framework
│   │           │       ├── FirebaseAnalytics
│   │           │       ├── Headers
│   │           │       │   ├── FIRAnalytics+AppDelegate.h
│   │           │       │   ├── FIRAnalytics+Consent.h
│   │           │       │   ├── FIRAnalytics+OnDevice.h
│   │           │       │   ├── FIRAnalytics.h
│   │           │       │   ├── FIREventNames.h
│   │           │       │   ├── FIRParameterNames.h
│   │           │       │   ├── FIRUserPropertyNames.h
│   │           │       │   ├── FirebaseAnalytics-Swift.h
│   │           │       │   ├── FirebaseAnalytics-umbrella.h
│   │           │       │   └── FirebaseAnalytics.h
│   │           │       ├── Info.plist
│   │           │       └── Modules
│   │           │           ├── FirebaseAnalytics.swiftmodule
│   │           │           │   ├── Project
│   │           │           │   │   └── arm64-apple-tvos.swiftsourceinfo
│   │           │           │   ├── arm64-apple-tvos.abi.json
│   │           │           │   ├── arm64-apple-tvos.private.swiftinterface
│   │           │           │   ├── arm64-apple-tvos.swiftdoc
│   │           │           │   └── arm64-apple-tvos.swiftinterface
│   │           │           └── module.modulemap
│   │           └── tvos-arm64_x86_64-simulator
│   │               └── FirebaseAnalytics.framework
│   │                   ├── FirebaseAnalytics
│   │                   ├── Headers
│   │                   │   ├── FIRAnalytics+AppDelegate.h
│   │                   │   ├── FIRAnalytics+Consent.h
│   │                   │   ├── FIRAnalytics+OnDevice.h
│   │                   │   ├── FIRAnalytics.h
│   │                   │   ├── FIREventNames.h
│   │                   │   ├── FIRParameterNames.h
│   │                   │   ├── FIRUserPropertyNames.h
│   │                   │   ├── FirebaseAnalytics-Swift.h
│   │                   │   ├── FirebaseAnalytics-umbrella.h
│   │                   │   └── FirebaseAnalytics.h
│   │                   ├── Info.plist
│   │                   └── Modules
│   │                       ├── FirebaseAnalytics.swiftmodule
│   │                       │   ├── Project
│   │                       │   │   ├── arm64-apple-tvos-simulator.swiftsourceinfo
│   │                       │   │   └── x86_64-apple-tvos-simulator.swiftsourceinfo
│   │                       │   ├── arm64-apple-tvos-simulator.abi.json
│   │                       │   ├── arm64-apple-tvos-simulator.private.swiftinterface
│   │                       │   ├── arm64-apple-tvos-simulator.swiftdoc
│   │                       │   ├── arm64-apple-tvos-simulator.swiftinterface
│   │                       │   ├── x86_64-apple-tvos-simulator.abi.json
│   │                       │   ├── x86_64-apple-tvos-simulator.private.swiftinterface
│   │                       │   ├── x86_64-apple-tvos-simulator.swiftdoc
│   │                       │   └── x86_64-apple-tvos-simulator.swiftinterface
│   │                       └── module.modulemap
│   ├── FirebaseCore
│   │   ├── FirebaseCore
│   │   │   ├── Extension
│   │   │   │   ├── FIRAppInternal.h
│   │   │   │   ├── FIRComponent.h
│   │   │   │   ├── FIRComponentContainer.h
│   │   │   │   ├── FIRComponentType.h
│   │   │   │   ├── FIRDependency.h
│   │   │   │   ├── FIRHeartbeatLogger.h
│   │   │   │   ├── FIRLibrary.h
│   │   │   │   ├── FIRLogger.h
│   │   │   │   ├── FIROptionsInternal.h
│   │   │   │   └── FirebaseCoreInternal.h
│   │   │   └── Sources
│   │   │       ├── FIRAnalyticsConfiguration.h
│   │   │       ├── FIRAnalyticsConfiguration.m
│   │   │       ├── FIRApp.m
│   │   │       ├── FIRBundleUtil.h
│   │   │       ├── FIRBundleUtil.m
│   │   │       ├── FIRComponent.m
│   │   │       ├── FIRComponentContainer.m
│   │   │       ├── FIRComponentContainerInternal.h
│   │   │       ├── FIRComponentType.m
│   │   │       ├── FIRConfiguration.m
│   │   │       ├── FIRConfigurationInternal.h
│   │   │       ├── FIRDependency.m
│   │   │       ├── FIRFirebaseUserAgent.h
│   │   │       ├── FIRFirebaseUserAgent.m
│   │   │       ├── FIRHeartbeatLogger.m
│   │   │       ├── FIRLogger.m
│   │   │       ├── FIROptions.m
│   │   │       ├── FIRVersion.m
│   │   │       └── Public
│   │   │           └── FirebaseCore
│   │   │               ├── FIRApp.h
│   │   │               ├── FIRConfiguration.h
│   │   │               ├── FIRLoggerLevel.h
│   │   │               ├── FIROptions.h
│   │   │               ├── FIRVersion.h
│   │   │               └── FirebaseCore.h
│   │   ├── LICENSE
│   │   └── README.md
│   ├── FirebaseCoreInternal
│   │   ├── FirebaseCore
│   │   │   └── Internal
│   │   │       └── Sources
│   │   │           ├── HeartbeatLogging
│   │   │           │   ├── Heartbeat.swift
│   │   │           │   ├── HeartbeatController.swift
│   │   │           │   ├── HeartbeatLoggingTestUtils.swift
│   │   │           │   ├── HeartbeatStorage.swift
│   │   │           │   ├── HeartbeatsBundle.swift
│   │   │           │   ├── HeartbeatsPayload.swift
│   │   │           │   ├── RingBuffer.swift
│   │   │           │   ├── Storage.swift
│   │   │           │   ├── StorageFactory.swift
│   │   │           │   ├── WeakContainer.swift
│   │   │           │   ├── _ObjC_HeartbeatController.swift
│   │   │           │   └── _ObjC_HeartbeatsPayload.swift
│   │   │           └── Resources
│   │   │               └── PrivacyInfo.xcprivacy
│   │   ├── LICENSE
│   │   └── README.md
│   ├── FirebaseInstallations
│   │   ├── FirebaseCore
│   │   │   └── Extension
│   │   │       ├── FIRAppInternal.h
│   │   │       ├── FIRComponent.h
│   │   │       ├── FIRComponentContainer.h
│   │   │       ├── FIRComponentType.h
│   │   │       ├── FIRDependency.h
│   │   │       ├── FIRHeartbeatLogger.h
│   │   │       ├── FIRLibrary.h
│   │   │       ├── FIRLogger.h
│   │   │       ├── FIROptionsInternal.h
│   │   │       └── FirebaseCoreInternal.h
│   │   ├── FirebaseInstallations
│   │   │   └── Source
│   │   │       └── Library
│   │   │           ├── Errors
│   │   │           │   ├── FIRInstallationsErrorUtil.h
│   │   │           │   ├── FIRInstallationsErrorUtil.m
│   │   │           │   ├── FIRInstallationsHTTPError.h
│   │   │           │   └── FIRInstallationsHTTPError.m
│   │   │           ├── FIRInstallations.m
│   │   │           ├── FIRInstallationsAuthTokenResult.m
│   │   │           ├── FIRInstallationsAuthTokenResultInternal.h
│   │   │           ├── FIRInstallationsItem.h
│   │   │           ├── FIRInstallationsItem.m
│   │   │           ├── FIRInstallationsLogger.h
│   │   │           ├── FIRInstallationsLogger.m
│   │   │           ├── IIDMigration
│   │   │           │   ├── FIRInstallationsIIDStore.h
│   │   │           │   ├── FIRInstallationsIIDStore.m
│   │   │           │   ├── FIRInstallationsIIDTokenStore.h
│   │   │           │   └── FIRInstallationsIIDTokenStore.m
│   │   │           ├── InstallationsAPI
│   │   │           │   ├── FIRInstallationsAPIService.h
│   │   │           │   ├── FIRInstallationsAPIService.m
│   │   │           │   ├── FIRInstallationsItem+RegisterInstallationAPI.h
│   │   │           │   └── FIRInstallationsItem+RegisterInstallationAPI.m
│   │   │           ├── InstallationsIDController
│   │   │           │   ├── FIRCurrentDateProvider.h
│   │   │           │   ├── FIRCurrentDateProvider.m
│   │   │           │   ├── FIRInstallationsBackoffController.h
│   │   │           │   ├── FIRInstallationsBackoffController.m
│   │   │           │   ├── FIRInstallationsIDController.h
│   │   │           │   ├── FIRInstallationsIDController.m
│   │   │           │   ├── FIRInstallationsSingleOperationPromiseCache.h
│   │   │           │   ├── FIRInstallationsSingleOperationPromiseCache.m
│   │   │           │   └── FIRInstallationsStatus.h
│   │   │           ├── InstallationsStore
│   │   │           │   ├── FIRInstallationsStore.h
│   │   │           │   ├── FIRInstallationsStore.m
│   │   │           │   ├── FIRInstallationsStoredAuthToken.h
│   │   │           │   ├── FIRInstallationsStoredAuthToken.m
│   │   │           │   ├── FIRInstallationsStoredItem.h
│   │   │           │   └── FIRInstallationsStoredItem.m
│   │   │           ├── Private
│   │   │           │   └── FirebaseInstallationsInternal.h
│   │   │           ├── Public
│   │   │           │   └── FirebaseInstallations
│   │   │           │       ├── FIRInstallations.h
│   │   │           │       ├── FIRInstallationsAuthTokenResult.h
│   │   │           │       ├── FIRInstallationsErrors.h
│   │   │           │       └── FirebaseInstallations.h
│   │   │           └── Resources
│   │   │               └── PrivacyInfo.xcprivacy
│   │   ├── LICENSE
│   │   └── README.md
│   ├── FirebaseMessaging
│   │   ├── FirebaseCore
│   │   │   └── Extension
│   │   │       ├── FIRAppInternal.h
│   │   │       ├── FIRComponent.h
│   │   │       ├── FIRComponentContainer.h
│   │   │       ├── FIRComponentType.h
│   │   │       ├── FIRDependency.h
│   │   │       ├── FIRHeartbeatLogger.h
│   │   │       ├── FIRLibrary.h
│   │   │       ├── FIRLogger.h
│   │   │       ├── FIROptionsInternal.h
│   │   │       └── FirebaseCoreInternal.h
│   │   ├── FirebaseInstallations
│   │   │   └── Source
│   │   │       └── Library
│   │   │           └── Private
│   │   │               └── FirebaseInstallationsInternal.h
│   │   ├── FirebaseMessaging
│   │   │   ├── Interop
│   │   │   │   └── FIRMessagingInterop.h
│   │   │   └── Sources
│   │   │       ├── FIRMessaging.m
│   │   │       ├── FIRMessagingAnalytics.h
│   │   │       ├── FIRMessagingAnalytics.m
│   │   │       ├── FIRMessagingCode.h
│   │   │       ├── FIRMessagingConstants.h
│   │   │       ├── FIRMessagingConstants.m
│   │   │       ├── FIRMessagingContextManagerService.h
│   │   │       ├── FIRMessagingContextManagerService.m
│   │   │       ├── FIRMessagingDefines.h
│   │   │       ├── FIRMessagingExtensionHelper.m
│   │   │       ├── FIRMessagingLogger.h
│   │   │       ├── FIRMessagingLogger.m
│   │   │       ├── FIRMessagingPendingTopicsList.h
│   │   │       ├── FIRMessagingPendingTopicsList.m
│   │   │       ├── FIRMessagingPersistentSyncMessage.h
│   │   │       ├── FIRMessagingPersistentSyncMessage.m
│   │   │       ├── FIRMessagingPubSub.h
│   │   │       ├── FIRMessagingPubSub.m
│   │   │       ├── FIRMessagingRemoteNotificationsProxy.h
│   │   │       ├── FIRMessagingRemoteNotificationsProxy.m
│   │   │       ├── FIRMessagingRmqManager.h
│   │   │       ├── FIRMessagingRmqManager.m
│   │   │       ├── FIRMessagingSyncMessageManager.h
│   │   │       ├── FIRMessagingSyncMessageManager.m
│   │   │       ├── FIRMessagingTopicOperation.h
│   │   │       ├── FIRMessagingTopicOperation.m
│   │   │       ├── FIRMessagingTopicsCommon.h
│   │   │       ├── FIRMessagingUtilities.h
│   │   │       ├── FIRMessagingUtilities.m
│   │   │       ├── FIRMessaging_Private.h
│   │   │       ├── FirebaseMessaging.h
│   │   │       ├── NSDictionary+FIRMessaging.h
│   │   │       ├── NSDictionary+FIRMessaging.m
│   │   │       ├── NSError+FIRMessaging.h
│   │   │       ├── NSError+FIRMessaging.m
│   │   │       ├── Protogen
│   │   │       │   └── nanopb
│   │   │       │       ├── me.nanopb.c
│   │   │       │       └── me.nanopb.h
│   │   │       ├── Public
│   │   │       │   └── FirebaseMessaging
│   │   │       │       ├── FIRMessaging.h
│   │   │       │       ├── FIRMessagingExtensionHelper.h
│   │   │       │       └── FirebaseMessaging.h
│   │   │       └── Token
│   │   │           ├── FIRMessagingAPNSInfo.h
│   │   │           ├── FIRMessagingAPNSInfo.m
│   │   │           ├── FIRMessagingAuthKeychain.h
│   │   │           ├── FIRMessagingAuthKeychain.m
│   │   │           ├── FIRMessagingAuthService.h
│   │   │           ├── FIRMessagingAuthService.m
│   │   │           ├── FIRMessagingBackupExcludedPlist.h
│   │   │           ├── FIRMessagingBackupExcludedPlist.m
│   │   │           ├── FIRMessagingCheckinPreferences.h
│   │   │           ├── FIRMessagingCheckinPreferences.m
│   │   │           ├── FIRMessagingCheckinService.h
│   │   │           ├── FIRMessagingCheckinService.m
│   │   │           ├── FIRMessagingCheckinStore.h
│   │   │           ├── FIRMessagingCheckinStore.m
│   │   │           ├── FIRMessagingKeychain.h
│   │   │           ├── FIRMessagingKeychain.m
│   │   │           ├── FIRMessagingTokenDeleteOperation.h
│   │   │           ├── FIRMessagingTokenDeleteOperation.m
│   │   │           ├── FIRMessagingTokenFetchOperation.h
│   │   │           ├── FIRMessagingTokenFetchOperation.m
│   │   │           ├── FIRMessagingTokenInfo.h
│   │   │           ├── FIRMessagingTokenInfo.m
│   │   │           ├── FIRMessagingTokenManager.h
│   │   │           ├── FIRMessagingTokenManager.m
│   │   │           ├── FIRMessagingTokenOperation.h
│   │   │           ├── FIRMessagingTokenOperation.m
│   │   │           ├── FIRMessagingTokenStore.h
│   │   │           └── FIRMessagingTokenStore.m
│   │   ├── Interop
│   │   │   └── Analytics
│   │   │       └── Public
│   │   │           ├── FIRAnalyticsInterop.h
│   │   │           ├── FIRAnalyticsInteropListener.h
│   │   │           ├── FIRInteropEventNames.h
│   │   │           └── FIRInteropParameterNames.h
│   │   ├── LICENSE
│   │   └── README.md
│   ├── GoogleAnalytics
│   │   └── Frameworks
│   │       └── GoogleAnalytics.xcframework
│   │           ├── Info.plist
│   │           ├── ios-arm64
│   │           │   └── GoogleAnalytics.framework
│   │           │       ├── GoogleAnalytics
│   │           │       ├── Headers
│   │           │       │   ├── GAI.h
│   │           │       │   ├── GAIDictionaryBuilder.h
│   │           │       │   ├── GAIEcommerceFields.h
│   │           │       │   ├── GAIEcommerceProduct.h
│   │           │       │   ├── GAIEcommerceProductAction.h
│   │           │       │   ├── GAIEcommercePromotion.h
│   │           │       │   ├── GAIFields.h
│   │           │       │   ├── GAILogger.h
│   │           │       │   ├── GAITrackedViewController.h
│   │           │       │   ├── GAITracker.h
│   │           │       │   └── GoogleAnalytics-umbrella.h
│   │           │       ├── Info.plist
│   │           │       └── Modules
│   │           │           └── module.modulemap
│   │           ├── ios-arm64_x86_64-maccatalyst
│   │           │   └── GoogleAnalytics.framework
│   │           │       ├── GoogleAnalytics
│   │           │       ├── Headers
│   │           │       │   ├── GAI.h
│   │           │       │   ├── GAIDictionaryBuilder.h
│   │           │       │   ├── GAIEcommerceFields.h
│   │           │       │   ├── GAIEcommerceProduct.h
│   │           │       │   ├── GAIEcommerceProductAction.h
│   │           │       │   ├── GAIEcommercePromotion.h
│   │           │       │   ├── GAIFields.h
│   │           │       │   ├── GAILogger.h
│   │           │       │   ├── GAITrackedViewController.h
│   │           │       │   ├── GAITracker.h
│   │           │       │   └── GoogleAnalytics-umbrella.h
│   │           │       ├── Info.plist
│   │           │       └── Modules
│   │           │           └── module.modulemap
│   │           └── ios-arm64_x86_64-simulator
│   │               └── GoogleAnalytics.framework
│   │                   ├── GoogleAnalytics
│   │                   ├── Headers
│   │                   │   ├── GAI.h
│   │                   │   ├── GAIDictionaryBuilder.h
│   │                   │   ├── GAIEcommerceFields.h
│   │                   │   ├── GAIEcommerceProduct.h
│   │                   │   ├── GAIEcommerceProductAction.h
│   │                   │   ├── GAIEcommercePromotion.h
│   │                   │   ├── GAIFields.h
│   │                   │   ├── GAILogger.h
│   │                   │   ├── GAITrackedViewController.h
│   │                   │   ├── GAITracker.h
│   │                   │   └── GoogleAnalytics-umbrella.h
│   │                   ├── Info.plist
│   │                   └── Modules
│   │                       └── module.modulemap
│   ├── GoogleAppMeasurement
│   │   └── Frameworks
│   │       ├── GoogleAppMeasurement.xcframework
│   │       │   ├── Info.plist
│   │       │   ├── ios-arm64
│   │       │   │   └── GoogleAppMeasurement.framework
│   │       │   │       ├── GoogleAppMeasurement
│   │       │   │       ├── Info.plist
│   │       │   │       └── Modules
│   │       │   │           └── module.modulemap
│   │       │   ├── ios-arm64_x86_64-maccatalyst
│   │       │   │   └── GoogleAppMeasurement.framework
│   │       │   │       ├── GoogleAppMeasurement
│   │       │   │       ├── Info.plist
│   │       │   │       └── Modules
│   │       │   │           └── module.modulemap
│   │       │   ├── ios-arm64_x86_64-simulator
│   │       │   │   └── GoogleAppMeasurement.framework
│   │       │   │       ├── GoogleAppMeasurement
│   │       │   │       ├── Info.plist
│   │       │   │       └── Modules
│   │       │   │           └── module.modulemap
│   │       │   ├── macos-arm64_x86_64
│   │       │   │   └── GoogleAppMeasurement.framework
│   │       │   │       ├── GoogleAppMeasurement
│   │       │   │       ├── Info.plist
│   │       │   │       └── Modules
│   │       │   │           └── module.modulemap
│   │       │   ├── tvos-arm64
│   │       │   │   └── GoogleAppMeasurement.framework
│   │       │   │       ├── GoogleAppMeasurement
│   │       │   │       ├── Info.plist
│   │       │   │       └── Modules
│   │       │   │           └── module.modulemap
│   │       │   └── tvos-arm64_x86_64-simulator
│   │       │       └── GoogleAppMeasurement.framework
│   │       │           ├── GoogleAppMeasurement
│   │       │           ├── Info.plist
│   │       │           └── Modules
│   │       │               └── module.modulemap
│   │       └── GoogleAppMeasurementIdentitySupport.xcframework
│   │           ├── Info.plist
│   │           ├── ios-arm64
│   │           │   └── GoogleAppMeasurementIdentitySupport.framework
│   │           │       ├── GoogleAppMeasurementIdentitySupport
│   │           │       ├── Info.plist
│   │           │       └── Modules
│   │           │           └── module.modulemap
│   │           ├── ios-arm64_x86_64-maccatalyst
│   │           │   └── GoogleAppMeasurementIdentitySupport.framework
│   │           │       ├── GoogleAppMeasurementIdentitySupport
│   │           │       ├── Info.plist
│   │           │       └── Modules
│   │           │           └── module.modulemap
│   │           ├── ios-arm64_x86_64-simulator
│   │           │   └── GoogleAppMeasurementIdentitySupport.framework
│   │           │       ├── GoogleAppMeasurementIdentitySupport
│   │           │       ├── Info.plist
│   │           │       └── Modules
│   │           │           └── module.modulemap
│   │           ├── macos-arm64_x86_64
│   │           │   └── GoogleAppMeasurementIdentitySupport.framework
│   │           │       ├── GoogleAppMeasurementIdentitySupport
│   │           │       ├── Info.plist
│   │           │       └── Modules
│   │           │           └── module.modulemap
│   │           ├── tvos-arm64
│   │           │   └── GoogleAppMeasurementIdentitySupport.framework
│   │           │       ├── GoogleAppMeasurementIdentitySupport
│   │           │       ├── Info.plist
│   │           │       └── Modules
│   │           │           └── module.modulemap
│   │           └── tvos-arm64_x86_64-simulator
│   │               └── GoogleAppMeasurementIdentitySupport.framework
│   │                   ├── GoogleAppMeasurementIdentitySupport
│   │                   ├── Info.plist
│   │                   └── Modules
│   │                       └── module.modulemap
│   ├── GoogleDataTransport
│   │   ├── GoogleDataTransport
│   │   │   ├── GDTCCTLibrary
│   │   │   │   ├── GDTCCTCompressionHelper.m
│   │   │   │   ├── GDTCCTNanopbHelpers.m
│   │   │   │   ├── GDTCCTUploadOperation.m
│   │   │   │   ├── GDTCCTUploader.m
│   │   │   │   ├── GDTCOREvent+GDTCCTSupport.m
│   │   │   │   ├── GDTCOREvent+GDTMetricsSupport.m
│   │   │   │   ├── GDTCORMetrics+GDTCCTSupport.m
│   │   │   │   ├── Private
│   │   │   │   │   ├── GDTCCTCompressionHelper.h
│   │   │   │   │   ├── GDTCCTNanopbHelpers.h
│   │   │   │   │   ├── GDTCCTUploadOperation.h
│   │   │   │   │   ├── GDTCCTUploader.h
│   │   │   │   │   ├── GDTCOREvent+GDTMetricsSupport.h
│   │   │   │   │   └── GDTCORMetrics+GDTCCTSupport.h
│   │   │   │   ├── Protogen
│   │   │   │   │   └── nanopb
│   │   │   │   │       ├── cct.nanopb.c
│   │   │   │   │       ├── cct.nanopb.h
│   │   │   │   │       ├── client_metrics.nanopb.c
│   │   │   │   │       ├── client_metrics.nanopb.h
│   │   │   │   │       ├── compliance.nanopb.c
│   │   │   │   │       ├── compliance.nanopb.h
│   │   │   │   │       ├── external_prequest_context.nanopb.c
│   │   │   │   │       ├── external_prequest_context.nanopb.h
│   │   │   │   │       ├── external_privacy_context.nanopb.c
│   │   │   │   │       └── external_privacy_context.nanopb.h
│   │   │   │   └── Public
│   │   │   │       └── GDTCOREvent+GDTCCTSupport.h
│   │   │   ├── GDTCORLibrary
│   │   │   │   ├── GDTCORAssert.m
│   │   │   │   ├── GDTCORClock.m
│   │   │   │   ├── GDTCORConsoleLogger.m
│   │   │   │   ├── GDTCORDirectorySizeTracker.m
│   │   │   │   ├── GDTCOREndpoints.m
│   │   │   │   ├── GDTCOREvent.m
│   │   │   │   ├── GDTCORFlatFileStorage+Promises.m
│   │   │   │   ├── GDTCORFlatFileStorage.m
│   │   │   │   ├── GDTCORLifecycle.m
│   │   │   │   ├── GDTCORLogSourceMetrics.m
│   │   │   │   ├── GDTCORMetrics.m
│   │   │   │   ├── GDTCORMetricsController.m
│   │   │   │   ├── GDTCORMetricsMetadata.m
│   │   │   │   ├── GDTCORPlatform.m
│   │   │   │   ├── GDTCORProductData.m
│   │   │   │   ├── GDTCORReachability.m
│   │   │   │   ├── GDTCORRegistrar.m
│   │   │   │   ├── GDTCORStorageEventSelector.m
│   │   │   │   ├── GDTCORStorageMetadata.m
│   │   │   │   ├── GDTCORTransformer.m
│   │   │   │   ├── GDTCORTransport.m
│   │   │   │   ├── GDTCORUploadBatch.m
│   │   │   │   ├── GDTCORUploadCoordinator.m
│   │   │   │   ├── Internal
│   │   │   │   │   ├── GDTCORAssert.h
│   │   │   │   │   ├── GDTCORDirectorySizeTracker.h
│   │   │   │   │   ├── GDTCOREventDropReason.h
│   │   │   │   │   ├── GDTCORLifecycle.h
│   │   │   │   │   ├── GDTCORMetricsControllerProtocol.h
│   │   │   │   │   ├── GDTCORPlatform.h
│   │   │   │   │   ├── GDTCORReachability.h
│   │   │   │   │   ├── GDTCORRegistrar.h
│   │   │   │   │   ├── GDTCORStorageEventSelector.h
│   │   │   │   │   ├── GDTCORStorageProtocol.h
│   │   │   │   │   ├── GDTCORStorageSizeBytes.h
│   │   │   │   │   └── GDTCORUploader.h
│   │   │   │   ├── Private
│   │   │   │   │   ├── GDTCOREndpoints_Private.h
│   │   │   │   │   ├── GDTCOREvent_Private.h
│   │   │   │   │   ├── GDTCORFlatFileStorage+Promises.h
│   │   │   │   │   ├── GDTCORFlatFileStorage.h
│   │   │   │   │   ├── GDTCORLogSourceMetrics.h
│   │   │   │   │   ├── GDTCORMetrics.h
│   │   │   │   │   ├── GDTCORMetricsController.h
│   │   │   │   │   ├── GDTCORMetricsMetadata.h
│   │   │   │   │   ├── GDTCORReachability_Private.h
│   │   │   │   │   ├── GDTCORRegistrar_Private.h
│   │   │   │   │   ├── GDTCORStorageMetadata.h
│   │   │   │   │   ├── GDTCORTransformer.h
│   │   │   │   │   ├── GDTCORTransformer_Private.h
│   │   │   │   │   ├── GDTCORTransport_Private.h
│   │   │   │   │   ├── GDTCORUploadBatch.h
│   │   │   │   │   └── GDTCORUploadCoordinator.h
│   │   │   │   └── Public
│   │   │   │       └── GoogleDataTransport
│   │   │   │           ├── GDTCORClock.h
│   │   │   │           ├── GDTCORConsoleLogger.h
│   │   │   │           ├── GDTCOREndpoints.h
│   │   │   │           ├── GDTCOREvent.h
│   │   │   │           ├── GDTCOREventDataObject.h
│   │   │   │           ├── GDTCOREventTransformer.h
│   │   │   │           ├── GDTCORProductData.h
│   │   │   │           ├── GDTCORTargets.h
│   │   │   │           ├── GDTCORTransport.h
│   │   │   │           └── GoogleDataTransport.h
│   │   │   └── Resources
│   │   │       └── PrivacyInfo.xcprivacy
│   │   ├── LICENSE
│   │   └── README.md
│   ├── GoogleTagManager
│   │   └── Frameworks
│   │       └── GoogleTagManager.xcframework
│   │           ├── Info.plist
│   │           ├── ios-arm64
│   │           │   └── GoogleTagManager.framework
│   │           │       ├── GoogleTagManager
│   │           │       ├── Headers
│   │           │       │   ├── GoogleTagManager-umbrella.h
│   │           │       │   └── TAGCustomFunction.h
│   │           │       ├── Info.plist
│   │           │       └── Modules
│   │           │           └── module.modulemap
│   │           ├── ios-arm64_x86_64-maccatalyst
│   │           │   └── GoogleTagManager.framework
│   │           │       ├── GoogleTagManager
│   │           │       ├── Headers
│   │           │       │   ├── GoogleTagManager-umbrella.h
│   │           │       │   └── TAGCustomFunction.h
│   │           │       ├── Info.plist
│   │           │       └── Modules
│   │           │           └── module.modulemap
│   │           └── ios-arm64_x86_64-simulator
│   │               └── GoogleTagManager.framework
│   │                   ├── GoogleTagManager
│   │                   ├── Headers
│   │                   │   ├── GoogleTagManager-umbrella.h
│   │                   │   └── TAGCustomFunction.h
│   │                   ├── Info.plist
│   │                   └── Modules
│   │                       └── module.modulemap
│   ├── GoogleUtilities
│   │   ├── GoogleUtilities
│   │   │   ├── AppDelegateSwizzler
│   │   │   │   ├── GULAppDelegateSwizzler.m
│   │   │   │   ├── GULSceneDelegateSwizzler.m
│   │   │   │   ├── Internal
│   │   │   │   │   ├── GULAppDelegateSwizzler_Private.h
│   │   │   │   │   └── GULSceneDelegateSwizzler_Private.h
│   │   │   │   └── Public
│   │   │   │       └── GoogleUtilities
│   │   │   │           ├── GULAppDelegateSwizzler.h
│   │   │   │           ├── GULApplication.h
│   │   │   │           └── GULSceneDelegateSwizzler.h
│   │   │   ├── Common
│   │   │   │   └── GULLoggerCodes.h
│   │   │   ├── Environment
│   │   │   │   ├── GULAppEnvironmentUtil.m
│   │   │   │   ├── GULHeartbeatDateStorage.m
│   │   │   │   ├── GULHeartbeatDateStorageUserDefaults.m
│   │   │   │   ├── GULSecureCoding.m
│   │   │   │   ├── NetworkInfo
│   │   │   │   │   └── GULNetworkInfo.m
│   │   │   │   ├── Public
│   │   │   │   │   └── GoogleUtilities
│   │   │   │   │       ├── GULAppEnvironmentUtil.h
│   │   │   │   │       ├── GULHeartbeatDateStorable.h
│   │   │   │   │       ├── GULHeartbeatDateStorage.h
│   │   │   │   │       ├── GULHeartbeatDateStorageUserDefaults.h
│   │   │   │   │       ├── GULKeychainStorage.h
│   │   │   │   │       ├── GULKeychainUtils.h
│   │   │   │   │       ├── GULNetworkInfo.h
│   │   │   │   │       ├── GULSecureCoding.h
│   │   │   │   │       ├── GULURLSessionDataResponse.h
│   │   │   │   │       └── NSURLSession+GULPromises.h
│   │   │   │   ├── SecureStorage
│   │   │   │   │   ├── GULKeychainStorage.m
│   │   │   │   │   └── GULKeychainUtils.m
│   │   │   │   └── URLSessionPromiseWrapper
│   │   │   │       ├── GULURLSessionDataResponse.m
│   │   │   │       └── NSURLSession+GULPromises.m
│   │   │   ├── Logger
│   │   │   │   ├── GULLogger.m
│   │   │   │   └── Public
│   │   │   │       └── GoogleUtilities
│   │   │   │           ├── GULLogger.h
│   │   │   │           └── GULLoggerLevel.h
│   │   │   ├── MethodSwizzler
│   │   │   │   ├── GULSwizzler.m
│   │   │   │   └── Public
│   │   │   │       └── GoogleUtilities
│   │   │   │           ├── GULOriginalIMPConvenienceMacros.h
│   │   │   │           └── GULSwizzler.h
│   │   │   ├── NSData+zlib
│   │   │   │   ├── GULNSData+zlib.m
│   │   │   │   └── Public
│   │   │   │       └── GoogleUtilities
│   │   │   │           └── GULNSData+zlib.h
│   │   │   ├── Network
│   │   │   │   ├── GULMutableDictionary.m
│   │   │   │   ├── GULNetwork.m
│   │   │   │   ├── GULNetworkConstants.m
│   │   │   │   ├── GULNetworkInternal.h
│   │   │   │   ├── GULNetworkURLSession.m
│   │   │   │   └── Public
│   │   │   │       └── GoogleUtilities
│   │   │   │           ├── GULMutableDictionary.h
│   │   │   │           ├── GULNetwork.h
│   │   │   │           ├── GULNetworkConstants.h
│   │   │   │           ├── GULNetworkLoggerProtocol.h
│   │   │   │           ├── GULNetworkMessageCode.h
│   │   │   │           └── GULNetworkURLSession.h
│   │   │   ├── Privacy
│   │   │   │   └── Resources
│   │   │   │       └── PrivacyInfo.xcprivacy
│   │   │   ├── Reachability
│   │   │   │   ├── GULReachabilityChecker+Internal.h
│   │   │   │   ├── GULReachabilityChecker.m
│   │   │   │   ├── GULReachabilityMessageCode.h
│   │   │   │   └── Public
│   │   │   │       └── GoogleUtilities
│   │   │   │           └── GULReachabilityChecker.h
│   │   │   └── UserDefaults
│   │   │       ├── GULUserDefaults.m
│   │   │       └── Public
│   │   │           └── GoogleUtilities
│   │   │               └── GULUserDefaults.h
│   │   ├── LICENSE
│   │   ├── README.md
│   │   └── third_party
│   │       └── IsAppEncrypted
│   │           ├── IsAppEncrypted.m
│   │           └── Public
│   │               └── IsAppEncrypted.h
│   ├── Headers
│   │   ├── Private
│   │   │   └── Firebase
│   │   │       └── Firebase.h -> ../../../Firebase/CoreOnly/Sources/Firebase.h
│   │   └── Public
│   │       └── Firebase
│   │           └── Firebase.h -> ../../../Firebase/CoreOnly/Sources/Firebase.h
│   ├── Kingfisher
│   │   ├── LICENSE
│   │   ├── README.md
│   │   └── Sources
│   │       ├── Cache
│   │       │   ├── CacheSerializer.swift
│   │       │   ├── DiskStorage.swift
│   │       │   ├── FormatIndicatedCacheSerializer.swift
│   │       │   ├── ImageCache.swift
│   │       │   ├── MemoryStorage.swift
│   │       │   └── Storage.swift
│   │       ├── Extensions
│   │       │   ├── ImageView+Kingfisher.swift
│   │       │   ├── NSButton+Kingfisher.swift
│   │       │   ├── NSTextAttachment+Kingfisher.swift
│   │       │   ├── TVMonogramView+Kingfisher.swift
│   │       │   ├── UIButton+Kingfisher.swift
│   │       │   └── WKInterfaceImage+Kingfisher.swift
│   │       ├── General
│   │       │   ├── ImageSource
│   │       │   │   ├── AVAssetImageDataProvider.swift
│   │       │   │   ├── ImageDataProvider.swift
│   │       │   │   ├── Resource.swift
│   │       │   │   └── Source.swift
│   │       │   ├── KF.swift
│   │       │   ├── KFOptionsSetter.swift
│   │       │   ├── Kingfisher.swift
│   │       │   ├── KingfisherError.swift
│   │       │   ├── KingfisherManager.swift
│   │       │   └── KingfisherOptionsInfo.swift
│   │       ├── Image
│   │       │   ├── Filter.swift
│   │       │   ├── GIFAnimatedImage.swift
│   │       │   ├── GraphicsContext.swift
│   │       │   ├── Image.swift
│   │       │   ├── ImageDrawing.swift
│   │       │   ├── ImageFormat.swift
│   │       │   ├── ImageProcessor.swift
│   │       │   ├── ImageProgressive.swift
│   │       │   ├── ImageTransition.swift
│   │       │   └── Placeholder.swift
│   │       ├── Networking
│   │       │   ├── AuthenticationChallengeResponsable.swift
│   │       │   ├── ImageDataProcessor.swift
│   │       │   ├── ImageDownloader.swift
│   │       │   ├── ImageDownloaderDelegate.swift
│   │       │   ├── ImageModifier.swift
│   │       │   ├── ImagePrefetcher.swift
│   │       │   ├── RedirectHandler.swift
│   │       │   ├── RequestModifier.swift
│   │       │   ├── RetryStrategy.swift
│   │       │   ├── SessionDataTask.swift
│   │       │   └── SessionDelegate.swift
│   │       ├── SwiftUI
│   │       │   ├── ImageBinder.swift
│   │       │   ├── KFImage.swift
│   │       │   └── KFImageOptions.swift
│   │       ├── Utility
│   │       │   ├── Box.swift
│   │       │   ├── CallbackQueue.swift
│   │       │   ├── Delegate.swift
│   │       │   ├── ExtensionHelpers.swift
│   │       │   ├── Result.swift
│   │       │   ├── Runtime.swift
│   │       │   ├── SizeExtensions.swift
│   │       │   └── String+MD5.swift
│   │       └── Views
│   │           ├── AnimatedImageView.swift
│   │           └── Indicator.swift
│   ├── Local Podspecs
│   ├── Manifest.lock
│   ├── Pods.xcodeproj
│   │   ├── project.pbxproj
│   │   └── xcuserdata
│   │       └── incross0915.xcuserdatad
│   │           └── xcschemes
│   │               ├── Alamofire-Alamofire.xcscheme
│   │               ├── Alamofire.xcscheme
│   │               ├── Differentiator.xcscheme
│   │               ├── Firebase.xcscheme
│   │               ├── FirebaseAnalytics.xcscheme
│   │               ├── FirebaseCore.xcscheme
│   │               ├── FirebaseCoreInternal-FirebaseCoreInternal_Privacy.xcscheme
│   │               ├── FirebaseCoreInternal.xcscheme
│   │               ├── FirebaseInstallations-FirebaseInstallations_Privacy.xcscheme
│   │               ├── FirebaseInstallations.xcscheme
│   │               ├── FirebaseMessaging.xcscheme
│   │               ├── GoogleAnalytics.xcscheme
│   │               ├── GoogleAppMeasurement.xcscheme
│   │               ├── GoogleDataTransport-GoogleDataTransport_Privacy.xcscheme
│   │               ├── GoogleDataTransport.xcscheme
│   │               ├── GoogleTagManager.xcscheme
│   │               ├── GoogleUtilities-GoogleUtilities_Privacy.xcscheme
│   │               ├── GoogleUtilities.xcscheme
│   │               ├── Kingfisher.xcscheme
│   │               ├── Pods-IncrossOfficeCafe-IncrossOfficeCafeUITests.xcscheme
│   │               ├── Pods-IncrossOfficeCafe-NotificationServiceExtension.xcscheme
│   │               ├── Pods-IncrossOfficeCafe.xcscheme
│   │               ├── Pods-IncrossOfficeCafeTests.xcscheme
│   │               ├── PromisesObjC-FBLPromises_Privacy.xcscheme
│   │               ├── PromisesObjC.xcscheme
│   │               ├── ReactorKit.xcscheme
│   │               ├── RxCocoa-RxCocoa_Privacy.xcscheme
│   │               ├── RxCocoa.xcscheme
│   │               ├── RxDataSources.xcscheme
│   │               ├── RxGesture.xcscheme
│   │               ├── RxOptional.xcscheme
│   │               ├── RxRelay-RxRelay_Privacy.xcscheme
│   │               ├── RxRelay.xcscheme
│   │               ├── RxSwift-RxSwift_Privacy.xcscheme
│   │               ├── RxSwift.xcscheme
│   │               ├── RxViewController.xcscheme
│   │               ├── SafeAreaBrush.xcscheme
│   │               ├── SnapKit-SnapKit_Privacy.xcscheme
│   │               ├── SnapKit.xcscheme
│   │               ├── Then.xcscheme
│   │               ├── Toast-Swift.xcscheme
│   │               ├── WeakMapTable.xcscheme
│   │               ├── nanopb.xcscheme
│   │               └── xcschememanagement.plist
│   ├── PromisesObjC
│   │   ├── LICENSE
│   │   ├── README.md
│   │   └── Sources
│   │       └── FBLPromises
│   │           ├── FBLPromise+All.m
│   │           ├── FBLPromise+Always.m
│   │           ├── FBLPromise+Any.m
│   │           ├── FBLPromise+Async.m
│   │           ├── FBLPromise+Await.m
│   │           ├── FBLPromise+Catch.m
│   │           ├── FBLPromise+Delay.m
│   │           ├── FBLPromise+Do.m
│   │           ├── FBLPromise+Race.m
│   │           ├── FBLPromise+Recover.m
│   │           ├── FBLPromise+Reduce.m
│   │           ├── FBLPromise+Retry.m
│   │           ├── FBLPromise+Testing.m
│   │           ├── FBLPromise+Then.m
│   │           ├── FBLPromise+Timeout.m
│   │           ├── FBLPromise+Validate.m
│   │           ├── FBLPromise+Wrap.m
│   │           ├── FBLPromise.m
│   │           ├── FBLPromiseError.m
│   │           ├── Resources
│   │           │   └── PrivacyInfo.xcprivacy
│   │           └── include
│   │               ├── FBLPromise+All.h
│   │               ├── FBLPromise+Always.h
│   │               ├── FBLPromise+Any.h
│   │               ├── FBLPromise+Async.h
│   │               ├── FBLPromise+Await.h
│   │               ├── FBLPromise+Catch.h
│   │               ├── FBLPromise+Delay.h
│   │               ├── FBLPromise+Do.h
│   │               ├── FBLPromise+Race.h
│   │               ├── FBLPromise+Recover.h
│   │               ├── FBLPromise+Reduce.h
│   │               ├── FBLPromise+Retry.h
│   │               ├── FBLPromise+Testing.h
│   │               ├── FBLPromise+Then.h
│   │               ├── FBLPromise+Timeout.h
│   │               ├── FBLPromise+Validate.h
│   │               ├── FBLPromise+Wrap.h
│   │               ├── FBLPromise.h
│   │               ├── FBLPromiseError.h
│   │               ├── FBLPromisePrivate.h
│   │               └── FBLPromises.h
│   ├── ReactorKit
│   │   ├── LICENSE
│   │   ├── README.md
│   │   └── Sources
│   │       ├── ReactorKit
│   │       │   ├── ActionSubject.swift
│   │       │   ├── Exports.swift
│   │       │   ├── IdentityEquatable.swift
│   │       │   ├── IdentityHashable.swift
│   │       │   ├── Pulse.swift
│   │       │   ├── Reactor+Pulse.swift
│   │       │   ├── Reactor.swift
│   │       │   ├── StateRelay.swift
│   │       │   ├── StoryboardView.swift
│   │       │   ├── Stub.swift
│   │       │   └── View.swift
│   │       └── ReactorKitRuntime
│   │           ├── ReactorKitRuntime.m
│   │           └── include
│   │               └── ReactorKitRuntime.h
│   ├── RxCocoa
│   │   ├── LICENSE.md
│   │   ├── Platform
│   │   │   ├── DataStructures
│   │   │   │   ├── Bag.swift
│   │   │   │   ├── InfiniteSequence.swift
│   │   │   │   ├── PriorityQueue.swift
│   │   │   │   └── Queue.swift
│   │   │   ├── DispatchQueue+Extensions.swift
│   │   │   ├── Platform.Darwin.swift
│   │   │   ├── Platform.Linux.swift
│   │   │   └── RecursiveLock.swift
│   │   ├── README.md
│   │   ├── RxCocoa
│   │   │   ├── Common
│   │   │   │   ├── ControlTarget.swift
│   │   │   │   ├── DelegateProxy.swift
│   │   │   │   ├── DelegateProxyType.swift
│   │   │   │   ├── Infallible+Bind.swift
│   │   │   │   ├── Observable+Bind.swift
│   │   │   │   ├── RxCocoaObjCRuntimeError+Extensions.swift
│   │   │   │   ├── RxTarget.swift
│   │   │   │   ├── SectionedViewDataSourceType.swift
│   │   │   │   └── TextInput.swift
│   │   │   ├── Foundation
│   │   │   │   ├── KVORepresentable+CoreGraphics.swift
│   │   │   │   ├── KVORepresentable+Swift.swift
│   │   │   │   ├── KVORepresentable.swift
│   │   │   │   ├── NSObject+Rx+KVORepresentable.swift
│   │   │   │   ├── NSObject+Rx+RawRepresentable.swift
│   │   │   │   ├── NSObject+Rx.swift
│   │   │   │   ├── NotificationCenter+Rx.swift
│   │   │   │   └── URLSession+Rx.swift
│   │   │   ├── Runtime
│   │   │   │   ├── _RX.m
│   │   │   │   ├── _RXDelegateProxy.m
│   │   │   │   ├── _RXKVOObserver.m
│   │   │   │   ├── _RXObjCRuntime.m
│   │   │   │   └── include
│   │   │   │       ├── RxCocoaRuntime.h
│   │   │   │       ├── _RX.h
│   │   │   │       ├── _RXDelegateProxy.h
│   │   │   │       ├── _RXKVOObserver.h
│   │   │   │       └── _RXObjCRuntime.h
│   │   │   ├── RxCocoa.h
│   │   │   ├── RxCocoa.swift
│   │   │   ├── Traits
│   │   │   │   ├── ControlEvent.swift
│   │   │   │   ├── ControlProperty.swift
│   │   │   │   ├── Driver
│   │   │   │   │   ├── BehaviorRelay+Driver.swift
│   │   │   │   │   ├── ControlEvent+Driver.swift
│   │   │   │   │   ├── ControlProperty+Driver.swift
│   │   │   │   │   ├── Driver+Subscription.swift
│   │   │   │   │   ├── Driver.swift
│   │   │   │   │   ├── Infallible+Driver.swift
│   │   │   │   │   └── ObservableConvertibleType+Driver.swift
│   │   │   │   ├── SharedSequence
│   │   │   │   │   ├── ObservableConvertibleType+SharedSequence.swift
│   │   │   │   │   ├── SchedulerType+SharedSequence.swift
│   │   │   │   │   ├── SharedSequence+Concurrency.swift
│   │   │   │   │   ├── SharedSequence+Operators+arity.swift
│   │   │   │   │   ├── SharedSequence+Operators.swift
│   │   │   │   │   └── SharedSequence.swift
│   │   │   │   └── Signal
│   │   │   │       ├── ControlEvent+Signal.swift
│   │   │   │       ├── ObservableConvertibleType+Signal.swift
│   │   │   │       ├── PublishRelay+Signal.swift
│   │   │   │       ├── Signal+Subscription.swift
│   │   │   │       └── Signal.swift
│   │   │   ├── iOS
│   │   │   │   ├── DataSources
│   │   │   │   │   ├── RxCollectionViewReactiveArrayDataSource.swift
│   │   │   │   │   ├── RxPickerViewAdapter.swift
│   │   │   │   │   └── RxTableViewReactiveArrayDataSource.swift
│   │   │   │   ├── Events
│   │   │   │   │   └── ItemEvents.swift
│   │   │   │   ├── NSTextStorage+Rx.swift
│   │   │   │   ├── Protocols
│   │   │   │   │   ├── RxCollectionViewDataSourceType.swift
│   │   │   │   │   ├── RxPickerViewDataSourceType.swift
│   │   │   │   │   └── RxTableViewDataSourceType.swift
│   │   │   │   ├── Proxies
│   │   │   │   │   ├── RxCollectionViewDataSourcePrefetchingProxy.swift
│   │   │   │   │   ├── RxCollectionViewDataSourceProxy.swift
│   │   │   │   │   ├── RxCollectionViewDelegateProxy.swift
│   │   │   │   │   ├── RxNavigationControllerDelegateProxy.swift
│   │   │   │   │   ├── RxPickerViewDataSourceProxy.swift
│   │   │   │   │   ├── RxPickerViewDelegateProxy.swift
│   │   │   │   │   ├── RxScrollViewDelegateProxy.swift
│   │   │   │   │   ├── RxSearchBarDelegateProxy.swift
│   │   │   │   │   ├── RxSearchControllerDelegateProxy.swift
│   │   │   │   │   ├── RxTabBarControllerDelegateProxy.swift
│   │   │   │   │   ├── RxTabBarDelegateProxy.swift
│   │   │   │   │   ├── RxTableViewDataSourcePrefetchingProxy.swift
│   │   │   │   │   ├── RxTableViewDataSourceProxy.swift
│   │   │   │   │   ├── RxTableViewDelegateProxy.swift
│   │   │   │   │   ├── RxTextStorageDelegateProxy.swift
│   │   │   │   │   ├── RxTextViewDelegateProxy.swift
│   │   │   │   │   └── RxWKNavigationDelegateProxy.swift
│   │   │   │   ├── UIActivityIndicatorView+Rx.swift
│   │   │   │   ├── UIApplication+Rx.swift
│   │   │   │   ├── UIBarButtonItem+Rx.swift
│   │   │   │   ├── UIButton+Rx.swift
│   │   │   │   ├── UICollectionView+Rx.swift
│   │   │   │   ├── UIControl+Rx.swift
│   │   │   │   ├── UIDatePicker+Rx.swift
│   │   │   │   ├── UIGestureRecognizer+Rx.swift
│   │   │   │   ├── UINavigationController+Rx.swift
│   │   │   │   ├── UIPickerView+Rx.swift
│   │   │   │   ├── UIRefreshControl+Rx.swift
│   │   │   │   ├── UIScrollView+Rx.swift
│   │   │   │   ├── UISearchBar+Rx.swift
│   │   │   │   ├── UISearchController+Rx.swift
│   │   │   │   ├── UISegmentedControl+Rx.swift
│   │   │   │   ├── UISlider+Rx.swift
│   │   │   │   ├── UIStepper+Rx.swift
│   │   │   │   ├── UISwitch+Rx.swift
│   │   │   │   ├── UITabBar+Rx.swift
│   │   │   │   ├── UITabBarController+Rx.swift
│   │   │   │   ├── UITableView+Rx.swift
│   │   │   │   ├── UITextField+Rx.swift
│   │   │   │   ├── UITextView+Rx.swift
│   │   │   │   └── WKWebView+Rx.swift
│   │   │   └── macOS
│   │   │       ├── NSButton+Rx.swift
│   │   │       ├── NSControl+Rx.swift
│   │   │       ├── NSSlider+Rx.swift
│   │   │       ├── NSTextField+Rx.swift
│   │   │       ├── NSTextView+Rx.swift
│   │   │       └── NSView+Rx.swift
│   │   └── Sources
│   │       └── RxCocoa
│   │           └── PrivacyInfo.xcprivacy
│   ├── RxDataSources
│   │   ├── LICENSE.md
│   │   ├── README.md
│   │   └── Sources
│   │       └── RxDataSources
│   │           ├── AnimationConfiguration.swift
│   │           ├── Array+Extensions.swift
│   │           ├── CollectionViewSectionedDataSource.swift
│   │           ├── DataSources.swift
│   │           ├── Deprecated.swift
│   │           ├── FloatingPointType+IdentifiableType.swift
│   │           ├── IntegerType+IdentifiableType.swift
│   │           ├── RxCollectionViewSectionedAnimatedDataSource.swift
│   │           ├── RxCollectionViewSectionedReloadDataSource.swift
│   │           ├── RxPickerViewAdapter.swift
│   │           ├── RxTableViewSectionedAnimatedDataSource.swift
│   │           ├── RxTableViewSectionedReloadDataSource.swift
│   │           ├── String+IdentifiableType.swift
│   │           ├── TableViewSectionedDataSource.swift
│   │           ├── UI+SectionedViewType.swift
│   │           └── ViewTransition.swift
│   ├── RxGesture
│   │   ├── LICENSE
│   │   ├── Pod
│   │   │   └── Classes
│   │   │       ├── GenericRxGestureRecognizerDelegate.swift
│   │   │       ├── GestureFactory.swift
│   │   │       ├── GestureRecognizer+RxGesture.swift
│   │   │       ├── SharedTypes.swift
│   │   │       ├── View+RxGesture.swift
│   │   │       └── iOS
│   │   │           ├── ForceTouchGestureRecognizer.swift
│   │   │           ├── TouchDownGestureRecognizer.swift
│   │   │           ├── TransformGestureRecognizers.swift
│   │   │           ├── UIHoverGestureRecognizer+RxGesture.swift
│   │   │           ├── UILongPressGestureRecognizer+RxGesture.swift
│   │   │           ├── UIPanGestureRecognizer+RxGesture.swift
│   │   │           ├── UIPinchGestureRecognizer+RxGesture.swift
│   │   │           ├── UIRotationGestureRecognizer+RxGesture.swift
│   │   │           ├── UIScreenEdgePanGestureRecognizer+RxGesture.swift
│   │   │           ├── UISwipeGestureRecognizer+RxGesture.swift
│   │   │           └── UITapGestureRecognizer+RxGesture.swift
│   │   └── README.md
│   ├── RxOptional
│   │   ├── LICENSE
│   │   ├── LICENSE.md
│   │   ├── README.md
│   │   └── Sources
│   │       └── RxOptional
│   │           ├── Observable+Occupiable.swift
│   │           ├── Observable+Optional.swift
│   │           ├── Occupiable.swift
│   │           ├── OptionalType.swift
│   │           ├── RxOptionalError.swift
│   │           ├── SharedSequence+Occupiable.swift
│   │           └── SharedSequence+Optional.swift
│   ├── RxRelay
│   │   ├── LICENSE.md
│   │   ├── README.md
│   │   ├── RxRelay
│   │   │   ├── BehaviorRelay.swift
│   │   │   ├── Observable+Bind.swift
│   │   │   ├── PublishRelay.swift
│   │   │   ├── ReplayRelay.swift
│   │   │   └── Utils.swift
│   │   └── Sources
│   │       └── RxRelay
│   │           └── PrivacyInfo.xcprivacy
│   ├── RxSwift
│   │   ├── LICENSE.md
│   │   ├── Platform
│   │   │   ├── AtomicInt.swift
│   │   │   ├── DataStructures
│   │   │   │   ├── Bag.swift
│   │   │   │   ├── InfiniteSequence.swift
│   │   │   │   ├── PriorityQueue.swift
│   │   │   │   └── Queue.swift
│   │   │   ├── DispatchQueue+Extensions.swift
│   │   │   ├── Platform.Darwin.swift
│   │   │   ├── Platform.Linux.swift
│   │   │   └── RecursiveLock.swift
│   │   ├── README.md
│   │   ├── RxSwift
│   │   │   ├── AnyObserver.swift
│   │   │   ├── Binder.swift
│   │   │   ├── Cancelable.swift
│   │   │   ├── Concurrency
│   │   │   │   ├── AsyncLock.swift
│   │   │   │   ├── Lock.swift
│   │   │   │   ├── LockOwnerType.swift
│   │   │   │   ├── SynchronizedDisposeType.swift
│   │   │   │   ├── SynchronizedOnType.swift
│   │   │   │   └── SynchronizedUnsubscribeType.swift
│   │   │   ├── ConnectableObservableType.swift
│   │   │   ├── Date+Dispatch.swift
│   │   │   ├── Disposable.swift
│   │   │   ├── Disposables
│   │   │   │   ├── AnonymousDisposable.swift
│   │   │   │   ├── BinaryDisposable.swift
│   │   │   │   ├── BooleanDisposable.swift
│   │   │   │   ├── CompositeDisposable.swift
│   │   │   │   ├── Disposables.swift
│   │   │   │   ├── DisposeBag.swift
│   │   │   │   ├── DisposeBase.swift
│   │   │   │   ├── NopDisposable.swift
│   │   │   │   ├── RefCountDisposable.swift
│   │   │   │   ├── ScheduledDisposable.swift
│   │   │   │   ├── SerialDisposable.swift
│   │   │   │   ├── SingleAssignmentDisposable.swift
│   │   │   │   └── SubscriptionDisposable.swift
│   │   │   ├── Errors.swift
│   │   │   ├── Event.swift
│   │   │   ├── Extensions
│   │   │   │   └── Bag+Rx.swift
│   │   │   ├── GroupedObservable.swift
│   │   │   ├── ImmediateSchedulerType.swift
│   │   │   ├── Observable+Concurrency.swift
│   │   │   ├── Observable.swift
│   │   │   ├── ObservableConvertibleType.swift
│   │   │   ├── ObservableType+Extensions.swift
│   │   │   ├── ObservableType.swift
│   │   │   ├── Observables
│   │   │   │   ├── AddRef.swift
│   │   │   │   ├── Amb.swift
│   │   │   │   ├── AsMaybe.swift
│   │   │   │   ├── AsSingle.swift
│   │   │   │   ├── Buffer.swift
│   │   │   │   ├── Catch.swift
│   │   │   │   ├── CombineLatest+Collection.swift
│   │   │   │   ├── CombineLatest+arity.swift
│   │   │   │   ├── CombineLatest.swift
│   │   │   │   ├── CompactMap.swift
│   │   │   │   ├── Concat.swift
│   │   │   │   ├── Create.swift
│   │   │   │   ├── Debounce.swift
│   │   │   │   ├── Debug.swift
│   │   │   │   ├── Decode.swift
│   │   │   │   ├── DefaultIfEmpty.swift
│   │   │   │   ├── Deferred.swift
│   │   │   │   ├── Delay.swift
│   │   │   │   ├── DelaySubscription.swift
│   │   │   │   ├── Dematerialize.swift
│   │   │   │   ├── DistinctUntilChanged.swift
│   │   │   │   ├── Do.swift
│   │   │   │   ├── ElementAt.swift
│   │   │   │   ├── Empty.swift
│   │   │   │   ├── Enumerated.swift
│   │   │   │   ├── Error.swift
│   │   │   │   ├── Filter.swift
│   │   │   │   ├── First.swift
│   │   │   │   ├── Generate.swift
│   │   │   │   ├── GroupBy.swift
│   │   │   │   ├── Just.swift
│   │   │   │   ├── Map.swift
│   │   │   │   ├── Materialize.swift
│   │   │   │   ├── Merge.swift
│   │   │   │   ├── Multicast.swift
│   │   │   │   ├── Never.swift
│   │   │   │   ├── ObserveOn.swift
│   │   │   │   ├── Optional.swift
│   │   │   │   ├── Producer.swift
│   │   │   │   ├── Range.swift
│   │   │   │   ├── Reduce.swift
│   │   │   │   ├── Repeat.swift
│   │   │   │   ├── RetryWhen.swift
│   │   │   │   ├── Sample.swift
│   │   │   │   ├── Scan.swift
│   │   │   │   ├── Sequence.swift
│   │   │   │   ├── ShareReplayScope.swift
│   │   │   │   ├── SingleAsync.swift
│   │   │   │   ├── Sink.swift
│   │   │   │   ├── Skip.swift
│   │   │   │   ├── SkipUntil.swift
│   │   │   │   ├── SkipWhile.swift
│   │   │   │   ├── StartWith.swift
│   │   │   │   ├── SubscribeOn.swift
│   │   │   │   ├── Switch.swift
│   │   │   │   ├── SwitchIfEmpty.swift
│   │   │   │   ├── Take.swift
│   │   │   │   ├── TakeLast.swift
│   │   │   │   ├── TakeWithPredicate.swift
│   │   │   │   ├── Throttle.swift
│   │   │   │   ├── Timeout.swift
│   │   │   │   ├── Timer.swift
│   │   │   │   ├── ToArray.swift
│   │   │   │   ├── Using.swift
│   │   │   │   ├── Window.swift
│   │   │   │   ├── WithLatestFrom.swift
│   │   │   │   ├── WithUnretained.swift
│   │   │   │   ├── Zip+Collection.swift
│   │   │   │   ├── Zip+arity.swift
│   │   │   │   └── Zip.swift
│   │   │   ├── ObserverType.swift
│   │   │   ├── Observers
│   │   │   │   ├── AnonymousObserver.swift
│   │   │   │   ├── ObserverBase.swift
│   │   │   │   └── TailRecursiveSink.swift
│   │   │   ├── Reactive.swift
│   │   │   ├── Rx.swift
│   │   │   ├── RxMutableBox.swift
│   │   │   ├── SchedulerType.swift
│   │   │   ├── Schedulers
│   │   │   │   ├── ConcurrentDispatchQueueScheduler.swift
│   │   │   │   ├── ConcurrentMainScheduler.swift
│   │   │   │   ├── CurrentThreadScheduler.swift
│   │   │   │   ├── HistoricalScheduler.swift
│   │   │   │   ├── HistoricalSchedulerTimeConverter.swift
│   │   │   │   ├── Internal
│   │   │   │   │   ├── DispatchQueueConfiguration.swift
│   │   │   │   │   ├── InvocableScheduledItem.swift
│   │   │   │   │   ├── InvocableType.swift
│   │   │   │   │   ├── ScheduledItem.swift
│   │   │   │   │   └── ScheduledItemType.swift
│   │   │   │   ├── MainScheduler.swift
│   │   │   │   ├── OperationQueueScheduler.swift
│   │   │   │   ├── RecursiveScheduler.swift
│   │   │   │   ├── SchedulerServices+Emulation.swift
│   │   │   │   ├── SerialDispatchQueueScheduler.swift
│   │   │   │   ├── VirtualTimeConverterType.swift
│   │   │   │   └── VirtualTimeScheduler.swift
│   │   │   ├── Subjects
│   │   │   │   ├── AsyncSubject.swift
│   │   │   │   ├── BehaviorSubject.swift
│   │   │   │   ├── PublishSubject.swift
│   │   │   │   ├── ReplaySubject.swift
│   │   │   │   └── SubjectType.swift
│   │   │   ├── SwiftSupport
│   │   │   │   └── SwiftSupport.swift
│   │   │   └── Traits
│   │   │       ├── Infallible
│   │   │       │   ├── Infallible+CombineLatest+Collection.swift
│   │   │       │   ├── Infallible+CombineLatest+arity.swift
│   │   │       │   ├── Infallible+Concurrency.swift
│   │   │       │   ├── Infallible+Create.swift
│   │   │       │   ├── Infallible+Debug.swift
│   │   │       │   ├── Infallible+Operators.swift
│   │   │       │   ├── Infallible+Zip+arity.swift
│   │   │       │   ├── Infallible.swift
│   │   │       │   └── ObservableConvertibleType+Infallible.swift
│   │   │       └── PrimitiveSequence
│   │   │           ├── Completable+AndThen.swift
│   │   │           ├── Completable.swift
│   │   │           ├── Maybe.swift
│   │   │           ├── ObservableType+PrimitiveSequence.swift
│   │   │           ├── PrimitiveSequence+Concurrency.swift
│   │   │           ├── PrimitiveSequence+Zip+arity.swift
│   │   │           ├── PrimitiveSequence.swift
│   │   │           └── Single.swift
│   │   └── Sources
│   │       └── RxSwift
│   │           └── PrivacyInfo.xcprivacy
│   ├── RxViewController
│   │   ├── LICENSE
│   │   ├── README.md
│   │   └── Sources
│   │       └── RxViewController
│   │           ├── NSViewController+Rx.swift
│   │           └── UIViewController+Rx.swift
│   ├── SafeAreaBrush
│   │   ├── LICENSE
│   │   ├── README.md
│   │   └── Sources
│   │       └── Classes
│   │           ├── SafeAreaPosition.swift
│   │           ├── SafeAreaView.swift
│   │           └── Utils
│   │               ├── CALayer+Animation.swift
│   │               └── UIViewController+SafeAreaBrush.swift
│   ├── SnapKit
│   │   ├── LICENSE
│   │   ├── README.md
│   │   └── Sources
│   │       ├── Constraint.swift
│   │       ├── ConstraintAttributes.swift
│   │       ├── ConstraintConfig.swift
│   │       ├── ConstraintConstantTarget.swift
│   │       ├── ConstraintDSL.swift
│   │       ├── ConstraintDescription.swift
│   │       ├── ConstraintDirectionalInsetTarget.swift
│   │       ├── ConstraintDirectionalInsets.swift
│   │       ├── ConstraintInsetTarget.swift
│   │       ├── ConstraintInsets.swift
│   │       ├── ConstraintItem.swift
│   │       ├── ConstraintLayoutGuide+Extensions.swift
│   │       ├── ConstraintLayoutGuide.swift
│   │       ├── ConstraintLayoutGuideDSL.swift
│   │       ├── ConstraintLayoutSupport.swift
│   │       ├── ConstraintLayoutSupportDSL.swift
│   │       ├── ConstraintMaker.swift
│   │       ├── ConstraintMakerEditable.swift
│   │       ├── ConstraintMakerExtendable.swift
│   │       ├── ConstraintMakerFinalizable.swift
│   │       ├── ConstraintMakerPrioritizable.swift
│   │       ├── ConstraintMakerRelatable+Extensions.swift
│   │       ├── ConstraintMakerRelatable.swift
│   │       ├── ConstraintMultiplierTarget.swift
│   │       ├── ConstraintOffsetTarget.swift
│   │       ├── ConstraintPriority.swift
│   │       ├── ConstraintPriorityTarget.swift
│   │       ├── ConstraintRelatableTarget.swift
│   │       ├── ConstraintRelation.swift
│   │       ├── ConstraintView+Extensions.swift
│   │       ├── ConstraintView.swift
│   │       ├── ConstraintViewDSL.swift
│   │       ├── Debugging.swift
│   │       ├── LayoutConstraint.swift
│   │       ├── LayoutConstraintItem.swift
│   │       ├── PrivacyInfo.xcprivacy
│   │       ├── Typealiases.swift
│   │       └── UILayoutSupport+Extensions.swift
│   ├── Target Support Files
│   │   ├── Alamofire
│   │   │   ├── Alamofire-Info.plist
│   │   │   ├── Alamofire-dummy.m
│   │   │   ├── Alamofire-prefix.pch
│   │   │   ├── Alamofire-umbrella.h
│   │   │   ├── Alamofire.debug.xcconfig
│   │   │   ├── Alamofire.modulemap
│   │   │   ├── Alamofire.release.xcconfig
│   │   │   └── ResourceBundle-Alamofire-Alamofire-Info.plist
│   │   ├── Differentiator
│   │   │   ├── Differentiator-Info.plist
│   │   │   ├── Differentiator-dummy.m
│   │   │   ├── Differentiator-prefix.pch
│   │   │   ├── Differentiator-umbrella.h
│   │   │   ├── Differentiator.debug.xcconfig
│   │   │   ├── Differentiator.modulemap
│   │   │   └── Differentiator.release.xcconfig
│   │   ├── Firebase
│   │   │   ├── Firebase.debug.xcconfig
│   │   │   └── Firebase.release.xcconfig
│   │   ├── FirebaseAnalytics
│   │   │   ├── FirebaseAnalytics-xcframeworks-input-files.xcfilelist
│   │   │   ├── FirebaseAnalytics-xcframeworks-output-files.xcfilelist
│   │   │   ├── FirebaseAnalytics-xcframeworks.sh
│   │   │   ├── FirebaseAnalytics.debug.xcconfig
│   │   │   └── FirebaseAnalytics.release.xcconfig
│   │   ├── FirebaseCore
│   │   │   ├── FirebaseCore-Info.plist
│   │   │   ├── FirebaseCore-dummy.m
│   │   │   ├── FirebaseCore-umbrella.h
│   │   │   ├── FirebaseCore.debug.xcconfig
│   │   │   ├── FirebaseCore.modulemap
│   │   │   ├── FirebaseCore.release.xcconfig
│   │   │   └── ResourceBundle-FirebaseCore_Privacy-FirebaseCore-Info.plist
│   │   ├── FirebaseCoreInternal
│   │   │   ├── FirebaseCoreInternal-Info.plist
│   │   │   ├── FirebaseCoreInternal-dummy.m
│   │   │   ├── FirebaseCoreInternal-prefix.pch
│   │   │   ├── FirebaseCoreInternal-umbrella.h
│   │   │   ├── FirebaseCoreInternal.debug.xcconfig
│   │   │   ├── FirebaseCoreInternal.modulemap
│   │   │   ├── FirebaseCoreInternal.release.xcconfig
│   │   │   └── ResourceBundle-FirebaseCoreInternal_Privacy-FirebaseCoreInternal-Info.plist
│   │   ├── FirebaseInstallations
│   │   │   ├── FirebaseInstallations-Info.plist
│   │   │   ├── FirebaseInstallations-dummy.m
│   │   │   ├── FirebaseInstallations-umbrella.h
│   │   │   ├── FirebaseInstallations.debug.xcconfig
│   │   │   ├── FirebaseInstallations.modulemap
│   │   │   ├── FirebaseInstallations.release.xcconfig
│   │   │   └── ResourceBundle-FirebaseInstallations_Privacy-FirebaseInstallations-Info.plist
│   │   ├── FirebaseMessaging
│   │   │   ├── FirebaseMessaging-Info.plist
│   │   │   ├── FirebaseMessaging-dummy.m
│   │   │   ├── FirebaseMessaging-umbrella.h
│   │   │   ├── FirebaseMessaging.debug.xcconfig
│   │   │   ├── FirebaseMessaging.modulemap
│   │   │   ├── FirebaseMessaging.release.xcconfig
│   │   │   └── ResourceBundle-FirebaseMessaging_Privacy-FirebaseMessaging-Info.plist
│   │   ├── GoogleAnalytics
│   │   │   ├── GoogleAnalytics-xcframeworks-input-files.xcfilelist
│   │   │   ├── GoogleAnalytics-xcframeworks-output-files.xcfilelist
│   │   │   ├── GoogleAnalytics-xcframeworks.sh
│   │   │   ├── GoogleAnalytics.debug.xcconfig
│   │   │   └── GoogleAnalytics.release.xcconfig
│   │   ├── GoogleAppMeasurement
│   │   │   ├── GoogleAppMeasurement-xcframeworks-input-files.xcfilelist
│   │   │   ├── GoogleAppMeasurement-xcframeworks-output-files.xcfilelist
│   │   │   ├── GoogleAppMeasurement-xcframeworks.sh
│   │   │   ├── GoogleAppMeasurement.debug.xcconfig
│   │   │   └── GoogleAppMeasurement.release.xcconfig
│   │   ├── GoogleDataTransport
│   │   │   ├── GoogleDataTransport-Info.plist
│   │   │   ├── GoogleDataTransport-dummy.m
│   │   │   ├── GoogleDataTransport-umbrella.h
│   │   │   ├── GoogleDataTransport.debug.xcconfig
│   │   │   ├── GoogleDataTransport.modulemap
│   │   │   ├── GoogleDataTransport.release.xcconfig
│   │   │   └── ResourceBundle-GoogleDataTransport_Privacy-GoogleDataTransport-Info.plist
│   │   ├── GoogleTagManager
│   │   │   ├── GoogleTagManager-xcframeworks-input-files.xcfilelist
│   │   │   ├── GoogleTagManager-xcframeworks-output-files.xcfilelist
│   │   │   ├── GoogleTagManager-xcframeworks.sh
│   │   │   ├── GoogleTagManager.debug.xcconfig
│   │   │   └── GoogleTagManager.release.xcconfig
│   │   ├── GoogleUtilities
│   │   │   ├── GoogleUtilities-Info.plist
│   │   │   ├── GoogleUtilities-dummy.m
│   │   │   ├── GoogleUtilities-umbrella.h
│   │   │   ├── GoogleUtilities.debug.xcconfig
│   │   │   ├── GoogleUtilities.modulemap
│   │   │   ├── GoogleUtilities.release.xcconfig
│   │   │   └── ResourceBundle-GoogleUtilities_Privacy-GoogleUtilities-Info.plist
│   │   ├── Kingfisher
│   │   │   ├── Kingfisher-Info.plist
│   │   │   ├── Kingfisher-dummy.m
│   │   │   ├── Kingfisher-prefix.pch
│   │   │   ├── Kingfisher-umbrella.h
│   │   │   ├── Kingfisher.debug.xcconfig
│   │   │   ├── Kingfisher.modulemap
│   │   │   ├── Kingfisher.release.xcconfig
│   │   │   └── ResourceBundle-Kingfisher-Kingfisher-Info.plist
│   │   ├── Pods-IncrossOfficeCafe
│   │   │   ├── Pods-IncrossOfficeCafe-Info.plist
│   │   │   ├── Pods-IncrossOfficeCafe-acknowledgements.markdown
│   │   │   ├── Pods-IncrossOfficeCafe-acknowledgements.plist
│   │   │   ├── Pods-IncrossOfficeCafe-dummy.m
│   │   │   ├── Pods-IncrossOfficeCafe-frameworks-Debug-input-files.xcfilelist
│   │   │   ├── Pods-IncrossOfficeCafe-frameworks-Debug-output-files.xcfilelist
│   │   │   ├── Pods-IncrossOfficeCafe-frameworks-Release-input-files.xcfilelist
│   │   │   ├── Pods-IncrossOfficeCafe-frameworks-Release-output-files.xcfilelist
│   │   │   ├── Pods-IncrossOfficeCafe-frameworks.sh
│   │   │   ├── Pods-IncrossOfficeCafe-umbrella.h
│   │   │   ├── Pods-IncrossOfficeCafe.debug.xcconfig
│   │   │   ├── Pods-IncrossOfficeCafe.modulemap
│   │   │   └── Pods-IncrossOfficeCafe.release.xcconfig
│   │   ├── Pods-IncrossOfficeCafe-IncrossOfficeCafeUITests
│   │   │   ├── Pods-IncrossOfficeCafe-IncrossOfficeCafeUITests-Info.plist
│   │   │   ├── Pods-IncrossOfficeCafe-IncrossOfficeCafeUITests-acknowledgements.markdown
│   │   │   ├── Pods-IncrossOfficeCafe-IncrossOfficeCafeUITests-acknowledgements.plist
│   │   │   ├── Pods-IncrossOfficeCafe-IncrossOfficeCafeUITests-dummy.m
│   │   │   ├── Pods-IncrossOfficeCafe-IncrossOfficeCafeUITests-frameworks-Debug-input-files.xcfilelist
│   │   │   ├── Pods-IncrossOfficeCafe-IncrossOfficeCafeUITests-frameworks-Debug-output-files.xcfilelist
│   │   │   ├── Pods-IncrossOfficeCafe-IncrossOfficeCafeUITests-frameworks-Release-input-files.xcfilelist
│   │   │   ├── Pods-IncrossOfficeCafe-IncrossOfficeCafeUITests-frameworks-Release-output-files.xcfilelist
│   │   │   ├── Pods-IncrossOfficeCafe-IncrossOfficeCafeUITests-frameworks.sh
│   │   │   ├── Pods-IncrossOfficeCafe-IncrossOfficeCafeUITests-umbrella.h
│   │   │   ├── Pods-IncrossOfficeCafe-IncrossOfficeCafeUITests.debug.xcconfig
│   │   │   ├── Pods-IncrossOfficeCafe-IncrossOfficeCafeUITests.modulemap
│   │   │   └── Pods-IncrossOfficeCafe-IncrossOfficeCafeUITests.release.xcconfig
│   │   ├── Pods-IncrossOfficeCafe-NotificationServiceExtension
│   │   │   ├── Pods-IncrossOfficeCafe-NotificationServiceExtension-Info.plist
│   │   │   ├── Pods-IncrossOfficeCafe-NotificationServiceExtension-acknowledgements.markdown
│   │   │   ├── Pods-IncrossOfficeCafe-NotificationServiceExtension-acknowledgements.plist
│   │   │   ├── Pods-IncrossOfficeCafe-NotificationServiceExtension-dummy.m
│   │   │   ├── Pods-IncrossOfficeCafe-NotificationServiceExtension-umbrella.h
│   │   │   ├── Pods-IncrossOfficeCafe-NotificationServiceExtension.debug.xcconfig
│   │   │   ├── Pods-IncrossOfficeCafe-NotificationServiceExtension.modulemap
│   │   │   └── Pods-IncrossOfficeCafe-NotificationServiceExtension.release.xcconfig
│   │   ├── Pods-IncrossOfficeCafeTests
│   │   │   ├── Pods-IncrossOfficeCafeTests-Info.plist
│   │   │   ├── Pods-IncrossOfficeCafeTests-acknowledgements.markdown
│   │   │   ├── Pods-IncrossOfficeCafeTests-acknowledgements.plist
│   │   │   ├── Pods-IncrossOfficeCafeTests-dummy.m
│   │   │   ├── Pods-IncrossOfficeCafeTests-umbrella.h
│   │   │   ├── Pods-IncrossOfficeCafeTests.debug.xcconfig
│   │   │   ├── Pods-IncrossOfficeCafeTests.modulemap
│   │   │   └── Pods-IncrossOfficeCafeTests.release.xcconfig
│   │   ├── PromisesObjC
│   │   │   ├── PromisesObjC-Info.plist
│   │   │   ├── PromisesObjC-dummy.m
│   │   │   ├── PromisesObjC-umbrella.h
│   │   │   ├── PromisesObjC.debug.xcconfig
│   │   │   ├── PromisesObjC.modulemap
│   │   │   ├── PromisesObjC.release.xcconfig
│   │   │   └── ResourceBundle-FBLPromises_Privacy-PromisesObjC-Info.plist
│   │   ├── ReactorKit
│   │   │   ├── ReactorKit-Info.plist
│   │   │   ├── ReactorKit-dummy.m
│   │   │   ├── ReactorKit-prefix.pch
│   │   │   ├── ReactorKit-umbrella.h
│   │   │   ├── ReactorKit.debug.xcconfig
│   │   │   ├── ReactorKit.modulemap
│   │   │   └── ReactorKit.release.xcconfig
│   │   ├── RxCocoa
│   │   │   ├── ResourceBundle-RxCocoa_Privacy-RxCocoa-Info.plist
│   │   │   ├── RxCocoa-Info.plist
│   │   │   ├── RxCocoa-dummy.m
│   │   │   ├── RxCocoa-prefix.pch
│   │   │   ├── RxCocoa-umbrella.h
│   │   │   ├── RxCocoa.debug.xcconfig
│   │   │   ├── RxCocoa.modulemap
│   │   │   └── RxCocoa.release.xcconfig
│   │   ├── RxDataSources
│   │   │   ├── RxDataSources-Info.plist
│   │   │   ├── RxDataSources-dummy.m
│   │   │   ├── RxDataSources-prefix.pch
│   │   │   ├── RxDataSources-umbrella.h
│   │   │   ├── RxDataSources.debug.xcconfig
│   │   │   ├── RxDataSources.modulemap
│   │   │   └── RxDataSources.release.xcconfig
│   │   ├── RxGesture
│   │   │   ├── RxGesture-Info.plist
│   │   │   ├── RxGesture-dummy.m
│   │   │   ├── RxGesture-prefix.pch
│   │   │   ├── RxGesture-umbrella.h
│   │   │   ├── RxGesture.debug.xcconfig
│   │   │   ├── RxGesture.modulemap
│   │   │   └── RxGesture.release.xcconfig
│   │   ├── RxOptional
│   │   │   ├── RxOptional-Info.plist
│   │   │   ├── RxOptional-dummy.m
│   │   │   ├── RxOptional-prefix.pch
│   │   │   ├── RxOptional-umbrella.h
│   │   │   ├── RxOptional.debug.xcconfig
│   │   │   ├── RxOptional.modulemap
│   │   │   └── RxOptional.release.xcconfig
│   │   ├── RxRelay
│   │   │   ├── ResourceBundle-RxRelay_Privacy-RxRelay-Info.plist
│   │   │   ├── RxRelay-Info.plist
│   │   │   ├── RxRelay-dummy.m
│   │   │   ├── RxRelay-prefix.pch
│   │   │   ├── RxRelay-umbrella.h
│   │   │   ├── RxRelay.debug.xcconfig
│   │   │   ├── RxRelay.modulemap
│   │   │   └── RxRelay.release.xcconfig
│   │   ├── RxSwift
│   │   │   ├── ResourceBundle-RxSwift_Privacy-RxSwift-Info.plist
│   │   │   ├── RxSwift-Info.plist
│   │   │   ├── RxSwift-dummy.m
│   │   │   ├── RxSwift-prefix.pch
│   │   │   ├── RxSwift-umbrella.h
│   │   │   ├── RxSwift.debug.xcconfig
│   │   │   ├── RxSwift.modulemap
│   │   │   └── RxSwift.release.xcconfig
│   │   ├── RxViewController
│   │   │   ├── RxViewController-Info.plist
│   │   │   ├── RxViewController-dummy.m
│   │   │   ├── RxViewController-prefix.pch
│   │   │   ├── RxViewController-umbrella.h
│   │   │   ├── RxViewController.debug.xcconfig
│   │   │   ├── RxViewController.modulemap
│   │   │   └── RxViewController.release.xcconfig
│   │   ├── SafeAreaBrush
│   │   │   ├── SafeAreaBrush-Info.plist
│   │   │   ├── SafeAreaBrush-dummy.m
│   │   │   ├── SafeAreaBrush-prefix.pch
│   │   │   ├── SafeAreaBrush-umbrella.h
│   │   │   ├── SafeAreaBrush.debug.xcconfig
│   │   │   ├── SafeAreaBrush.modulemap
│   │   │   └── SafeAreaBrush.release.xcconfig
│   │   ├── SnapKit
│   │   │   ├── ResourceBundle-SnapKit_Privacy-SnapKit-Info.plist
│   │   │   ├── SnapKit-Info.plist
│   │   │   ├── SnapKit-dummy.m
│   │   │   ├── SnapKit-prefix.pch
│   │   │   ├── SnapKit-umbrella.h
│   │   │   ├── SnapKit.debug.xcconfig
│   │   │   ├── SnapKit.modulemap
│   │   │   └── SnapKit.release.xcconfig
│   │   ├── Then
│   │   │   ├── Then-Info.plist
│   │   │   ├── Then-dummy.m
│   │   │   ├── Then-prefix.pch
│   │   │   ├── Then-umbrella.h
│   │   │   ├── Then.debug.xcconfig
│   │   │   ├── Then.modulemap
│   │   │   └── Then.release.xcconfig
│   │   ├── Toast-Swift
│   │   │   ├── ResourceBundle-Toast-Swift-Toast-Swift-Info.plist
│   │   │   ├── Toast-Swift-Info.plist
│   │   │   ├── Toast-Swift-dummy.m
│   │   │   ├── Toast-Swift-prefix.pch
│   │   │   ├── Toast-Swift-umbrella.h
│   │   │   ├── Toast-Swift.debug.xcconfig
│   │   │   ├── Toast-Swift.modulemap
│   │   │   └── Toast-Swift.release.xcconfig
│   │   ├── WeakMapTable
│   │   │   ├── WeakMapTable-Info.plist
│   │   │   ├── WeakMapTable-dummy.m
│   │   │   ├── WeakMapTable-prefix.pch
│   │   │   ├── WeakMapTable-umbrella.h
│   │   │   ├── WeakMapTable.debug.xcconfig
│   │   │   ├── WeakMapTable.modulemap
│   │   │   └── WeakMapTable.release.xcconfig
│   │   └── nanopb
│   │       ├── ResourceBundle-nanopb_Privacy-nanopb-Info.plist
│   │       ├── nanopb-Info.plist
│   │       ├── nanopb-dummy.m
│   │       ├── nanopb-prefix.pch
│   │       ├── nanopb-umbrella.h
│   │       ├── nanopb.debug.xcconfig
│   │       ├── nanopb.modulemap
│   │       └── nanopb.release.xcconfig
│   ├── Then
│   │   ├── LICENSE
│   │   ├── README.md
│   │   └── Sources
│   │       └── Then
│   │           └── Then.swift
│   ├── Toast-Swift
│   │   ├── LICENSE
│   │   ├── README.md
│   │   └── Toast
│   │       └── Toast.swift
│   ├── WeakMapTable
│   │   ├── LICENSE
│   │   ├── README.md
│   │   └── Sources
│   │       └── WeakMapTable
│   │           └── WeakMapTable.swift
│   └── nanopb
│       ├── LICENSE.txt
│       ├── README.md
│       ├── pb.h
│       ├── pb_common.c
│       ├── pb_common.h
│       ├── pb_decode.c
│       ├── pb_decode.h
│       ├── pb_encode.c
│       └── pb_encode.h
├── container
│   └── GTM-KK4GLVB4.json
└── fastlane
    └── report.xml

```

<br>



## 4. 역할 분담

### 🧑🏻‍💻신승욱

- iOS 개발 및 카페 키오스크(AOS) 개발

# iOS

### 1. **Login / Logout**
1. 로그인
- 기본적인 유효성 검사를 진행후 로그인 버튼을 활성화 하는 방식입니다.
- UserDefaults를 이용하여 한번이라도 로그인하고 로그아웃 이력이 없는 사용자면 90일 자동로그인을 걸어놓았습니다.

<img src="https://github.com/user-attachments/assets/0ad813b1-130b-4688-9476-e0be3ecfcedf" width="300">
<br><br>

2. 로그아웃
- 로그아웃을 할 경우 UserDefaults에 저장되어 있던 유저 정보를 삭제하고 로그인 화면으로 이동합니다.

<img src="https://github.com/user-attachments/assets/fe47d289-918d-43bd-a424-1b0b76bc0362" width="300">
<br><br>



### 2. **홈 화면**
1. 당월 몇잔 주문 했는지 카운팅
2. 상단, 하단 배너(자동 슬라이드 배너)
- 상단 배너는 카페의 공지사항으로 이동하는 클릭 이벤트가 있습니다.
- 하단 배너 광고는 귀사의 프로젝트인 Tdeal로 이동하는 광고 배너입니다.


3. 현재 카페 상황 체크(현재 대기 건수, 예상 대기 시간)
- 웹훅을 이용하여 현재 대기 건수 및 예상 대기시간을 체크합니다(대기 건수 당 30초를 잡아 대략적으로 체크합니다.)

<img src="https://github.com/user-attachments/assets/b0d8dd58-609e-4074-8f78-d7b833329799" width="300">

<br><br>

### 3. **주문 화면**
1. 개인, 업무용 주문 분기
- 개인, 업무용 주문을 나누었습니다. 개인 주문이지만 일회용기로 주문 할 경우 돈을 받고 텀블러는 무료로 제공 됩니다. 업무용 주문은 사유와 회의실을 적어야 주문이 가능합니다.
- 업무용 주문은 카트에 담아 주문이 가능합니다.
<img src="https://github.com/user-attachments/assets/4b148984-0ebd-4ab6-b983-0fe1f41992a2" width="300">
<br><br>

2. 주문 프로세스
- 퍼스널 옵션은 서버에서 정해주는 default값이 있으며 컵이나 HOT / ICE를 선택할 경우 자동으로 선택됩니다.

<img src="https://github.com/user-attachments/assets/f33e4698-9261-4dc3-98a7-a0abce126399" width="300">
<img src="https://github.com/user-attachments/assets/65d72d31-13aa-4728-bf72-f87f1b172282" width="300">
<img src="https://github.com/user-attachments/assets/785317d3-ec26-4297-a836-6efe72e7d890" width="300">



<br><br>

### 4. **마이 페이지**
1. 주문내역(기간을 설정하여 해당 기간에 주문 내역을 확인 가능)
- 이번달, 저번달, 오늘로부터 365일 확인 가능합니다.

<img src="https://github.com/user-attachments/assets/755efcb7-af46-4f87-810d-389d61ce145b" width="300">
<br><br>

2. 공지사항(사내 공지사항이나 카페 공지사항을 해당 탭에서 확인 가능)
<img src="https://github.com/user-attachments/assets/fcd4104f-a790-4f96-b312-2434e236b9d6" width="300">
<br><br>
   
3. 버전 정보(현재 앱의 버전을 확인하고 업데이트가 필요하다면 앱스토어로 이동)
<img src="https://github.com/user-attachments/assets/32435976-5dca-4a77-b286-c6c8f49975c7" width="300">
<br><br>

4. 만든 사람들
<img src="https://github.com/user-attachments/assets/079c5bfb-3889-40d9-9f4a-1d09e42a9801" width="300">
<br><br>

### 5. **푸시 알림 커스텀**
1. 푸시 이미지와 아이콘을 변경하여 푸시 커스텀 하여 노출

#### 커스텀 Push
- NoitificationServiceExtension과 Intents를 이용하여 카카오톡 처럼 메신저 형태의 푸시 알림이 가능한데 해당 기능을 이용

<img src="https://github.com/user-attachments/assets/162e7221-d1f3-4cf4-9135-041dee4d15ab" width="300">
<br><br>

#### 일반 Push
<img src="https://github.com/user-attachments/assets/8a74f3cb-747b-4428-8404-ac492da4affe" width="300">

<br><br>

### 6. **알림 센터**
1. 푸시 알림 모아보기
<img src="https://github.com/user-attachments/assets/ef71b009-60ce-4456-aebd-674bfd9bdcab" width="300">

<br><br>

<br><br>

# AOS
### 안드로이드
- Kiosk를 안드로이드로 만들었습니다. 주문을 진행하고 마지막에 사원증을 NFC Reader에 태그해 결제하는 방식의 개발을 진행했습니다.

## 5. 개발 기간 및 작업 관리

### 개발 기간

- 전체 개발 기간 : 2023-07-11 ~ 2024-12-10
- 기능 구현 : 2024-08-01 ~ 2024-12-05

<br>

## 6. 신경 쓴 부분

1. 실 사용자가 사웓들이기 때문에 팀 단위로 테스트 진행한 결과 주문이 몰리면 딜레이 된다는 것을 파악하였고 백엔드에서 대응했습니다. 앱도 맞추어 주문이 많아 딜레이가 된다는 Toast 노출
2. 개발자이자 실 사용자이기 때문에 클라이언트 입장에서 어느것이 더 불편할 것인지 체크하며 개발하였습니다.
3. 키오스크는 안드로이드로 만들어졌는데 NFC태그시에 payload에 NFC 고유번호를 실어 서버에 요청하는 방식입니다. 중복되는 NFC는 없는지 물리적으로 NFC Reader가 빠지진 않는지 체크했습니다.


<br>

## 7. 개선 목표

- 앱 위젯 개발 예정
- 단축어 개발 예정
- 퀵메뉴 개발 예정
- 워치 연동 개발 예정
- 각종 사내에서 필요한 기능들 추가 개발 예정
    
<br>

## 8. 프로젝트 후기

### 🧑🏻‍💻 신승욱

- 처음 입사하게되어 PL로 참여하게된 프로젝트 입니다. 개발한 다른 앱 보다 애정이 더 많이 가는 앱이고, 성공적으로 앱 배포하게 되어 정말 기뻤습니다 출근길에 제가 만든 앱을 사용하는 사원들을 보니 뿌듯함이 이루 다 말할 수 없습니다 😃 
- 안드로이드도 동시 개발이 이루어졌는데 폰트, 버튼 모양, 버튼 크기 등.. 싱크를 맞추려고 많이 노력했습니다.
- 결과적으로 사내 정책으로 텀블러 사용을 앱이 제작됨과 동시에 텀블러 사용을 도모했으며 나름 환경 운동에 참여한 셈이 되었습니다..☺️
<br>

