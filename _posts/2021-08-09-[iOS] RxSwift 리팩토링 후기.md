---
layout: post
title: "[iOS] RxSwift 리팩토링 후기"
author: "이남준"
date: 2021-08-09
background: ''
tags: iOS
---

오프오프는 수 많은 사용자가 수 많은 네트워크 통신을 해야하는 커뮤니티 어플리케이션 이기 때문에,  
사용자 입장에서는 네트워크 요청, UI 작업이 끊기지 않고 지속되는 것처럼 보여야 더 좋은 UX를 제공할 수 있습니다.

이를 위해 비동기 처리, 메모리 관리를 위해 RxSwift를 도입하였는데,  
RxSwift에 대한 간단한 설명과 함께 후기를 남겨보겠습니다.

# RxSwift ❓

<p align="center"><img src="https://github.com/ReactiveX/RxSwift/raw/main/assets/RxSwift_Logo.png" width="40%" alt="RxSwift Observable Example of a price constantly changing and updating the app's UI" /></p>

> https://github.com/ReactiveX/RxSwift

RxSwift는 Swift를 위한 Reactive extension으로
Observable을 통해 변수, 이벤트의 변화를 감지하고, 그에 대응할 수 있도록 해주는 generic abstraction입니다.

예를들어 기존에 delegate, NotificationCenter 등을 사용해 API response, button action 등과 같은 이벤트를 처리하는 것을 더 빠르고, 가독성 좋게 코드를 작성할 수 있도록 해주고,
DispatchQueue, OperationQueue를 통해 처리하던 비동기 작업을 더 직관저이고 적은 코드로 구현할 수 있도록 해줍니다.

# Observable
> Observable Sequence

RxSwift의 Observable은 뜻 그대로, 무언가를 '관찰할 수 있다'라는 의미를 가집니다.  
예를 들어 ```Observable<Int>```라면 상태 변화를 관찰할 수 있는 Int 변수라는 의미입니다.

## Observable Life Cycle

비동기 처리를 하는 어떤 함수를 실행하고, 결과 값을 받아오기 위해서
Swift에서는 completion handler를 주로 사용했습니다.  

completion handler의 단점은 비동기 처리를 하는 함수로부터 리턴 값을 전달 받을 수 없다는 단점이 있고,  
RxSwift의 Observable을 이용하면 이 단점을 극복할 수 있습니다.

Observable은 next, error, completed 3가지 상태로 구성되어 있습니다.

### next 

<img width="796" alt="스크린샷 2021-08-09 오후 4 53 03" src="https://user-images.githubusercontent.com/22260098/128674876-97297812-b22a-4858-9bc1-f6caad7f92f5.png">

next는 스트림을 통해 연속된 값들을 배출합니다.
Observer는 이 배출된 값들을 관찰하고 구독해서 원하는 행동을 취할 수 있습니다.

### error

<img width="400" alt="스크린샷 2021-08-09 오후 4 53 03" src="https://i.imgur.com/zQrUSLZ.png">

error는 스트림에 값을 배출하던 중 에러가 발생하면 error를 배출하고 스트림을 끝냅니다.

### completed

<img width="400" alt="스크린샷 2021-08-09 오후 4 53 03" src="https://i.imgur.com/SHqnLWn.png">

completed는 더 이상 스트림에 배출할 값이 없을 때 completed를 배출하고 스트림을 끝냅니다.

# Observer (Subscribe)

위에서 설명한 Observable은 연속되거나 단일 값들을 스트림을 통해 방출을 합니다.  
방출되는 값들을 관찰하고, 활용하기 위해서는 ```subscribe```를 사용합니다.

```swift
let observable = Observable.from([1, 2, 3, 4, 5]) // 1, 2, 3, 4, 5를 방출하는 observable
observable.subscribe(
    // next로 오는 값들 구현
    onNext: {
        print($0) // 1, 2, 3, 4, 5
    }
)
```

위의 예시처럼 observable을 subscribe 하면 observable에서 방출하는 값을 확인하고, 가공하여 원하는 동작(함수)를 실행할 수 있습니다.

# RxSwift 활용

간단히 서버로부터 데이터를 받아와 테이블 뷰에 표시하는 예를 통해 RxSwift 활용법에 대해 알아보겠습니다.

먼저 Swift를 사용하면
```swift
class ViewController: UITableViewController {
    let viewModel = MyViewModel()

    override func viewDidLoad() {
        super.viewDidLoad()
        viewModel.fetchData()
        
    }

    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        viewModel.dataCount
    }
    
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let myCell = tableView.dequeueReusableCell(withIdentifier: "MyCell", for: indexPath)
        myCell.configure(indexPath.row)

        return myCell
    }

    ...
}
```
위의 코드와 같이 UITableViewDatasource의 함수들을 통해, table view cell을 정의하고 필요한 작업을 해줬습니다.

하지만 RxSwift를 이용해서 같은 작업을 구현하면
```swift
class ViewController: UITableViewController {
    let viewModel = MyViewModel()

    override func viewDidLoad() {
        super.viewDidLoad()
        viewModel.fetchData()

        viewModel.data
            .bind(to: self.tableView.rx.items(cellIdentifier: "MyCell", cellType: UITableViewCell.self)) { (row, element, cell) in
                cell.titleLabel.text = element.title
            }
            .disposed(by: disposeBag)
    }
    ...
}
```
위의 코드처럼 RxCocoa tableview를 활용하여 viewModel의 data를 구독하고, data의 변화가 있을 때 (서버로부터 데이터를 다 받아왔을 때)  
cell을 정의하고 표현하는 코드를 간결하게 작성할 수 있습니다.

# 결론

RxSwift는 러닝커브가 다소 높아 적용하는데 어려움이 있었지만, Swift로 구현해둔 코드들을 RxSwift로 바꾸면서,  
확실히 코드가 더 간결해지고, Thread, 비동기 처리 등 일일히 신경쓰기 귀찮았던 작업들을 알아서 처리해주기 때문에 좋은면이 있었습니다.