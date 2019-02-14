---
layout: post
title: iOS 12.1 rootlessJB3 Xcode 빌드하여 root 접속
description: "rootlessJB3 Xcode 빌드하여 root 접속"
modified: 2019-02-14
tags: [mobile, iOS]
---

* 디바이스 정보 : iPhone 6s (iOS 12.1)


## rootlessJB란?
> ..?
* rootlessJB3 지원 버전 : iOS 12.0 - 12.1.2

## rootlessJB 다운
[rootlessJB3](https://github.com/jakeajames/rootlessJB3) : https://github.com/jakeajames/rootlessJB.git

![](https://user-images.githubusercontent.com/16396760/52755226-92c5fd80-3040-11e9-960e-68ae37707626.png)



## Xcode 빌드
1. Xcode에서 Clone을 이용하여 git repository에 있는 소스코드 다운로드
    ![](https://user-images.githubusercontent.com/16396760/52755223-8e99e000-3040-11e9-81b7-13201982ebfa.png)

2. `https://github.com/jakeajames/rootlessJB3.git` 입력
    ![](https://user-images.githubusercontent.com/16396760/52755230-95285780-3040-11e9-9a60-9e0a300627f2.png)

3. 프로젝트 선택하여 Identity, Signing(Apple 계정) 입력 후 빌드 
    ![](https://user-images.githubusercontent.com/16396760/52755234-99547500-3040-11e9-96b8-4477b4bd19e5.png)

## iphone 실행
1. 정상 빌드 후 `설정 -일반-프로파일` 신뢰하는 개발자 인증서로 확인
    ![](https://user-images.githubusercontent.com/16396760/52759751-631ef180-3050-11e9-8aef-86dfb3f25ad5.PNG)

2. 터미널 접속 필요 시 iSuperSu 앱에서 ServerAuditor를 Unsandbox 후 root/alpine 터미널 접속 가능
    ![](https://user-images.githubusercontent.com/16396760/52759752-64501e80-3050-11e9-91b9-849ba7b86e8f.PNG)

