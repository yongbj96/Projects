# Attendance-Function

## 04 Development 3



```
Development 2에서 발생한 문제점
1. 출석기능 자체를 잘못 동작시킬 수 있기 때문에 강의 주의 출석 데이터들을 삭제할 수 있어야 한다.
2. 공휴일이 포함되어있으면 3주차 수업을 건너뛰고 4주차 수업으로 출석체크를 진행한다.
```



---

1. #### 출석관리페이지

   ##### 1.1. 출석관리 페이지 삭제기능

   ![attendance41](https://user-images.githubusercontent.com/43952470/106374742-27120200-63c9-11eb-9ccd-8f0b4e204b85.PNG)

   상단에 `삭제`버튼을 통해 동작시킬 수 있습니다.

   

   ##### 1.2. 출석데이터 삭제(RoomServiceImpl)
   
   ``` java
   @Override
   public void delete_Attendance(Map<String, Object> map) throws Exception {
       List<Map<String, Object>> stu_list = roomDAO.detailAtt(map) ;
   
       for(int a=0; a<stu_list.size(); a++) { /* room_code, week, id */
           map.put("id", stu_list.get(a).get("id")) ;
           if(stu_list.get(a).get("attendance").toString().equals("O")) {
               roomDAO.minus_total_circle(map) ;
           } else if(stu_list.get(a).get("attendance").toString().equals("X")) {
               roomDAO.minus_total_x(map) ;
           } else if(stu_list.get(a).get("attendance").toString().equals("/")) {
               roomDAO.minus_total_slash(map) ;
           }
           map.remove("id") ;
       }
       roomDAO.delete_stu_Att(map) ;
       roomDAO.delete_room_Att(map) ;
   }
   ```
   
   `room_code`와 `week`값을 파라미터로 전송받아서 저장된 `class_att_data`의 데이터들을 삭제시키는 기능을 제작하였습니다.
   
   반복문을 통해 수강중인 학생들의 `id`와 파라미터로 받은 `room_code`, `week`값을 통해 `class_att`의 출석 데이터들을 삭제시키는 기능을 추가하였습니다.
   
      

---

2. #### 출석기능동작 페이지

   ##### 2.1. 수업주차 선택

   ![attendance42](https://user-images.githubusercontent.com/43952470/106374746-2bd6b600-63c9-11eb-8bf5-cb20e2fd9ef7.PNG)

   `문제점 2`를 해결하기 위해 수업 주를 입력할 수 있는 기능을 다시 작성하게 되었습니다.
   `default=last_week+1`를 사용하여 자동으로 다음 강의의 주차를 입력할 수 있도록 기능을 구현하였습니다.

   

   ##### 2.2. 기능동작

   ![attendance43](https://user-images.githubusercontent.com/43952470/106374749-309b6a00-63c9-11eb-8f9a-6f76b8ec89af.PNG)

   

   ##### 2.3. 난수입력(학생계정)

   ![attendance44](https://user-images.githubusercontent.com/43952470/106374752-34c78780-63c9-11eb-938c-cfdebd21c7fe.PNG)

   ``` java
   @Override
   public boolean insert_stuAtt(Map<String, Object> map, HttpServletRequest req) throws Exception {
       boolean flag = false ;

       Map<String, Object> room_data = roomDAO.Att_week_data(map) ; //방에있는 마지막 출석정보를 가져온 리스트 week, number, date
       if(room_data.get("number").toString().equals(map.get("number").toString())) { //room_data와 map의 number가 같으면 출석
           SimpleDateFormat format_type = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss") ; /* 선언부 */
           Calendar cal = Calendar.getInstance() ;
           Date now = new Date() ;
           String room_time = room_data.get("date").toString() ;

           Date _late_time = format_type.parse(room_time) ;
           cal.setTime(_late_time) ;
           cal.add(Calendar.MINUTE, 3) ; /* 지각시간 설정, 3분*/
           Date late_time = cal.getTime() ;

           Date _end_time = format_type.parse(room_time) ;
           cal.setTime(_end_time) ;
           cal.add(Calendar.MINUTE, 10) ; /* 결석시간 설정, 10분 */
           Date end_time = cal.getTime() ;

           Map<String, Object> resultMap = new HashMap<>() ;
           resultMap.put("room_code", map.get("room_code")) ;
           resultMap.put("week", room_data.get("week")) ;
           resultMap.put("id", ((HashMap)(req.getSession().getAttribute("user_info"))).get("id")) ;
           if(now.getTime() <= late_time.getTime()) {
               resultMap.put("attendance", "O") ;
               roomDAO.insert_stuAtt(resultMap) ;
               roomDAO.add_total_circle(resultMap) ;
               flag = true ;
           } else if(now.getTime() <= end_time.getTime() && now.getTime() >= late_time.getTime()) {
               resultMap.put("attendance", "X") ;
               roomDAO.insert_stuAtt(resultMap) ;
               roomDAO.add_total_x(resultMap) ;
               flag = true ;
           } else {
               resultMap.put("attendance", "/") ;
               roomDAO.insert_stuAtt(resultMap) ;
               roomDAO.add_total_slash(resultMap) ;
               flag = true ;
           }
       }

       return flag ;
   }
   ```

   `2.2.`의 기능이 동작하면서 난수와 난수가 발생한 시간을 `class_att_data`에 저장하게 되는데, 학생이 난수를 입력하면 입력한 시간과 저장된 시간을 비교하고 출석값을 자동으로 저장합니다.

   

   `-`: 기능동작 후 0 - 10분동안 유지되는 값으로 출석기능이 동작중일 때, 나타나는 값입니다.(default)
   `O`: 기능동작 후 0 - 3분에 난수를 입력하면 발생하는 값으로 출석을 의미합니다.
   `X`: 기능동작 후 3 - 10분에 난수를 입력하면 발생하는 값으로 지각을 의미합니다.
   `/`: 기능동작 후 10분이 지나도 난수를 입력하지 않으면 발생하는 값으로 결석을 의미합니다.

   

---

3. #### 출석현황 페이지(개편)

   `2.1.`에서 특정 주차를 넘기고 출석체크를 할 수 있도록 구현하면서 순서대로 데이터가 저장되지 않아 사용자가 조회할 때 한 눈에 데이터들을 파악할 수 없다고 생각하여 전체 주차별 출석체크를 출력하도록 페이지를 수정하였습니다.

   

   ##### 3.1. 출석현황 페이지(교수계정)

   ![attendance51](https://user-images.githubusercontent.com/43952470/106374753-398c3b80-63c9-11eb-9907-10ee6ce917d6.PNG)

   

   ##### 3.2. 출석현황 페이지(학생계정)

   ![attendance52](https://user-images.githubusercontent.com/43952470/106374758-3db85900-63c9-11eb-8dfb-5d0db10ff9cc.PNG)

   
