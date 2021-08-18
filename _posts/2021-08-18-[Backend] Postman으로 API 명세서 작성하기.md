---
layout: post
title: "[Backend] Postman으로 API 명세 작성하기"
author: "김소연"
date: 2021-08-18
background: ''
tags: python
---
<br>


# STEP1. API 명세 작성 방법 선택  

```FrontEnd``` 와 ```BackEnd```는 API를 통해 소통합니다.  
따라서 API 명세서 작성이 **굉장히** 중요합니다.  

API 명세서를 작성하는 방식은 크게 다음과 같이 구분할 수 있습니다.
1. 엑셀 파일 
2. restx 라이브러리를 활용한 방식 
3. postman을 활용한 방식

<br>

저희 팀이 ```postman```을 선택한 이유는 다음과 같습니다.

1. 백앤드 팀이 api를 테스트 하면서 바로 명세서를 작성할 수 있다 ! 
2. ```request```시 ```url```, ```params(query string)```, ```method```, ```headers```, ```body```와 ```response``` 를 한 눈에 볼 수 있다 ! 


<br>
<br>

# STEP2. Postman 명세서 작성 방법
<br>

### 1. ```WorkSpace```, ```Collection```, ```Request``` 생성  
   
- 우선 워크스페이스와 컬랙션을 생성합니다.  
- 주로 컬랙션은 프로젝트 단위로 생성합니다.
- 하지만, 저희 팀에서는 한 문서에서 모든 api를 확인하기 위해 폴더를 활용하였습니다.  

<img width="1739" alt="시작" src="https://user-images.githubusercontent.com/84752537/129885983-dd05cf07-e8f2-44bd-ae7d-f45b46b259e0.png">


<br>
<br>
<br>

### 2. ```global 변수``` 설정

- 여러 종류의 api를 테스트 해야하는 경우, uri에서 authority 부분을 변수로 설정해두면 편리합니다.  
- host를 0.0.0.0으로 설정하고 main.py를 실행시키면, 네트워크에 따라 authority 부분이 변경됩니다.  
- 이때 authority를 변수로 설정해두면 일일이 uri를 수정할 필요 없이 변수의 value만 수정하면 됩니다.

<img width="1728" alt="global 변수 설정 및 사용" src="https://user-images.githubusercontent.com/84752537/129886532-3525253d-c621-441d-8649-dbd6b946e3ff.png">

<br> 
<br> 
<br> 

### 3. response 저장
- response(요청 결과)를 예시로 저장할 수 있습니다.

<img width="512" alt="response 저장" src="https://user-images.githubusercontent.com/84752537/129887724-a17bf1a0-656d-4f41-b9ab-4a5277b19bc9.png">

<br>
<br>
<br>

### 4. 컬랙션 publish 
- postman은 3인까지 무료로 워크스페이스를 공유할 수 있습니다.
- 컬렉션을 공유해서 api 명세를 확인하는 방법 이외에도, publish를 통해 api 명세서를 공유할 수 있습니다.

<br>
<br>

#### 1) 도큐먼트 생성
<br>
<img width="1746" alt="document 생성 시작" src="https://user-images.githubusercontent.com/84752537/129888071-a437a248-6cfd-4c66-a7ca-b3115c968107.png">

<br>
<br>
<br>

#### 2) 컬렉션 선택
<br>
<img width="1463" alt="document 생성" src="https://user-images.githubusercontent.com/84752537/129888129-96e60585-4373-4a76-94d6-826fc9d86c8f.png">

<br>
<br>
<br>

#### 3) 도큐먼트 상세보기
<br>
<img width="518" alt="도큐먼트 상세보기" src="https://user-images.githubusercontent.com/84752537/129888213-303333c6-12b5-4e8b-9a19-d96864c0b762.png">

<br>
<br>
<br>

#### 4) publish 클릭
<br>

<img width="1735" alt="publish 클릭" src="https://user-images.githubusercontent.com/84752537/129888262-540e893e-74f5-4d1d-806b-8c226b1d4616.png">
<img width="622" alt="api 명세 publish" src="https://user-images.githubusercontent.com/84752537/129888356-68c91d8f-8aaa-480e-a746-8377a311a5d6.png">

<br>
<br>
<br>

#### 5) url 확인
<br>
<img width="768" alt="url 확인" src="https://user-images.githubusercontent.com/84752537/129888401-26e8cf0c-bf0e-48d1-a9e1-c59114f96a5c.png">

- 컬렉션에서 새로운 사항을 추가, 수정하는 경우 이 도큐먼트에 즉시 반영됩니다
- 따라서 해당 url로 api 명세서를 바로 확인할 수 있습니다.  

<br>
<br>

_다음 시간에는 pymongo의 리스트 관련 연산자에 대해 다루겠습니다_  
_비밀번호 암호화(bcrypt), 토큰(jwt)에 대한 내용은 다다음 시간 !!_ 

##  감사합니다 :)
