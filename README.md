# 현직 대기업 개발자 푸와 함께하는 진짜 백엔드 시스템 실무!By. 푸 강의 실습
# 젠킨스 배포자동화

# 서버 세팅 명령어

- GCP에서 서버인스턴스 생성수 아래 명령어로 설치한다
- GCP기준으로 속도가 너무 느리거나 타임아웃이나면 젠킨스 인스턴스를 micro → medium으로 변경할것

```bash
sudo yum install -y wget;
sudo yum install -y maven;
sudo yum install -y git;
sudo yum install -y docker;https://pkg.jenkins.io/redhat-stable/jenkins.repo;
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key;
sudo yum install -y jenkins;
sudo wget -O /etc/yum.repos.d/jenkins.repo 
sudo systemctl start jenkins;
sudo systemctl status jenkins;
```

# GCP 방화벽 정책 추가

젠킨스는 8080 포트로 뜨기 떄문에 서버의 8080 포트를 열어줘야한다.

- VM인스턴스 페이지로 이동
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eaa70742-58c3-4f11-837c-8948be6732e9/Untitled.png)
    
- 방화벽 규칙 만들기 선택

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/270a8858-dc82-42ca-85e6-5e58dcfab72e/Untitled.png)

- 방화벽 규칙 생성
    - 대상 : 네트워크의 모든 인스턴스
    - 소스 IPv4 범위 : 0.0.0.0/0
    - 프로토콜 및 포트 → 지정된 프로토몰 및 포트 → tcp : 8080 지정
    - 생성 버튼 클릭
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d1ec884a-1a1f-4ed6-9bbd-4604a058b603/Untitled.png)
    

# 젠킨스 설정

- 젠킨스 어드민 계정 비밀전호 설정
    
    젠킨스 인스턴스의 아래경로에 비밀번호를 확인후 입력
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0fb90a11-4d5b-44fa-b3da-1a9c69918c18/Untitled.png)
    
    ```bash
    sudo cat /var/lib/jenkins/secrets/initialAdminPassword 
    ```
    
- 추천 플러그인 설치
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7d3879fb-3011-4973-b0d8-b3d038ab1b0d/Untitled.png)
    
- SSH 플러그인 추가
    - 젠킨스관리 → 플러그인 관리
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d8a9ee8d-7b51-4f31-8843-44171137cea5/Untitled.png)
        
    - 설치가능 → 검색창 publish over ssh 입력 → 체크후 install without restart 입력
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/289e17e7-5e60-47c6-8cd0-bbd312632860/Untitled.png)
        

# SSH 생성

- 공개키-개인키 암호화 방식
    - 서로를 복호화 할 수 있는 공개키 개인키 쌍을 활용한 암호화 방식
    - 공개키로 암호화한 데이터는 개인키로 복호화 가능
    - 개인키로 암호화한 데이터는 공개키로 복호화 가능
    - 서버에서 개인키로 암호화한 데이터를 클라이언트에서 공개키로 복호화함으로서 요청한 서버가 인증된 서버라는것을 알수 있다
    - https에서 사용하는 암호화 방식
- 젠킨스 서버에서 공개키, 개인키쌍 생성
    - .ssh 폴더 하위에 id_rsa 파일로생성
    - id_rsa : 개인키
    - id_rsa.pub : 공개키
    
    ```bash
    # 키 생성
    ssh-keygen -t rsa -f ~/.ssh/id_rsa
    
    # 생성된 폴더로 이동
    cd ~/.ssh
    
    # 공개키 열기
    vi id_rsa.pub
    
    # 공개키 복사
    ```
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b63fdad4-569a-4781-9fd3-0cd92340dd49/Untitled.png)
    

# 클라이언트 서버 도커 설정

- 도커설치
    
    ```bash
    sudo yum install -y docker;
    ```
    
- 도커 실행권한 변경. 권한을 변경하지 않으면 젠킨스로 빌드시 sudo를 입력하지 않고 배포스크립트에 docker 입력시 권한 에러가 발생한다
    
    ```bash
    sudo chmod 666 /var/run/docker.sock;
    ```
    

# 클라이언트 서버에 젠킨스 서버 공개키 등록

- gcp에서 authorized_keys 파일을 삭제하는 이슈가 있는경우 아래 항목참조
- ssh 접속

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b9bd9a74-059d-453f-afd0-7ff47fd42fa8/Untitled.png)

- ssh 키 관리 파일을 연 뒤 젠킨스 서버의 공개키를 등록
    
    GCP에서 해당 파일을 삭제하므로 정상적으로 등록되지 않을수 있음
    
    ```bash
    vi ~/.ssh/authorized_keys
    ```
    
