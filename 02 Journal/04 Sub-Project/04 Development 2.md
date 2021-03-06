# Sub-Project

## 03 Django Restframework-jwt



```
로그인기능을 효율적으로 사용하기 위해 jwt 모듈을 사용
```



---

1. ### 설치

   - 모듈설치

     `pip install dajangorestframework dajangorestframework-jwt`

   

   - Python 패키지 환경을 쉽게 관리할 수 있도록 작성하는 것. 

     `pip freeze > requirements.txt`

---

2. ### JWT 설정

   - ##### {프로젝트}/settings.py

     ```
     ...
     REST_FRAMEWORK = {
         'DEFAULT_PERMISSION_CLASSES': (
             'rest_framework.permissions.IsAuthenticated', # 로그인 여부확인 클래스
         ),
         'DEFAULT_AUTHENTICATION_CLASSES': (
             'rest_framework_jwt.authentication.JSONWebTokenAuthentication', # 로그인관련
             'rest_framework.authentication.SessionAuthentication',
             'rest_framework.authentication.BasicAuthentication',
         ),
     }
     
     JWT_AUTH = {
         'JWT_SECRET_KEY': SECRET_KEY,
         'JWT_ALGORITHM': 'HS256',
         'JWT_ALLOW_REFRESH': True,
         'JWT_EXPIRATION_DELTA': datetime.timedelta(hours=1), # 토큰 유효기간
         'JWT_REFRESH_EXPIRATION_DELTA': datetime.timedelta(hours=3), # 토큰갱신의 유효기간
     }
     ...
     ```

     `datetime` 사용법

     ​	days=? : 일, hours=? : 시, minutes=? : 분, seconds=? : 초

     

   - ##### {프로젝트}/urls.py

     ```python
     ...
     from rest_framework_jwt.views import obtain_jwt_token, verify_jwt_token, refresh_jwt_token
     
     path('api/token/', obtain_jwt_token), #발행
     path('api/token/verify/', verify_jwt_token), #검증
     path('api/token/refresh/', refresh_jwt_token), #갱신
     path('앱/', include('앱.urls')), # member/~를 member/urls파일에서 관리
     ...
     ```

     해당 프로젝트에서는 앱을 `member`로 사용

     

   - ##### {앱}/urls (해당 프로젝트에서는 `member/urls` 로 사용)

     ```python
     from django.urls import path
     from . import views
     
     urlpatterns = [
         path('posts/', views.posts, name='posts'),
     ]
     ```

     



---

3. 

---

4. 

