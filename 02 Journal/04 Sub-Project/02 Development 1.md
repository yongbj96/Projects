# Sub-Project

## 02 Development 1



```
# https://velog.io/@teddybearjung
링크를 참고로 제작중인 Django기반 게시판페이지 입니다.

회원가입기능(02 Development)
```



---

1. #### member startapp 생성 및 환경설정

   ##### 1.1. Django - startapp 생성

   ```shell
   python manager.py startapp member
   ```

   

   Django 개발환경설정 후, 프로젝트를 생성하고 `member` 앱을 생성하였습니다.

   

   ##### 1.2. 기본 환경설정

   ###### 		1.2.1.  settings.py 설정

   ​	`member` 앱을 사용할 수 있도록 선언해주었습니다.

   

   ###### 		1.2.2. Project.urlpatterns 설정

   ```python
   urlpatterns = [
   	path('member/', include('member.urls')),
   ]
   ```
   
   ​	`localhost:PORT/member` 을 해당 앱의 `urls.py`에서 관리할 수 있도록 선언하였습니다.
   
   
   
   ###### 		1.2.3. Member.urlpatterns 설정
   
   ​	`member`폴더에 `urls.py` 파일을 생성하였고 `urlpatterns`를 작성하였습니다.
   
   ```python
   urlpatterns = [
   	path('register/', views.register),
   ]
   ```
   
   ​	해당 명령어를 통해 `localhost:PORT/member/register` 로 들어오는 `url`을 `views.register` 함수로 반환됩니다.
   
   

---

2. #### Model, Controller, View 작성

   ##### 2.1. Model 설정

   `member`에서 사용할 데이터모델은 다음과 같이 설정하였습니다.

   ```python
   class BoardMember(models.Model):
       username    = models.CharField(max_length=100, verbose_name='유저Id')
       password    = models.CharField(max_length=100, verbose_name='유저PW')
       email       = models.EmailField(max_length=100, verbose_name='유저메일')
       create_at   = models.DateTimeField(auto_now_add=True, verbose_name='가입날짜')
       update_at   = models.DateTimeField(auto_now=True, verbose_name='마지막수정일')
   
       # 생성되는 객체의 타입을 문자열로 변환해서 보여주는 역할
       def __str__(self):
           return self.username
   
       class Meta:
           db_table            = 'boardmembers'
           verbose_name        = '게시판멤버' # 해당 테이블을 조회할 때 테이블이름
           verbose_name_plural = '게시판멤버들' # 해당 테이블을 조회할 때 테이블이름 (기본값 = "verbose_name"+s)
   ```
   
   ###### 		2.1.1. 기본문법
   
   ​	`db_table`: 데이터베이스에 저장될 테이블의 명칭.
   ​	`verbose_name`: 해당 테이블을 가져올 때, 값들의 명칭.
   ​	`verbose_name_plural`: 해당 테이블을 가져올 때, 테이블의 명칭. (기본값 = `verbose_name`+`s`)
   
   ###### 		2.1.2. 데이터  저장방식
   
   ​	`CharField`: Datatype은 `VARCHAR`으로 데이터를 저장. (길이: 100)
   ​	`EmailField`: Datatype은 `VARCHAR`으로 데이터를 저장. (길이: 100)
   ​	`DateTimeField`: Datatype은 `DATETIME`으로 데이터를 저장.
   ​		`auto_now_add` - 생성당시의 날짜/시간 입력
   ​		`auto_now` - 데이터를 조작한 당시의 날짜/시간 입력
   
   
   
   ##### 2.2. Controller 설정
   
   ```python
   def register(request):
       if request.method == 'GET':
           return render(request, 'register.html')
       elif request.method == 'POST':
           username    = request.POST.get('username', None)
           password    = request.POST['password']
           email       = request.POST.get('email', None)
   
           res_data = {}
           if not (username and email):
               res_data['error'] = '모든 값을 입력해주세요.'
           else:
               member = BoardMember(
                   username = username,
                   password = password,
                   email = email,
               )
               member.save() # 데이터베이스에 저장
   
           # res_data를 넘겨서 html을 request
           return render(request, 'register.html', res_data)
   ```
   
   `GET`방식으로 `member.regiser`가 호출되면 `register.html`를 반환.
   
   
   
   `POST`방식으로 호출되면 `request`받은 값들을 데이터베이스에 저장하게됩니다.
   
   
   
   ##### 2.3. View
   
   ![01](https://user-images.githubusercontent.com/43952470/107841951-e96a9b80-6e02-11eb-9f8d-8b88e7db7b43.PNG)
   
   비밀번호 길이 및 `Confirm`확인을 위해 `Javascript`를 작성하였습니다.
   
   ```javascript
   function isSame() {
     var PW = document.getElementById('PW');
     var confirmPW = document.getElementById('confirmPW');
     if(PW.value.length < 6 || PW.value.length > 16) {
       document.getElementById('length').innerHTML = 'The length must be between 6 and 16.';
       document.getElementById('length').style.color='red';
     } else {
       document.getElementById('length').innerHTML = '';
       if(PW.value!='' && confirmPW.value!='') {
         if(PW.value==confirmPW.value) {
           document.getElementById('same').innerHTML='Match';
           document.getElementById('same').style.color='blue';
         } else {
           document.getElementById('same').innerHTML='Missmatch';
           document.getElementById('same').style.color='red';
         }
       }
     }
   }
   ```
   
   

---

3. #### Css설정

   `css`파일을 다운받고 `member/static`폴더아래에 복사, 해당 파일을 사용하기위해 `settings.py`에 선언하였습니다.

   ```python
   STATIC_URL = '/static/'
   STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static'), # import os 입력시 사용가능
   ]
   ```
   
   
   
   
   
   