- 권한 설정
    
    GCP가 authorized_keys 파일을 지웠을수 있으므로 에러가 나더라도 무시한다.
    
    ```bash
    chmod 700 ~/.ssh;
    chmod 600 ~/.ssh/authorized_keys;
    ```
    

# GCP에서 SSH 키 추가

젠킨스 서버에서 각 워커 인스턴스로 SSH접근시 젠킨스에 사전에 설정한 개인키로 암호화된 값을 메타데이터에 저장된 SSH키(젠킨스 서버의 공개키)로 복호화 함으로써 SSH접근하는 주체가 사전에 허가된 서버임을 인증한다.

- 대시보드 → 설정 → 메타데이터 → 메타데이터추가 → SSH키 선택
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dfe2d110-48c7-4537-a42b-5f4af2c69ada/Untitled.png)
    
- 젠킨스서버의 공개키 추가

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fb79be3c-d60b-4bd7-8ff8-3c7f7faace84/Untitled.png)

# 젠킨스 SSH 플러그인 설정

- 젠킨스관리 → 시스템설정
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0a15e824-836e-48b5-af1d-998397cebd24/Untitled.png)
    
- 시스템 설정 최하단에 아래 설정값 입력
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c7c7bc6b-e9a4-4278-bc66-4bb0dd4a9bcb/Untitled.png)
    
    - key : 젠킨스 서버의 ssh 개인키 (——— 로 표시된 부분까지 전부 복사해야함)
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/30178e69-3cb9-41b1-81eb-29a70d7ef729/Untitled.png)
    
    - 클라이언트 서버 정보 입력
        - Name : 임의값 입력 (인스턴스이름등)
        - Hostname : 서버 내부 IP (젠킨스와 같은 네트워크 이기때문)
        - username : 클라이언트 계정명
            
            ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1f28f465-ce94-418b-9e45-4d122a5c3346/Untitled.png)
            
        - Remote Directory : 임의의 폴더 지정
    - 모두 입력후 Test Configuration 클릭해 설정확인 “Success” 가 나오면 성공

# 젠킨스 배포 스크립트 작성

- 대시보드 → 새로운 Item 선택
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ee3d301f-7168-42d3-88cb-6f56ab31d237/Untitled.png)
    
- 배포 아이템 이름 입력, Freestyle project 선택후 ok버튼 클릭
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bfab20a8-d693-4cd6-aa12-ba655b4fe341/Untitled.png)
    
- General → 하단 Build → 빌드 후 조치 → Send build artifacts over SSH 선택
    - **Verbose output in console : 로그를 자세히 찍어줌. 체크**
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6b775dc1-83ad-4d99-9d0e-0bcbf64f4279/Untitled.png)
        
    - 빌드 스크립트 입력
        - 80포트를 사용하려면 웰노운 포트이기 떄문에 sudo를 추가해야함
        - 로그를 출력하지 않으려면 /dev>null 2&1 로 변경
        
        ```bash
        nohub docker run -p 8080:80 devswsong/spring-boot-cpu-bound nohub.out 2&1 &
        ```
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ae3a1b27-75b3-4a15-a850-8940ac8071bd/Untitled.png)
        

# 젠킨스 빌드 실행

- 대시보드 → 아이템 → Build Now 선택
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8bace9ad-44c1-477a-b331-9ed145fde8cf/Untitled.png)
    
- 도커 실행권한 에러 발생시
    
    ```
    /usr/bin/docker-current: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post http://%2Fvar%2Frun%2Fdocker.sock/v1.26/containers/create: dial unix /var/run/docker.sock: connect: permission denied.
    ```
    
- 스프링 실행 로그가 뜨면서 배포가 멈추기 않는경우 → 백그라운드로 실행하도록 변경
    - 2>&1 는 표준에러를 표준출력으로 redirection 하라는 의미
    - rm test > /dev/null은 표준 출력을 /dev/null로 redirection 하라는 의미 (출력하지 않기위함)
    
    ```bash
    nohup docker run -p 8080:80 devswsong/spring-boot-cpu-bound nohup.out 2>&1 &
    ```


# 젠킨스 빌드시 추가한 인스턴스에도 배포되도록 수정

- 대시보드 → 아이템 선택 → 설정 → 빌드후 조치 → Add Server 선택
- 추가후 Build Now 클릭시 3개 서버에 동시에 배포됨
    - 2,3번 인스턴스에 도커가 실행 되지 않은경우 에러발생하므로 도커 실행할것
    - 젠킨스에서는 빌드성공으로 출력되더라도 서버로그 확인시 에러가 발생했을수 있음

# 엔진엑스 인스턴스 생성

- 서버 인스턴스 생성
    - 서버 스팩 미디움으로 설치
    - 서울리전은 4개밖에 인스턴스 추가가 안되므로 다른 리전에 생성할것
