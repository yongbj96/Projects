# ERROR: 502 Bad Gateway



* ### 오류발생

​						Https://icnet.kornu.ac.kr/schoolware

![Bad Gateway11](https://user-images.githubusercontent.com/43952470/106335518-58f36d80-62d0-11eb-908e-df6949bb6ab2.PNG)

스쿨웨어 홈페이지를 유지보수 하던중에 메인 페이지에서 502 Bad Gateway오류가 발생하는것을 확인하였습니다. 따라서, 서버컴퓨터(centOS)의 작업관리자(top)를 확인하였고 `java`와 `mysqld`에서 과부하가 걸리고있다는 것을 확인할 수 있었습니다.



![Bad Gateway01](https://user-images.githubusercontent.com/43952470/106335544-614ba880-62d0-11eb-9cd4-1100f46197f8.PNG)



---

- ### 디버깅

이후에 곧바로 메인페이지에서 오류가 날 수 있는 부분을 디버깅을 진행하였습니다.



![Bad Gateway12](https://user-images.githubusercontent.com/43952470/106335560-6872b680-62d0-11eb-86c8-eb5f7e648c20.PNG)

`공지글부분`: 나사렛대학교 홈페이지를 크롤링해서 사용자에게 출력해주는 기능

`수업시간표`: 학생의 강의내용을 DB에서가져와 테이블로 출력해주는 기능

`공지&QnA&FAQ`: 홈페이지의 게시판의 글을 출력해주는 기능

`학사일정`: 학교 홈페이지의 학사일정 부분을 크롤링해서 사용자에게 출력하는 기능

`모임방썸네일`: 수강중인 강의들을 불러와 썸네일방식으로 출력하여 포워딩시켜주는 기능



위의 기능들 중, `모임방썸네일` 기능에서 SQL의 구문으로 인해 `Duration`이 높게 측정되는것을 확인하였고 코드를 확인했습니다.



---

- ### 원인 분석 및 해결방안

- #### 변경 전 코드

```java
<!-- 수강중인 모임방의 최신 글 10개만 -->
	
		(
			SELECT
				p.id,p.title,p.write_date,p.post_num,p.room_code,n.room_name,@room_type:='normal' as room_type,b.board_type
			FROM
				request_room re
			JOIN
				normal_room_info n
			ON
				re.room_code = n.room_code
			JOIN
				post_info p
			ON
				re.room_code = p.room_code
			JOIN
				board_info b
			ON
				b.board_code = p.board_code
			WHERE
				b.notices_flag = '1'
			AND
				re.accept_flag= '1'
			AND
				n.setup = '1'
			AND
				re.student_id = #{id}
			ORDER BY
				write_date DESC
		)
		UNION
		(
			SELECT
				p.id,p.title,p.write_date,p.post_num,p.room_code,n.subject_name as room_name,@room_type:='subject' as room_type,b.board_type
			FROM
				request_room re
			JOIN
				subject_info n
			ON
				re.room_code = n.room_code
			JOIN
				post_info p
			ON
				re.room_code = p.room_code
			JOIN
				board_info b
			ON
				b.board_code = p.board_code
			WHERE
				b.notices_flag = '1'
			AND
				re.accept_flag= '1'
			AND
				n.setup = '1'
			AND
				re.student_id = #{id}
			<if test="open_year != null">
				AND n.open_year = #{open_year}
			</if>
			<if test='season != null'>
				AND n.season = #{season}
			</if>
			ORDER BY
				write_date DESC
		)
		ORDER BY
		write_date DESC
		<if test='limit != null'>
			limit #{limit}
		</if>
		<if test='limit == null'>
			limit 10
		</if>
	</select>
```

코드가 한눈에 들어오지 않아 연산과정을 도식화 시켜보게 되었고 아래와 같은 결과를 얻을 수 있었습니다.

![Bad Gateway13](https://user-images.githubusercontent.com/43952470/106335571-70caf180-62d0-11eb-81fa-cc7699ad7911.PNG)



위와 같은 방식으로 DB가 구성되어있기 때문에 `p`랑 연결된 부분에서 `FULL OUTER JOIN`이 발생하게 되었고 많은 연산량으로 인해 Duration이 높게 측정되었다는 것을 알게 되었습니다.



- #### 변경 후 코드

```java
<!-- 수강중인 모임방의 최신 글 10개만 -->
<select id="selectRegRoomPostInfoLimit10" parameterType="hashmap" resultType="hashmap">
	(
		SELECT
			p.write_date, p.post_num, p.room_code, @room_type:='normal' as room_type, p.title
		FROM
			normal_room_info n
		JOIN
			post_info p
		ON
			p.room_code = n.room_code
		JOIN
			request_room re
		ON
			p.room_code = re.room_code
		JOIN
			board_info b
		ON
			p.board_code = b.board_code
		WHERE
			re.student_id = #{id}
		AND
			re.accept_flag = 1
		AND
			b.notices_flag = 1
		AND
			n.setup = 1
	)
	UNION
	(
		SELECT
			p.write_date, p.post_num, p.room_code, @room_type:='subject' as room_type, p.title
		FROM
			subject_info n
		JOIN
			post_info p
		ON
			p.room_code = n.room_code
		JOIN
			request_room re
		ON
			p.room_code = re.room_code
		JOIN
			board_info b
		ON
			p.board_code = b.board_code
		WHERE
			re.student_id = #{id}
		AND
			re.accept_flag = 1
		AND
			b.notices_flag = 1
		AND
			n.setup = 1
		<if test="open_year != null">
			AND n.open_year = #{open_year}
		</if>
		<if test='season != null'>
			AND n.season = #{season}
		</if>
	)
	ORDER BY
		write_date DESC
	<if test='limit != null'>
		limit #{limit}
	</if>
	<if test='limit == null'>
		limit 10
	</if>
</select>
```

위 코드의 연산과정을 도식화 시킨 결과입니다.

![Bad Gateway14](https://user-images.githubusercontent.com/43952470/106335589-77f1ff80-62d0-11eb-8c45-19a1c28b26f7.PNG)

`p` 테이블과 모든 테이블에서 조인키를 사용하기 때문에 위 코드와 비교했을 때, 훨씬 적은량으로 테이블들을 JOIN하게 되었습니다.

---

- ### 최종결과

첫 번째 코드의 동작시간(Duration)입니다.

![Bad Gateway15](https://user-images.githubusercontent.com/43952470/106335595-7cb6b380-62d0-11eb-9930-3dd136537756.PNG)

(0.875 + 0.890 + 0.890 + 0.875 + 0.875) / 5 = 0.881



두 번째 코드의 동작시간(Duration)입니다.

![Bad Gateway16](https://user-images.githubusercontent.com/43952470/106335601-804a3a80-62d0-11eb-8b80-3f724b08b6be.PNG)

(0.063 + 0.047 + 0.062 + 0.063 + 0.063) / 5 = 0.0596

∴0.881 / 0.0596 ≒ 14.78,

해당 코드에 대해서 약 14.8배 동작속도를 빠르게 수정하였으며 전체적인 SQL구문을 재코딩하게 되었고 성능을 향상시킬 수 있었습니다.