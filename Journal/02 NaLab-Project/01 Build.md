# Na-Lab Project

## 01 Build



```
동아리활동으로 `스쿨웨어`를 유지보수하던 도중, 나사렛대학교 Na-Engage 센터에서 캡스톤디자인 수업을 진행하면서 사용할 수 있는 홈페이지를 제작해달라는 요청을 받게 되었습니다.
동아리원들과 함께 프로젝트를 참여했고 02파일에서는 사용한 개발환경선정이유와 버전, 기초코딩에 대하여 작성을 했습니다.
```



---

1. #### 개발환경

   ##### 1.1. IntelliJ

   ![intelliJ](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d5/IntelliJ_IDEA_Logo.svg/64px-IntelliJ_IDEA_Logo.svg.png)

   기존 프로그래밍 통합 개발 환경으로 `eclipse(이클립스)`를 사용하고 있었지만 `Spring Framework`를 사용하면서 많은 페이지를 코딩하게 되었고 점점 이클립스가 무거워지더니 동작속도에서도 눈에띄게 느려졌으며 종종 종료되는 현상을 겪게되었습니다. 다른 개발 환경을 찾던도중 `IntelliJ`가 `Spring`도 지원을 하고 교육을 목적으로 학생인증을 받으면 `Ultimate`버전도 무료로 배포하기 때문에 해당 개발 환경을 사용하게 되었습니다.

   

   ##### 1.2. Spring Framework

   ![Spring](https://upload.wikimedia.org/wikipedia/commons/thumb/4/44/Spring_Framework_Logo_2018.svg/200px-Spring_Framework_Logo_2018.svg.png)

   `스쿨웨어`홈페이지를 유지보수 하면서 지속적으로 접하게 되었던 프레임워크로 동아리원들도 가장 많이 사용하고 있던 오픈 소스 애플리케이션 프레임워크이다. 국내 여러 기업에서도 해당 프레임워크를 사용하고 있기 때문에 졸업후에 취업에도 도움을 줄 수 있다고 생각하여 사용하게 되었습니다.

   

   사용버전: `springframework 5.2.2`, `aspectj 1.9.2`

   

   ##### 1.3. mariaDB

   ![mariaDB](https://upload.wikimedia.org/wikipedia/commons/thumb/c/c9/MariaDB_Logo.png/220px-MariaDB_Logo.png)

   `스쿨웨어`홈페이지에서 사용하고있던 데이터베이스로 이미 서버컴퓨터에 개발환경이 구축되어있었고 가장 많이 사용한 언어이기 때문에 사용하게되었습니다.

   

   사용버전: `maria 10.1.21`

   

   ##### 1.4. Xshell

   ![Xshell](https://upload.wikimedia.org/wikipedia/commons/thumb/6/6d/Xshell_6_logo.png/64px-Xshell_6_logo.png)

   개발초기에는 직접 서버컴퓨터에서 작업을 하였으나 `에뮬레이터 클라이언트`를 사용하면 개인컴퓨터에서 서버컴퓨터에 접근할 수 있다는 것을 알게 되었다. 이후 Xshell이라는 클라이언트를 설치하였습니다.

   

---

2. #### 개발환경구축, 참고링크([흔한개발자](Https://addio3305.tistory.com/category/Spring))

   ##### 2.1. 프로젝트생성

   프로젝트는 `Spring MVC Project`로 생성을 하고 `Tomcat 8`을 사용했습니다.

   

   ##### 2.2. Interceptor(인터셉터) 작성

   ```java
   public class LoggerInterceptor extends HandlerInterceptorAdapter {
       protected Log log = LogFactory.getLog(LoggerInterceptor.class);
   
       @Override
       public boolean preHandle(HttpServletRequest request,
                                HttpServletResponse response,
                                Object handler) throws Exception {
           if (log.isDebugEnabled()) {
               log.debug(" Request URI \t:  " + request.getRequestURI());
           }
           log.debug("[Request URI]," + request.getRequestURI());
           return super.preHandle(request, response, handler);
       }
   
       @Override
       public void postHandle(HttpServletRequest request,
                              HttpServletResponse response,
                              Object handler,
                              ModelAndView modelAndView) throws Exception {
           if (log.isDebugEnabled()) {
               log.debug("==============           END          ===============\n");
           }
       }
   }
   ```

   `흔한개발자`블로그를 토대로 작성했으며 사용자가 url을 요청했을 때 동작과정을 확인할 수 있도록 `Terminal`에 보여주는 기능입니다.

   

   ##### 2.3. 데이터베이스 연동

   ```java
   <!--&lt;!&ndash; 디버그용 &ndash;&gt;
   <bean id="dataSourceSpied" class="org.apache.commons.dbcp.BasicDataSource">
      <property name="driverClassName" value="org.mariadb.jdbc.Driver" />
         &lt;!&ndash; 테스트는 NaLab_beta &ndash;&gt;
      <property name="url" value="...생략..." />
         <property name="username" value="root" />
         <property name="password" value="...생략..." />
         &lt;!&ndash; 유효성 검사용 쿼리 &ndash;&gt;
      <property name="validationQuery" value="select 1" />
         &lt;!&ndash; 커넥션을 사용하지않을 때, 유효성 검사 여부판단 &ndash;&gt;
      <property name="testWhileIdle" value="true" />
         &lt;!&ndash; 해당 밀리초마다 유효성 검사 진행 &ndash;&gt;
      <property name="timeBetweenEvictionRunsMillis" value="7200000" />
   </bean>
   
   <bean id="dataSource" class="net.sf.log4jdbc.Log4jdbcProxyDataSource">
     <constructor-arg ref="dataSourceSpied" />
     <property name="logFormatter">
        <bean class="net.sf.log4jdbc.tools.Log4JdbcCustomFormatter">
            <property name="loggingType" value="MULTI_LINE" />
            <property name="sqlPrefix" value="SQL : " />
         </bean>
      </property>
   </bean>-->
   
      <!-- 배포용 -->
      <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
          <property name="driverClassName" value="org.mariadb.jdbc.Driver" />
          <!-- 테스트는 NaLab_beta -->
          <property name="url" value="...생략..." />
          <property name="username" value="root" />
          <property name="password" value="...생략..." />
          <!-- 유효 검사용 쿼리 -->
          <property name="validationQuery" value="select 1"/>
          <!-- 커넥션을 사용하지않을 때, 유효성 검사 여부판단 -->
          <property name="testWhileIdle" value="true"/>
          <!-- 해당 밀리초마다 유효성 검사 진행 -->
          <property name="timeBetweenEvictionRunsMillis" value="7200000"/>
       </bean>
   ```

   서버용 dataBase 테이블과 로컬 테스트용 dataBase 테이블을 따로 생성을 했기 때문에 `배포용`, `디버그용`으로 나눠서 사용할 수 있도록 코드를 작성하였습니다.

   
