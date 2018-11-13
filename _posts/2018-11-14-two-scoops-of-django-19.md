---
layout: post
title: "two scoops of django 19장"
description: "TIL:django"
date: 2018-11-14
tags: [TIL]
comments: true
share: false
    
---

## Two scoops of django 19장 - 장고 어드민 이용하기

* 객체의 문자열 표현

  * 장고 어드민 페이지에서 객체를 'IceCreamBar Object'로 보여줌 (객체의 이름을 저렇게 문자열로 표현하기 때문)
    * 모든 장고 모델에 대해 항상 `__str()__` 메서드를 구현하자. 이는 좀 더 나은 문자열 표현을 어드민뿐 아니라 여러곳에서 보여 줄 것이다. (객체에서 `unicode()`가 호출될 때마다 `__str()__`메서드가 호출됨)
    * 어드민 리스트 항목들이 해당 객체의 문자열 값(이름)을 보여주는 것이 아니라면 `list_display`를 이용하자

* 장고 어드민 클래스에 호출자 더하기

  * 예) 실제 객체의 url과 달라서  `get_absolute_url`이 의미가 없어질 때(특히 rest api일 때) 타깃의 url의 링크를 제공하기위한 호출자 구현

    ```python
    class IceCreamBarAdmin(admin.ModelAdmin):
        list_display = ("name")
        readonly_fields= ("show_url", )
        
        def show_url(self, instance):
            url = reverse("ice_cream_bar_detail", kwargs={"pk": instance.pk})
            response = format_html("""<a href="{0}">{1}</a>""", url, url)
            return response
        show_url.short_description = "Ice Cream Bar URL"
        show_url.allow_tags=True # 사용자 입력 데이터의 allow_tags의 값을 절대 True로 하지말 것!
    ```

    * `allow_tags` 속성 사용할 때 주의하기
      * 기본적으로 `False`
      * `True`로 하면 보안 이슈를 일으킬 수 있음 
        * HTML태그가 어드민에서 보이게 됨 
      * 프라이머리 키, 날짜, 연산된 값 등 오직 시스템이 생성한 데이터일 경우에 이용하자 

* django.contrib.admin.ModelAdmin.list_editable을 이용할 때 주의할 점 

  * => 다중 사용자가 이용하는 환경에서 `list_editable`  이용은 피하자  
  * `django.contrib.admin` 기능은 사용자가 여러 레코드를 한 거번에 수정할 수 있게 하기 위해 기본 어드민 리스트 뷰를 폼으로 대체해 보여 주는 것 but 레코드는 레코드의 프라이머리 키가 아니라 레코드의 순서로 식별됨
    * => 다중 사용자가 사용해서 동시에 리스트를 편집하는 경우 문제가 발생할 수 있음(누군가 편집하는데 다른 누군가 또 편집해서 순서가 뒤틀리면 디스플레이 순서가 다 깨져버림)

* 커스텀 장고 스킨에 대한 충고

  * => 기본적으로 잘 작동하는 기능들이 커스텀 스킨에서 깨지거나 제대로 작동하지 않을 수 있으니 테스트 케이스 작성해보기 