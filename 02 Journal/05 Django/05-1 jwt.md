# JWT

토큰 기반 인증 시스템(Stateless 서버)

``` 
참고블로그: https://jpadilla.github.io/django-rest-framework-jwt/ # 실패
		https://moondol-ai.tistory.com/173 # 실패
		https://velog.io/@lemontech119/DRF%EB%A1%9C-%EC%84%9C%EB%B2%84-%EA%B0%9C%EB%B0%9C1-%ED%9A%8C%EC%9B%90%EA%B8%B0%EB%8A%A5 # 실패
```



## 01 jwt 설정



- django rest_framework 및 jwt 설치

`pip install djangorestframework djangorestframework-jwt`



"<프로젝트이름>/settings.py"

``` python
from datetime import timedelta

...

INSTALLED_APPS = [
    ...
    'rest_framework',
]

REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticated',
    ),
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_jwt.authentication.JSONWebTokenAuthentication',
    ),
}

JWT_AUTH = {
    'JWT_ENCODE_HANDLER': 'rest_framework_jwt.utils.jwt_encode_handler',
    'JWT_DECODE_HANDLER': 'rest_framework_jwt.utils.jwt_decode_handler',
    'JWT_PAYLOAD_HANDLER': 'rest_framework_jwt.utils.jwt_payload_handler',
    'JWT_PAYLOAD_GET_USER_ID_HANDLER': 'rest_framework_jwt.utils.jwt_get_user_id_from_payload_handler',
    'JWT_RESPONSE_PAYLOAD_HANDLER': 'rest_framework_jwt.utils.jwt_response_payload_handler',

    'JWT_SECRET_KEY': SECRET_KEY,
    'JWT_GET_USER_SECRET_KEY': None,
    'JWT_PUBLIC_KEY': None,
    'JWT_PRIVATE_KEY': None,
    'JWT_ALGORITHM': 'HS256',
    'JWT_VERIFY': True,
    'JWT_VERIFY_EXPIRATION': True,
    'JWT_LEEWAY': 0,
    'JWT_EXPIRATION_DELTA': timedelta(days=30),
    'JWT_AUDIENCE': None,
    'JWT_ISSUER': None,
    'JWT_ALLOW_REFRESH': False,
    'JWT_REFRESH_EXPIRATION_DELTA': timedelta(days=30),
    'JWT_AUTH_HEADER_PREFIX': 'JWT',
    'JWT_AUTH_COOKIE': None,
}

REST_USE_JWT = True
```



이후 실습 몇가지를 추가로 진행하였으나 실패..



새로운 md파일로 작성할 예정입니다.