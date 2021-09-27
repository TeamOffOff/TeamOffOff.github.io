---
layout: post
title: "[Android] Android에서 Realm DB로 데이터 관리하기"
author: "박유진"
date: 2021-09-28
background: ''
tags: Android
---
![image](https://user-images.githubusercontent.com/57751515/134942276-4790f42d-5283-4e8d-8670-16158b59f45f.png)

Team OFF 안드로이드 앱에서 스케줄 관리 기능을 구현하면서, 로컬에 데이터를 저장하기 위해 어떤 DB를 사용해야 할지 고민이 많았습니다. 

안드로이드에서 데이터를 저장하는 방식에는 여러가지가 있습니다.

**1. SharedPreferences**
- 적은 양의 원시 데이터를 (key, value) 형태로 앱 안에 XML 파일로 저장합니다.

**2. Room**
   - AAC(Android Architecture Component) 중 하나입니다.
- SQLite 위에 있는 추상화 레이어를 제공해 SQLite의 기능을 사용하면서 데이터베이스 접근을 보다 편리하게 합니다. 
- 객체의 맵핑을 통해 DB에 접근하는 ORM 방식의 라이브러리입니다.

**3. Realm**
- 쿼리문을 통해 테이블내 컬럼에 데이터를 저장하는 SQLite의 방식과 달리, 데이터를 객체 형태로 저장합니다.
- 크로스 플랫폼 데이터베이스로 Android, iOS 간 DB 공유가 가능합니다.
- 타 데이터베이스들에 비해 데이터 처리 속도가 빠르다.

➡️ 이렇게 여러 방식을 찾아보고 비교해보면서, **최종적으로는 Realm을 선택하게 되었습니다.**

# Why Realm ❓

우선, SharedPreference는 자동 로그인 여부와 같은 간단한 값을 저장할 때엔 적합하지만, 저장할 데이터의 양이 많고 간단하지 않은 경우에는 적합하지 않다는 생각이 들었습니다.

Room과 Realm 사이에서 고민이 많았지만, Realm은 모델 구조 자체가 객체 형식으로 구성되어 있기 때문에 사용이 직관적인 것이 큰 장점이라고 생각해 Realm을 선택하게 되었습니다.

데이터 컨테이너 모델을 사용해 직접 객체를 데이터베이스에 저장하는 방식이기 때문에 저장, 불러오기와 같은 동작들도 좀 더 빠르게 처리할 수 있습니다.
   
# Realm 사용하기

## ① 초기 설정
- app 레벨의 build.gradle에 플러그인 추가
```kotlin
plugins {
    id "realm-android" // 가장 하단에 위치시키기
}
```
- project 레벨의 build.gradle에 의존성 추가
```kotlin
dependencies {
        classpath 'io.realm:realm-gradle-plugin:10.8.0'
        // 최신 버전은 realm 릴리즈 노트 확인
    }
```

## ② Realm 초기화하기
realm 초기화는 앱을 실행하는 동안 한 번만 하면 되므로 Application class에서 해주는 것이 좋습니다.
```kotlin
class OffoffApplication : MultiDexApplication() {

    override fun onCreate() {
        super.onCreate()

        // Realm 초기화
        Realm.init(this)
        val config : RealmConfiguration = RealmConfiguration.Builder()
            .allowWritesOnUiThread(true)
            .name("user.realm") // 생성할 realm db 이름
            .deleteRealmIfMigrationNeeded()
            .build()

        Realm.setDefaultConfiguration(config)
    }
}
```

## ③ 객체 클래스 모델 만들기
```kotlin
open class User  (
    @PrimaryKey
    var id: Int? = 0,
    var name: String? = null,
    var age: Int? = 0,
) : RealmObject()
```
- 클래스는 open 키워드를 붙여주고 RealmObject를 상속받아야 합니다.
- PK로 사용할 필드는 @PrimaryKey 어노테이션을 붙여줍니다.

## ④ Realm 인스턴스 얻기
realm을 사용할 곳에서 realm 인스턴스를 가져와 사용할 수 있습니다.
```kotlin
val realm = Realm.getDefaultInstance()
```

## ⑤ 데이터 다루기 (CRUD)
**Create, Update - 데이터 저장 또는 갱신하기** 
```kotlin
// 사용자 저장 또는 갱신하기
fun insert(user: User) {
     realm.executeTransactionAsync {
          it.insertOrUpdate(user)
     }
}
```
**Read - 데이터 읽기** 
```kotlin
// 모든 사용자 읽어오기
fun getAllUser(): RealmResults<User> {
     return realm.where<User>()
         .findAll()
         .sort("id", Sort.ASCENDING)
}
        
// 특정 사용자 읽어오기
fun getUser(id: Int): User? = realm.where<User>()
    .equalTo("id", id)
    .findFirst()
```
**Delete - 데이터 삭제하기** 
```kotlin
// 모든 사용자 삭제
fun deleteAllUser() {
    realm.executeTransaction {
        it.where<User>().findAll().deleteAllFromRealm()
    }
}

// 특정 사용자 삭제
fun deleteUser(id: Int) {
    realm.executeTransaction {
        it.where<User>().equalTo("id", id).findAll().deleteAllFromRealm()
    }
}
```

## 마치며...
간단하게 안드로이드에서의 Realm 사용 방법을 알아보았습니다. 처음 접해보는 것이다 보니 초반에 Realm과 정을 붙이는 데 조금 힘들었지만 지금은 훨씬 익숙해진 것 같아 뿌듯합니다 :) 

> 참고:  
> https://developer.android.com/training/data-storage/room
> 
> https://docs.mongodb.com/realm-legacy/docs/kotlin/latest/index.html
> 
> https://github.com/realm/realm-kotlin