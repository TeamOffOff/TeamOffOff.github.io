---
layout: post
title: "[iOS] iOS에 Realm 도입하기"
author: "이남준"
date: 2021-09-15
background: ''
tags: iOS
---

이번 포스트에서는 오프오프 iOS 프로젝트에 채택한 Realm의 특징과 간단한 사용법에 대해 알아보겠습니다.

# Realm ❓

> 렘? 렐름?

Realm은 모바일에 특화된 NoSQL 데이터베이스로 Swift, Objective-C, Java, Kotlin 등 다양한 SDK를 제공합니다.  

iOS에서 Realm을 사용할 경우, UserDefaults와 CoreData를 대체해 Persistent data를 저장하고 관리할 수 있습니다.

대표적인 특징으로는 ORM이 아닌 데이터 컨테이너 모델을 사용하고, 데이터 객체를 Realm에 객체 형태로 저장합니다. 그렇기 때문에 DB에서 가져온 데이터를 복잡한 가공과정 없이 바로 사용할 수 있다는 장점이 있습니다.

# Realm을 선택한 이유

iOS에서 영구적인 데이터는 저장하기 위해 사용할 수 있는 도구들은
UserDefaults, CoreData, SQLite, Realm 등이 있습니다.

UserDefaults는 String, Int 등의 간단한 단일 테이터를 저장하는데 적합하기 때문에, 객체 형태로 여러 데이터를 저장하고 관리하는데는 어려움이 있고,

CoreData는 기본으로 제공되는 데이터 저장용 프레임워크로, 객체 형태로 데이터를 관리할 수 있다는 장점이 있지만, XCode를 통해서 Entity를 생성하고, 코드로 데이터를 Read, Write하는 과정이 직관적이지 않고 사용이 불편했기 때문에 후보에서 제외했습니다.

Realm은 무료로 제공되는 데이터베이스로 
1. SQLite와 CoreData보다 작업 속도가 빠르고 
2. Cross Platform을 지원해서 안드로이드 OS와 DB 파일을 공유할 수 있고
3. Realm Studio를 통해서 DB 상태를 편하게 확인할 수 있고
4. 직관적인 코드로 작업할 수 있고
5. Rx를 지원하는 RxRealm이 존재

한다는 장점이 있어서 Realm을 채택하여 사용했습니다.

# Realm 사용법 

## 1. 설치

Realm을 Swift 프로젝트에 설치할 때 SPM, CocoPods, Carthage 모두를 사용해 설치를 할 수 있습니다.

### SPM

ProjectName.xcodeproj -> Frameworks, Libraries, and Embedded Content
에서 + 버튼을 클릭하고 아래의 url을 추가하면 됩니다.

https://github.com/realm/realm-cocoa.git

### CocoaPods

프로젝트에 podfile이 존재한다면,   
```pod 'RealmSwift', '~>10'```
을 추가하고 ```pod install```을 실행하면 됩니다

자세한 설명과 Carthage를 사용한 예시는 아래 링크에서 확인할 수 있습니다
https://docs.mongodb.com/realm/sdk/ios/install/

## 2. Model 정의

Realm에 저장할 데이터 객체를 정의할 때는 일반적인 모델을 정의하는 것처럼  
class를 만들고 필요한 property를 정의하면 됩니다.

이때 Realm에서 사용할 수 있는 형태로 만들어 주려면 class가 ```Object```를 상속해야 하고,  
Property를 정의할 때 앞에 ```@Persisted```를 붙이기만 하면 됩니다.

### Example
```swift
class Phonebook: Object {
    @Persisted(PrimaryKey: true) var number: String? // primary key로 지정
    @Persisted var name: String = ""
    @Persisted var status: String = ""

    convenience init(number: String) {
        self.init()
        self.number = number
    }
}
```

## 3. CRUD

Realm 데이터베이스에 접근하고, 원하는 작업을 하기 위해서는 

```swift
let realm = try! Realm()
```

를 통해 local default Realm 객체를 열어 사용하면 됩니다.

### Create

```swift
// 새로운 객체 생성
let phonebook = Phonebook(number: "010-1234-5678")

// write transaction block
try! realm.write { 
    realm.add(phonebook)
}
```

### Retrieve

```swift
// 모든 객체 얻기
let books = realm.objects(Phonebook.self)

// 필터링
let predicateQuery = NSPredicate(format: "name = %@", "kim") // 쿼리
let result = savedShifts.books(predicateQuery)
```

### update
```swift
let toUpdate = books[0]

try! realm.write {
    books.name = "Lee"
}

// or

try! realm.write {
    // PrimaryKey가 있는 경우, 새로운 객체의 PK와 맞는 객체를 자동으로 업데이트
    realm.add(newBook, update: .modified)
}
```

### delete
```swift
let toDelete = book[0]

try! realm.write {
    realm.delete(toDelete)
}
```


## Write Transactions

위 CRUD Example에서 add, delete와 같은 메소드를 ```realm.write``` 블락 안에서 사용한 것을 볼 수 있습니다.  
Realm은 create, update, delete 동작을 read-write operation으로 이루어진 transaction으로 관리하고 Realm을 오픈하고 객체를 수정할 때는 반드시 write transaction block 안에서 수정을 해야합니다.

데이터베이스를 수정하는 동작은 순서에 매우 예민하고, Sync를 맞추는 것이 필 수 있기 때문에 Realm에서는 Write Transaction을 사용합니다.  
Transaciton은 read-write operation의 리스트로 Realm은 한 transaciton을 하나의 operation으로 간주합니다. 

Transaction은 동시에 하나만 열릴 수 있고, 열려있는 transaction이 complete되기 전까지 Realm은 다른 write을 block 합니다.  
Transaction이 완료된 경우, Realm은 Commit을 하고 디스크에 데이터를 쓰거나, cancel하고 error를 방출합니다.

> 참고:  
>  https://docs.mongodb.com/realm/sdk/ios/quick-start/  
>  https://realm.io