- 서버 인스턴스에 엔진엑스 설치

    ```bash
    sudo yum install -y nginx
    
    # 엔진엑스 실행
    sudo systemctl start nginx
    ```


# 엔진엑스 로드벨런싱 설정

- 엔진엑스 공식문서

  [HTTP Load Balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)

- 설정파일 열기

    ```bash
    sudo vi /etc/nginx/nginx.conf
    ```

- 로드벨런싱 설정추가
    - upstream : 로드벨런싱할 서버 IP포트 설정정보 입력
    - location : 로드벨런싱할 URL정보 입력

    ```bash
    ```

  include /etc/nginx/conf.d/*.conf;

  upstream cpu-bound-app {
  server {instance_1번의_ip}:8080 weight=100 max_fails=3 fail_timeout=3s;
  server {instance_2번의_ip}:8080 weight=100 max_fails=3 fail_timeout=3s;
  server {instance_3번의_ip}:8080 weight=100 max_fails=3 fail_timeout=3s;
  }

  server {

    ```
    include /etc/nginx/default.d/*.conf;
    
    location / {
      proxy_pass http://cpu-bound-app;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection 'upgrade';
      proxy_set_header Host $host;
      proxy_cache_bypass $http_upgrade;
    }
    
    error_page 404 /404.html;
    location = /404.html {
    }
    
    ```
    ```
    
- 엔진엔스 connect() 함수사용할수 있도록 설정변경

  엔진엑스는 기본적으로 connect() 함수를 사용불가능하므로 사용할수 있게 변경처리해야함

  변경하지 않으면 엔진엑스 접근시 로드벨런싱이 되지 않고 아레 에러가 로그에 출력됨

    ```bash
    connect() to 10.178.0.8:8080 failed (13: Permission denied) while connecting to upstrea
    ```

  아래 명령어 실행해 connect() 함수 사용할수 있도록 변경

    ```bash
    sudo setsebool -P httpd_can_network_connect on
    ```

- 엔진엑스 서버 접근
    - [http://서버주소](http://서버주소)
- 로드벨런싱이 적용되었는지 확인
    - [http://서버주소](http://서버주소)/hash/123

    ```bash
    # 엔진엑스 서버상태 확인
    sudo systemctl status nginx.service
    ```

  에러로그 확인

    ```bash
    sudo tail -f /var/log/nginx/error.log;
    ```
# 로드벨런서 적용후 스트레스 테스트

- 시나리오
  1. 5분간 TPU 8~16사이로 인스턴스 1~3개를 이용해 테스트
  2. 5분간 TPU를 높여보면서 서버가 언제 요청을 받지 못하는지(500, 502에러 리턴) 확인
- 설정

    ```yaml
    config:
      target: "http://34.64.184.213/" # 테스트 호스트정보
      phases:
        - duration: 360 # 테스트시간
          arrivalRate: 8 # 목표 TPU
          name: Warm up
    
    scenarios:
      - name: "just hash"
        flow:
          - get:
              url: "/hash/123" # 테스트 URI
    ```

# Git 사용법
## 원하는 시점으로 커밋 롤백하기

1. 원하는 시점 선택

   ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/54034652-58e7-4e90-a279-1750836f5c32/Untitled.png)

2. 이 커밋하까지 현재 브랜치를 초기화 버튼 클릭 - 하드 선택
  - 하드 선택시 깃커밋 내용과 소스작업내용이 모두 삭제되고 이전 커밋상태가됨

   ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4faa6183-7744-4e3a-aacd-f6478c1a42f8/Untitled.png)

# git에서 jar파일 빌드

1. 대시보드 → 아이템 → 설정 → 소스코드관리 → git선택

   ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/974a558c-2ea9-4bfd-a7f5-661be5222f5d/Untitled.png)

   ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/438ae655-d6eb-483b-a3ab-1fa7c37cb2dd/Untitled.png)

2. Build → add build step 선택 → excute shell 선택

   기본적으로 mvnw 파일이 윈도우에서 생성되어 실행권한이 없으므로 실행권을 추가해줘야함

    ```yaml
    chmod 544 ./mvnw
    ./mvnw clean package
    ```


![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3ea766a4-c2a9-4c03-a65f-2a1227ff1d72/Untitled.png)

# 깃허브 웹훅 설정

- 레파지토리 → settings → webhooks → add webhooks

  ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2c197b75-86b3-4edb-953d-aa55f412d563/Untitled.png)

- 설정입력
  - payload url : 젠킨스 호스트 정보 입력
    - ex) [http://34.64.184.213:8080/github-webhook/](http://34.64.184.213:8080/github-webhook/)

  ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9131a001-bc6c-49c5-8c10-bdbe75e97a29/Untitled.png)