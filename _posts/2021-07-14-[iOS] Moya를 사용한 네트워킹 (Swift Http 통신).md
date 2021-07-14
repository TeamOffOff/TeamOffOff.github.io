---
layout: post
title: "[iOS] Moya를 사용한 네트워킹 (Swift Http 통신)"
author: "이남준"
date: 2021-07-14
background: ''
tags: iOS
---
# Why Moya 🤔

![](https://github.com/Moya/Moya/blob/master/web/diagram.png?raw=true)

iOS에서 네트워킹을 구현하는 가장 기본적인 방법은 ```URLSession```을 사용하는 것입니다.  

```URLSession```은 로우레벨의 코드를 작성할 수 있고, 다른 프레임워크를 사용할 필요가 없다는 장점이 있지만, 사용이 복잡하고 코드의 가독성이 좋지 않아서 Foundation Networking을 기반으로한 인터페이스를 제공해 네트워킹 작업을 단순화 해주는 라이브러리인```Alamofire```를 많이 사용합니다.  

```Alamofire```는 iOS 앱을 개발할 때 가장 보편적으로 많이 사용되는 방법이지만, 유지보수와 유닛 테스트가 힘들다는 단점이 있습니다.

그래서 등장한 ```Moya```는 ```URLSession```을 추상화한 ```Alamofire```를 다시 추상화한 프레임워크로 Network Layer를 템플릿화 해서 재사용성을 높히고, 개발자가 request, response에만 신경을 쓰도록 해줍니다.

# Moya 사용해보기 ⌨️

```Moya```의 사용법을 알아보고 실제 사용을 해보기 위해 샘플 프로젝트를 하나 만들어 테스트 해보겠습니다.

## 준비

### 1. 프로젝트 생성
![스크린샷 2021-07-14 오전 10 08 16](https://user-images.githubusercontent.com/22260098/125550836-33b150e1-d42c-46e6-8f77-64ebde477edb.png)

### 2. Moya 설치

```Moya```는 Swift Package Manager, CocoaPods, Carthage를 사용하여 인스톨할 수 있기 때문에 본인에게 가장 익숙하고 편한 방법으로 설치하면 됩니다.

### 3. 사용할 API

```Moya```는 네트워킹 프레임워크이기 때문에, 테스트를 하기 위해서는 Http 통신을 지원하는 api가 꼭 필요합니다.

![스크린샷 2021-07-14 오전 11 28 02](https://user-images.githubusercontent.com/22260098/125551134-86eb5172-b105-4217-beb4-6fb2a51ad46b.png)
[ICNDb.com](http://www.icndb.com)

이번 포스트에서는 
___Chuck Norris once ordered a Big Mac at Burger King, and got one.___
와 같은 척 노리스 밈(조크)를 모아둔 ```ICNDb.com```에서 랜덤한 조크를 리턴해주는 api를 사용하겠습니다.

### 4. UI 세팅

![스크린샷 2021-07-14 오전 11 36 54](https://user-images.githubusercontent.com/22260098/125551914-f4cff463-6b0b-43da-9df8-2233b038b5a2.png)

## 코드 작성

[템플릿](https://github.com/Moya/Moya/blob/master/docs/Examples/Basic.md) ```Moya```에서 제공해주는 템플릿을 따라 코드를 작성해보겠습니다.

### 1. API target enum

새로운 파일을 하나 만들고 ```JokeAPI.swift```  
enum을 하나 선언해서 사용될 target들을 작성합니다.

```swift
// JokeAPI.swift
import Moya

enum JokeAPI {
    case randomJokes(_ firstName: String? = nil, _ lastName: String? = nil)
}
```

이 예제에서는 이름을 파라미터로 받아 조크의 이름을 만들어주는 target을 하나만 사용하겠습니다.

### 2. ```TargetType``` 구현

```Moya```는 ```MoyaProvider<TargetType>```으로 request를 수행하기 때문에, 위에서 정의한 API가 ```TargetType``` 프로토콜을 구현해야합니다.

extension을 만들어 ```TargetType``` 프로토콜을 채택하면, 아래와 같은 프로퍼티들을 구현 해야합니다.

- baseURL: 서버의 endpoint 도메인
- path: 도메인 뒤에 추가 될 path (/users, /documents, ...)
- method: HTTP method (GET, POST, ...)
- sampleData: 테스트용 Mock Data
- task: request에 사용될 파라미터 .requestPlain: no param, .requestParameters(parameter:, encoding:)
- headers: HTTP Header

```swift
extension JokeAPI: TargetType {
    var baseURL: URL {
        return URL(string: "https://api.icndb.com")!
    }
    
    var path: String {
        switch self {
        case .randomJokes(_, _):
            return "/jokes/random"
        }
    }
    
    var method: Moya.Method {
        switch self {
        case .randomJokes(_, _):
            return .get
        }
    }
    
    var sampleData: Data {
        switch self {
        case .randomJokes(let firstName, let lastName):
            let firstName = firstName
            let lastName = lastName
            
            return Data(
                """
                {
                    "type": "success",
                    "value": {
                        "id": 107,
                        "joke": "\(firstName)\(lastName) can retrieve anything from /dev/null."
                    }
                }
                """.utf8
            )
        }
    }
    
    var task: Task {
        switch self {
        case .randomJokes(let firstName, let lastName):
            let params: [String: Any] = [
                "firstName": firstName,
                "lastName": lastName
            ]
            return .requestParameters(parameters: params, encoding: URLEncoding.queryString)
        }
    }
    
    var headers: [String : String]? {
        return ["Content-type": "application/json"]
    }
}

```
위처럼 ```switch```문을 통해 여러 target에 맞는 프로퍼티들을 정의해줄 수 있습니다.

### 3. Request 보내고 Response 받아서 처리하기

이제 위에서 정의한 ```TargetType```을 이용해 request를 보내는 코드를 작성하겠습니다.

response 시 받는 JSON의 형태는 아래와 같습니다.
```json
{ 
    "type": "success", 
    "value": {
         "id": 268, 
         "joke": "Time waits for no man. Unless that man is John Doe." 
    }
}
```

이와 대응되는 Decodable struct를 정의하고:

```swift
struct Joke: Decodable {
    var type: String
    var value: Value

    struct Value: Decodable {
        var id: Int
        var joke: String
    }
}
```

이제 request를 보내고 받은 response를 text view에 표시해보도록 하겠습니다.

```swift
class ViewController: UIViewController {
    @IBOutlet var jokeTextView: UITextView!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let provider = MoyaProvider<JokeAPI>()
        provider.request(.randomJokes("GilDong", "Hong")) { (result) in
            switch result {
            case let .success(response):
                let result = try? response.map(Joke.self)
                self.jokeTextView.text = result?.value.joke
            case let .failure(error):
                print(error.localizedDescription)
            }
            
        }
    }
}
```

# 결과

<img height="600" src="https://user-images.githubusercontent.com/22260098/125572709-df2ae53c-eaaf-43f9-b523-9e2f58cbf537.png">

이처럼 네트워크 요청을 ```Moya```를 통해 처리하면 enum을 활용해 안전한 방식으로 캡슐화된 요청을 할 수 있다는 장점이 있고, 코드를 직관적이고 쉽게 작성할 수 있다는 장점이 있습니다.

> 참고: 
> [Moya Github](https://github.com/Moya/Moya)
> [Moya basic usage](https://github.com/Moya/Moya/blob/master/docs/Examples/Basic.md)