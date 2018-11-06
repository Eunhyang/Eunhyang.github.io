---
layout: post
title: "two scoops of django 10장"
description: "TIL:django"
date: 2018-10-25
tags: [TIL]
comments: true
share: false
    
---

## Two scoops of django 10장 - 클래스 기반 뷰의 모범적인 이용

- 믹스인과 클래스 기반 뷰를 이용하는 방법

  - 클래스 기반 뷰를 이용할 때의 **가이드라인** (함수뷰와 거의 비슷)

    - 뷰 코드의 양은 적으면 적을수록 좋다
    - 뷰 안에서 같은 코드를 반복적으로 이용하지 말자
    - 뷰는 프레젠테이션 로직에서 관리하도록 하고, 비즈니스 로직은 모델에서 처리하자. (매우 특별한 경우 폼에서 처리)
    - 뷰는 간단 명로해야 한다.
    - 403, 404, 500 에러 핸들링에는 클래스 기반 뷰가 아닌, 함수뷰 사용하기
    - 믹스인은 간단 명료 해야 한다. 

  - 믹스인 사용하기

    - 믹스인: 실체화된 클래스가 아니라 상속해 줄 기능들을 제공하는 클래스
    - 사용할 때 **<u>규칙</u>**
      - 장고가 제공하는 <u>기본 뷰</u>는 항상 **오른쪽**으로 진행
      - <u>믹스인</u>은 기본 뷰에서부터 **왼쪽**으로 진행
      - 믹스인은 파이썬의 기본 객체 타입(object)을 상속

    ```python
    from django.views.generic import TemplateView
    
    class FreshFruitMixin(object):
    
        def get_context_data(self, **kwargs):
            context = super(FreshFruitMixin, self).get_context_data(**kwargs)
            context["has_fresh_fruit"] = True
            return context
    
    
    class FruitFlavorView(FreshFruitMixin, TemplateView):
        template_name = "fruite_flavor.html"
    ```

- 어떤 작업에 장고의 클래스 기반 뷰가 이용되어야 하는가

| 이름              | 목적                                |
| ----------------- | ----------------------------------- |
| Views             | 어디든 이용 가능한 기본 뷰          |
| RedirectView      | 사용자를 다른 RUL로 리다이렉트      |
| TeplateView       | 장고 HTML템플릿을 보여줄 때         |
| ListView          | 객체 목록                           |
| DetailView        | 객체를 보여줄 때                    |
| FormView          | 폼 전속                             |
| CreateView        | 객체를 만들 때                      |
| UpdateView        | 객체를 업데이트 할 때               |
| DeleteView        | 객체를 삭제                         |
| generic date view | 시간 순서로 객체를 나열해 보여줄 때 |

- 클래스 기반 뷰 사용에 대한 일반적인 팁들
  - 인증된 사용자에게만 장고 클래스 기반 뷰/ 제네릭 클래스 기반 뷰 접근 가능하게 하기 =>  `django-braces`에서 제공하는 `LoginRequiredMixin` 을 상속받기
  - 뷰에서 유요한, 혹은 부적합한 폼을 이용해 커스텀 액션 구현하기 => `from_valid()` 혹인 `form_invalid()` 함수를 제너릭 클래스 기반 뷰가 요청을 보내는 곳에 작성
- 클래스 기반 뷰를 폼에 연동하기
- 베이스 `django.view.generic.View` 이용하기
