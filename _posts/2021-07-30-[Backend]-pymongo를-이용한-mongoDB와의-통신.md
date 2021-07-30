---
layout: post
title: "[Backend] Pymongo를 이용한 mongoDB와의 통신"
author: "김유민"
date: 2021-07-30
background: ''
tags: python
---

<br>

## mongoDB란?

![image](https://user-images.githubusercontent.com/23094883/127590401-c8244b9a-ebc5-4733-8a8a-03312a3cbbdf.png)

mongoDB는 NoSQL 기반의 document 지향 데이터베이스입니다.

MySQL과 같은 관계형 데이터베이스가 아니며 SQL을 사용하지 않습니다.

기존 관계형 데이터베이스와 mongoDB에서 사용되는 용어는 다음과 같이 대입됩니다.

| RDBMS       | MongoDB     |
| ----------- | ----------- |
| Database    | Database    |
| Table       | Collection  |
| Record(Row) | Document    |
| Column      | Field       |
| Index       | Index       |
| primary key | _id field   |
| group by    | aggregation |

관계형이 아니기 때문에 mongoDB는 데이터를 더 유연하게 다룰 수 있고 확장성이 좋은 것이 장점입니다.

또한 데이터의 생김새들이 제각각이더라도 데이터베이스 Schema에 맞춰서 작업할 필요가 없는 Schema-less한 구조이기 때문에 대용량 데이터 작업에도 강점을 가지고 있습니다.



## mongoDB & pymongo 설치

mongoDB를 사용하기 위해서는 첫 번째로 mongoDB를 설치해주셔야합니다.

설치는 아래 링크에서 하실 수 있습니다.

https://www.mongodb.com/try/download/community



python에서 mongoDB와 연동하기 위해서는 pymongo라는 라이브러리가 필요합니다.

설치는 아래와 같이 하시면 됩니다.

```bash
$ pip install pymongo
```



## pymongo 연결하기

```python
from pymongo import MongoClient

client = MongoClient('localhost', 27017)

if __name__ == "__main__":
    print(client.list_database_names())
```

python에서 mongoDB에 연결하려면 먼저 `MongoClient` 객체를 생성해야 합니다.

이때 넘겨주어야할 파라미터는 실행할 서버의 IP 주소와 port 번호 두 가지를 넘겨주면 됩니다.

이 파라미터 대신 서버의 URI를 적어주어도 됩니다.

결과는 아래와 같이 나옵니다.

![image](https://user-images.githubusercontent.com/23094883/127606799-6659c4be-fa5e-4d65-8b16-b17cb6e89192.png)

지금 test_db로 나와있는 것은 제가 테스트하면서 만들어 놓은 db이고 원래는 admin, config, local 이 세 가지가 기본적으로 나오게 됩니다.

이를 확인하셨다면 정상적으로 mongoDB에 연결되었다고 볼 수 있습니다.

그럼 mongoDB안에 있는 데이터베이스에 접근하는 방법은 무엇일까요?

```python
db = client.test_db
db = client["test_db"]
```

두 개의 구문은 같은 동작을 합니다.

mongodb에 접근하는 방법은 두 가지 방법이 있습니다.

하나는 메소드 형태로 접근하는 방법과 나머지는 string 형태로 접근하는 방법입니다.

이는 collection에서도 똑같이 작동합니다.

아래 두 개의 구문은 같은 동작을 합니다.

```python
collection = db.test_collection
collection = db["test_collection"]
```



## mongoDB에 데이터 추가, 조회, 수정, 삭제

이제 collection에 접근하는 방법을 알았으니 데이터를 추가해보도록 합시다.

데이터를 추가하는 방법은 아래와 같습니다.

```python
data = {
        "text": "Hello World!",
        "date": "2021-07-30"
    }

db = client["data_db"]
collection = db["data_collection"]

inserted_id = collection.insert_one(data).inserted_id
```

mongoDB에서는 데이터를 json으로 다루기 때문에 dictionary 형태의 데이터를 준비합니다.

그리곤 pymongo에서 제공하는 `insert_one()` 함수를 사용하면 쉽게 데이터를 넣을 수 있습니다.

만약 해당 이름의 데이터베이스나 collection이 없다면 자동으로 생성되니 따로 생성하실 필요는 없습니다.

이때 id가 따로 지정되어 있지 않다면 자동으로 `_id` 가 생성됩니다.

그것을 받을 수 있는 것이 `inserted_id`인데 값은 아래와 같이 나옵니다.

![image](https://user-images.githubusercontent.com/23094883/127623792-472f24ba-642b-4c0c-ae34-0844aae97b26.png)

해당 `_id`는 ObjectId 타입으로 document 내에 자동으로 생성이 됩니다.

얼핏보면 그저 랜덤하게 나온 문자열 같지만 `_id`는 해당 document가 생성된 시간에 대한 정보를 담고 있습니다.



![image](https://user-images.githubusercontent.com/23094883/127624328-ca2bb54f-9cd1-49f1-a56e-00f7e0ea6df8.png)

<center>https://www.mongodb.com/blog/post/pymongo-monday-pymongo-create</center>



```python
from bson import ObjectId

ObjectId("6103b602b01f0d727bcca409").generation_time
```

![image](https://user-images.githubusercontent.com/23094883/127624925-7016739e-05db-4fc2-8d9f-6258732bac68.png)

bson 라이브러리(pymongo 설치 시 같이 설치)에서 ObjectId를 import 하신 후 `_id` 가 언제 생성됐는지 `generation_time` 을 통하여 구할 수 있습니다.

이를 통해 데이터를 최신순으로 불러오거나 특정 날짜에 쓰여진 글을 가져올 수 있습니다.



```python
db = client["data_db"]
collection = db["data_collection"]

collection.find_one({"_id": ObjectId("6103b602b01f0d727bcca409")})
```

위 코드는 특정 `_id`의 document를 찾아주는 역할을 합니다.

결과는 아래와 같이 나옵니다.

![image](https://user-images.githubusercontent.com/23094883/127626428-fd01fd86-9a82-4da9-820b-ab3b260386bb.png)

주의하실 점은 `_id`로 찾으실땐 string 타입이 아닌 ObejctId 타입으로 찾아줘야한다는 것입니다.

또한 쿼리문 없이 `collection.find_one()` 만 실행시켜줄 경우 가장 처음 만들어진 document를 반환합니다.

`find_one()` 함수 자체가 가장 처음 만나는 결과를 반환해주는 것이라 그런 특성이 있는 것 같습니다.

만약 다량의 데이터를 찾고 싶다면 `collection.find()` 함수를 사용하면 되고, 결과는 iterable한 Cursor 타입이 반환됩니다.

해당 함수는 쿼리문에 부합하는 조건을 가진 데이터를 모두 반환해주는 것입니다.



```python
collection.update_one({"_id": ObjectId("6103b602b01f0d727bcca409")}, {"$set": {"text": "Goodbye World!"}})
collection.find_one({"_id": ObjectId("6103b602b01f0d727bcca409")})
```

위 코드는 해당 `_id`를 가진 document의 text라는 field를 Goodbye World!로 수정하라는 뜻입니다.

![image](https://user-images.githubusercontent.com/23094883/127628227-c20bc3a0-bacf-4246-b443-96679dc91a3a.png)

결과는 위와 같습니다.

update_one의 쿼리문 뒤 인자는 `update` 라는 인자인데 이 인자는 `{"$set":{수정할 내용 dictionary}}` 의 형태로 작동합니다.

마찬가지로 `update_many()` 함수로 조건에 맞는 다수의 문서를 업데이트합니다.

```python
collection.update_one({"text": "Hello Hello World!"}, update={"$set": {"text": "Goodbye World!"}}, upsert=True)
```

`update_one()` 함수에는 `upsert` 라는 인자가 있습니다.

이는 해당 조건이 없으면 `update` 라는 인자의 내용을 가진 document를 생성하라는 뜻입니다.

![image](https://user-images.githubusercontent.com/23094883/127629178-96c1db96-f86f-460f-84e3-921537a2cbcb.png)

저희의 collection에는 `text` 가 `Hello Hello World!` 인 documnet는 없기 때문에 위와 같이 새로운 document를 생성해주는 것을 확인할 수 있습니다.

해당 코드의 `update` 인자에는 `text` 에 관한 내용밖에 없으므로 `date` field는 없는 document가 생성되었습니다.

참고로 `update()` 함수도 있긴하지만 deprecated한 상태라 `update_one()` 과 `update_many()` 를 사용하는게 좋아보입니다.



```python
collection.delete_one({"_id": ObjectId("6103bf33785e866dc3e5f530")})
collection.find_one({"_id": ObjectId("6103bf33785e866dc3e5f530")})
```

이제 `date` field가 없는 데이터를 삭제하고 싶으니 삭제해봅시다.

`delete_one()` 함수에 해당 `_id` 를 가진 document를 삭제하라고 한 뒤 `find_one()` 함수를 이용하여 찾아보면 해당 document를 찾을 수 없기 때문에 None이 반환됩니다.

데이터 삭제도 마찬가지로 `delete_many()` 함수로 조건에 맞는 다량의 데이터를 삭제할 수 있습니다.



## 마무리

지금까지 mongoDB와 python에서 이를 사용하게 해주는 pymongo 라이브러리를 알아보았습니다.

기본적인 데이터 추가, 조회, 수정, 삭제에 대해 알아보았는데, 이를 이용하여 간단하게 데이터베이스를 사용하고 싶을때 손쉽게 사용할 수 있을 것 같네요!



```
> 참고: https://coding-start.tistory.com/273
```

