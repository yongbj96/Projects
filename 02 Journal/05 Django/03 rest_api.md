# Rest_api

## 01 rest_framework 환경구축 

파이썬 패키지 설치

`pip install djangorestframework`



"settings.py"

``` python
INSTALLED_APPS = [
    ...
    'rest_framework',
]
```



## 02 앱생성

`python manage.py startapp <앱이름>`



"settings.py"

``` python
INSTALLED_APPS = [
    ...
    '<앱이름>',
]
```



## 03 Model 및 Serializer 생성

"<앱이름>/models.py"

``` python
class Member(models.Model):
    username = models.CharField(max_length=20)
    age = models.IntegerField(null=True)

    def __str__(self):
        return self.username

    class Meta:
        db_table            = 'members' # 데이터베이스에 저장되는 테이블명
        verbose_name        = '회원' # 해당 테이블을 불러오는 이름
        verbose_name_plural = '회원목록' # 해당 테이블을 불러오는 그룹이름
```



마이그레이션 진행

`python manage.py makemigrations`

`python manage.py migrate`



"<앱이름>/serializers.py" 파일생성 후

``` python
from rest_framework import serializers
from .models import Member

class MemberSerializers(serializers.ModelSerializer):
    class Meta:
        model = Member
        fields = ('id', 'username', 'age')
```



## 04 View 작성

"<앱이름>/views.py"

``` python
from rest_framework import viewsets
from .serializers import MemberSerializer
from .models import Member

class MemberViewSet(viewsets.ModelViewSet):
    queryset = Member.objects.all()
    serializer_class = MemberSerializer
```



## 05 Urls 작성

"<프로젝트이름>/urls.py"

``` python
from django.conf.urls import url, include
from rest_framework import routers
from rest_app.views import MemberViewSet

router = routers.DefaultRouter()
router.register('member', MemberViewSet)

urlpatterns = [
    path('admin/', admin.site.urls),
    url(r'^', include(router.urls)),
]
```



## 06 실행화면

- 리스트조회

![image](https://user-images.githubusercontent.com/43952470/111594762-d20e3c00-880e-11eb-8073-0c08af786ced.png)

- 멤버조회

![image](https://user-images.githubusercontent.com/43952470/111594906-f702af00-880e-11eb-8c3a-afddd4e3282b.png)

- DB에는 자동으로 저장

![image-20210318172644395](C:\Users\yong_\AppData\Roaming\Typora\typora-user-images\image-20210318172644395.png)