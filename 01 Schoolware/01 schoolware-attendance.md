# 출석기능 제작





##### 목차

> 1. [교수계정](#professor)
>    - [출석현황페이지](#att-list)
>
>    - [출석관리페이지](#att-mgmt)
> 2. [학생계정](#student)
> 3. [동작과정](#process)

------



동아리에 참가하여 처음으로 담당했던 기능입니다. 출석체크를 할 때, 수강하는 학생들이 많아서 강의시간을 소비하면서 까지 출석을 부르는 것이 불편하여 이것을 대신할 출석 시스템을 개발해달라고 교수님께서 요청을 하셨습니다. 

동작과정에서 난수를 발생시키고 학생들이 숫자를 입력하여 입력한 시간에 따른 출석값을 DB에 자동으로 저장하는 기능을 개발했습니다.



1. <div id="professor">교수계정</div>
- <div id="att-list">출석현황페이지</div>
  
  ![attendance01_1](https://user-images.githubusercontent.com/43952470/106319410-c09bbf80-62b4-11eb-9d51-529155c8b398.PNG)
  
  ```출석``` 버튼을 통해 출석기능을 동작시킬 수 있고 상단 `숫자`를 통해 해당 주의 출석을 관리하는 페이지로 이동할 수 있습니다.
  
  
  
     ```java
  //출석체크
  @RequestMapping( value="/room/manager/selectAttendance.do")
  public ModelAndView roomAttendancePage(CommandMap commandMap, HttpServletRequest req) {
  	ModelAndView mv = new ModelAndView("/room/manage/room_attendance");
  	List<Map<String, Object>> week_total = new ArrayList<>();
  	List<Map<String, Object>> att_data = roomService.Attroom_data(commandMap.getMap());
  	List<Map<String, Object>> att_user = roomService.Attroom_user(commandMap.getMap());
  
      if(att_user.get(att_user.size() - 1).containsKey("count")) { /* 출석현황 주별 통계 연산 */
          int count = Integer.parseInt(att_user.get(att_user.size() - 1).get("count").toString());
          att_user.remove(att_user.size() - 1);
          for (int i = 0; i <= count - 1; i++) {
              week_total.add(att_user.get(att_user.size() - 1));
              week_total.get(i).put("week", att_data.get(i).get("week"));
              att_user.remove(att_user.size() - 1);
          }
      }
      
      mv.addObject("room_code", commandMap.get("room_code"));
      mv.addObject("board_info", roomService.selectBoardList(commandMap.getMap()));
      mv.addObject("data", att_data);
      mv.addObject("user", att_user);
      mv.addObject("week_total", week_total);
      mv.addObject("total", roomService.Attroom_total(commandMap.getMap()));
      
      return mv;
  }
     ```
  
  ######      기능정리
  
  `att_data`: 모임방의 출석 데이터를 불러와 포워딩하는 페이지에 값을 전송하기 위한 변수
  
  `att_user`: 학생들의 출석 값을 주 단위로 가져오는 변수
  
  `week_total`: 하단의 출석, 결석, 지각 통계값을 나타내기위한 변수
  
  `total`: 모임방에서 학생이 갖는 최종 출결값을 나타내기위한 변수
  
  
  
     ```sql
  <!-- 모임방 회원, 출석값 가져오기 -->
  <select id="Attroom_user" parameterType="hashmap" resultType="hashmap">
  	<![CDATA[
      SELECT
      	att.room_code, u.id, u.name, group_concat(week order by week) week, group_concat(attendance order by week) count
      FROM
      	class_att att
      JOIN
      	user_info u
      ON
      	att.id = u.id
      WHERE
      	att.room_code = #{room_code}
      GROUP BY
      	u.id
      order by
      	stu.grade, u.id
  	]]>
  </select>
     ```
  
  `week_total`의 경우 값을 따로 저장하고 있지 않기 때문에 `attendance`를 통해 학생들의 출석값들을 불러와 Impl에서 연산을 하고 Controller에서 `total`로 분리하여 JSP페이지에 전송하였습니다.
  
  
  
  `교수계정에서 출석기능을 잘못 동작시키거나 학생들의 사정으로 인해 출석 값을 임의로 변경해야되는 상황이 있을 수 있기 때문에 출석을 관리할 수 있는 페이지를 추가로 제작하였습니다. `
  
  
  
- <div id="att-mgmt">출석관리페이지</div>
  
  ![attendance01_2](https://user-images.githubusercontent.com/43952470/106319533-ecb74080-62b4-11eb-8621-91a3f55ac43e.PNG)
  

[출석현황페이지](#att-list)의 상단 `숫자`를 통해 이동할 수 있습니다. 해당 페이지에서는 입력한 숫자에 해당하는 주의 출석현황을 삭제할 수 있고 학생들의 출석 값을 임의로 수정할 수 있습니다.



   ```java
@RequestMapping(value="/room/manager/del_Att.do")
public ModelAndView del_Att(CommandMap commandMap) throws Exception {
    ModelAndView mv = new ModelAndView("redirect:/room/manager/selectAttendance.do");
    roomService.delete_Attendance(commandMap.getMap());
    return mv;
}
   
@RequestMapping(value="/room/manager/Att_edit_func.do")
public ModelAndView Att_edit_func(CommandMap commandMap) throws Exception {
    ModelAndView mv = new ModelAndView("redirect:/room/manager/detailAtt.do");
    Map<String, Object> map = new HashMap<>() ;
    
    try {
        for(int i = 0; i < ((String[]) commandMap.get("id")).length; i++) {
            map.put(((String[]) commandMap.get("id"))[i], ((String[]) commandMap.get("attendance"))[i]);
        }
    } catch(Exception e) {
        map.put((String)commandMap.get("id"), (String) commandMap.get("attendance")) ;
    }
    
    roomService.Att_edit_func(map) ;
    
    return  mv;
}
   ```

`delete_Attendance`: `room_code`와 `week`값을 통해 저장된 주의 출석값이나 학생들에게 임의로 입력된 출석값들을 삭제하기 위한 기능.

학생의 출석값을 임의로 변경을 하면 값들을 배열형태로 받아오는데 반복문을 통해서 `map`에 입력하고 `Att_edit_func`에 전송하여 출석 값들을 변경했습니다.



2. <div id="student">학생계정</div>

   ![attendance05](https://user-images.githubusercontent.com/43952470/106319554-f50f7b80-62b4-11eb-84b5-eaa955dfaa01.PNG)

   학생계정의 출석현황페이지에서는 `출석`버튼을 통해 난수를 입력할 수 있고 출석현황을 조회할 수 있습니다.

   

3. <div id="process">동작과정</div>

   1. 교수계정에서 `출석`기능 동작

      ![attendance02](https://user-images.githubusercontent.com/43952470/106319570-fb055c80-62b4-11eb-91d6-85e908e04a19.PNG)

      수업의 주(default=마지막 출결날짜+1)와 수업날짜를 입력

      

      ```java
      @Override
      public Map<String, Object> attendance_action(Map<String, Object> map) throws Exception {
          map.put("number", (int)(Math.random() * 100) + 1) ;
          
          List<Map<String, Object>> studentList = roomDAO.Attroom_new_user(map) ;
          List<Map<String, Object>> student_oldList = roomDAO.student_oldList(map) ;
          
          for(int i=0; i<=studentList.size()-1; i++) { /* 현재 수강중인 학생들 */
              int count = -1 ;
              for(int j=0; j<=student_oldList.size()-1; j++){ /* 출석값이 있는 학생들 */
                  if(studentList.get(i).containsValue(student_oldList.get(j).get("id"))) {
                      studentList.get(i).put("room_code", map.get("room_code")) ;
      				studentList.get(i).put("week", map.get("week")) ;
      				roomDAO.add_stuAttList(studentList.get(i)) ; //기존 수강생들은 출석 입력
      				break ;
      			} else { //새로들어온 학생인지 연산
      				count++ ;
      			}
      		}
      		if(count == student_oldList.size()-1) { //새로들어온 학생일 때
      			List<Map<String, Object>> oldAttdata = roomDAO.Attroom_data(map) ;
      			Map<String, Object> saveNewStudent = new HashMap<>() ;
      
      			saveNewStudent.put("room_code", map.get("room_code")) ;
      			saveNewStudent.put("id", studentList.get(i).get("id")) ;
      			for(int z=0; z<=(oldAttdata.size()-1); z++) {
      				saveNewStudent.put("week", oldAttdata.get(z).get("week")) ;
      				roomDAO.add_stuAttList(saveNewStudent) ; //att테이블에 이전 주까지 출석 값 (-)로 입력
      				saveNewStudent.remove("week") ;
      			}
      			saveNewStudent.put("week", map.get("week")) ;
      			roomDAO.add_stuAttList(saveNewStudent) ; //att테이블에 당일 주 출석 값 (-)로 입력
      			if(roomDAO.stu_totalAtt(saveNewStudent).isEmpty()) {
      				roomDAO.add_newStu_total(saveNewStudent); //total에 명단추가 --------> 값이 있을경우 생략
      			}
      		}
      	}
      	roomDAO.attendance_action(map) ;
          
          return map ;
      }
      ```

      `확인` 버튼을 누르게되면 동작하는 기능으로 기본적으로 난수를 발생시키고 학생들의 출석값에 `-`을 입력합니다.

      

      강의를 진행하면 많은 예외적인 상황이 발생하게 됩니다.  해당 기능에서는 강의진행 중, 새로운 학생들이 강의에 들어오게되면 1, 2주차의 출석값이 없는 문제를 해결하기위해 `새로들어온 학생일 때`조건문을 동작시켜 출석기능 동작이전의 값들을 `-`로 입력해주는 기능을 구현했습니다.

      

   2. 임의의 난수 발생

      ![attendance03](https://user-images.githubusercontent.com/43952470/106319583-0193d400-62b5-11eb-96c8-0141c1e5afbb.PNG)

      

   3. 학생계정에서 임의의 난수를 입력

      ![attendance06](https://user-images.githubusercontent.com/43952470/106319603-0789b500-62b5-11eb-8a6c-03e916c47793.PNG)
   
      ```java
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
   
      학생이 숫자를 입력할 때 입력된 난수와 동작시간을 불러와 학생들의 입력한 값과 비교를 하여 시간에 따른 출석 값을 자동으로 저장합니다.

      

   4. 시간에 따른 출석값 입력

      ![attendance07](https://user-images.githubusercontent.com/43952470/106319617-0d7f9600-62b5-11eb-88ef-7938dc35f615.PNG)

      `-`: 기능동작 후 0~10분동안 유지되는 값으로 출석기능이 동작중일 때, 나타나는 값입니다.

      `O`: 기능동작 후 0~3분에 난수를 입력하면 발생하는 값으로 출석을 의미합니다.

      `X`: 기능동작 후 3~10분에 난수를 입력하면 발생하는 값으로 지각을 의미합니다.
   
      `/`: 기능동작 후 10분이 지나도 난수를 입력하지 않으면 발생하는 값으로 결석을 의미합니다.