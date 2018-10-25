---
layout: post
title: "two scoops of django 9장"
description: "TIL:django"
date: 2018-10-24
tags: [TIL]
comments: true
share: true
    
---

## Two scoops of django 9장 - 함수 기반 뷰의 모범적인 이용

- 함수 기반 뷰로 작성할 때의 **가이드라인**

  - 뷰 코드는 **작을수록** 좋다
  - 뷰에서 절대 코드를 **반복해서 사용하지 말자**
  - 뷰는 **프레젠테이션 로직**을 처리해야 한다. 비즈니스 로직은 가능한 한 모델 로직에 적용시키고 만약 해야 한다면 폼 안에 내재시켜야 한다. 
  - 뷰를 가능한 한 **단순하게** 유지하자.
  - 403, 404, 500을 처리하는 커스텀 코드를 쓰는 데 이용하라.
  - 복잡하게 중첩된 `if` 블록 구문을 피하자.

- HttpReuqest 객체 전달하기

  - 뷰에서 코드를 재사용하기 원하는 경우: 미들웨어나 콘텍스트 프로세서 같은 글로벌 액션에 연동되어있지 않은경우 재사용에 문제가 생김

  - => 프로젝트 전체를 아우르는 유틸리티 함수 만들기: `django.http.HttpRequest`객체의 속성을 가져와서 데이터를 가공

  - `HttpRequest` 를 주된 인자로 삼는 것의 장점 

    - 많은 메서드와 인자 구성을 단순하게 해 줌!

      ```python
      # sprinkles/utils.py
      
      from django.core.exceptions import PermissionDenied
      
      def check_sprinkle_rights(request):
          if request.user.can_sprinkle or request.user.is_staff:
              return request
          
          raise PermissionDenied
          
      ```

       - `return request` 에서 `HttpRequest` 객체가 반환되고 있음 

       - 반환되는 `request` 에 추가적인 속성을 덧붙일 수도 있음

         ```python
         request.can_sprinkle = True
         return request
         ```

         => 이렇게 반환하는 경우 템플릿에서 {% raw %}`{% request.can_sprinkle %}`{% endraw %} 이렇게 더 간단히 사용할 수 있음

      ```python
      # sprinkles/views.py
      
      def sprinkle_list(request):
          request = check_sprinkles(rquest) # view에서 이렇게 사용
          return render(rquest, "sprinkles/sprinkle_list.html", {"sprinkles:Sprinkle.objecs.all()"})
      ```

    - 클래스 기반 뷰로 통합하기도 쉽다! (클래스 기반 뷰에서도 함수에서  `request = check_sprinkles(rquest)` 식으로 활용

    - but 함수에서 함수 이용하기를 반복하고 있음 => 자동으로 처리하려면? **<u>데코레이터</u>**!

- 편리한 데코레이터

  - 코드를 좀 더 간결하게 해 주고 사람이 읽기에 좋아지는 기능

  ```python
  import functools
  
  def decorator(view_func):
      @functools.wraps(view_func)
      def new_view_func(request, *args, **kwargs):
          # 여기에서 request(HttpRequest) 객체를 수정
          response = view_func(request, *args, **kwargs)
          # 여기에서 response(HttpResponse) 객체를 수정
          return response
      return new_view_func()
  ```

  => check_sprinkles 예제 함수를 데코레이터로 사용할 수 있도록 구현

  ```python
  # sprinkles/decorators.py
  
  import functools
  from . import utils
  
  def check_sprinkles(view_func):
      @functools.wraps(view_func)
      def new_view_func(request, *args, **kwargs):
          requset = utils.can_sprinkle(request)
          
          # view함수를 호출
          response = view_func(request, *args, **kwargs)
          return response
      return new_view_func()
  ```

  ```python
  # sprinkles/views.py
  
  from .decorators import check_sprinkles
  
  @check_sprinkles
  def sprinkle_list(request):
      return render(rquest, "sprinkles/sprinkle_list.html", {"sprinkles:Sprinkle.objecs.all()"})
  ```

  * `functools.wraps()` ??
    * docstrings같은 중요한 데이터를 포함한 메타데이를 새로이 데코레이션되는 함수에 복사해주는 도구
    * 반드시 필요한 것은 아니지만 프로젝트 관리가 매우 편해짐
  * 데코레이터 남용하지 않기
  * `HttpResponse`객체도 `HttpRequest`와 비슷하게 활용할 수 있다. 특히 데코레이터와 함꼐!
