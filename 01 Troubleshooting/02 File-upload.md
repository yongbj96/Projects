# File upload to Server



```
두 홈페이지를 유지보수 하던 중, 스쿨웨어의 경우 파일업로드를 하면 서버와 nginx에 저장되지만
Na-Lab의 경우 서버와 nginx에 저장되지 않고 Tomcat에만 파일이 저장되었습니다.
Tomcat에만 파일이 저장되면 서버를 재구동하거나 홈페이지를 재배포하면 파일들이 사라지는것을 확인하였고
심각한 문제를 발생시킬 수 있다고 생각하여 해결하기 위해 `docker container`를 새로 구축했습니다.
```



---

1. #### docker run (-v) 에 대하여

   처음 `docker container`를 구축할때, 기존에 생성되어 있던 컨테이너를 `inspect`해서 컨테이너의 정보를 확인했습니다.

   ``` nginx
   {
        ...생략...
           
        "HostConfig": {
            "Binds": [
                "/etc/letsencrypt:/etc/nginx/ssl",
                "/etc/localtime:/etc/localtime",
                "/usr/etc/dockers/web/tomcat/schoolware/files:/etc/nginx/sch/files",
                "/usr/etc/dockers/web/tomcat/nalab/files:/etc/nginx/nalab/files"
            ],
            "NetworkMode": "default",
            "PortBindings": {
                "443/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "443"
                    }
                ],
                "80/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "80"
                    }
                ]
            },
            "Links": [
                "/schoolware:/nginx_server/schoolware",
                "/eportfolio:/nginx_server/e-portfolio",
                "/nalab:/nginx_server/nalab",
                "/push:/nginx_server/push",
                "/file:/nginx_server/file",
                "/chat:/nginx_server/chat"
            ],

         ...생략...
    }
   ```

   위는 현재 `docker container`를 `inspect`한 것으로
   
   ```
   -p 443:443 -p 80:80
   -v /etc/letsencrypt:/etc/nginx/ssl
   -v /etc/localtime:/etc/localtime
   -v /usr/etc/dockers/web/tomcat/schoolware/files:/etc/nginx/sch/files
   -v /usr/etc/dockers/web/tomcat/nalab/files:/etc/nginx/nalab/files
   -l ~
   ```

   의 명령어를 사용했다는 것을 유추할 수 있었고 해당 명령어들을 사용하여 새 `docker container`를 생성했습니다.

   

   파일업로드 시, `Tomcat`에는 파일이 전송되지만 `server`와 `nginx`에는 전송되지 않은 것을 통해 `tomcat`과 `server` 사이에 바인드가 되어있지 않다는 것을 알게되었습니다. 이후 `-v`옵션을 추가하여 `nalab`서버를 동작시킬 `tomcat`컨테이너를 생성했습니다.

   
   
   ``` nginx
   ...생략...
   
   "Binds": [
                   "/usr/etc/dockers/web/tomcat/nalab/files/file:/usr/local/tomcat/files/file",
                   "/etc/localtime:/etc/localtime",
                   "/usr/etc/dockers/web/tomcat/nalab/webapps:/usr/local/tomcat/webapps/"
               ],
   
   ...생략...
   ```
   
   다음과 같이 정상적으로 컨테이너를 동작시키게 되었고 파일업로드를 하게되면 `/usr/local/tomcat/files/file`위치에 파일이 저장되는 것도 확인했습니다.
   
   

---

2. #### url을 통해 파일을 불러오기

   정상적으로 파일을 `서버`와 `nginx`에 저장했지만 파일들을 불러오는 과정에서 403오류가 발생했습니다.

   확인결과 nginx 내부에 있는 `nalab/files/file`에 사용자가 접근할 수 없는 것이 문제였고 사용자가 `nginx 내부에 있는 file폴더`에 접근할 수 있도록 `default` 수정하여 이미지와 컨테이너를 새로 생성했습니다.

   

   ``` nginx
   ...생략...
      
      server {
      	listen 443;
      	server_name icnet.kornu.ac.kr;
      
          ...생략...
      
          location ^~/sch/files/file/{
          	root /etc/nginx/;
          }
      
          location ^~/sch/files/video/{
          	root /etc/nginx/;
          }
      
      	location ^~/nalab/files/file/{
      		root /etc/nginx/;
      	}
      }
      
      server {
          listen       80;
          server_name  icnet.kornu.ac.kr;
          
          location ^~/schoolware/ {
          	proxy_pass http://schoolware/schoolware/;
          }
      
          location ^~/e-portfolio/ {
          	proxy_pass http://e-portfolio/e-portfolio/;
          }
          location ^~/nalab/ {
          	proxy_pass http://nalab/nalab/;
          }
      
          ...생략...
      }
   ```

   이후 정상적으로 홈페이지에서 파일을 업로드하고 불러올 수 있게 되었습니다.

   

---

3. #### 결과

   ##### 3.1. 사진불러오기

   ![File-uploading01](https://user-images.githubusercontent.com/43952470/106359989-74568b00-6359-11eb-9672-b91d5e134cb1.PNG)

   

   ##### 3.2. 불러온 사진의 경로 조회

   ![File-uploading02](https://user-images.githubusercontent.com/43952470/106359996-79b3d580-6359-11eb-961f-8909541eedbf.PNG)

   

