---
layout: post
title: "[Android] ViewBinding을 통해 뷰와 상호작용하기"
author: "박유진"
date: 2021-07-30
background: ''
tags: Android
---

이전에는 안드로이드 코드단에서 view의 컴포넌트들을 객체로 만들어 사용하고 컨트롤하기 위해서 `findViewById()` 메소드를 많이 사용했었습니다.

```kotlin
val textView = findViewById<TextView>(R.id.tv_id)
```
위 코드와 같이 뷰를 참조하기 위해 `findViewById()` 메소드를 이용하는 방법은 몇 가지 문제가 있습니다.

1. 컴포넌트가 100개라면, 컴포넌트의 개수대로 위와 같은 코드를 100번 추가해 주어야 합니다. 🤮
   -> 이로 인해 코드 양이 불필요하게 많아집니다.
2. 레이아웃의 파일 트리를 순회하며 해당 컴포넌트를 찾기 때문에 무겁고 속도가 느립니다.
3. 실수로 존재하지 않는 컴포넌트의 id를 인자로 전달하면 Null Pointer Exception이 발생하기 때문에 null-safety하지 못합니다.

이러한 `findViewById()`를 대체하고 문제점들을 해결하기 위해 `viewBinding`이 등장하게 됩니다. 

# View Binding ❓

`viewBinding`은 뷰의 컴포넌트를 객체화시키지 않고 바로 사용할 수 있어 불필요한 코드 작업을 줄입니다. 
- `findViewById()`와의 비교
  1. Null Safety 보장
  binding class에서 이미 뷰의 직접 참조를 가지므로 존재하지 않는 id를 사용할 일이 없어 Null Pointer Exception 발생 위험이 줄어듭니다.
  2. Type Safety 보장
  binding class에 있는 필드의 타입이 뷰와 일치하므로 Class Cast Exception의 발생 위험이 줄어듭니다.
  
# View Binding 사용 방법

## 초기 세팅

module 수준의 build.gradle에서 `viewBinding` 사용을 설정합니다.
```kotlin
android {
        ...
        viewBinding {
            enabled = true
        }
    }
```
## 사용

위의 설정을 완료하면 각 XML 레이아웃 파일마다 binding class가 생성됩니다.
  - binding class의 이름은 XML 파일의 이름을 카멜 표기법으로 변환한 것 + 'Binding'의 형태입니다.
  
예시로 `user_info.xml` 레이아웃 파일이 있고, 그 파일의 코드는 아래와 같습니다.
```xml
<LinearLayout ... >
    <ImageView android:cropToPadding="true" />
  	<TextView android:id="@+id/tv_id" />
  	<Button android:id="@+id/bt_save" />
</LinearLayout>
```
- binding class의 이름은 `UserInfoBinding`이 되고, 자동으로 id가 존재하지 않는 ImageView를 제외한 TextView와 Button의 참조를 가지게 됩니다. 
- 모든 binding class는 레이아웃 파일의 root view에 관한 직접 참조를 제공하는 `getRoot()` 메소드를 가집니다. 예시의 경우 `getRoot()` 메소드는 LinearLayout을 리턴하게 됩니다.

## Activity에서의 사용

```kotlin
    // 1. lateinit 예약어를 사용해 선언
    private lateinit var binding: UserInfoBinding
    
    // 2. activity의 onCreate() 메소드에서 생성
    override fun onCreate(savedInstanceState: Bundle) {
        super.onCreate(savedInstanceState)
        
        // 3. inflate() 메소드를 호출해 인스턴스 생성
        binding = UserInfoBinding.inflate(layoutInflater)
        
        // 4. getRoot() 메소드를 호출해 root view 참조를 가져옴
        val view = binding.root
        
        // 5. root view를 setContentView()에 전달해 화면상의 활성 뷰로 만들기
        setContentView(view)
    }
```

이제 binding class의 인스턴스를 사용해 view 참조가 가능합니다.
```kotlin
 binding.tvId.text = viewModel.name
 binding.btSave.setOnClickListener { viewModel.userClicked() }
```

## Fragment에서의 사용

```kotlin
    // 1. fragment binding class를 null로 초기화
    private var _binding: UserInfoBinding? = null
    private val binding get() = _binding!!

    // 2. onCreateView() 메소드에서 binding class 인스턴스 설정하기
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
    
        // 3. inflate() 메소드를 호출해 인스턴스 생성
        _binding = UserInfoBinding.inflate(inflater, container, false)
        
        // 4. getRoot() 메소드를 호출해 root view 참조를 가져옴
        val view = binding.root
        
        // 5. root view를 리턴해 화면상의 활성 뷰로 만들기
        return view
    }
	
    // 6. binding class를 null 처리
    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
```
이제 위의 `Activity에서의 사용` 에서 설명한 것과 같이 binding class의 인스턴스를 사용해 view 참조가 가능합니다.

⚠️ 주의할 점

프래그먼트는 뷰보다 수명이 깁니다.
-> 따라서 프래그먼트의 lifecycle에서 뷰가 제거될 때 호출되는 `onDestroyView()` 콜백 메소드에서 binding class의 인스턴스를 null 값으로 변경하여 참조를 정리해주어야 합니다. 

# +) Data Binding ❓ 

`dataBinding`은 [Android JetPack 라이브러리](https://developer.android.com/jetpack?hl=ko) 중 하나로 xml 레이아웃 파일에 data를 결합해 `findViewById()` 메소드를 사용하지 않아도 되고, 결합된 데이터가 변할 때 view에 변경된 데이터를 쉽게 반영할 수 있는 장점이 있습니다. 
- MVVM 패턴을 구현할 때 `LiveData`와 함께 많이 쓰입니다.
- `viewBinding`과 같이 뷰를 직접 참조하는데 사용 가능한 binding class를 제공합니다.

# Data Binding VS View Binding

`viewBinding`과 `dataBinding`은 서로 비교했을 때, 각각의 이점을 가지고 있습니다.

## View Binding의 이점

- 컴파일이 더 빠릅니다.
- `dataBinding`의 경우, xml 레이아웃 파일에 따로 tag 처리를 해주어야 하지만 `viewBinding`은 그러한 과정이 필요하지 않기 때문에 사용이 편리합니다.

## Data Binding의 이점

- Two-way binding 을 이용할 수 있어 뷰의 데이터가 변경 될 때마다 데이터를 가져올 수 있습니다.

# 느낀 점

   ![tenor](https://user-images.githubusercontent.com/57751515/127556101-a5274a10-dfe0-4b9e-999f-a0f16a4bb324.gif)
- 현재 오프오프의 Android 앱에서는 `viewBinding`과 `dataBinding`을 통해 뷰를 참조하고 있는데, 이전에 `findViewById()` 메소드를 사용해 뷰를 참조했을 때보다 훨씬 더 편리하고 안전하게 코드를 짤 수 있었습니다.
- 프로젝트에서 `viewBinding`과 `dataBinding` 중 무조건 하나만 골라 사용하기 보다는 상황에 따라서 적절한 바인딩 방법을 선택해 사용하는 것이 좋은 방법인 것 같습니다.

> 참고: 
> [Android Developer 공식문서](https://developer.android.com/topic/libraries/view-binding?hl=ko)