# 포팅 매뉴얼 상세
Detail Porting Manual for Team. Santa production.(A.K.A MemoryCapsule) <br>
You can enjoy MemoryCapsule service with WEB envrionment. <br>
Visit and enjoy your memory! <br>
[MemoryCapsule](https://memorycapsule.site). <br>



## 빌드/배포 시나리오

- 전체 프로젝트 구조
![docker-ver](https://github.com/MemoryCapsuleSanta/.github/assets/33599065/36648cc1-7198-4f13-a7d1-42612630ed11)

![archi_internal](https://github.com/MemoryCapsuleSanta/.github/assets/33599065/902fa119-f286-4fe1-807e-567f9034693a)

- 파일 구조
```
repository
|
|---/BE
|   |-- ProjectService
|   |   |-- Dockerfile
|   |   |-- feature.sh
|   |   |-- pom.xml
|   |   |-- src
|   |   `-- target
|   |
|   |-- alarm-service
|   |   |-- Dockerfile
|   |   |-- alarm
|   |   |-- pom.xml
|   |   |-- setup.sh
|   |   |-- src
|   |   `-- target
|   |
|   |-- apigateway-service
|   |   |-- Dockerfile
|   |   |-- pom.xml
|   |   |-- setup.sh
|   |   |-- src
|   |   `-- target
|   |
|   |-- board-service
|   |   |-- Dockerfile
|   |   |-- pom.xml
|   |   |-- setup.sh
|   |   |-- src
|   |   `-- target
|   |
|   |-- config-service
|   |   |-- Dockerfile
|   |   |-- pom.xml
|   |   |-- setup.sh
|   |   |-- src
|   |   `-- target
|   |
|   |-- discovery-service
|   |   |-- Dockerfile
|   |   |-- pom.xml
|   |   |-- setup.sh
|   |   |-- src
|   |   `-- target
|   |
|   |-- user-service
|   |   |-- Dockerfile
|   |   |-- pom.xml
|   |   |-- setup.sh
|   |   |-- src
|   |   `-- target
|   |
|   |-- pom.xml
|   `-- target
|      `-- sonar
|
`---/FE
   |-- Dockerfile
   |-- nginx.conf
   |-- node_modules
   |-- package.json
   |-- public
   |-- src
   `-- task.sh

```
### 사용한 버전관리

---

#### <i>OS</i>
OS : 5.4.0-1018-aws

---

#### <i>Servers</i>
Nginx : nginx/1.25.1 built by gcc 12.2.1 20220924 (Alpine 12.2.1_git20220924-r4)

JVM : openjdk 11 2018-09-25 / Open JDK Runtime Envrionment (build 11+11-Debian-2) / OpenJDK 64-Bit Server VM (build 11+11-Debian-2, mixed mode, sharing)

Spring boot : 2.7.14

React : 18.2.0

NodeJS : 18.17.0

---

#### <i>Build</i>
Maven : Apache Maven 3.6.3

NPM : 9.6.7

---

#### <i>DB</i>


MariaDB : 11.0.2-MariaDB-1:11.0.2+maria~ubu2204

MongoDB : MongoDB 6.0.8 Community

Redis : 7.0.12


---

#### <i>extra</i>

Docker : version 20.10.21, build 20.10.21-0ubuntu1~20.04.2

Jenkins : 2.415

prometheus : 2.46.0

Grafana : 10.0.3

RabbitMQ : 3.8.11

---

#### <i>IDEA</i>

Intellij IDEA 2023.1.3 (Ultimate Edition)

---


### 빌드 시 사용되는 환경 변수

JENKINS_PORT=8088

CONFIG_PORT=8888

DISCOVERY_PORT=8761

API_GATEWAY_PORT=8000

PROMETHEUS_PORT=8760

GRAFANA_PORT=8001

CONFIG_URL=config-service:8888

DISCOVERY_URL=discovery-service:8761/eureka

RABBITMQ_URL=rabbitmq

### 배포시 특이사항

모든 서비스(Application Server, Web Server, DB 등) 이 Docker로 관리되어 있습니다. 또한 내부 docker network 로 묶여 있어 특정 서버만 외부로부터 접근이 가능합니다.

Memory-Capsule(Team. santa) 의 서버내 컨테이너의 목록은 다음과 같이 요약할 수 있습니다. 
 - service Container 목록
    - config-service
    - discovery-service
    - apigateaway-service
    - user-service
    - project-service
    - notification-service
    - board-service
 - DB Container 목록
    - MariaDB
    - Redis
    - MongoDB
 - ETC
    - RabbitMQ
 - Monitoring Container 목록
    - Prometheus
    - Grafana
 - deploy Container 목록
    - Jenkins
    - santa-service (Nginx)
    
각 서비스들은 Jenkins 가 저장소(gitLab)으로부터 소스를 받아 빌드를 합니다. 빌드를 통해서 나온 archive 파일들은 각 프로젝트(마이크로서비스)의 shell script(setup.sh) 를 통해서 docker container 로 빌드를 진행합니다.

젠킨스는 이전커밋과의 변경을 비교하여 변동사항이 있는 서비스만 pull 및 빌드를 진행하며, 빌드 결과를 <b>sonarqube</b> 에 빌드결과 및 분석결과를 전송합니다.

각 빌드 파일들을 docker 이미지로 빌드 및 container 로 만드는 과정은 shell 파일을 실행하면서 build profile 변수를 입력받습니다.

 (예시- ./setup.sh dev -> dev 프로파일로 shell 파일 실행)

shell을 실행하면서 받은 인자값(프로파일)을 통해서 각 service 들을 프로파일에 맞는 build를 진행합니다. 실행 최초의 서버는 Config-service 로부터 해당되는 프로파일을 repository로부터 실시간으로 불러와 빌드를 진행합니다.

빌드 이후의 서버에 대해서는, rabbitMQ 와 연동되어 있는 모든 서버들은 이벤트를 받게 되는 경우에 repository의 변경된 프로파일의 yml 파일을 체크하여 리빌드 없이 서버에 반영됩니다.

---

- <i><b>SonarQube</b></i>

https://sonarqube.ssafy.com
에서  <i>memory-capsule</i> 의 검색을 통해서 빌드결과를 확인할 수 있습니다.

- <i><b>Grafana</b></i>
 
http://i9a608.p.ssafy.io:8001 
에서 <> / <> 로 로그인 후에 서버의 상태(JVM, Gateway 상태)를 확인할 수 있습니다.



## 프로젝트에서 사용한 외부 서비스 정보

1. Kakao 공유하기 
2. Kakao 로그인 

## DB 덤프파일 최신본


[santa__1_.sql](/uploads/17ef5e8952ea769a40c282b686826dc5/santa__1_.sql)


## 시연 시나리오

### 배경화면

안녕하세요~ 메모리캡슐에 오신것을 환영합니다.!

메모리캡슐은 사용자들의 과정의 추억을 종합하여 고객에게 최고의 감동을 선사하는 서비스입니다.

제가 직접 메모리캡슐을 시연 및 이미지를 통해서 보여드리겠습니다.

클릭을 해서 로그인을 해볼게요




[초기화면 -> 로그인페이지]

![1_첫페이지_로그인페이지](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/6828c041-d6b0-4315-a0e1-4288a65f0cb7)

로그인은 두가지 방법을 사용해서 할 수 있습니다.

[카카오 로그인 페이지로 이동]

![4_카카오로그인페이지이동](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/4a92718d-f89f-4b7c-988e-c585097b7b5a)

[로그인시 이메일 형식이 아닐 경우]

![5_로그인_이메일형식X](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/68d604ce-5ab2-415f-b498-334e7e112b2f)

[가입된 이메일이 아닐 경우]

![6_로그인_가입X](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/b6042037-d38b-458a-b2fd-78468da01e55)

[정상적인 로그인]

![7_로그인_정상](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/33afd00d-d1b3-475d-8e54-73d41c068d78)

### 회원가입


자체 회원가입을 해보겠습니다.

[회원가입 페이지]

![2_회원가입페이지](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/04ea621f-4add-4176-938b-f2964e2cbbc0)


[이메일 인증 화면]

![3_회원가입_이메일인증](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/87ee0969-9c1e-4bda-a0fb-aed18316d39d)


이메일은 중복체크는 반드시 확인해주세요!

중복체크를 하면 인증메일이 날라오는데요, 이 인증코드를 적어줘야 사용할 수 있습니다.



### Main Page


로그인을 하면 예쁜 메인페이지를 확인할 수 있습니다.
시작하기 버튼을 통해서 메인페이지를 접속해 보겠습니다.


[로그인 후 메인 페이지와 네비바]

![8_로그인후_첫페이지_네비바](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/acdfddb3-c4a6-464b-a6dd-94ce9b5e5d99)

메인페이지를 통해서 현재 유저가 진행중인 프로젝트 목록을 확인할 수 있습니다.


### Navi Bar

네비바를 통해서 다음의 페이지로 이동할 수 있습니다.
   - 마이페이지
      
      사용자의 정보 및 친구, 프로젝트 초대 등을 확인 할 수 있습니다.
   - My Memory 페이지

      내가 진행중인 캡슐 프로젝트를 확인할 수 있는 페이지 입니다.
   - Capsule Box 페이지

      내가 과거에 진행을 완료했던 캡슐들을 확인할 수 있는 페이지 입니다.
   - 리뷰 페이지

      MemoryCapsule 서비스에 대하여 사용자들의 후기, 사용기 등을 볼 수 있는 페이지입니다.
   - 공지사항 페이지

      관리자가 게시한 공지사항을 확인할 수 있는 페이지 입니다.

네비바를 통해서 사용자는 보유포인트를 확인할 수 있습니다. 포인트는 최초회원가입시 500 포인트, 이후 일일출석마다 100 포인트가 지급됩니다.

지급된 포인트를 사용하여 사용자는 프로젝트에서 글을 작성할 때 추가사진을 등록할 수 있습니다. (무료 1장, 최대 4장 등록가능)

[페이지 이동]

![20_페이지_이동](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/9d9f5b9b-c5d4-4419-bffb-344b063ba15c)


프로젝트 상세 페이지를 통해서 프로젝트를 자세히 살펴보겠습니다.


### 프로젝트 생성

다시 메인페이지로 돌아와서, "새로운 캡슐 생성하기" 버튼을 통해서 새로운 프로젝트를 생성해보겠습니다.

프로젝트 생성은 대표이미지/제목/기간/설명/프로젝트타입 의 5가지를 설정을 통해서 생성할 수 있습니다. 

대표이미지는 해당 프로젝트를 대표하여 나타내는 이미지를 뜻하며, png/jpeg 뿐만 아니라 gif 도 지원하여 사용자가 다양하고 개성있는 프로젝트를 생성할 수 있습니다.





또한 "혼자할게요" 와 "여러명이서 할게요!" 를 통해서 개인/그룹 프로젝트를 설정할 수 있습니다. 

"그룹프로젝트" 로 진행하는 경우, 친구추가(친구 이메일)을 통해서 친구를 추가할 수 있습니다.
추가한 친구에 대해서 클릭을 통해서 이미 추가한 친구를 삭제하여 그룹생성 이전에 그룹원들을 관리할 수 있습니다.

제작기간을 선택하면 달력이 팝업되는데, 프로젝트의 시작일은 반드시 사용자 기준 오늘날짜 이후부터 설정이 가능하고 종료일은 반드시 시작일 + 1 일 이후로 설정이 가능합니다. 종료일이 시작일보다 앞서는 상황으로 프로젝트를 생성할 수 없습니다!

[개인 캡슐 만들기]

![9_캡슐목록_혼자_캡슐_만들기](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/97b8a8ea-f032-4c3e-a805-4adbaace04cf)

[단체 캡슐 만들기]

![12_다같이_캡슐_만들기](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/990c2c4d-910e-4235-a827-215d0292675f)


### Project Detail

[캡슐 상세 페이지]

![10_캡슐상세페이지](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/bde547e0-9d85-426f-82fa-d067dfae3b61)

프로젝트 목록을 클릭해서 사용자는 프로젝트의 세부 내용을 확인할 수 있습니다. 또한 프로젝트 세부내용 내에서 캡슐의 진행도를 확인할 수 있습니다.

사용자는 세부 페이지에서 내가 작성한 게시글들을 확인할 수 있으며, 해당 프로젝트에서 쌓인 전체 게시글 수와 내가 작성한 게시글 수를 확인할 수 있습니다. 

프로젝트당 게시글은 하루에 <b><i>단 한번</i></b> 만 작성이 가능하므로, 작성에 유의해주세요!


### Project 내에 글작성

프로젝트 내에서 글작성을 누르면, 위와같은 화면을 확인할 수 있습니다.
사진추가 를 통해서 기본 사진 1장 업로드 가능하고 그 이상은 일정 포인트를 지불하여 추가 사진의 업로드가 가능합니다.(최대 4장)

현재 SSAFY에서 많은것을 학습하고 있기에, 이미지는 싸피로 사진을 올려보겠습니다.

이후 오늘의 스탬프 (도장) 을 통해서 그날의 기분을 표현할 수 있습니다. 스탬프는 글 작성을 위해 반드시 필요하니 잊지말고 선택해주세요!

최종적으로 필자는 위와같은 글을 작성해보았습니다. 글은 최대 100글자까지 작성이 가능하기때문에 반드시 유의해주세요! 이제 이 글을 우측 상단의 "게시글 등록"을 통해서 등록을 진행해보겠습니다.

게시글이 정상적으로 등록이 되면, 해당 프로젝트 자세히 보기 페이지로 이동이 됩니다.
해당 페이지내에서 History 영역을 통해서 내가 작성한 글들만 확인을 할 수 있습니다.

[글 작성 및 사진 1장 업로드]

![11_캡슐_글쓰기_사진1장](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/38ddd3e7-64c6-43a3-b07e-6d4a5149853d)

[글 작성 및 사진 4장 업로드]

![11_2_글쓰기_사진_4장](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/990f12d9-458e-479a-8051-636742d9506c)


### 마이페이지

마이페이지를 통해서 유저는 이메일/닉네임, 친구등의 정보를 확인 할 수 있습니다.
달력에서 내가 접속한 날짜들을 확인할 수 있고, 내가 진행/완료한 캡슐프로젝트들을 브리핑 받을 수 있습니다.

또한 초대받은 프로젝트들을 확인하고 이를 수락/거절할수 있으며, 제작중인 프로젝트들도 확인이 가능합니다.

보관함을 통해서는 완료한 프로젝트들의 종합서비스를 열어볼수 있습니다.

[마이페이지 전체적인 모습]

![13_마이페이지](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/70782322-0ba3-4a57-b4f9-1c25d809649e)

[마이페이지 캡슐 초대를 거절 및 수락]

![13_2_마이페이지_캡슐초대_거절_수락](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/80e527fe-42e7-4808-9d6d-95f9bdd95efb)

[마이페이지에서 공지사항 글 보기]

![13_3_마이페이지_공지사항글보기](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/33003858-3c78-43e4-a439-e9f5f084d7c4)


### 친구기능

마이페이지의 친구목록 버튼을 통해서 친구들을 관리할 수 있습니다. 이떄 친구목록에는 "등록된 친구" 로, 총 몇명의 친구가 등록되어있는지 확인할 수 있습니다. 또한 검색을 통해서 이미 맺어진 친구들을 검색할 수 있습니다.

이제 새로운 친구를 찾아, 친구 추가를 하겠습니다. 우층 상단의 "친구찾기" 버튼을 눌러 친구추가를 합니다.

친구 검색창에 친구의 가입이메일을 검색하여 친구를 검색할 수 있고, 해당 친구에게 친구신청을 보낼 수 있습니다. 지금 상황은 "KJH@ssafy.com" 으로 검색한 유저를 검색하여, 친구신청까지 진행하겠습니다.

친구신청을 성공적으로 수행하면, Success 와 함께 친구신청이 완료된 피드백을 받아볼 수 있습니다. 다시 친구목록으로 돌아가 친구목록을 확인해봅니다.

친구목록에서 기존의 친구 1명과, 새로 친구신청을 한 유저를 확인할 수 있습니다. 친구프로필 우측의 아이콘 상태를 통해서 요청수락/ 요청 등의 상태를 확인할 수 있습니다.

이미 친구로 맺어져 있던, 요청을 수락한 친구의 오른쪽 아이콘을 누르면 위와같은 친구가 작성한 총 게시물 수 와 참여했던/참여중인 프로젝트의 총 수를 확인할 수 있습니다.

[친구 페이지에서 친구 요청보내기 및 친구 요청 취소]

![14_친구페이지_친구요청_요청취소](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/0ce527a6-a27b-4a2b-a9c2-8b86df3eaf72)

[친구 페이지에서 친구 요청 받기 및 거절]

![14_2_친구페이지_친구요청_수락_거절_친구삭제](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/a0e26ce2-31f5-4913-a926-511a06e82314)

### 프로필 수정

다음은 프로필 변경을 진행하겠습니다. 프로필 변경은 Navi 바 -> MyPage 로 이동하겠습니다.
이동후, 우측 상단에 "프로필변경"을 통해 프로필을 변경할 수 있습니다.

프로필변경을 클릭하여 들어오면, 기존 사용자가 사용하고 있던 프로필 사진과 닉네임이 업로드 됩니다. 이번 변경은 프로필을 "수원킹_after변경" 으로 프로필을 수정해보겠습니다.



프로필을 수정한 후, 하단의 "프로필 변경" 을 통해서 프로필을 변경할 수 있습니다. 또한 비밀번호 변경도 진행할 수 있습니다.



프로필 수정을 누르면 자동으로 MyPage로 돌아오게 됩니다. 마이페이지에서 내가 수정한 프로필들이 정상적으로 변경되었음을 확인할 수 있습니다.

[프로필 변경]

![15_프로필변경](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/3523a64a-4d25-49b8-9dda-af1e6bb7dcc9)

[비밀번호 변경]

![16_비밀번호변경](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/6d2fc662-97d6-49fd-80fa-cd760f74e024)


### 캡슐박스 페이지

캡슐박스를 통해서 사용자는 지난 프로젝트(이미 완료된 프로젝트) 들을 확인하고 링크를 통해서 공유할 수 있습니다. 이를 확인하기 위해 "Navi Bar" > "Memory Capsule" 로 이동하겠습니다.

해당 페이지를 통해서 과거의 프로젝트 목록들을 확인할 수 있고, 클릭을 통해서 해당 프로젝트들을 상세히 다시보기를 진행할 수 있습니다.

[완료된 캡슐]

![19_완료된캡슐](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/b5df5d6c-67fb-4dd8-af57-0c6b317bb0d2)

해당 url 을 가지고 있는 모든 유저들이 선물페이지를 열람할 수 있습니다. 이미지, 텍스트, gif 등을 시간 순으로 아름답게 나열하여 사용자들이 쌓은 추억을 프로젝트에 참여했거나 공유 url을 받은 모든 유저들이 공감하고 추억할 수 있습니다.



### 공지사항 및 리뷰

[공지사항 페이지]

![18_공지사항페이지](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/792445fe-d77f-4bc6-8e1e-4b8cf2cd2c74)

[리뷰 페이지 및 리뷰 글 좋아요 및 취소]

![17_리뷰페이지_좋아요_하기_취소](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/1e6d390f-a1a1-4ade-9cd0-254dc6c68f56)

[리뷰 페이지에서 글 작성 및 수정, 삭제]

![17_2_리뷰페이지_글쓰기_수정_삭제](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/b13520d9-3602-4539-b14d-de5621a505ee)

공지사항은 관리자에 의하여 긴급한 패치 소식이나 정보를 확인할 수 있는 게시글 페이지입니다. 또한 리뷰페이지를 통해서 사용자들을 이 서비스를 사용한 후기나 꿀팁등을 사용자들과 공유하며 글을 남길 수 있습니다.

[로그아웃]

![21_로그아웃](https://github.com/MemoryCapsuleSanta/.github/assets/75843997/c788a016-f8b4-4742-84dd-2f6283aad38f)
