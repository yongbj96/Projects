# Attendance-Function

## 02 Development 1



```
가이드라인을 기반으로 페이지를 코딩하였습니다.
```



---

1. #### 출석현황페이지

   ##### 1.1. 출석현황페이지

   ![attendance21](https://user-images.githubusercontent.com/43952470/106374681-bec32080-63c8-11eb-8d27-d3c27efce16f.PNG)

   학생들의 출석값을 저장하는 테이블 `class_att`에는 값을 임의로 입력하였습니다.

   

   ##### 1.2. 코드(SQL)

   ![attendance22](https://user-images.githubusercontent.com/43952470/106374682-c7b3f200-63c8-11eb-812b-2e6de542ac52.PNG)

   `class_att`에 입력되어있는 `attendance`를 id로 그룹화하여 Model을 호출하였습니다.

   

   ##### 1.3. 코드(JSP)

   ![attendance23](https://user-images.githubusercontent.com/43952470/106374705-e87c4780-63c8-11eb-83e0-cfcb0a011484.PNG)

   그룹화 되어있는 `attendance`를 `split(',')`를 사용하여 `att`배열로 다시 받고 출력하는 코드입니다.

   

---

2. #### 출석기능동작 페이지

   ##### 2.1. 페이지 화면(페이지만 구현)

   ![attendance24](https://user-images.githubusercontent.com/43952470/106374710-edd99200-63c8-11eb-8c53-c1cc43983b8e.png)

   상단, `selectbox`를 통해 주를 선택할 수 있고 입력한 주차의 랜덤한 난수를 발생시킬 수 있도록 구성하였습니다.

   

---

3. #### 문제점

   3.1. 학생이 잘못 입력하는 등의 문제가 발생할 수 있기 때문에 교수가 임의로 출석값을 변경할 수 있어야 한다.

   3.2. 랜덤번호를 생성할 때, 자동으로 다음 강의의 주차를 입력할 수 있어야 한다.

   3.3. `요구사항 3번`을 해결할 수 없다.

   

