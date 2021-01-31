# Attendance-Function

## 03 Development 2



```
Development 1에서 발생한 문제점
1. 학생이 잘못 입력하는 등의 문제가 발생할 수 있기 때문에 교수가 임의로 출석값을 변경할 수 있어야 한다.
2. 랜덤번호를 생성할 때, 자동으로 다음 강의의 주차를 입력할 수 있어야 한다.
3. `요구사항 3번`을 해결할 수 없다.
	-> 강의를 진행한 주의 날짜를 확인할 수 있어야 한다.
```



---

1. #### 출석관리페이지

   ##### 1.1. 페이지 이동

   ![attendance31](https://user-images.githubusercontent.com/43952470/106374716-f500a000-63c8-11eb-8bfa-e38535a06d17.PNG)

   기존 `출석현황페이지`상당에 주차별 링크를 통해 `출석관리페이지`로 이동할 수 있습니다.

   

   ##### 1.2. 출석관리 페이지

   ![attendance32](https://user-images.githubusercontent.com/43952470/106374719-faf68100-63c8-11eb-8e90-bb3333af99a1.PNG)

   `출석현황페이지`보다 학생들에 대한 자세한 정보(사진, 학년, 학과)를 조회할 수 있고 우측 `selectbox`를 통해 학생들의 출석값을 변경할 수 있습니다.

   

   ##### 1.3. 출석값 변경
   
   ```java
   @RequestMapping(value="/room/manager/Att_edit_func.do")
   public ModelAndView Att_edit_func(CommandMap commandMap) throws Exception {
       ModelAndView mv = new ModelAndView("redirect:/room/manager/detailAtt.do");
       Map<String, String> map = new HashMap<>() ;
       
       try {
           for (int i = 0; i < ((String[]) commandMap.get("id")).length; i++) {
               map.put(((String[]) commandMap.get("id"))[i],
                           ((String[]) commandMap.get("attendance"))[i]);
           }
       } catch(Exception e) {
           map.put((String)commandMap.get("id"), (String) commandMap.get("attendance")) ;
       }
   
       roomService.Att_edit_func(commandMap.getMap()) ;
   
       return  mv;
   }
   ```
   
   여러 학생들을 모임방에 생성하고 값들을 주고받으면서 코딩을 진행하여 기능을 정상적으로 제작하였으나
   `학생이 없거나 1명일 때` 오류가 발생하였고 파라미터의 갯수에 따라
   
   `1개 이하`: String
   
   `2개 이상`: String[]
   
   으로 값들을 불러오게 되는것을 확인하였습니다.
   
   
   
   이후, `try ~ catch ~`부분을 추가로 작성하여 문제를 해결했습니다.
   
   

---

2. #### 출석현황 페이지(학생계정)

   ##### 2.1. 출석현황 페이지(학생계정)

   ![attendance34](https://user-images.githubusercontent.com/43952470/106374733-15c8f580-63c9-11eb-8218-9932aad8492a.PNG)

   `출석현황페이지(교수계정)`를 기반으로 학생의 `session`값을 사용하여 로그인 한 학생의 출석 데이터만 가져올 수 있도록 제작하였습니다.



---

3. #### 출석기능동작 페이지(수정)

   ##### 3.1. 페이지 화면

   ![attendance33](https://user-images.githubusercontent.com/43952470/106374738-1eb9c700-63c9-11eb-9bde-8653b6e40e00.PNG)

   이전, 강의 주차를 입력하고 랜덤난수를 생성했지만 자동으로 입력될 수 있도록 수정하였으며 선택한 주차의 날짜를 입력할 수 있도록 기능을 추가하였습니다.

   

---

4. #### 문제점

   4.1. 출석기능 자체를 잘못 동작시킬 수 있기 때문에 강의 주의 출석 데이터들을 삭제할 수 있어야 한다.

   4.2. 공휴일이 포함되어있으면 3주차 수업을 건너뛰고 4주차 수업으로 출석체크를 진행한다.

