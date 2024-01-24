# AWS_Server_Practice
 
### 24/01/15 ~ 

#### :video_camera: [강의 유튜브](https://www.youtube.com/playlist?list=PLHQvFs5CMVoQMcglHmtPz9ShY058H3veh)    &    :page_facing_up: [강의 참고 블로그](https://cholol.tistory.com/482)

---

## 서버 개발자가 하는 일
1. API(= TR or RPC) 만들기
2. 배치 만들기
3. DB 테이블 만들기

### 1. 개발 환경 구성 1
* AWS 서버 생성
  - 플랫폼 : Linux
  - SSH 프로토콜 사용
  - 보안 규칙
    + 서버 접속 : 22번 포트 사용
    + 서버에서 띄운 Django에 접속 : 8000번 포트 사용

* PuTTY를 통한 SSH 서버 연결
  - 서버 IP 주소 & 서버 구성 시 생성한 키를 기반으로 연결

* 서버(Ubuntu) 환경 구성
  - 로컬 환경에서 Django 설치 및 구성 후 github에 commit & push
  - 서버에서 github에 등록된 파일 pull하여 구성
  - `sudo apt update` (설치 가능한 패키지 리스트를 최신화)
  - `sudo apt install python3` (파이썬 설치)
  - `sudo apt install python3-pip` (pip 설치)
  - `pip3 install virtualenv` & `sudo apt install virtualenv` (가상 환경 virtualenv 설치)
  - `virtualenv venv --python=python3` (python3 버전으로 가상 환경 venv 구성)
  - `source venv/bin/activate` (가상 환경 실행)
  - 필요 모듈 추가 
    + [로컬 환경] `pip freeze > requirements.txt` (로컬 환경에서 사용된 모듈들을 txt 파일로 내보내기)
    + github에 commit & push
    + [서버 환경] pull -> `pip install -r requirements.txt` (필요 모듈을 서버 환경에 추가)
    + 
---

