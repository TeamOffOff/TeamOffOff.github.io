---

layout: post
title: "[Backend] Python Flask를 활용한 Restful API 서버 구축"
author: "김유민"
date: 2021-07-18
background: ''
tags: python
---
<br>

## Flask란?

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/3c/Flask_logo.svg/1920px-Flask_logo.svg.png" alignment="center">



Python ```Flask```는 간단한 웹 사이트나 API 서버를 만들때 사용되는 Web Framework입니다.

서버를 간단하게 작성할 수 있고 확장하는 것이 용이하여 Micro Web Framework라고도 불립니다.

흔히 파이썬을 활용해 웹 서버를 구축할때 사용하는 ```Django```와 함께 가장 많이 사용되는 라이브러리라고 볼 수 있습니다.

```Django```에 비하면 비교적 가볍게 작성된 프레임워크이기 때문에 ``` Flask```에서 자체적으로 제공하는 기능이 많지는 않지만,

사용자가 커스터마이징하고 싶은 부분을 자유롭게 구현할 수 있다는 장점이 있습니다.



## Flask 설치

```bash
$ pip install flask
```

가상 환경을 사용하고 있다면 위 한 줄로 쉽게 설치가 가능합니다.



## Flask 사용법

앞서 설명한 바와 같이 ```Flask```에서는 매우 간단하게 서버를 만들 수 있습니다.

```python
from flask import Flask

app = Flask(__name__)


@app.route('/')
def hello():
    return 'Hello World!'


if __name__ == "__main__":
    app.run(debug=True, host="0.0.0.0", port=5000)
```

위 코드는 "Hello World!" 라는 내용을 띄우는 ```Flask``` 서버입니다.

`app = Flask(__name__)` 는 ```Flask``` 객체를 생성한다는 의미입니다.

main에서 `app.run` 함수를 실행시키게 되면 설정된 host와 port로 서버가 실행됩니다.

로컬 환경에서 테스트 할 경우 0.0.0.0으로 설정하게 되면 알아서 127.0.0.1(localhost) 환경으로 실행됩니다.

따라서 `localhost:5000` 으로 들어가 보면 다음 화면을 볼 수 있습니다.



