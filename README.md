## 현재까지 학습했던 것들
자격증: 정보처리 기사
학원: kgit bank 자바 백엔드 국비지원 교육
인강:
- Inflearn
- 파이썬 알고리즘 문제풀이 입문 (김태원)
- 스프링 데이터 JPA (김영한)
- 스프링 부트와 JPA활용1 (김영한)
- 스프링 부트와 JPA활용2 (김영한)
- 스프링 핵심원리 - 기본편(김영한)
- 코딩으로 학습하는 리팩토링 (백기선)
- SW 개발자를 위한 성능 좋은 SQL 쿼리 작성법 (김정선)
- Udemy
- Docker & Kubernetes (Maximillian Schwarzmuller (현재 수강중)

책
- 객체지향의 사실과 오해
- 하루 3분 네트워크 교실
- got design patterns
- Kotlin in action
- 자바 ORM 표준 JPA 프로그래밍
- 실전 카프카 개발부터 운영까지
- 클린코드 (현재 읽는 중)

공식문서
- [Kotlin Programming Language](https://kotlinlang.org/)
- [The Go Programming Language](https://go.dev/)
- [dgraph-io/badger: Fast key-value DB in Go.](https://github.com/dgraph-io/badger)
- (이 외에는 찾아보았지만 지금 당장 생각나지 않아서... 더 못적었습니다;;)


## HTBeyond (2021.09 ~ 2022.09)

### 비지니스 설명
- 프리미엄 아파트 입주민들이 사용하는 앱을 만들었습니다.
- 아파트 내에서 제공하는 서비스 들과 IoT 장비들에서 나오는 데이터를 수집하고 가공하여 입주민들에게 필요한 정보를 보여주도록 합니다.
- 입주민은 앱을 통하여 아파트 내에서 제공하는 여러 서비스를 예약, 결제, 상태 등을 확인할 수 있습니다.
- 또한, 아파트 내에 커뮤니티에 참여할 수 있고, 불편사항 등 민원을 앱으로 처리할 수 있는 등의 편의 기능을 제공합니다.

### CICD
- Code 관리 - Git, Github
- AWS Code pipe line 이용
  - github에서 개발, 메인 branch에 merge 시 aws code pipeline이 감지, 바로 빌드 후 배포
  - 필요시 배포 전 AWS 서비스 내에서 승인 분기
- Blue/Green으로 무중단 배포

### 인증/인가
- AWS에서 제공하는 Cognito User 사용
- API Gateway에서 유저 정보 확인 후 토큰 발급
- 해당 토큰을 가지고 서버는 유저의 정보 확인
- 따로 Spring Security 를 사용 하여 개발 x (사업 초기 외주 받은 애플리케이션의 경우 Spring Security을 이용하여 개발 됨)

### 사용 기술
- Kotlin, Spring Boot, Spring Data JPA, QueryDsl, Postgresql, Coroutine, FeignClient, Active MQ, Postgresql, Git

### 개발 
1. Paging처리 
   1. Querydsl을 사용하여 fetch join 하여 조회할 때, 1쪽에서 N을 조회할 때 조회된 데이터를 모두 어플리케이션에 올려서
      조회되는 문제 발견
      -> application.yml에 jpa 설정 중 default_batch_fetch_size 의 사이즈를 설정하여 문제 해결
   2. paging과 fetch join 사용 시 limit 조건이 걸리지 않은 상태로 쿼리가 발생, pagination 제네릭 함수를 따로 만들어
      limit 조건도 포함 될 수 있게 변경

2. Optional 메서드 처리
   1. 외부 업체와 협업 시 문제 발생
   2. 외부 업체에서 사용하는 프레임워크가 option 메서드를 먼저 요청
   3. 예전에 만들어 놓은 프로그램에 interceptror 부분에 회원을 확인하는 부분이 있어서 option 요청이 들어왔을 때,
      interceptor에서 에러 발생
   4. 기존 시스템들은 aws gateway option 요청이 막혀있어서 오류가 발생하지 않음. 하지만, 예전 애플리케이션은 해당 부분이 막혀있지 않음
   5. aws에서 다른 cloud 서비스로 옮길 계획이 없었고, 해당 오래된 애플리케이션은 계약이 종료되면 없어질 수 있던 서버이기 때문에,
      코드에 별다른 추가 없이 aws gateway 설정으로 해결

3. Feign Client 성능
   1. 해당 회사에서 마이크로 서비스 아키텍처를 지향하여, 다른 서버에 정보가 필요할 때 기존에 있는 API를 활용하기 위해 Feign Client로 요청
   2. 문제는 Feign Client 를 사용한 클래스의 함수를 호출할 때, 함수가 Blocking 되어 성능이 떨어짐 (한개 일때는 어쩔 수 없으나 2개 이상의 다른 서버의 요청이 필요할 때 서능이 저하)
   3. 해당 부분을 Coroutine을 사용하여 여러개의 Feign Client 함수가 호출 될 때, 동시에 호출될 수 있도록 변경

4. 리팩토링
   1. 레거시 프로젝트의 Layer가 너무 많아 문제가 발생
   2. 기존 Contoller, Service, Respository 형태가 아닌 Contoller, User Service, Handler, Service, Repository 식으로 레이어가 많아짐
   3. 여기서 문제는 레이어가 너무 많아 코드가 파편화, 이 때문에 단방향으로 의존하는 것이 아닌 양방향으로 의존되는 경우가 자주 발생
   4. 또한, 불필요한 레이어가 깊어짐으로 인하여 프로젝트 파악이 힘들고, 이미 있는 기능을 새로 정의하는 일도 발생 및 가독성이 떨어짐
   5. 불필요한 레이어를 줄이고 책임을 분리, 파편화 되어 있는 함수들을 응집하여 재배치 가독성과 유지보수성을 높힘
   6. 또한 책임이 분리되어, 테스트하기 어려웠던 부분의 테스트 코드 작성도 용이해짐

### 협업
- 사용툴: Jira, Confluence, Slack, Teams, Github

1. PR Review
   1. 버스팩터가 낮아지는 것을 방지, 가독성, 코드 로직 확인 등을 위한 코드리뷰 활성화
2. Git flow 전략
   1. [배달의 민족 Git Flow 전략](https://techblog.woowahan.com/2553/)
   2. [뱅크 샐러드 Git Flow 전략](https://blog.banksalad.com/tech/become-an-organization-that-deploys-1000-times-a-day/)
   3. 위의 두 회사의 전략을 참고하여 GitFlow 전략 수립
   4. 두 가지를 참고하였지만, 뱅크 샐러드쪽의 전략과 흡사하였습니다.
3. Agile
   1. Daily Scrum
      1. 매일 스크럼을 통하여 현재 작업 상황 공유 및 이슈 공유
      2. Jira를 통하여 작업을 작은 단위로 나눠 이슈 공유
   2. Sprint 
      1. 2주간 진행할 Stroy를 선정 계획을 세우고, 하위 이슈 생성
      2. Sprint 회고를 통해 2주간 진행 상황 공유
4. Code Convention
   1. 각자의 코드의 특성이 너무 다르고 네이밍이 제각각인 경우가 많아, 처음 프로젝트를 접할 시 시간 소요가 너무 많음을 인지
   2. 모든 프로젝트의 코딩 컨벤션을 수립하고 차근 차근 적용할 수 있는 프로젝트부터 적용
5. Github Pull Request, Commit Message Convention
   1. Jira의 이슈의 번호를 사용하여 Git commit message 컨벤션을 맞춤 (이슈의 번호로 찾기 용이)
   2. Github PR시 Jira의 이슈 번호를 앞에 두어 링크가 걸릴 수 있도록 컨벤션 맞춤
6. Confluence
   1. 개발 시 나온 문제 상황 혹은 기획이 변경 혹은 비지니스 부분의 이해를 돕기 위하여 Confluence를 활용해 위키화
   2. 인수 인계 자료, 그룹 스터디에 관한 내용도 기록하여 후임자 혹은 신규 입사자가 확인할 수 있도록 위키화

## Emotiontek (2023.01 ~ 현재)

### 비지니스 설명
- 모션 제어기 프로그램을 현재는 Window에 종속된 프로그램으로만 고객들에게 제공하였습니다.
- 이 프로그램을 웹으로 만들어 Cross Platform이 가능하도록 하기위하여 시작되었습니다.
- 또한, 윈도우 프로그램의 UI적 한계를 깨기 위하여 시작되었습니다.

### 사용기술
- Golang, Echo, Redis, Badger Database, Git

### 개발
1. Badger Database 추상화
    1. 비지니스 특성상 File DB를 쓸 수밖에 없었습니다.
    2. 그로인하여 언제 업데이트가 멈출지 모르는 라이브러리 사용하게 되고, 그로 인해 언제든 다른 데이터 베이스로 갈아 끼울 수 있도록
       추상화 했습니다.
    3. 사용성에 가장 초점을 두고 만들었고, Transaction 처리, DB 동시접근 제어 등을 신경쓰면서 작업하였습니다.
2. 객체지향스럽게 개발
   1. 기존에 경력이 별로 없는 동기가 작업한 코드를 보고 유지 보수성이 고려되지 않고, 가독성이 많이 떨어지고 객체 책임이 분리되어 있지 않은
      코드가 많았습니다. 클린 코드의 중요성을 전파하고 직접 리팩토링을 시켜보며 중요성을 느끼게 하여 구조부터 코드 컨벤션까지 다시
      수립하게 하였습니다.

### 협업
1. Git
   1. 기존에 Git Flow 전략을 사용하지 않아서 전략을 수립하고 적용하였습니다.
2. Code Convention
   1. 기존에 작성된 코드를 보고 유지보수성이 너무 떨어진다는 것을 느꼈습니다.
   2. 따라서, 코딩 컨벤션을 적용하였습니다.

