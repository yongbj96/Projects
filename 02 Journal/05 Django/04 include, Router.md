# include, Router

## 01 include

주로 `urls.py`에서 사용하며 다른 `urls`들을 포함시키거나 `router`를 포함할 때 사용한다.



패키지는 `django.urls`내부에 존재하며 `path`와 함께 선언하여 사용이 가능하다.

``` python
from django.urls import path, include
```



``` python
1 from django.urls import path, include
2 from . import views
3
4 urlpatterns = [
5    path('admin/', admin.site.urls),
6    path('member/', include('member.urls')),
7    path('member/<int:id>', views.detail),
8    path('', include(router.urls)),
9]
```

Line 5: Django의 관리자페이지이며 `admin` 페이지와 연동한다.

Line 6: `<url>/member/`로 들어오는 요청을 해당 파일에서 작성한 링크와 연동한다.

​	변수로 선언할 수 있다.(참고링크: https://velog.io/@ddusi/django-2)

Line 7: `<int:id>`로 자원의 `primary_Key`를 호출하여 회원별 상세정보를 조회할 수 있다.

​	단, 이 경우 상세정보를 조회할 `view`를 따로 작성해야한다.

Line 8: `router`에 작성한 `url`로 연동한다.(router란?)



##### * urlpatterns 사용방법

url을 `views.py`에 작성한 페이지와 연결하는 방식.



##### * url? path?

프로젝트를 진행하다보면 `path`외에도 `url`로 작성한 것을 확인할 수 있는데

`url`의 경우 2.0 버전 이전에 사용하던 모듈로 이후 버전에서 `path`라는 간결한 모듈을 사용한다.

참고링크: https://medium.com/@lyoungh2570/django-url%EA%B3%BC-path-f75ec1754460



## 02 Router

##### * `rest_framework`에서 제공하는 기능, `url`을 자동으로 매핑해주는 역할을 한다.

참고링크: https://www.django-rest-framework.org/api-guide/routers/



"<프로젝트이름>/urls.py"

``` python
1  from rest_framework import routers
2  from rest_app.views import MemberViewSet
3
4  router = routers.DefaultRouter() # SimpleRouter(), etc..
5  router.register('member', MemberViewSet)
6
7  urlpatterns = [
8      ...
9      path('', include(router.urls)),
10 ]
```

Line 4: `DefaultRouter`와 `SimpleRouter`를 주로 사용하게 되며 `DefaultRouter`의 경우 어떤 `url`을 기본 라우터로 설정했는지 보여줄 수 있다. (개인적으로는 더 많이 사용할 듯..)

Line 5: `router`의 기본 url을 `member`로 설정하였고 `<url>/member`로 접속하면 리스트 View를 보여준다.

​	* `'member'`을 `''`로 변경하면 Line 9번에서 연동한 링크만 사용해도 리스트를 조회할 수 있다.

Line 9: `<url>/`을 `router`와 연동하였다.



실행화면

- `DefaultRouter`로 선언하였을 때,

![image](https://user-images.githubusercontent.com/43952470/111716195-db43eb00-8898-11eb-953e-1169375ccccc.png)

`localhost:8000`으로 접속을 하였으나 `router`의 기본 url인 `member`를 반환해주는 것을 확인할 수 있다.



- `localhost:8000/member`으로 접속한 화면

![image](https://user-images.githubusercontent.com/43952470/111716397-4b527100-8899-11eb-94c9-4acc48a09b98.png)



- Line 5번의 `'member'`를 `''`로 변경하였을 때,

![image](https://user-images.githubusercontent.com/43952470/111716458-6d4bf380-8899-11eb-9fdf-c307d53830f2.png)

`localhost:8000`으로 접속을 하였음에도 기존 `localhost:8000/member`의 목록을 불러온다.