![helloworld_flask](https://user-images.githubusercontent.com/23094883/126060276-bd6a10b9-2f49-43d8-8bab-1e83967e5403.PNG)



`@app.route("/")` 라는 데코레이터는 app 객체를 실행하였을때 URI를 통해 어떠한 함수로 routing을 해줄지를 표시하는 함수입니다.

해당 부분을 ```@app.route("/hello")``` 라는 경로로 바꾼 후 다시 접속해보겠습니다.

참고로 `debug=True` 로 설정해놓으면 파일을 수동으로 다시 실행시키지 않아도 잠시 후 스스로 다시 실행되어 코드가 적용됩니다. 



![notfound_flask](https://user-images.githubusercontent.com/23094883/126060330-37d3fde2-b979-4407-8abf-d5298c081945.PNG)



그럼 다시 ```localhost:5000``` 이라는 경로로 접속했을때 Not Found라는 화면을 보여줍니다.

현재 실행된 ```app``` 객체에서는 해당 경로로 routing을 해줄 위치가 없기 때문입니다.

그렇다면 바꿔준 URI로 접속한다면 어떻게 될까요?



![helloworld2](https://user-images.githubusercontent.com/23094883/126060392-fc6a6356-8198-4431-b3e5-c88d35c63ee0.PNG)



뒤에 ```/hello```를 붙여주니 정상적으로 Hello World!가 출력된 것을 볼 수 있었습니다.

이처럼 실행할 각 함수에 ```app.route("")``` 데코레이터를 붙여주면 원하는 URI로 원하는 함수를 통해 사용자에게 무엇을 보여줄지를 구현할 수 있습니다.



## Flask Restful API 서버 만들기

그럼 지금까지는 ```Flask```를 이용해 일반 웹 서버를 구축하는 방법에 대해 알아보았습니다.

또한 웹 서버와 마찬가지로 ```Flask```를 이용하면 Restful API 서버 또한 간단하게 구현할 수 있습니다.

각 API를 가볍게 구현할 수 있고 기능 확장이 쉽기 때문에 API 서버로 많이 이용이 된다고 하네요!

그렇다면 지금부터 ```Flask```를 이용하여 데이터 통신을 위한 Restful API 서버를 구축하는 방법을 알아봅시다.



## Flask Restful API 서버를 위한 라이브러리 설치

```bash
$ pip install flask-restx
```

```flask_restx```는 ```Flask``` 내에서 간단하게 restful api를 구현할 수 있게 해주는 라이브러리입니다.



## Flask Restful API 서버 사용법

```python
from flask import Flask
from flask_restx import Api, Resource

app = Flask(__name__)
api = Api(app)


@api.route("/hello")
class HelloWorld(Resource):
    def get(self):
        return {"hello": "world!"}


if __name__ == "__main__":
    app.run(debug=True, host='0.0.0.0', port=5000)
```

위에서 웹 서버와는 다르게 변한 부분이 조금씩 보입니다.

첫 번째로 `api = Api(app)` 이라는 코드가 생겼는데, 이는 윗 줄에서 만든 ```Flask``` 객체에 ```Api``` 객체를 등록하는 역할을 합니다.

또한 `@api.route("/hello")`로 바뀐 것을 볼 수 있습니다.

이는 위 `@app.route("/hello")`와 같은 역할을 한다고 보시면 됩니다.



그리고 함수대신에 클래스에 데코레이터를 사용합니다.

`HelloWorld`라는 클래스는 import한 `Resource`를 상속받아 만들어집니다.

함수 이름은 restful api에서 지원하는 method인 get, post, put, delete로 설정해주시면 해당 method로 들어온 요청이 함수 명에 맞게 실행됩니다.

이제 테스트를 해봅시다!



![image](https://user-images.githubusercontent.com/23094883/126061238-3db92859-6f8c-4ed2-8b5d-3a73a768ccac.png)

api 테스트로는 advanced rest client라는 프로그램을 이용하였습니다.

```Flask``` 객체를 실행시킨 host, port와 api 객체에 등록해준 경로를 담은 URI를 GET method로 보냈습니다.

결과는 json type으로 받아지는데 ```{"hello":"world!"}```로 잘 통신된 것을 볼 수 있었습니다.

현재는 SSL(Secure Sockets Layer)을 따로 설정해주지 않았기때문에 http로만 통신이 가능합니다.

추후에 SSL 인증을 통하여 키를 제공해주면 https를 사용하여 보안성을 더욱 높일 수 있다고 합니다.



## Flask Restful API Method 테스트

```python
from flask import Flask, request
from flask_restx import Api, Resource

app = Flask(__name__)
api = Api(app)

data = {}
count = 1


@api.route("/data")
class DataPost(Resource):
    def post(self):
        global data
        global count

        idx = count
        count += 1
        data[idx] = request.get_json()["data"]

        return {
            "data_id": idx,
            "data": data[idx]
        }


@api.route("/data/<int:data_id>")
class DataControl(Resource):
    def get(self, data_id):
        try:
            return {
                "data_id": data_id,
                "data": data[data_id]
            }
        except KeyError:
            return{
                "mission": "fail"
            }

    def put(self, data_id):
        data[data_id] = request.get_json()["data"]
        return {
            "data_id": data_id,
            "data": data[data_id]
        }

    def delete(self, data_id):
        global count

        del data[data_id]
        count -= 1
        return {
            data_id: "delete"
        }


if __name__ == "__main__":
    app.run(debug=True, host='0.0.0.0', port=5000)

```

위 코드는 data를 보내 저장하고 저장한 값을 받아오고, 수정하고, 삭제하는 기능을 하는 코드입니다.

```DataPost``` 클래스는 post method를 이용하여 body로만 값을 받고, ```DataControl``` 클래스는 get, put, delete method를 이용하여 URI로부터 값을 받아 분리하여 구현하였습니다.

통상적으로 각 method들이 하는 역할은 다음과 같습니다.

| Method  | POST               | GET              | PUT              | DELETE           |
| ------- | ------------------ | ---------------- | ---------------- | ---------------- |
|         | Create             | Read             | Update           | Delete           |
| example | 새로운 사용자 추가 | 사용자 정보 조회 | 사용자 정보 수정 | 사용자 정보 삭제 |

위 코드에서도 각 method들은 이 동작에 대한 기능을 하고 있습니다.

그럼 각각의 method를 테스트해보겠습니다.



![image](https://user-images.githubusercontent.com/23094883/126062048-18f251c9-6eb8-4a6c-9a46-f5761960ae92.png)



첫 번째로 POST method입니다.

hello라는 데이터를 보내주면 내용을 읽어와 ```data``` 라는 dictionary에 저장해주게 됩니다.



![image](https://user-images.githubusercontent.com/23094883/126062097-954f8661-d857-4a1a-9f72-bddcf8b1eb8c.png)



다음은 GET method입니다.

```/data``` 뒤에 찾고자 하는 data의 id를 입력해주게 되면 그것을 인자로 받아 ```data``` dictionary에서 그 내용을 찾아 리턴해주는 기능을 합니다.



![image](https://user-images.githubusercontent.com/23094883/126062134-444b6837-a6be-4ce6-8386-d7b59186f4d2.png)



세 번째는 PUT method입니다.

수정하고자 하는 id를 마찬가지로 뒤에 입력해준 후 수정할 내용을 전달하면, 해당 id를 찾아 수정하는 기능을 합니다.



![image](https://user-images.githubusercontent.com/23094883/126062195-58304e7e-ca3f-4efc-8b8d-477e8c1a7e5f.png)



![image](https://user-images.githubusercontent.com/23094883/126062253-5e5a8f84-c7c0-4c9b-aa93-098f96d73552.png)



마지막으로 DELETE method입니다.

삭제하고자 하는 id를 입력해주고 나면 해당 id의 data는 삭제됩니다.

만약 삭제한 id를 다시 찾으려고 한다면 ```data``` dictionary 내에 해당 id가 없으므로 KeyError가 발생할 것 입니다.

그럼 이것을 try, except 문으로 처리하여 해당 아이디가 없다는 것을 사용자에게 리턴해줄 수 있습니다.



## 마무리

지금까지 ```Flask```를 이용하여 기본 웹 서버 구성과 이를 이용한 Restful APi 서버를 구축하는 작업을 해보았습니다.

```Flask```를 이용하면 편하게 Restful API를 구축할 수 있다는 장점답게 매우 간단한 코드로도 잘 통신되는 API 서버를 구축한 것을 확인할 수 있었습니다.

그럼 다음 시간에는 이를 이용하여 데이터베이스와 통신하는 방법을 포스팅하도록 하겠습니다.



> 참고: https://justkode.kr/python/flask-restapi-1

