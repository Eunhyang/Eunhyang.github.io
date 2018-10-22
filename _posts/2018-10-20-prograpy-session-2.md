---
layout: post
title: "[프로그라피 수업준비]  Q. Django Authentication"
description: "프로그라피: 수업 준비"
date: 2018-10-19
tags: [prography]
comments: true
share: true목

---



> 10월 20일 프로그라피 Django session에서 공유한 내용입니다.

## Q2. Django Authentication

### Django에 내장되어있는 기본 인증

- ```
  django.contrib.auth
  ```

   인증 관련 모델이 존재 

  - `django.contrib.auth.models` 에 User모델 존재

  - `django.contrib.auth.views` 인증관련 View

  - `django.contrib.auth.urls` 인증관련 URL패턴

  - ⇒ 직접 까서 봅시다!

    ```python
    # django.contrib.auth.urls.py
    
    urlpatterns = [
        path('login/', views.LoginView.as_view(), name='login'),
        path('logout/', views.LogoutView.as_view(), name='logout'),
    
        path('password_change/', views.PasswordChangeView.as_view(), name='password_change'),
        path('password_change/done/', views.PasswordChangeDoneView.as_view(), name='password_change_done'),
    
        path('password_reset/', views.PasswordResetView.as_view(), name='password_reset'),
        path('password_reset/done/', views.PasswordResetDoneView.as_view(), name='password_reset_done'),
        path('reset/<uidb64>/<token>/', views.PasswordResetConfirmView.as_view(), name='password_reset_confirm'),
        path('reset/done/', views.PasswordResetCompleteView.as_view(), name='password_reset_complete'),
    ]
    ```

------

### 실습

>  django 2.1.1, python 3.6.0 환경

- user app만들기

  ```shell
  $ python [manage.py](http://manage.py) startapp user
  ```