### 2. 개발 환경 구성 2
![image](https://github.com/NamSangwon/AWS_Server_Practice/assets/127469500/94441b73-7c72-4e21-b219-10baa3d8bb2d)

#### [pip를 이용하기 위해 가상 환경에서 진행 (`source venv/bin/activate`)]
#### `ps -ef | grep (프로그램명)` &rightarrow; 설정 적용 확인

* uwsgi (+ socket)
  - wsgi(Web Server Gateway Interface) == 웹서버와 웹 프레임워크 사이에 통신을 담당하는 인터페이스
    
  - `pip install uwsgi` (uwsgi 설치)
    
  - `uwsgi --http :8000 --module server_dev.wsgi` -> 서버에서 Django 실행
    + -- http :8000 == 0.0.0.0:8000
    + --module server_dev.wsgi == Django를 wsgi로 실행
      
  - uwsgi 커스텀 설정
    + etc 디렉토리에 uwsgi/sites/ 디렉토리 생성
    + `sudo vi AWS_Server_Prac` (/etc/uwsgi/sites/에 파일 작성) 
    + ![image](https://github.com/NamSangwon/AWS_Server_Practice/assets/127469500/8f60c94d-1411-4728-a661-3d3b252a0094)
    + `sudo mv AWS_Server_Practice AWS_Server_Practice.ini` (.ini로 파일명 변경)
      
  - `uwsgi -i /etc/uwsgi/sites/AWS_Server_Prac.ini` (uwsgi 커스텀 설정 적용)
    + `AWS_Server_Prac.ini` 파일은 **# 주석 처리 불가**이므로 **유의!**
    + `uwsgi -i /etc/uwsgi/sites/AWS_Server_Prac.ini -http :8000`으로 **config파일을 이용해서 uwsgi를 실행**
    + ***/tmp/ 디렉토리에서 `tail -f uwsgi.log`을 통해 로그 찍기***
  
* Nginx
  - `sudo apt-get install nginx` (nginx 설치) &rightarrow; 설치 시 기본 디렉토리 생성됨
    
  - `/etc/nginx/nginx/nginx.conf` &rightarrow; ***[nginx의 config](https://cholol.tistory.com/485)는 서버 개발 역량에 필수이므로 확인 요망!!***
    
  - `/etc/nginx/sites-enabled/default` 에서 실질적인 설정을 조정함 &rightarrow; 포트 번호를 80에서 8080으로 변경함
    
  - nginx의 virtual host config로 config파일 생성
    
  - ![image](https://github.com/NamSangwon/AWS_Server_Practice/assets/127469500/eadd8972-b0ca-49b5-a3a8-dfa180fd4e74)
    
  - ### nginx 가동 &rightarrow; /etc/nginx/nginx.conf에서 /etc/nginx/sites-enables/AWS_Server_Practice를 읽어서 80(http) 포트번호로 들어오는 요청을 uwsgi로 전송
    
  - ***`sudo systemctl start nginx`으로 nginx 실행***
    + AWS 인스턴스에 인바운드 규칙(HTTP, HTTPS, etc.) 추가

---

### 3. Docker로 서버 띄우기

**micro service architecture(MSA) = 각각의 기능별로 서버를 쪼개는 개념**
**docker가 등장 &rightarrow; 컨테이너로 서버를 운영/배포/관리가 쉬워짐**
***해당 실습에서는 Django와 nginx를 각각의 Docker로 여는 것이 목표!!***

* Docker 설치
  - `curl -fsSL https://get.docker.com/ | sudo sh` (docker 설치)
    
  - `sudo usermod -aG docker $USER` (현재 접속중인 사용자에게 권한 주기)
    
  - `mkdir docker-server` (docker의 이미지들을 관리할 디렉토리 생성)

* Django docker container 구성
  - 해당 github를 클론하여 django 불러오기
    
  - `vi Dockerfile`로 docker 사전 설정 작성 (docker 실행 시 사전 설정 파일이 실행됨)
    + ![image](https://github.com/NamSangwon/AWS_Server_Practice/assets/127469500/26beb411-6140-4e5c-94c0-37aacbb18c0a)
      
  - `docker build -t aws_server_prac/django .` (django의 docker 빌드하기 **경로명은 반드시 소문자로!!**)
    + `apt-get update`에서의 **에러** 발생 &rightarrow; `RUN echo "deb http://archive.debian.org/debian stretch main" > /etc/apt/sources.list`의 명령어를 추가하여 해결 [[참고](https://www.sysnet.pe.kr/2/0/13331)]
    + `RUN pip install -r requirements.txt`에서의 **에러** 발생 &rightarrow; requirements.txt 내의 django와 asgief의 버전을 다운그레이드하여 해결
    + `docker image ls` (docker 이미지 확인)
      
  - `docker run -p 8000:8000 aws_server_prac/django` (docker 실행 &rightarrow; 사전 설정 파일에 작성된 CMD를 통해 바로 django 서버 실행)
    + `docker run -d -p 8000:8000 aws_server_prac/django` (***-d == 데모 실행 (백그라운드 실행)*** &rightarrow; `docker ps`를 통해 실행 유무 확인)
    + `-p 8000:8000`은 ubuntu 서버와 docker 서버 간의 연결을 의미

* Nginx docker container 구성
  - *nginx와 django의 연결을 위한 uwsgi를 requirements.txt에 추가하여 설치*
    
  - *`/docker-server/AWS_Server_Prac/Dockerfile`의 EXPOSE & CMD 불필요하므로 주석 처리*
    
  - *`/docker-server/AWS_Server_Prac/uwsgi.ini` 추가*
    + ![image](https://github.com/NamSangwon/AWS_Server_Practice/assets/127469500/419b1216-62f7-4949-8ecb-784b8b9ac1e4)
   
  - **nginx 설정** 
    + nginx.conf & nginx-app.conf & Dockerfile 추가 [[참고](https://cholol.tistory.com/489)]
      - ***`./nginx/nginx-app.conf`의 upstream uwsgi 내의 server를 unix:/srv/docker-server/django.sock
 &rightarrow; unix:/srv/docker-server/django.sock 변경 必 (./AWS_Server_Prac/django.sock과 ./nginx/nginx-app.conf 내의 server의 .sock를 일치시켜야 함)***
    + `docker build -t docker-server/nginx .` (docker 빌드하기)
    + `docker run -p 80:80 docker-server/nginx` (nginx docker 실행하기)
 
* docker-compose 구성
  - 여러 개의 docker 이미지를 한 번에 관리하는 툴인 docker-compose 설치
    + ```
      sudo curl -L https://github.com/docker/compose/releases/download/1.25.0-rc2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
      ```
    + `sudo chmod +x /usr/local/bin/docker-compose` (실행 권한 주기)

  - 환경 관리 파일 docker-compose.yml 작성
    + ***`depends_on`이 유의해야할 점 (DB &rightarrow; django &rightarrow; nginx 순으로 실행하는 것이 오류를 발생시키지 않아 안전하기 때문)***
    +  ![image](https://github.com/NamSangwon/AWS_Server_Practice/assets/127469500/8ee9d92e-7987-4514-938e-7efe764c4d68)

  - `docker-compose up -d --build` (up == docker-compose를 실행 &rightarrow; 실행 시 빌드하여 django & nginx의 docker image 생성 및 실행)
    
  - 실행 == `docker-compose up` & 종료 == `docker-compose down`
    + django의 소스 파일만을 업데이트 시 `./AWS_Server_Prac/` 내에서 git을 pull하고 docker-compose를 재실행 해주면 됨

---

### 4. API 구성 1

* Docker에 MySQL 띄우기
  - `docker-server` 디렉토리와 같은 위치에 `mkdir mysql` (*docker-server내에 DB를 생성하면 docker 종료 시 DB가 모두 날라가기 때문*)
    
  - `docker-server` 디렉토리와 같은 위치에 `mkdir scripts` (쉘 및 배치 프로그램 저장할 용도로 생성)
    + `scripts` 디렉토리 내에 `mysql-docker.sh` 생성 (mysql 실행 스크립트 파일)
    + ![image](https://github.com/NamSangwon/AWS_Server_Practice/assets/127469500/c2486c12-ed77-4b48-87a2-81b1d49e1554)
    + `chmod +x mysql-docker.sh`로 스크립트 파일을 실행 파일로 변경 &rightarrow; `./mysql-docker.sh`로 바로 실행 가능

  - *AWS 서버 내에서 DB를 사용하기 위해 **3306 포트** 권한 설정*
 
  - Django 프로젝트의 사용 DB 변경
    + Django 프로젝트의 setting.py 내의 설정 변경 (DATABASE & INSTALLED_APPS)
      - INSTALLED_APPS에 rest_framework 추가
      - DATABASE를 sqlite에서 mysql로 설정 변경
      ```python
      # Database (in setting.py)
      DATABASES = {
          'default': {
             'ENGINE': 'django.db.backends.mysql',
             'HOST': '43.201.60.58',
             'NAME': 'AWS_Server_Prac',
             'USER': 'root',
             'PASSWORD': 'admin123!',
             'PORT': '3306',
             'OPTIONS': {'charset': 'utf8mb4'},
         }
      }
      ```
    + `pip install mysqlclient` (setting.py를 위와 같이 변경시키기 위해 필요한 모듈 설치)
    + `python manage.py makemigrations` & `python manage.py migration` (migrate 진행 &rightarrow; django에서 기본으로 제공하는 테이블 생성)
      - MYSQL과 Django 간의 버전 오류 발생 &rightarrow; ***Django의 버전을 3.2.23으로 다운그레이드로 해결*** [[참고](https://stackoverflow.com/questions/75986754/django-db-utils-notsupportederror-mysql-8-or-later-is-required-found-5-7-33)]
 
  - ***로컬에서 Django를 띄워도 setting.py 내의 DATABASE 설정에 의해 AWS EC2 내의 MySQL DB를 보게 됨***

* Django & DRF(Django Restful-Framework)를 통한 API 구현
  - **`pip install djangorestframework` (DRF 설치)**
 
  - `python manage.py startapp login` (로그인 앱 생성) [[영상](https://www.youtube.com/watch?v=NCRFC5lo8WA&list=PLHQvFs5CMVoQMcglHmtPz9ShY058H3veh&index=10) 및 [블로그](https://cholol.tistory.com/497) 구현 참고!]
    + ***setting.py의 INSTALLED_APPS에 해당 앱 추가 필요***
    + models.py 구성 (DB 테이블 구성) &rightarrow; `python manage.py makemigrations` & `python manage.py migrate`를 통해 DB 업데이트 (LoginUser 테이블 추가)
    + views.py (api call 구성)
    + urls.py (url 구성) (**AWS_Server_Prac/urls.py와 include를 통해 연결 必**)

---

### 4. API 구성 2

* Serialization (직렬화)
  - 다수의 데이터를 입력 받을 시 `user_id = request.data.get('user_id')`와 같은 코드를 통해 입력 받기 힘들기 때문에 Serialization을 사용하여 해결
 
  - 입력 받은 데이터(ex. 코드 내의 객체, 해시, 딕셔너리 등)를 JSON의 데이터를 자동으로 변환
    + `serialize.py` 파일 생성 후 *데이터 직렬화 시키는 코드* 작성
    + `serializer = LoginUserSerializer(request.data)`를 통해 serializer를 사용하여 `views.py`에 적용
    + ***[자세한 실습 사항은 github에 commit된 내용 및 강의 블로그와 영상을 참고!!]***
    + 역직렬화 : 반대로 Json 데이터를 객체, 해시, 딕셔너리 등으로 변환

---

### 5. Todo-List 앱 구성 

**Django 구성을 위주로 진행할 예정이기 때문에 *강의 블로그 및 영상* 참고!!**
***Todo-List의 프론트엔드는 [해당 github 링크](https://github.com/tkdlek11112/todo-list)을 docker에 띄워서 실습을 진행!! (3000번 포트에서 3001번 포트로 docker에 전송)***

1. Login & Regist 구현
2. Todo Task (Create & Select & Delete & Toggle) 구현
3. 페이징 처리
4. 
