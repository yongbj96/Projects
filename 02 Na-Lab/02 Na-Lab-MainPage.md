# Na-Lab 메인페이지 개발



##### 메인페이지

> 1. [썸네일](#thumbnail)
>    1. [페이징처리](#paging)
>    2. [학과 Search](#search)
>    3. Modal
> 2. [로그인테이블](#table)
>    1. [매니저](#manager)
>    2. [교수](#professor)
>    3. [조교](#assistant)
>    4. [학생](#student)

---



- 메인페이지(좌측하단: 썸네일, 우측하단: 로그인테이블)

![MainPage01](https://user-images.githubusercontent.com/43952470/106341449-b2fc2f00-62e0-11eb-959f-399cf74c1d89.PNG)

`썸네일`: 좌측하단에 위치하며 진행중인 강의목록과 간략한 정보들을 조회

`로그인테이블`: 우측하단에 위치하며 session이 없을땐 `Login`기능을 갖는다.([권한별 다른 테이블](#table)구성)



---

- <div id="thumbnail">썸네일</div>

  <img src="https://user-images.githubusercontent.com/43952470/106341464-bbed0080-62e0-11eb-8465-3d1e13338446.PNG" alt="MainPage02" style="zoom:80%;" />

  `썸네일`테이블을 구성할 때 `4*2`, `5*2`, `6*2`등 여러 관점에서 구성을 해본 결과 모든 화면크기에서 `4*2`가 보기 편하다는 의견이 많아 8개의 썸네일을 화면에 배치하게 되었습니다.

  

  - <div id="paging">페이징처리</div>

```java
@RequestMapping(value="/main_page.do")
    public ModelAndView main_page(CommandMap commandMap, HttpServletRequest req) throws Exception {
    ... 생략 ...
    List<Map<String, Object>> _thumbnail = commonService.thumbnail(commandMap.getMap());// Lab list를 받아오는 변수

        List<Map<String, Object>> thumbnail = new ArrayList<>();//Lab list를 8개씩 끊는 변수
        List<List<Map<String, Object>>> result = new ArrayList<>(); //최종값을 전달할 변수
        int index=0; //index는 div의 개수를 설정
        for(int a=0; a<_thumbnail.size(); a++) {
            if(a!=0 && a%8==0) {
                result.add(thumbnail);
                thumbnail = new ArrayList<>();
                index++;
            }
            if(!_thumbnail.get(index*8+(a%8)).isEmpty()) {
                thumbnail.add(_thumbnail.get((index)*8+(a%8)));
            }
        }
        if(!thumbnail.isEmpty()) {
            result.add(thumbnail);
            index++;
        }
        
        mv.addObject("major_list", commonService.major_list(commandMap.getMap()));
        mv.addObject("thumbnail_list", result);
        mv.addObject("search", commandMap.getMap());
        
    ... 생략 ...
 }
```

`SQL`을 통해 받아온 `Model`을 `Controller`에서 8개씩 끊어 새로운 List를 구성하고 구성된 List들을 `thumbnail_list` 변수로 JSP페이지에 전달을 하였습니다.



```html
<%-- 테이블을 나누어 페이징처리 -->
<div style="min-width: 710px; max-width: 870px; height: 450px;" class="card-body">
    <div>
        <c:if test="${empty thumbnail_list}">
            <div style="margin-top: 100px; text-align: center;">
                <i class="fas fa-chalkboard-teacher" style="font-size: 70px;"></i>
                <p>아직 등록된 모임방이 없습니다.</p>
                <p>모임방을 만들면 온라인에서 과제관리, 재정관리, 물품요청을 할 수 있습니다.</p>
            </div>
        </c:if>

        <c:forEach items="${thumbnail_list}" var="list">
            <div class="mySlides fade2">
                <div class="thumb_test_body">
                <c:forEach items="${list}" var="thumbnail">
                    <a class="thumb_body" href="#${thumbnail.lab_code}_show" data-toggle="modal">
                        <div class="imgdiv">
                            <c:choose>
                                <c:when test="${thumbnail.photo ne null}">
                                    <img class="thumb_img" src="https://icnet.kornu.ac.kr/nalab/files/file/${thumbnail.photo}">
                                </c:when>
                                <c:otherwise>
                                    <img class="thumb_img" src="${pageContext.request.contextPath}/assets/img/img.jpg">
                                </c:otherwise>
                            </c:choose>
                        </div>
                        <article class="article">
                            <h1 class="title">
                                ${thumbnail.lab_title}
                            </h1>
                            <span class="contentpost">
                                (${thumbnail.name}교수님)
                            </span>
                        </article>
                    </a>
                </c:forEach>
                </div>
            </div>
        </c:forEach>
    </div>
</div>
<div style="text-align: center; height: 48px;">
    <c:forEach var="index" begin="1" step="1" end="${max_index}">
        <span class="dot" onclick="currentSlide(${index})"></span>
    </c:forEach>
</div>
```

`thumbnail_list`통해 받아온 `썸네일`을 분리된 테이블로 구성하여 코딩하였고 사용자가 클릭한 `index`의 테이블만 보일 수 있도록 `javascript`코딩하였습니다.

하단, javascript를 이용한 코딩

```javascript
var slideIndex = 1;
    showSlides(slideIndex);

    function plusSlides(n) {
        showSlides(slideIndex += n);
    }

    function currentSlide(n) {
        showSlides(slideIndex = n);
    }

    function showSlides(n) {
        var i;
        var slides = document.getElementsByClassName("mySlides");
        var dots = document.getElementsByClassName("dot");

        if (n > slides.length) {slideIndex = 1}

        if (n < 1) {slideIndex = slides.length}

        for (i = 0; i < slides.length; i++) {
            slides[i].style.display = "none";
        }

        for (i = 0; i < dots.length; i++) {
            dots[i].className = dots[i].className.replace(" active", "");
        }

        slides[slideIndex-1].style.display = "block";
        dots[slideIndex-1].className += " active";
    }
```

---

- <div id="table">로그인테이블</div>
  - 기본적으로 session값이 없을때, 로그인기능을 수행할 수 있도록 [코딩](#coding)하였습니다.

  ![MainPage03](https://user-images.githubusercontent.com/43952470/106341477-c7402c00-62e0-11eb-8f84-cd5c322a6576.PNG)

  

  - 권한(auth)

    - <div id="manager">매니저(4)</div>

    ![MainPage04](https://user-images.githubusercontent.com/43952470/106341479-cad3b300-62e0-11eb-8b42-066ebfd796ae.PNG)

    매니저 권한에서는 센터에서 운영하는 `Lab예산` 및 `캡스톤디자인예산`을 조회할 수 있으며 운영중인 모임방현황을 확인할 수 있도록 테이블을 구성하였습니다.

    

    - <div id="professor">교수(3)</div>

    ![MainPage05](https://user-images.githubusercontent.com/43952470/106341483-cf986700-62e0-11eb-9845-74d09f262947.PNG)

    교수계정에서는 교수가 운영중인 `Lab모임방` 및 `캡스톤디자인`의 예산사용현황을 조회할 수 있으며 강의에서 구성된 팀의 수를 조회할 수 있도록 테이블을 구성하였습니다.

    

    - <div id="assistant">조교(2)</div>

    ![MainPage06](https://user-images.githubusercontent.com/43952470/106341488-d45d1b00-62e0-11eb-9511-f95a18a9de90.PNG)

    조교계정에서는 학과에서 운영중인 `Lab예산` 및 `캡스톤디자인예산`을 조회할 수 있습니다. 추가로 최하단에 학생들이 신청한 물품요청알림을 메인페이지에서 바로 확인할 수 있도록 테이블을 구성하였습니다.

    

    - <div id="student">학생(1)</div>

    ![MainPage07](https://user-images.githubusercontent.com/43952470/106341495-da52fc00-62e0-11eb-85c8-9a5be2b665ff.PNG)

      학생계정에서는 학생팀의 예산사용내역을 조회할 수 있도록 구현하였으며 `Lab모임방` 또는 `캡스톤디자인`의 과제목록을 확인할 수 있도록 테이블을 구성하였습니다. 추가로 과제목록은 미제출한 과제들을 대상으로 날짜 순서대로 나타나도록 구현하였습니다.



- <div id="coding">코딩</div>

  - main.jsp(로그인테이블)

    ```html
    <div class="col-xl-4 col-lg-5">
        <div class="card shadow mb-4 login_size">
            <c:choose>
                <c:when test="${user_info eq null}"> <%-- Session에 값이 없을 때 Start -->
                    <div class="card-header py-3 d-flex flex-row align-items-center justify-content-between" style="height: 71px;">
                        <h6 class="m-0 font-weight-bold text-primary">로그인</h6>
                    </div>
                    <div class="card-body">
                        <form action="${pageContext.request.contextPath}/loginCK.do" class="sky-form" novalidate="novalidate" method="post">
                            <fieldset>
                                <div>
                                    <section>
                                        <div class="row input_size">
                                            <label class="label col col-4">학번/사번</label>
                                            <div class="col col-8">
                                                <label class="input">
                                                    <i class="icon-append fa fa-user"></i>
                                                    <input type="text" name="id" autocomplete="off" autofocus>
                                                </label>
                                            </div>
                                        </div>
                                    </section>
                                    <section>
                                        <div class="row input_size">
                                            <label class="label col col-4">비밀번호</label>
                                            <div class="col col-8">
                                                <label class="input">
                                                    <i class="icon-append fa fa-lock"></i>
                                                    <input type="password" name="pw" autocomplete="current-password">
                                                </label>
                                                <div class="note">
                                                	<a href="#" onclick="javascript:alert('준비중입니다.')">비밀번호 찾기</a>
                                                </div>
                                            </div>
                                        </div>
                                    </section>
                                </div>
                            </fieldset>
                            <footer style="position: absolute; bottom: 0px; width: 90%;">
                                <button class="btn btn-info btn-sm" type="submit">로그인</button>
                                <button class="btn btn-secondary btn-sm" type="button" onclick="location.href='${pageContext.request.contextPath}/registration_page.do'">회원가입</button>
                            </footer>
                        </form>
                    </div>
                </c:when> <%-- Session에 값이 없을 때 End -->
                <c:otherwise> <%-- 로그인 공통 -->
                    <div class="card-header py-3 d-flex flex-row align-items-center justify-content-between" style="height: 71px;">
                        <h6 class="m-0 font-weight-bold text-primary">${user_info.name}</h6>
                    </div>
                    <div class="card-body">
                        <div class="table-responsive" style="white-space:nowrap; width:100%; overflow:auto">
                            <div style="font-weight: 500;">예산사용현황</div>
                            <table class="table table-bordered" style="font-size: 12px;" width="100%" cellspacing="0">
                                <thead>
                                <tr style="background-color: #e0ffff;">
                                    <th>유형</th>
                                    <th>총 지원금</th>
                                    <th>사용</th>
                                    <th>잔액</th>
                                </tr>
                                </thead>
                                <tbody>
                                <tr>
                                    <td style="font-size: 14px; text-align: center;">Lab</td>
                                    <td><fmt:formatNumber value="${info.lab_total_price}" pattern="#,###"/></td>
                                    <td><fmt:formatNumber value="${info.lab_use_price}" pattern="#,###"/></td>
                                    <td>
                                        <fmt:formatNumber value="${info.lab_total_price - info.lab_use_price}" pattern="#,###"/>
                                    </td>
                                </tr>
                                <tr>
                                    <td style="font-size: 14px; text-align: center;">캡스톤</td>
                                    <td><fmt:formatNumber value="${info.cap_total_price}" pattern="#,###"/></td>
                                    <td><fmt:formatNumber value="${info.cap_use_price}" pattern="#,###"/></td>
                                    <td>
                                        <fmt:formatNumber value="${info.cap_total_price - info.cap_use_price}" pattern="#,###"/>
                                    </td>
                                </tr>
                                </tbody>
                            </table>
                            <p style="font-size: 11px; text-align: right;">＊(단위 : 원)</p>
                        </div>
                        <c:if test="${user_info.auth ne 1}">
                            <div class="table-responsive" style="white-space:nowrap; width:100%; overflow:auto">
                                <div style="font-weight: 500;">모임방현황</div>
                                <table class="table table-bordered" style="font-size: 14px; text-align: center;" width="100%" cellspacing="0">
                                    <thead>
                                    <tr style="background-color: #ffffe0;">
                                        <th width="26%">유형</th>
                                        <th width="37%">Lab</th>
                                        <th width="37%">캡스톤</th>
                                    </tr>
                                    </thead>
                                    <tbody>
                                    <c:choose>
                                        <c:when test="${user_info.auth eq 3}">
                                            <tr>
                                                <td>팀 수</td>
                                                <td><fmt:formatNumber value="${info.lab_team}" pattern="#,###"/></td>
                                                <td><fmt:formatNumber value="${info.cap_team}" pattern="#,###"/></td>
                                            </tr>
                                            <tr>
                                                <td>수강생</td>
                                                <td><fmt:formatNumber value="${info.lab_people}" pattern="#,###"/></td>
                                                <td><fmt:formatNumber value="${info.cap_people}" pattern="#,###"/></td>
                                            </tr>
                                        </c:when>
                                        <c:otherwise>
                                            <tr>
                                                <td>모임방</td>
                                                <td><fmt:formatNumber value="${info.lab}" pattern="#,###"/></td>
                                                <td><fmt:formatNumber value="${info.cap}" pattern="#,###"/></td>
                                            </tr>
                                            <tr>
                                                <td>팀 수</td>
                                                <td><fmt:formatNumber value="${info.lab_team}" pattern="#,###"/></td>
                                                <td><fmt:formatNumber value="${info.cap_team}" pattern="#,###"/></td>
                                            </tr>
                                        </c:otherwise>
                                    </c:choose>
                                    </tbody>
                                </table>
                            </div>
                        </c:if>
                        <c:if test="${user_info.auth eq 1}"> <%-- 학생계정 Start -->
                            <div style="font-weight: 500; width: 50%;">잔여과제현황</div>
                            <c:if test="${empty stu_report}">
                                <span style="font-size: 13px; color: red;">남은 과제가 없습니다.</span>
                            </c:if>
                            <c:forEach items="${stu_report}" var="list">
                                <div>
                                    <form action="${pageContext.request.contextPath}/LabBoardDetail.do" method="post">
                                        <input type="hidden" name="board_num" value="${list.board_num}">
                                        <input type="hidden" name="room" value="${list.room}">
                                        <button class="btn over" type="submit">${list.room} ${list.title}</button>
                                    </form>
                                </div>
                            </c:forEach>
                        </c:if> <%-- 학생계정 End -->
                        <c:if test="${user_info.auth eq 2}"> <%-- 조교계정 Start -->
                            <div style="display: flex; text-align: center;">
                                <div style="font-weight: 500; width: 50%;">물품요청상황</div>
                                <div style="width: 50%;">
                                    <a href="${pageContext.request.contextPath}/labList.do?paging=1">${info.alert}</a>개
                                </div>
                            </div>
                        </c:if> <%-- 조교계정 End -->
                        <c:if test="${user_info.auth eq 4}"> <%-- 매니저계정 Start -->
                            <div style="display: flex; text-align: center;">
                                <div style="font-weight: 500; width: 50%;">전체사용자<sub>(교수 / 학생)</sub></div>
                                <div style="width: 50%;"><span style="font-size: 20px;">${info.pro}</span>명
                                    / <span style="font-size: 20px;">${info.stu}</span>명</div>
                            </div>
                        </c:if> <%-- 매니저계정 End -->
                    </div>
                </c:otherwise>
            </c:choose>
        </div>
    </div>
    ```

    

  - 학생계정이 아닌 계정의 데이터를 가져오는 SQL

    ```sql
    <select id="info" parameterType="hashmap" resultType="hashmap">
        <![CDATA[
            SELECT
                ifnull(SUM(case when l.lab_code like 'L%' then t.total_price else 0 end), 0) lab_total_price,
                ifnull(SUM(case when l.lab_code like 'L%' then t.use_price else 0 end), 0) lab_use_price,
                ifnull(SUM(case when l.lab_code like 'C%' then t.total_price else 0 end), 0) cap_total_price,
                ifnull(SUM(case when l.lab_code like 'C%' then t.use_price else 0 end), 0) cap_use_price
            FROM
                lab_info l
            JOIN
                total_devPrice t
            ON
                l.lab_code = t.lab_code
            JOIN
                season_info s
            ON
                (l.open_year = s.now_year
                AND l.season = s.now_season)
            JOIN
                user_info u
            ON
                l.id = u.id
            JOIN
                major_info m
            ON
                u.major_code = m.major_code
        ]]>
        <if test="auth == 3">
            WHERE
                u.id = #{id}
        </if>
        <if test="auth == 2">
            WHERE
                CASE
                    WHEN
                        '-' IN (SELECT @code:=school_code FROM major_info WHERE major_code LIKE '${major_code}%')
                    THEN
                        m.major_code LIKE '${major_code}%'
                    ELSE
                        m.school_code LIKE @code
                END
        </if>
    </select>
    ```

    기본적으로 매니저계정을 대상으로 코딩을 하였으며 교수계정이나 조교계정에 맞춰 조건문을 동작시키도록 코딩하였습니다.

    

  - 학생계정의 데이터를 가져오는 SQL

    ```sql
    <select id="info_stu" parameterType="hashmap" resultType="hashmap">
        <![CDATA[
            SELECT
                ifnull(SUM(case when dt.lab_code like 'L%' then dt.team_total_price else 0 end), 0) lab_total_price,
                ifnull(SUM(case when dt.lab_code like 'L%' then uh.use_price else 0 end), 0) lab_use_price,
                ifnull(SUM(case when dt.lab_code like 'C%' then dt.team_total_price else 0 end), 0) cap_total_price,
                ifnull(SUM(case when dt.lab_code like 'C%' then uh.use_price else 0 end), 0) cap_use_price
            FROM
                del_teamPrice dt
            JOIN
                lab_info l
            ON
                dt.lab_code = l.lab_code
            JOIN
                season_info s
            ON
                (l.open_year = s.now_year
                AND l.season = s.now_season)
            JOIN
                team_member tm
            ON
                dt.team_code = tm.team_code
            JOIN
                (
                SELECT
                    team_code, SUM(use_price) use_price
                FROM
                    use_history
                WHERE
                    accept_code = 3
                GROUP BY team_code
                ) uh
            ON
                tm.team_code = uh.team_code
            WHERE
                tm.student_id = #{id};
        ]]>
    </select>
    ```

    동작방식이 다른계정들과 다른부분이 많아서 해당 SQL은 따로 코딩하였습니다.