- [`settings.py`](http://settings.py)에 user앱 등록

  ```python
  INSTALLED_APPS = [
      ...
      'user',
  ]
  ```

- `<project경로>/urls.py` 에 user url경로 추가

  ```python
  urlpatterns = [
      url(r'^user/', include('user.urls', namespace="user")),
  ]
  ```

- user앱에 `urls.py` 만들고 url입력하기

  ```python
  from django.conf.urls import url, include
  from . import views
  
  app_name = 'users'
  
  urlpatterns = [
      url(r'^', include('django.contrib.auth.urls')), # 장고 내장 기본 인증 url경로 추가
      url(r'^register/$', views.UserCreateView.as_view(), name='register'), # 회원 가입
      url(r'^register/done/$', views.UserCreateDoneTemplateView.as_view(), name='register_done'), # 가입 완료
  ]
  ```

- `user/views.py`

  ```python
  from django.contrib.auth.forms import UserCreationForm
  from django.urls import reverse_lazy
  from django.views.generic import TemplateView
  from django.views.generic.edit import CreateView
  
  
  class UserCreateView(CreateView):
      template_name = 'registration/register.html'
      form_class = UserCreationForm
      success_url = reverse_lazy('user:register_done')
  
  
  class UserCreateDoneTemplateView(TemplateView):
      template_name = 'registration/register_done.html'
  ```

- user앱에 `templates/registration` 폴더에 필요한 템플릿 만들기

  - `user/templates/rgistration/register.html`

    ```html
        <h1>Registration</h1>
        <form method="POST" class="post-form">{% csrf_token %}
            {{ form.as_p }}
            <button type="submit" class="save btn btn-default">Save</button>
        </form>
    ```

  * url / view/ template 패턴

    | URL 패턴                    | View이름                  | 템플릿 파일명                             |
    | --------------------------- | ------------------------- | ----------------------------------------- |
    | /user/login/                | login()                   | registration/login.html                   |
    | /user/logout/               | logout()                  | registration/logged_out.html              |
    | /user/password_change/      | password_change()         | registration/password_change_form.html    |
    | /user/password_change/done/ | password_change_done()    | registration/password_change_done.html    |
    | /user/password_reset/       | password_reset()          | registration/password_reset_form.html     |
    |                             |                           | registration/password_reset_email.html    |
    |                             |                           | registration/password_reset_subject.html  |
    | /user/password_reset/done/  | password_reset_done()     | registration/password_reset_done.txt      |
    | /user/reset/                | password_reset_confirm()  | registration/password_reset_confirm.html  |
    | /user/reset/done/           | password_reset_complete() | registration/password_reset_complete.html |

  - `settings.py` 에 url 설정

    ```python
    INSTALLED_APPS = [
            'django.contrib.admin',
            'django.contrib.auth', # 기본 설치됨
            'django.contrib.contenttypes',
            'django.contrib.sessions',
            'django.contrib.messages',
            'django.contrib.staticfiles',
        ]
        
        LOGIN_URL = '/accounts/login/' # 기본값
        LOGOUT_URL = '/accounts/logout/' # 기본값
        LOGIN_REDIRECT_URL = '/' # 반드시 정의할 것!
    ```

- 장고 기본 인증의 로그인 원리

  - 로그인하면 장고서버에서 cookie에 sessionid값을 발급
  - 유저의 브라우저 쿠키에 sessionid값 저장
  - 요청과 함께 sessionid값이 날라옴
  - 서버에서 sessionid값을 검증하고 유저정보를 알 수 있음 
    - user.is_authenticated = True
    - request.user로 user접근가능
  - but!!! 보안이 취약(csrf공격에 노출당할 수 있음)
    - ⇒ csrf token을 사용하거나
    - ⇒ request header에 인증 token값으로 유저를 검증하는 방식으로 사용하자!

## 인증 Customizing 하기

- User모델 확장 종류

  - 프록시 모델 사용하기
  - User모델과 일대일관계로 새로운 모델 정의
  - `AbstractUser` 모델 상속한 사용자 정의 User 모델 사용하기
  - `AbstractBaseUser` 모델 상속한 사용자 정의 User 모델 사용하기

- 프록시 모델 사용하기

  - DB에 user관련된 부가적인 필드를 넣을 필요가 없을 때 사용함

    ```python
    from django.contrib.auth.models import User
    from .managers import PersonManager
    
    class Person(User):
        objects = PersonManager()
    
        class Meta:
            proxy = True
            ordering = ('first_name', )
    
        def do_something(self):
            ...
    ```

  - 추가적인 행동(메소드)을 정의할 필요가 있을 때 주로 프록시모델을 사용합니다.

- `User` 모델과 일대일 관계로 새로운 모델 정의

  - 새로운 필드를 삽입할 수 있다.

  - 장고 기본 인증 시스템을 그대로 사용한다 ⇒ 권한, 로그인 로직을 변경할 수 없다!

    ```python
    from django.db import models
    from django.contrib.auth.models import User
    
    class CustomUser(models.Model):
        user = models.OneToOneField(User, on_delete=models.CASCADE) # 장고 기본 User모델
        bio = models.TextField(max_length=500, blank=True)
        location = models.CharField(max_length=30, blank=True)
        birth_date = models.DateField(null=True, blank=True)
    ```

- `AbstractUser` 모델 상속한 사용자 정의 User 모델 사용하기

  - AbstractUser 모델을 상속한 User 모델을 만들어 settings.py에 참조를 수정해야 한다.

    - 이 기법의 사용 여부는 프로젝트 시작 전에 하는 것이 좋다. 추후에 settings.AUTH_USER_MODEL 변경시 데이터베이스 스키마를 알맞게 재수정해야 하는데 사용자 모델 필드에 추가나 수정으로 끝나지 않고 완전히 새로운 사용자 객체를 생성하는 일이 된다.	

  ```python
  ## <project_name>/settings.py	
  
  AUTH_USER_MODEL = 'user.User' # user모델 참조 설정
  ```

  - 새로운 필드를 삽입할 수 있다.

  - 장고 기본 인증 시스템을 그대로 사용한다 ⇒ 권한, 로그인 로직을 변경할 수 없다!

    ```python
    from django.contrib.auth.models import AbstractUser
    
    class User(AbstractUser):
        name = models.CharField('이름', max_length=30, blank=True)
        sms_receiving_consent = models.BooleanField('SMS 수신 동의', default=False)
        email_receiving_consent = models.BooleanField('이메일 수신 동의', default=False)
    
        def to_dict_for_api(self):
            return {
                'username': self.username,
                'email': self.email,
                'name': self.name,
                'isStaff': self.is_staff,
                'lastLogin': self.last_login,
                'dateJoined': self.date_joined,
                'email_receiving_consent': self.email_receiving_consent,
                'sms_receiving_consent': self.sms_receiving_consent,
            }
    ```

- `AbstractBaseUser` 모델 상속한 사용자 정의 User 모델 사용하기

  - `AbstractBaseUser` 모델을 상속한 User 모델을 만들고 `settings.py`에 참조를 수정하는 것은 `AbstractUser` 모델을 상속하는 것과 같다. 따라서 마찬가지로 프로젝트 시작 전에 이 기법의 사용 여부를 결정하는 것이 바람직하다.

  -  `AbstractUser` 모델을 상속하는 방법과 달리 로그인 아이디로 이메일 주소를 사용하도록 하거나 Django 로그인 절차가 아닌 인증 절차를 직접 구현하고자 할 때 사용할 수 있다. 

    - 예를 들면, sns로 로그인을 구현


참고자료

------

[위키독스](https://wikidocs.net/6650)
