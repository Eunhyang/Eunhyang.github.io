---
layout: post
title: "two scoops of django 4장"
description: "TIL:django"
date: 2018-10-15
tags: [TIL]
comments: true
share: true

---

## Two scoops of django 4장

- 장고 앱 디자인의 황금률
  - 좋은 장고 앱을 정의하고 개발하는 것은 **한 번에 한 가지 일을 하고 그 한 가지 일을 매우 충실히 하는 프로그램을 짜는 것**
    - 어떤 앱의 성격과 기능을 한 문장으로 설명할 수 없거나 그 앱을 설명하기 위해 '그리고/또한'이란 단어를 한 번 이상 사용해야 한다면, 이미 그 앱은 너무 커질 대로 커져서 여러 개로 나누어야할 때!
    - 앱들을 될 수 있으면 작게 유지하자. 거대한 크기의 앱 두세 개를 구성하는 것보다 작은 앱 여러 개를 구성하는 것이 훨씬 나은 구성이다
- 비공통 앱 모듈
  - 일반적으로 공통 모듈로 존재하지않는 모듈이 있다.
  - behaviors.py, constants.py context_processors.py, decorators.py, db/, exceptions, fields.py, factories.py, helpers.py, mangaers.py, middleware.py, signals.py, utils.py, viewmixins.py
