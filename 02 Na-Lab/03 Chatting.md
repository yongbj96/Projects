# Chatting

>- 채팅시스템
>    - [채팅 불러오기](#read)
>    - [채팅 전송하기](#send)
>    - [채팅방 만들기](#make)



---

- 채팅시스템

  LMS시스템을 서비스하면서 채팅시스템을 제작해보자는 의견이 제시되었고 JSP페이지에서도 `Ajax`와 `JSON`을 사용하여 비동기적으로 채팅시스템을 제작할 수 있다는 자료들을 찾을 수 있었습니다. 이후 자료들을 토대로 채팅시스템을 제작해보게 되었습니다.

  

  - <div id="read">채팅내용 불러오기</div>

    ![chatting02](https://user-images.githubusercontent.com/43952470/106357368-0a35ea00-6349-11eb-9950-cf99762b02ce.PNG)

    채팅 시스템을 구현하기 위해 실시간으로 `DB`에 저장된 값을 가져와서 갱신할 수 있도록 코딩하였습니다.

    

    매 3초마다 `getchatMessage`함수를 동작시키게 되고 `MessageList`함수를 호출하여 `chatController.Message.do`에서 DB에 저장된 채팅내용을 불러오는 기능을 작성했습니다.

    

    이후에는 `addChat`함수를 사용하여 불러온 채팅의 `id`값을 기반으로 상대방과 사용중인 유저의 채팅내역을 다르게 구성할 수 있도록 작성하였습니다.

    ```javascript
    function getChatMessage(room_num) {
        roomNum = room_num; //새로 대화하는 상대 설정
        lastNUM = 0;
        clearInterval(intervalId); //이전 함수 종료
        $('#Message').empty(); //이전 채팅내용 비우기
        MessageList();
        intervalId = setInterval(function() { //새로운 함수를 변수로 할당, 3.0초마다 호출
            MessageList();
        }, 3000);
    }
    function MessageList() {
        $.ajax({
            type: "POST",
            url: "Message.do", //ChatController.java
            data: { //Controller로 보내는 데이터
                id: encodeURIComponent(${user_info.id}),
                room_num: encodeURIComponent(roomNum),
                lastnum: encodeURIComponent(lastNUM)
            },
            datatype: "json",
            success: function(data) {
                if(lastNUM==0 && data == "") {
                    $('#Message').empty();
                    $('#Message').append("채팅을 시작하세요");
                }
                var parsed = JSON.parse(data);
                var result = parsed.result;
                for(var i=0; i<result.length; i++) {
                    addChat(result[i][0].value, result[i][1].value);
                }
                lastNUM = Number(parsed.last);
            },
            error: function(error) {
                console.log("error");
            }
        });
    }
    function addChat(id, message) { //UI꾸미는 곳
        if(id == ${user_info.id}) {
            $('#Message').append('<div style="display: flow-root;">' +
                '<div style="float: right; padding: 10px; margin: 10px; border-radius: 15px; background-color: #f5deb390;"">' +
                message +
                '</div></div>');
        } else {
            $('#Message').append('<div style="display: flow-root;">' +
                '<div style="float: left; padding: 10px; margin: 10px; border-radius: 15px; background-color: #b0e0e690;">' +
                message +
                '</div></div>');
        }
        $('#Message').scrollTop($('#Message')[0].scrollHeight);
    }
    ```

    

  - <div id="send">내용 전송하기</div>

    ![chatting02](https://user-images.githubusercontent.com/43952470/106357368-0a35ea00-6349-11eb-9950-cf99762b02ce.PNG)

    전송버튼을 통해서 `sendMessage`를 호출하고 입력된 내용과 채팅방id, 전송될 유저의 id 파라미터들을`chatController.sendMessage.do`로 전송하게 되고 DB에 저장합니다.

    

    ```javascript
    function sendMessage() { //메시지 보내기
        $.ajax({
            type: "POST",
            url: "sendMessage.do",
            data: {
                id: encodeURIComponent(${user_info.id}),
                room_num: encodeURIComponent(roomNum),
                message: encodeURIComponent($('#MessageData')[0].value)
            },
            datatype: "json",
            error: function(error) {
                console.log("error");
            }
        });
        $('#MessageData').val(''); //입력한값 지우기
    }
    ```

    

  - <div id="make">채팅방 생성하기</div>

    ![chatting02](https://user-images.githubusercontent.com/43952470/106357368-0a35ea00-6349-11eb-9950-cf99762b02ce.PNG)

    좌측 `채팅방 만들기`버튼을 클릭하면

    

    ![chatting03](https://user-images.githubusercontent.com/43952470/106357373-14f07f00-6349-11eb-8a58-b8439fb7f339.PNG)

    모달창이 열리게되고 학생을 선택하여 채팅방을 제작할 수 있는 기능을 작성했습니다.
  
    
    
    ```java
    function addUser($target) {
        var pickUser = {
            id : $target.data('userid'),
            name : $target.data('username')
        };
        var html = '<div class="item" id="pick_'+pickUser.id+'" data-userID="'+pickUser.id+'">\
        <a href="javascript:;" style="text-align: center; display: table-caption;">\
        <input type="hidden" name="member" value="'+pickUser.id+'">\
        <img style="height: 120px; padding: 10px;" src="https://icnet.kornu.ac.kr/nalab/files/file/'+pickUser.id+'.png" alt="회원 사진" onerror=\'this.src="${pageContext.request.contextPath}/assets/img/user.jpg"\'>\
        \
        <small>'+pickUser.name+'</small>\
        </a>\
        </div>';
        $('#Member').append(html);
    }
    //선택된 회원에 회원 삭제
    function delUser($target) {
        $('#pick_'+$target.data('userid')).remove();
    }
    //검색 결과 체크박스로 회원 선택
    $(document).on('click','.ckbox',function () {
        var $target = $(this);
        //체크 상태 이면 선택 회원 리스트에 추가
        $target.is(":checked") ? addUser($target) : delUser($target);
    });
    //선택된 회원에서 네모난 회원 선택시 회원 삭제
    $(document).on('click','.item',function () {
        var target_id = $(this).data('userid');
        if(confirm('선택 취소하시겠습니까?')){
            $('.ckbox').each(function(){
                var $target = $(this);
                if($target.is(":checked") && $target.data('userid') == target_id){
                    $(this).prop('checked',false);
                }
            });
            $(this).remove();
        }
    });
    ```
    
    
  
  ![chatting04](https://user-images.githubusercontent.com/43952470/106357423-508b4900-6349-11eb-8858-78745a9b06a6.PNG)
  
  학생을 클릭하면 우측 선택칸이 활성화되며 상단에 해당 학생이 설정한 사진과 이름이 나타납니다. 선택을 해제하기 위해서 선택칸과 상단의 학생ui를 동기화시켜서 어느 한쪽을 클릭하더라도 해당하는 학생의 선택을 취소할 수 있습니다.