# Build Project



> - [개발환경](#project)
>   - [Spring Framework](#spring)
>   - [mariaDB](#database)
>   - [Xshell](#emul)
>
> - [개발환경구축](#develop), 참고링크(흔한개발자)
>   - [CommonMap](#map)
>   - [AbstractDAO](#dao)
>   - [Interceptor](#interceptor)
>   - [database연동](#database)
> - [부트스트랩 연동](#bootstrap)

---



- <div id="project">개발환경구축</div>

  개발한 언어를 선정하게된 이유와 버전을 작성했습니다.

  

  - <div id="spring">Spring Framework</div>

    스쿨웨어 홈페이지를 유지보수 하면서 `MVC패턴`에 대해 파악할 수 있었습니다. 또한 유지보수를 하면서 `xml`파일을 자주 접했기 때문에 쉽게 개발환경을 구축할 수 있다고 생각했고 `Spring Framework`를 개발언어로 선택하게 되었습니다.

    사용버전: `springframework 5.2.2`, `aspectj 1.9.2`

    

  - <div id="database">mariaDB</div>

    데이터베이스의 경우 서버컴퓨터에 이미 `mariaDB`개발환경이 구축되어 있었기 때문에 `mariaDB`를 사용했습니다. 서버에서 사용할 `NaLab`, 로컬에서 사용할 테스트용 `NaLab_beta`스키마를 생성하였고 테이블들을 구성하였습니다.

    사용버전: `maria 10.1.21`

    

  - <div id="emul">Xshell</div>

    개발초기에는 서버컴퓨터에서 직접 작업을 했지만 졸업한 선배와 연락을 하던 중에 `에뮬레이터 클라이언트`를 알게되었습니다.  `에뮬레이터 클라이언트`중 Xshell을 설치하여 개인컴퓨터로 서버컴퓨터에 접근하여 작업을 진행했습니다.

    

---

- <div id="develop">개발환경구축</div>

  스쿨웨어 홈페이지에서 환경설정 오류가 날때 참고한 사이트로 개발환경 구축에 대하여 작성한 포스트를 활용했습니다. 링크: [흔한개발자](Https://addio3305.tistory.com/category/Spring)블로그

  

  - <div id="map">CommomMap</div>

    ```java
    public class CommandMap {
    	Map<String, Object> map = new HashMap<String, Object>();
    	
    	public Object get(String key) {
    		return map.get(key);
    	}
    	
    	public void put(String key, Object value) {
    		map.put(key, value);
    	}
    	
    	public Object remove(String key) {
    		return map.remove(key);
    	}
    	
    	public boolean containsKey(String key) {
    		return map.containsKey(key);
    	}
    	
    	public boolean containsValue(Object value) {
    		return map.containsValue(value);
    	}
    	
    	public void clear() {
    		map.clear();
    	}
    	
    	public Set<Entry<String, Object>> entrySet() {
    		return map.entrySet();
    	}
    	
    	public Set<String> keySet() {
    		return map.keySet();
    	}
    	
    	public boolean isEmpty() {
    		return map.isEmpty();
    	}
    	
    	public void putAll(Map<? extends String, ? extends Object> m) {
    		map.putAll(m);
    	}
    	
    	public Map<String, Object> getMap() {
    		return map;
    	}
    }
    ```

    `HandlerMethodArgumentResolver`를 사용하여 사용자의 요청이 Controller에 전달되기 전에 개발자가 확인, 수정할 수 있도록 구성하기위해 `CommandMap`을 생성하였고 명령어들을 정의했습니다.

    

  - <div id="dao">AbstractDAO</div>

    ```java
    public class AbstractDAO {
    	
        protected Log log = LogFactory.getLog(AbstractDAO.class);
    
    	@Autowired
    	private SqlSessionTemplate sqlSession;
    
        protected void printQueryId(String queryId) {
    		if(log.isDebugEnabled()){
    			log.debug("\t QueryId  \t:  " + queryId);
    		}
    	}
    
        public Object insert(String queryId, Object params){
    		printQueryId(queryId);
    		return sqlSession.insert(queryId, params);
    	}
    
    	public Object update(String queryId){
    		printQueryId(queryId);
    		return sqlSession.update(queryId);
    	}
    
        public Object update(String queryId, Object params){
    		printQueryId(queryId);
    		return sqlSession.update(queryId, params);
    	}
    
        public Object delete(String queryId, Object params){
    		printQueryId(queryId);
    		return sqlSession.delete(queryId, params);
    	}
    
        public Object selectOne(String queryId){
    		printQueryId(queryId);
    		return sqlSession.selectOne(queryId);
    	}
    
        public Object selectOne(String queryId, Object params){
    		printQueryId(queryId);
    		return sqlSession.selectOne(queryId, params);
    	}
    
        @SuppressWarnings("rawtypes")
    	public List selectList(String queryId){
    		printQueryId(queryId);
    		return sqlSession.selectList(queryId);
    	}
    
        @SuppressWarnings("rawtypes")
    	public List selectList(String queryId, Object params){
    		printQueryId(queryId);
    		return sqlSession.selectList(queryId,params);
    	}
    }
    ```

    `printQueryId` 메서드를 사용해 입력한 SQL구문을 확인할 수 있고 받아오는 데이터들을 조회할 수 있도록 `AbstractDAO`페이지를 작성하였습니다.

    

  - <div id="interceptor">Interceptor</div>

    ```java
    public class LoggerInterceptor extends HandlerInterceptorAdapter {
    	protected Log log = LogFactory.getLog(LoggerInterceptor.class);
    
    	@Override
    	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    		if (log.isDebugEnabled()) {
    			log.debug(" Request URI \t:  " + request.getRequestURI());
    		}
    		log.debug("[Request URI]," + request.getRequestURI());
    		return super.preHandle(request, response, handler);
    	}
    
    	@Override
    	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    		if (log.isDebugEnabled()) {
    			log.debug("======================================           END          ======================================\n");
    		}
    	}
    }
    ```

    사용자가 url을 요청했을 때 동작과정을 확인할 수 있도록 `Interceptor`파일을 작성했습니다.

    

  - <div id="database">DataBase 연동</div>

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
    			<property name="sqlPrefix" value="SQL : "></property>
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

    서버용 dataBase 테이블과 로컬 테스트용 dataBase 테이블을 따로 사용하기 때문에 `디버그용`, `배포용` 연결방식을 다르게 코딩했습니다.



---

- <div id="bootstrap">부트스트랩 연동</div>

  `웹퍼블리셔`나 `프론트엔드담당자`가 없고 개발자들끼리 환경을 구축하기 때문에 UI에 많을 시간 소모를 할 수 없어서 부트스트랩을 사용하여 홈페이지 기본 디자인을 구성했습니다.

  

  - 부트스트랩 적용화면

    ![bootstrap01](https://user-images.githubusercontent.com/43952470/106357338-ee324880-6348-11eb-847d-9998fce29040.PNG)

    

  - 수정한 홈페이지 ui

    ![bootstrap02](https://user-images.githubusercontent.com/43952470/106357342-f4c0c000-6348-11eb-9676-387b889abd9a.PNG)

    

