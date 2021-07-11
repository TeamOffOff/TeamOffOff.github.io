---
layout: post
title: "[Android] Retrofit2 라이브러리로 서버와 통신하기"
author: "Park Yu Jin"
date: 2021-07-12
background: 'https://media.vlpt.us/images/bonimddal2/post/d5947941-13d4-4772-ac13-927fb2cf1ba7/0_nZkN6lvnF2cGhLof.png'
tags: Android
---
<br>

## ❓ Retrofit2?
- [Retrofit](https://github.com/square/retrofit)은 Square사에서 만든 라이브러리로 서버와 통신을 하기 위해 HTTP API를 자바나 코틀린의 인터페이스 형태로 변환해 사용할 수 있도록 해줍니다.
- 요즘은 안드로이드 개발 시, 통신 부분은 대부분 Retrofit 라이브러리를 사용한다고 합니다.
- AsyncTask나 Volley와 비교했을 때 응답 속도가 훨씬 빠릅니다.

# 예시

오프오프의 Android 앱은 서버와의 통신을 Retrofit2 라이브러리를 사용해 구현할 것이기 때문에, Test로 Github API를 통해 리포지토리 목록을 불러와 보려고 합니다.

## ◻️ 준비 !
### ① Internet 퍼미션 추가
- Manifest 파일에 Internet permission 추가해야 합니다.
- 서버와의 통신이 인터넷을 거쳐서 이루어지기 때문입니다.
```xml
    <uses-permission android:name="android.permission.INTERNET"/>

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
```

### ② Retrofit2 라이브러리 추가
- build.gradle(module 수준)에 Retrofit2 라이브러리 종속성을 추가해줍니다.
- [Retrofit Github](https://github.com/square/retrofit)에서 최신 버전을 확인할 수 있습니다.
```
    // Retrofit2 라이브러리
    def retrofit_version = "2.9.0"
    implementation "com.squareup.retrofit2:retrofit:$retrofit_version"
    implementation "com.squareup.retrofit2:converter-gson:$retrofit_version"
```
- 첫 번째 implementation : Retrofit 라이브러리
- 두 번째 implementation : Retrofit에 Converter로 함께 사용되는 Gson 라이브러리
   - gson이 Json을 자바 클래스로 바꿔 주는 역할을 해 따로 변환 객체를 구현하지 않아도 되는 것이 큰 장점입니다.
   
### ③ API 문서 확인하기

- `BASE URL`? 데이터를 요청할 서버의 이름이고,
- `BASE URL` 뒤에 붙는 것에 따라 서버에 어떤 것을 요청하는 API인지 정해집니다.

## ◻️ 본격적으로 Retrofit 사용해보기 !
### ① Retrofit을 사용하기 위한 API 인터페이스 생성
```kotlin
interface GithubService {
    @GET("/users/{username}/repos?sort=created")
    @Headers("Autorization: token 발급받은 키")
    fun getRepos(@Path("username") username: String): Call<List<GithubRepo>>
}
```
- 인터페이스를 생성하고 안에 Request 메소드를 작성합니다.
- GET Request를 요청해야 하므로 @GET 어노테이션을 이용해 서버에 GET 요청을 할 주소를 입력합니다.
- @Path 어노테이션은 동적으로 바뀔 수 있는 URL의 부분, Request 요청을 보낼 때의 매개변수를 뜻합니다.
   - 사용자 이름을 입력 받고(getRepos메소드의 파라미터로 넘어온 값), 그 입력받은 이름을 요청보내야 합니다.
- Request 메소드가 호출된 후 리턴하는 Call< T >타입
   - Call< T > ? 여러 메소드들을 정의하고 있는 인터페이스
   - T는 성공적으로 응답될 <u>response body type의 클래스</u>-> 안드로이드에서 data class로 사용. 즉, <> 안에는 성공적으로 응답될 body type의 data class 넣어주면 됩니다.
   - 반환을 비동기식으로 처리합니다.

### ② data class 작성
응답 데이터 형식은 `json`형식
-> 서버에서 응답 받는 실데이터는 json 형식, 클라이언트에서 응답 받을 데이터 타입은 data class 객체
- 따라서, 서버로부터 받은 json 형식의 데이터를 클라이언트에서는 data class 객체로 변환되어야 통신이 가능합니다.
- 서버에 데이터를 보낼 때에도 클라이언트에서 data class 객체를 json 형식의 데이터로 변환해서 보내야 서버가 받을 수 있습니다.
- json <-> data class 를 왔다 갔다 변환하는 기능은 Retrofit 라이브러리에 아까 추가한 Gson 라이브러리를 추가로 사용하면 가능합니다.
```kotlin
data class GithubRepo(
    @SerializedName("name")
    val repoName: String,
    @SerializedName("created_at")
    val date: String,
)
```
- SerializedName은 json 형식 데이터로 변환된 이름이라는 것입니다.
   - @SerializedName 어노테이션은 data class 변수 이름을 서버 쪽 json 포맷 데이터의 key 이름과 동일하게 사용하고 싶지 않을 때 작성해주면 이름을 다르게 사용할 수 있게 도와줍니다.
- test로 리포지토리의 이름과, 생성된 날짜를 받아 출력해보려고 합니다.
   
### ③ Retrofit 인터페이스 구현체 생성
- 앱에서 서버 호출이 필요한 곳마다 인터페이스를 사용해야 하는데, 인터페이스를 여러 번 구현하지 않고, 한 번만 구현해 놓고 필요한 곳에서 사용하려면?
   - 코틀린의 `object` 사용
   - `object`는 singleton 이기 때문에 전역 변수처럼 앱의 모든 곳에서 접근 가능
  
```kotlin
// singleton(메모리를 하나만 씀)
object RetrofitClient {
    // retrofit client 선언

    // BASE_URL을 private const 변수로 저장
    private const val BASE_URL = "https://api.github.com/"

    val retrofit = Retrofit.Builder()
    .baseUrl(BASE_URL)
    .addConverterFactory(GsonConverterFactory.create())
    .build()

    val githubService: GithubService = retrofit.create(GithubService::class.java)
}
```
- Retrofit.Builder를 통해 baseUrl을 설정하고, converter를 GSON Converter로 설정해 build()메소드 호출 -> Retrofit 객체 생성
- request하기 위해서는 client가 필요합니다.
```kotlin
val githubService: GithubService = retrofit.create(GithubService::class.java)
```
retrofit.create(클라이언트 클래스(인터페이스))를 통해 클라이언트 생성
- 만든 클라이언트에서 정의된 메소드(여기선 getRepos 메소드)를 통해 Request를 보냅니다.

### ④ Request 메소드를 호출하고, 서버로부터 응답 받아 처리하기
```kotlin
RetrofitClient.githubService.getRepos(username).enqueue(object : Callback<List<GithubRepo>> {
			// 서버 통신 실패 시의 작업
            override fun onFailure(call: Call<List<GithubRepo>>, t: Throwable) {
                Log.e("실패", t.toString())
            }
            
			// 서버 통신 성공 시의 작업
            // 매개변수 Response에 서버에서 받은 응답 데이터가 들어있음.
            override fun onResponse(call: Call<List<GithubRepo>>, response: Response<List<GithubRepo>>) 
            {
            	// 응답받은 데이터를 가지고 처리할 코드 작성
                val repos:List<GithubRepo>? = response.body()
                repos?.forEach { it ->
                    Log.d("성공", it.toString())
                }
            }
        })
```
- 앞의 과정에서 API 인터페이스를 구현했고, 이 구현체는 object를 사용해 singleton으로 구현해놨었습니다.
- 따라서 그 인터페이스 안에 구현된 getRepos()라는 Request 메소드를 호출해줍니다.
- 그 다음은 Response를 받기 위해 enqueue 메소드를 사용해줍니다.
   - Response를 받는 방식은 동기`execute()` / 비동기`enqueue(Callback<T> callback)`가 있습니다.
   - 서버와 통신하는 것은 시간이 오래걸리는 작업이다. 따라서 동기방식으로 통신하면 통신이 완료될 때까지 다른 작업이 이루어지지 못하고 앱 상에서 UI가 멈춰보이게 되며 모든 기능이 중단됩니다.
   - 따라서, **비동기식 통신**을 해 다른 작업을 처리하다가 통신 결과를 받으면 다시 작업을 처리할 수 있도록 해주어야 효율적입니다.
- 서버로부터 Response를 받지 못했을 때 onFailure(), 그 외에 onResponse()를 호출
- 위의 코드에서는 response의 body()를 로그에 출력해보았습니다.

## ◻️ 결과 !

![image](https://user-images.githubusercontent.com/57751515/124610481-83fb7200-deab-11eb-9da6-3382f7617d49.png)

이렇게 Retrofit2 라이브러리를 통해 서버와 통신해 데이터를 불러와 출력해보았습니다.
response의 body()가 로그에 출력되는 것을 볼 수 있었습니다.