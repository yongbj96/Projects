# Sub-Project

## 03 Django Restframework-jwt



```
로그인기능을 효율적으로 사용하기 위해 jwt 모듈을 사용
참고: https://dev-yakuza.posstree.com/ko/django/jwt/
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
     path('app/', include('app.urls')), # member/~를 member/urls파일에서 관리
     ...
     ```

     해당 프로젝트에서는 `app`을 `member`로 사용

     

   - ##### {app}/urls (해당 프로젝트에서는 `member/urls` 로 사용)

     ```python
     from django.urls import path
     from . import views
     
     urlpatterns = [
         path('posts/', views.posts, name='posts'),
     ]
     ```



---

3. ### Models 작성

   - ##### {app}/models.py

     ```python
     class Post(models.Model):
         author = models.ForeignKey('auth.User', on_delete=models.CASCADE)
         title = models.CharField(max_length=100)
         content = models.TextField()
         created_at = models.DateTimeField(auto_now_add=True)
         updated_at = models.DateTimeField(auto_now=True)
         published_at = models.DateTimeField(blank = True, null = True)
     
         def publish(self):
             self.published_at = timezone.now()
             self.save()
     
         def __str__(self):
             return self.title
     
         class Meta:
             db_table            = 'board'
             verbose_name        = '게시판'
             verbose_name_plural = '게시판목록'
     ```

     참고한 블로그의 모델을 그대로 사용 (추후에 변경 후 실습해볼 예정)

     

   - ##### Migration

     `python manager.py makemigrations`

     `python manager.py migrate`

     

   - ##### {app}/admin.py

     ``` python
     ...
     from .models import Post
     
     admin.site.register(Post)
     ...
     ```

     `admin`에서 해당 모델을 사용할 수 있도록 코드 추가작성



---

4. ### View 작성

   - ##### {app}/views.py

     ```python
     from django.core import serializers
     from rest_framework.decorators import api_view, permission_classes, authentication_classes
     from rest_framework.permissions import IsAuthenticated # 로그인여부 확인
     from rest_framework_jwt.authentication import JSONWebTokenAuthentication # JWT인증 확인
     from .models import Post
     
     @api_view(['GET'])
     @permission_classes((IsAuthenticated, ))
     @authentication_classes((JSONWebTokenAuthentication,))
     def posts(request):
         posts = Post.objects.filter(published_at__isnull=False).order_by('-published_at')
         post_list = serializers.serialize('json', posts)
         return HttpResponse(post_list, content_type="text/json-comment-filtered")
     ```

     `@api_view(['GET'])` : 요청이 GET인지 확인하여 JSON 타입으로 반환

     `@permission_classes` : 권한을 체크(로그인 했는지 여부만 체크)

     `@authentication_classes` : JWT토큰 확인, 토큰에 이상이 있으면 JSON으로 반환



---

5. ### 동작확인

   - GET, POST를 확인할 수 있도록 `Postman`을 사용하였습니다.

   

   1. ##### 보통 방식으로 url조회

   ![01](https://user-images.githubusercontent.com/43952470/110213306-cfdce100-7ee2-11eb-8ffc-2cc34222208f.PNG)

   다른 `app`들과 동일하게 `url`을 조회할 경우 다음과같이 `자격 인증데이터`가 제공되지 않아 조회가 불가능

   

   2. ##### `token`생성 (현재 admin 계정으로 생성)

   ![02](https://user-images.githubusercontent.com/43952470/110213489-a1abd100-7ee3-11eb-9fca-8ef38131f759.PNG)

   `username`과 `password`에 `admin계정` 입력 후 POST로 전송하면 token 생성

   

   3. ##### 기존 `url`에 `Authorization`에 `jwt {token}`으로 값을 전송

   ![03](https://user-images.githubusercontent.com/43952470/110213516-c30cbd00-7ee3-11eb-9bc8-49a3e57babd7.PNG)

   해당 계정으로 조회

   

   4. ##### token 확인: {url}/api/token/verify

   ![04](https://user-images.githubusercontent.com/43952470/110213598-0ff09380-7ee4-11eb-8cb9-24a5bd263cc2.PNG)

   `token`이 존재하면 동일한 값을 `response`

   

   ![05](https://user-images.githubusercontent.com/43952470/110213683-64940e80-7ee4-11eb-8858-7177d4071698.png)

   `token`이 없다면 다음과 같은 `JSON`을 반환

   

   5. ##### token 갱신: {url}/api/token/refresh

   ![06](https://user-images.githubusercontent.com/43952470/110213717-85f4fa80-7ee4-11eb-916a-5841159b6600.PNG)

   새로운 `token`을 반환