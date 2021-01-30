# Server-Build



동아리 자체적으로 서버컴퓨터를 보유하고 있기 때문에 스쿨웨어 홈페이지를 서비스하기위해 서버 유지보수를 진행했고 나랩 홈페이지를 사용자들에게 새롭게 서비스할 수 있도록 컨테이너를 생성했습니다.

기본적으로 `docker container`를 기반으로 구성되어있으며 여러 `Tomcat`파일을 동작시키기 위한 목적으로 `nginx`를 사용했습니다.

![Server-Build 01](https://user-images.githubusercontent.com/43952470/106358470-cd212600-634f-11eb-8a11-3c7eee4c69ed.png)



---

- ##### nginx에서 nalab을 사용할 수 있도록 설정

  - nginx.conf 파일의 경우 기존 이미파일을 생성할 때 사용했던 코드를 그대로 사용했습니다.

    

  - default.conf

    ```nginx
    upstream schoolware {
        server schoolware:8080;
    }
    upstream e-portfolio {
        server e-portfolio:8080;
    }
    upstream nalab {
        server nalab:8080;
    }
    
    ...생략...
    
    server {
    	listen 443;
    	server_name icnet.kornu.ac.kr;
    
    	ssl ...생략...
    
    ssl_ciphers '...생략...';
    
    	ssl_prefer_server_ciphers on;
    
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

    `nginx`에서 `nalab`이 동작할 수 있도록 `stream`, `80 server`, `443 server`에 선언해준 코드입니다.

    

  - Dockerfile

    ![Server-Build 02](https://user-images.githubusercontent.com/43952470/106358466-c692ae80-634f-11eb-96f7-96e8b72a199d.png)

    `nginx`를 `Docker image`로 생성하기위해 `Dockerfile`을 수정했고 생성된 이미지를 통해 `컨테이너`를 동작시켰습니다.

    

---

- SSL 갱신 자동화

  `Let's Encrypt Authority X3`에서 인증서를 발급받아 SSL을 사용하고 있으나 3개월씩 유효기간이 생기고 유효기간 2주전부터 갱신이 가능하기 때문에 날짜를 체크하여 갱신하거나 인증이 만료된 후에 갱신을 진행했습니다. 이후, `crontab`스케줄러 패키지를 사용하여 인증서를 갱신하고 있다는 얘기를 듣게 되었고 코드상에 문제가 발생하여 갱신하지 못하는 시기가 생긴다고 생각하여 코드를 수정했습니다.

  

  ![Server-Build 03](https://user-images.githubusercontent.com/43952470/106358464-c2669100-634f-11eb-9b49-4d9dfc9a975a.png)

  

  다음과 같이 매달 1일 인증서를 갱신하고 있었으나 11일에 추가로 갱신할 수 있도록 코드를 추가했습니다.

  