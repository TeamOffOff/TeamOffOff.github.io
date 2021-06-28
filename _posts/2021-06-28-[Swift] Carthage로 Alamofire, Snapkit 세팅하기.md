---
layout: post
title: "[Swift] Carthage로 Snapkit 세팅하기"
author: "Lee Nam Jun"
date: 2021-06-28
background: 'https://cdn2.civitatis.com/tunez/tunez/cartago-sidi-bou-said.jpg'
tags: iOS
---  
## Carthage (카르타고) ?

Carthage(카르타고)는 iOS(Cocoa)프로젝트에 외부 프레임워크를 추가하고, 관리할 수 있는 방법입니다.  
직접 프레임워크 파일을 프로젝트에 추가하거나, CocoaPods, Swift Package Manager를 사용할 수 있지만 카르타고를 사용하는 이유는 뭘까요??  

CocoaPods을 통해 프레임워크를 추가하면,  
프로젝트를 빌드할 때마다 다른 라이브러리도 같이 빌드가 되기 때문에 브레이크포인트를 줘서 디버깅할 수 있다는 장점도 있지만  
추가된 라이브러리가 많아질수록 빌드타임이 길어진다는 단점이 있습니다.  

반면 카르타고는 라이브러리 소스를 미리 빌드하고, 프로그램이 동작하는 도중 해당 모듈이 필요할 때 불러서 사용하는 방식이기 때문에 한번 빌드 후 빌드 속도가 빠르다는 장점이 있습니다.

## Carthage 사용법  

### Carthage 설치  

설치는 ```brew```를 통해 진행합니다.  

```
$ brew update
$ brew install carthage
```
  
### Cart file 생성  

CocoaPods에서 Pod file을 생성했던 것 처럼 사용할 라이브러리의 정보를 담을 Cart file을 .xcodeproj 가 있는 경로에 생성합니다.  


```
$ cd [프로젝트 경로]
$ touch Cartfile
```

### Cartfile 수정

Xcode로 Cartfile을 수정합니다.  

```
$ open -a Xcode ./Cartfile
```

### 사용할 라이브러리 작성

위 단계에서 Cartfile을 수정하기 위해 열었다면,
아래 처럼 사용하고 싶은 라이브러리를 작성하면 됩니다.
작성 양식은 아래와 같습니다.

```
github "[githubaccount]/[repo name]"
```

![스크린샷 2021-05-31 오후 6 56 09](https://user-images.githubusercontent.com/22260098/120176130-e5dc1280-c241-11eb-8740-2f7aee167cae.png)

작성 후
```
carthage update --use-xcframeworks
```
을 실행하면 프로젝트 폴더에 ```Cartfile.resolved``` 와 .xcframework 가 생성된걸 볼 수 있습니다.

![스크린샷 2021-05-31 오후 7 25 04](https://user-images.githubusercontent.com/22260098/120179724-e8406b80-c245-11eb-85bb-2fe90daa9752.png)

### Frameworks and Liabraries로 xcframework 옮기기

![image](https://user-images.githubusercontent.com/22260098/120179995-3a818c80-c246-11eb-978d-f0a24a317a45.png)

아까 생성된 .xcframework 폴더를 Xcode의 해당 부분에 드래그앤 드롭 해줍니다.

### 완료

![스크린샷 2021-05-31 오후 7 28 20](https://user-images.githubusercontent.com/22260098/120180128-5dac3c00-c246-11eb-8510-c66087648de8.png)

이제 .swift 파일에서 ```SnapKit```을 import 하고 성공적으로 빌드가 되는 것을 확인할 수 있습니다.

## 끝

이렇게 카르타고를 통해 SnapKit을 세팅해봤습니다.  
이 방법을 통해 다른 여러 라이브러리들을 추가하고 관리할 수 있습니다.