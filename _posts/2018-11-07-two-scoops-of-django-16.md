---
layout: post
title: "two scoops of django 16장"
description: "TIL:django"
date: 2018-11-06
tags: [TIL]
comments: true
share: false
    
---

## Two scoops of django 16장 - REST API 구현하기

* 기본 REST API 디자인의 핵심

  | 요청 목적                                      | HTTP 메서드 | 대략적으로 매칭되는 SQL 명령어 |
  | ---------------------------------------------- | ----------- | ------------------------------ |
  | 새로운 리소스 생성                             | POST        | INSERT                         |
  | 리소스 읽기                                    | GET         | SELECT                         |
  | 리소스의 메타데이터 요청                       | HEAD        |                                |
  | 리소스의 데이터 업데이트                       | PUT         | UPDATE                         |
  | 리소스의 부분 변경                             | PATCH       | UPDATE                         |
  | 리소스 삭제                                    | DELETE      | DELETE                         |
  | 특정 URL에 대해 지원되는 HTTP 메서드 출력      | OPTIONS     |                                |
  | 요청에 대한 반환 에코                          | TRACE       |                                |
  | TCP/IP 터널링(일반적으로 구현되어 있지는 않음) | CONNECT     |                                |

  * 참고) PUT vs PATCH
    * PUT은 해당 자원이 **통채로** 교체되는 것
    * PATCH는 해당 자원의 **일부분**만 변경되는 것

* REST API 아키텍처

  * 상호 긴밀히 연관된 작은 앱으로 구성된 프로젝트이고, API가 어디에 위치하고 있는지 찾기 힘들 때 => API만 전담하는 앱 따로 만들기

    * 단점) API앱이 너무 커질 수 있고, 각 API를 지원하는 개별 앱으로 부터 해당 API앱이 단절 됨

  * 너무 많은 REST API 뷰 클래스를 포함하는 대규모 프로젝트의 경우 **분할하기**

    * 단점) 작게 나뉜 상호 연관되는 앱이 너무 많이 존재하게 된다는 것. 모듈들이 너무 작게 나눠지고 뷰가 많아지면 API 컴포넌트들을 관리하기 어려워짐

* 서비스 지향 아키텍쳐

  * 서비스 지항 아키텍처의 관점에서 웹 어플리케이션은 독립적이고 분리된 컴포넌트로 구성됨
    * =>많은 엔지니어가 상호 충돌없이 서로 분리된 컴포넌트를 작업하기 수월하게 하기위해

* API에 접속 제한하기

  * 제한없는 API접속은 위험함
  * REST Framework는 반드시 접속 제한을 해야 함

