# Admin 계정

## 01 Admin 계정생성 

`python manage.py createsuperuser`



## 02 Model 정의

모델은 `app`에서 정의할 수 있다.

앱생성: `python manage.py startapp <앱이름>`



`<앱이름>/models.py` 작성

``` python
class People(models.Model):
    userID      = models.CharField(max_length=20, primary_key=True, verbose_name='유저ID')
    password    = models.CharField(max_length=12, verbose_name='유저PW')

    # 생성되는 객체의 타입을 문자열로 변환
    def __str__(self):
        return self.userID

    class Meta:
        db_table            = 'members' # 데이터베이스에 저장되는 테이블명
        verbose_name        = '회원' # 해당 테이블을 불러오는 이름
        verbose_name_plural = '회원목록' # 해당 테이블을 불러오는 그룹이름
```



## 03 모듈에 등록

Admin 모듈에 등록하여 관리자페이지에서 Table을 관리할 수 있음.



`<앱이름>/admin.py`

``` python
from member.models import People
admin.site.register(People)
```



## 04 Migration으로 DB반영

`python manage.py makemigrations`

`python manage.py migrate`