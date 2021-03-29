# Django 생성

## 01 Django 설치

1. pip 설치
2. 가상환경 설치



가상환경 실행 후,

1. 파이썬 설치
2. Django 설치
3. (djangorestframework 설치)



## 02 프로젝트 생성

`django-admin startproject <프로젝트이름>`



## 03 DB연동

연동 전, 스키마는 미리 생성.



Python에서 사용할 수 있도록 mysqlclient 설치

`pip install mysqlclient` 



"settings.py "

``` python
DATABASES = {
    'default': {
        #'ENGINE': 'django.db.backends.sqlite3',
        #'NAME': BASE_DIR / 'db.sqlite3',

        'ENGINE': 'django.db.backends.mysql', # 사용할 DB에 맞는 ENGINE 작성
        'NAME': 'DBconnection', # 사용할 스키마 작성
        'USER': 'root', # 계정
        'PASSWORD': '', # 비밀번호
        'HOST': '', # default=localhost
        'PORT': '' # default=3306
    }
}

# 하단의 다국어 및 지역 시간 설정(20210329 추가)
LANGUAGE_CODE = 'ko-kr'

TOME_ZONE = 'Asia/Seoul'
```





DB 연동

`python manage.py migrate`

