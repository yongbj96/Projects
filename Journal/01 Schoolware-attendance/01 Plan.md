# Attendance-Function

## 01 Plan



```
강의를 시작할 때, 학생들이 많아 출석체크하는 시간이 길어져 강의시간을 소비하는 문제가 생기는데
이를 해결할 출석시스템 개발을 교수님께서 요청하셨습니다.

요구사항
1. 출석현황페이지에서 모든학생들의 출석을 조회할 수 있어야 한다.
2. 학생들의 출결상황의 Total을 확인할 수 있어야 한다.
	-> 나사렛대학교 출석데이터(출석: O, 결석: /, 지각: X)
3. 강의를 진행한 주의 날짜를 확인할 수 있어야 한다.
4. 자리에 있지 않은 학생은 출석을 할 수 없어야 한다.
```



---

1. #### DB구성

   ##### 1.1. 위와 같은 요구사항을 만족시키는 기능을 제작하기 위해 DB를 구상하였습니다.

   ![attendance11](https://user-images.githubusercontent.com/43952470/106374668-ac48e700-63c8-11eb-9049-9bd013fcd893.PNG)

   

   ##### 1.2. 구상한 테이블을 바탕으로 테이블을 생성하였습니다.

   ```
   DB tables
   class_att: room_code(pk), id(pk), week(pk), attendance(NN, default='-')
   class_att_data: room_code(pk), week(pk), number(NN), date(NN, default='datetime')
   class_att_total: room_code(pk), id(pk), circle(default=0), slash(default=0), x(default=0)
   ```

   

---

2. #### 페이지 가이드라인 구상

   ##### 2.1. 교수계정 출석체크 페이지

   ​	`class_att`에서 학생들의 출석 데이터조회

   ​	`출석기능`을 통해 랜덤난수 생성

   

   ##### 2.2.학생계정 출석체크 페이지

   ​	`class_att`에서 본인의 출석 데이터조회

   ​	`출석하기`를 통해 난수를 입력, 시간에 따른 출석 데이터를 자동으로 입력

   
   
   ##### 2.3. 가이드라인
   
   ![attendance12](https://user-images.githubusercontent.com/43952470/106374676-b4a12200-63c8-11eb-9801-e76f779aa33f.png)
   
   `메모장을 사용하여 빈 화면에 출석현황페이지의 가이드라인을 제작하였습니다.`

