---
layout: post
title: "[iOS] viewDidLoad() VS loadView()"
author: "이남준"
date: 2021-07-06
background: 'https://developer.apple.com/swift/images/swift-og.png'
tags: 
    - iOS
---

오프오프의 iOS 앱은 앱의 UI 작업을 Interface Builder가 아닌 SnapKit과 Then의 조합으로 코드를 통해 구현했습니다.  
코드로 view를 그렸기 때문에 보통 IB를 통해 view를 구성할 때와 여러가지 차이점이 있지만 그중 한가지인 ```loadView()```메소드 사용에 대해 ```viewDidLoad()```와 비교를 통해 알아보겠습니다.

## UIViewController의 Life-Cycle

iOS의 ViewController는  
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=http%3A%2F%2Fcfile26.uf.tistory.com%2Fimage%2F2613D13C58C64DE32C838B">  

위와 같은 단계로, 생성부터 해제까지 큰 줄기의 생명주기를 가지고,  
View Controller가 가려졌다가 다시 보일 때, 다시 가려질 때 또한 특정 함수가 실행됩니다.

## viewDidLoad()

```viewDidLoad()```는 가장 익숙한 메소드 중 하나로 Xcode에서 View Controller를 새로 만들면 자동으로 생성되어있고, 여러가지 초기화 작업을 이 메소드 안에서 하게됩니다.

<img width="914" alt="스크린샷 2021-07-06 오후 2 30 39" src="https://user-images.githubusercontent.com/22260098/124547198-bdf65500-de66-11eb-965d-334b90b82d7d.png">

애플 공식문서 설명에 따르면 ```viewDidLoad()```는 view가 메모리에 로드된 후 실행이 되고,  
view가 nib file을 통해 정의가 되었던, loadView()에서 코드를 통해 정의가 되었던 상관 없이 호출이 됩니다.  

이렇게 ```viewDidLoad()```에서는 이미 로드된 view에 추가적인 초기화 작업이 필요할 때 사용이 되는 것을 알 수 있습니다.

## loadView()

```loadView()```메소드는 ```viewDidLoad()```의 설명에서처럼, viewControlleer.view를 만드는 역할을 하는 메소드입니다.  

애플 공식문서 설명에 따르면 ```loadView()```는 직접 호출하면 안되고, View Controller가 view를 요청받았는데, view가 정의되어 있지 않다면 실행이 되고, 시점상 ```viewDidLoad()```보다 먼저 실행됩니다.  

<img width="933" alt="스크린샷 2021-07-06 오후 2 56 34" src="https://user-images.githubusercontent.com/22260098/124549613-5d691700-de6a-11eb-9691-22edf451c1f0.png">  

또한 Interface Builder를 통해 View를 정의할 경우 이 메소드를 오버라이드 하면 안되기 때문에, 내가 코드로 정의한 커스텀 뷰를 사용해서 view를 정의하고 싶을 때만 오버라이드 해야합니다.  

그리고 ```loadView()```로 직접 view를 정의 할 때는 ```super.loadView()```를 호출 하면 안되고, ```self.view = SomeView()```처럼 명시적으로 초기화를 해줘야합니다.

### 예시

```swift
class MyClass: UIViewController {
    override func loadView() {
        // super.loadView() x
        self.view = .init()         // 방법 1 - 기본 UIView() 리턴
        self.view = MyCustomView()  // 방법 2 - 커스텀 뷰로 지정
    }
}
```

## 결론

코드로 view를 구성할 때, ```loadView()``` 메소드를 통해 필요한 view를 먼저 초기화 시키고, 그 이후에 호출되는 ```viewDidLoad()```에서 남은 작업을 해준다.