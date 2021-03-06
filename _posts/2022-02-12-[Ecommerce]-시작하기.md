---
layout: post
title:  "[Ecommerce]-시작하기"
date:   2022-02-12 12:34:53 +0900
categories: Ecommerce
tags : django MongoDB Vue.js
---

오늘부터 E-commerce project를 진행하려고 한다. 아키텍쳐 설계부터 차근차근 S/W 공학에서 배운대로 작성해볼 계획이다!
열심히 해서 우리나라 최고의 CTO가 되든, 내 사업을 할때든 꼭 필요한거 같아서! 일단 해봅시다!


S/W 개발계획에 필수적인 3가지 산출물은 아래와 같다

   * [S/W 개발 계획서](#S/W-개발-계획서-작성)
   * 요구사항 분석서
   * 시스템 설계서


## S/W 개발 계획서 작성
---
S/W 개발 계획서에 필요한 내용 어떤 방법론인지에 따라서
다르지만 혼자 하는 Project이기에 에자일 방법론(Xp)는 제한된다

따라서 전통적인 S/W 개발 방법론을 따를 것이며 작성해야할 사항은 아래와 같다

    1. 개요
      1) 프로젝트 개요
        1. 제목
        2. 개요
        3. 관련 어플리케이션
        4. 유사시스템 분석
      2) 예상효과
    2. 일정계획
        2.1. 업무분해도(WBS)
        2.2. CPM 소작업 리스트 
        2.3. CPM 네트워크
        2.4. 임계경로
        2.5. 간트 차트
    3. 자원 및 비용
        3.1. 자원
            3.1.1. 인력자원
        3.2. 비용
            3.2.1. 인건비
            3.2.2. H/W 및 구입비용
            3.2.3. 총 소요 비용
    3. 타당성 분석
        4.1. 경제적 타당성
            4.1.1. 시장성 
            4.1.2. 재무예측 및 수익성 분석 
            4.1.3. 손익분기점 
        4.2. 기술적 타당성
        4.3. 법적 타당성 
        4.4. 운영적 타당성 
    4. 조직구성 및 인력배치
        1. 조직구성
            5.1.1. 프로젝트 구조 
            5.1.2. 조직 구조도
        5.2. 프로젝트 팀 담당자 및 역할
    5. 표준 및 개발 절차
        6.1. 개발 방법론
        6.2. 개발 환경
    6. 기술관리 방법
        7.1. 변경관리 
        7.2. 위험관리 
        7.3. 비용 및 진도관리 
        7.4. 문제점 해결 방안 
            7.4.1. Boehm이 제시한 10대 위험 요소에 따른 해결 방안 
            7.4.2. 기타 위험요소에 따른 해결 방안
    7. 검토회의
        8.1. 검토회의 일정 
        8.2. 검토회의 진행 방법 
        8.3. 검토회의 후속조치 

하지만 나는 혼자하는 프로젝트 이기에! 아래사항만 작성해서
진행 하려고 한다!

  * [일정(Gantt Chart)](#Gantt-Chart)
  * [개발 환경 설정](#개발-환경-설정)

개발 계획서에 작성되는 내용은 다양하다. 본인이 어떤 사항을 추가하고 진행할지는 PM의 몫 인듯!


### Gantt-chart
---

간트차트 작성은 꼭 필요한것 같다. 예를 들면...입시 준비를 위해
공부를 할때 쓰는 학습계획서 같달까! 항상 선생님들이 꼭 쓰라고 하는데 귀찮아서 쓰지 않지만 쓰면 진짜 좋은 .. 그런거랄까 ?

혼자하는 프로젝트여서 과할 수 있지만 일단 가즈아!

간트차트 작성 툴은 정말 많다 그중에 나는 [moday.com](https://monday.com/lang/ko/)을 썼다.


간트차트 작성 :
![이미지](/assets/images/Gantt_chart.jpg)


위에서 보듯이 2.14 ~ 3.7까지 총 22일간 서비스 배포를 목표로
작성하였다 구현내용의 세부내용은 IA가 작성되면 구체화 될것 같다.



### 개발 환경 설정
---

일단 사용할 서버를 결정해야한다.
`Azure`,`AWS` 등등 많겠지만, 나는 `digital ocean` 을 쓰기로 했다.

https://cloud.digitalocean.com/

위 링크는 참고하시길!
```
  Server : Ubuntu 20.04
  Backend : Django 3.2.12
  Frontend : Vue.js
  DataBase : MongoDB
```

일단 이렇게 사용할 예정이다! 

이렇게 까지 하는 이유는.. 사실 일반적인 코딩학원들에서 프로젝트로
진행하는 걸 혼자도 할 수있다는 걸 보여주고 싶달까 ... 찾으면 다나오니꺄 요세는 쨌든 3.2일까지 빠이팅 하는걸로!! 그럼 이만!

다음포스트에선 Spring 과 django 양대 산맥 중 django 를 선택한이유와
MongoDB를 사용한 이유에 대해서 써볼예정이다