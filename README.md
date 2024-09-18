# 실전! Github Actions 와 Nginx 로 따라해보는 무중단 배포
<div align="center">

![책 이미지.png](docs%2F%EC%B1%85%20%EC%9D%B4%EB%AF%B8%EC%A7%80.png)

</div>

# 1️⃣ 저자
|         [박정균](https://github.com/junggyun)          |       [박주현](https://github.com/Programmer-may)     |            [이현준](https://github.com/little6523)            |         [조성윤](https://github.com/syeej)         |         [최혜미](https://github.com/ghrltjdtprbs)          |
|:---------------------------------------------------:|:----------------------------------------------------------:|:---------------------------------------------------------:|:---------------------------------------------------------:|:-------------------------------------------------------:|
| ![](https://avatars.githubusercontent.com/junggyun) | ![](https://avatars.githubusercontent.com/Programmer-may) | ![](https://avatars.githubusercontent.com/little6523) | ![](https://avatars.githubusercontent.com/syeej) | ![](https://avatars.githubusercontent.com/ghrltjdtprbs) |
<br>

# 2️⃣ 무중단 배포란?
무중단 배포는 애플리케이션을 배포할 때 서비스의 가용성을 유지하면서 사용자에게 영향을 미치지 않고 새로운 버전을 배포하는 방법입니다. 이는 애플리케이션의 업타임을 최대화하고, 배포 과정에서 발생할 수 있는 다운타임을 최소화하여 서비스의 지속성과 사용자 경험 안정을 향상시키는 것을 목표로 합니다.
### **A. 사용자 경험**

사용자는 서비스 중단 없이 항상 애플리케이션에 접근할 수 있기를 기대합니다. 특히, 글로벌 서비스나 금융 서비스와 같이 365일 24시간 내내 운영이 필수적인 시스템에서는 서비스 중단이 심각한 문제를 야기할 수 있습니다.

### **B. 비즈니스 연속성**

비즈니스 연속성을 유지하기 위해서는 언제 어디서나 서비스가 제공되어야 합니다. 배포 과정에서 발생하는 서비스 중단은 비즈니스 연속성에 큰 위협이 됩니다.

### **C. 신뢰성 및 안정성**

무중단 배포는 시스템의 신뢰성과 안정성을 높입니다. 언제든지 안전하게 새로운 기능을 배포하고, 문제가 발생할 경우 신속하게 롤백할 수 있는 환경을 제공합니다.


### **D. 경쟁력 유지**

빠르게 변화하는 시장에서 경쟁력을 유지하기 위해서는 지속적으로 새로운 기능과 개선사항을 배포해야 합니다. 무중단 배포는 이를 가능하게 하여, 경쟁사보다 앞서 나갈 수 있도록 도와줍니다.

![다운타임.png](docs%2F%EB%8B%A4%EC%9A%B4%ED%83%80%EC%9E%84.png)

**다운타임(downtime)** 이란 서비스가 내려간 시간을 말합니다. 구 버전 프로세스를 내리고 새로운 버전의 프로세스를 실행하는 동안 사용자에게 지속적인 서비스를 제공하지 못하게 됩니다.

# 3️⃣ 프로젝트 실습 진행 소개
![실습 흐름.png](docs%2F%EC%8B%A4%EC%8A%B5%20%ED%9D%90%EB%A6%84.png)

스프링부트 애플리케이션을 로컬 IDE를 통해 작성하고, GitHub 레포지토리에 소스코드를 푸시합니다. 각 배포 전략에 맞는 브랜치에 푸시가 이루어지면, GitHub Actions가 해당 브랜치에 설정된 배포 스크립트를 자동으로 실행합니다. 이 과정에서 Gradle을 사용해 애플리케이션을 빌드하고, 생성된 `.jar` 파일을 AWS EC2 인스턴스로 전송한 후 무중단 배포 전략이 도입 전이라면 배포 스크립트를 통해 명령어로 애플리케이션을 실행시키고, 무중단 배포 전략 도입 후라면  `deploy.sh` 스크립트를 통해 무중단 배포가 이루어집니다.

Nginx는 로드 밸런서를 역할을 하여, 8081과 8082 포트에서 각각 구동 중인 스프링 부트 애플리케이션의 트래픽을 분산시킵니. `deploy.sh` 스크립트 실행되는 과정에서 설정 파일에 적힌 트래픽을 라우팅할 포트가 바뀌며 무중단 배포를 구현합니다.

Termius를 사용해, EC2 서버에 접속하여 JAVA 와 Nginx 등 인프라를 구축하고 deploy.sh 파일을 작성, 배포 상태를 확인하고 관리합니다.

배포 후에는 Apache JMeter 의 Simple HTTP Request 템플릿을 활용하여 애플리케이션에 HTTP Request 를 잦은 간격 주기적으로 보내 테스트 API 응답값과 실행 포트를 확인하며 다운타임이 없는지 테스트합니다. Postman을 사용하여 헬스 체크 API 요청을 보내 구 버전의 애플리케이션이 안전히 내려갔는지 확인하며 무중단 배포 후의 애플리케이션 상태를 점검합니다.

# 4️⃣ 브랜치 전략
![브랜치 전략.png](docs%2F%EB%B8%8C%EB%9E%9C%EC%B9%98%20%EC%A0%84%EB%9E%B5.png)
1. 프로젝트를 생성하고, main 브랜치에 기본 셋팅을 합니다.
2. main 브랜치를 본따 bluegreen, rolling, canary 브랜치를 생성합니다. Github Actions 배포 파일에 bluegreen, rolling, canary 브랜치 push 트리거를 설정합니다.
3. 각 배포 전략의 before, after 브랜치 내용을 push하여 자동 배포가 실행되게 합니다.
   전략이름_before 브랜치는 해당 배포 전략 도입 전 셋팅을, 배포전략_after 브랜치는 해당 배포 전략을 도입하여 무중단 배포를 수행하는 브랜치입니다.
4. before 브랜치는 빌드 결과물인 JAR 파일을 서버의 `cicd` 디렉토리로 옮깁니다. 이전 프로세스를 종료 시킨 후 구 버전의 JAR 파일을 `old_build` 디렉토리로 옮기고, 신 버전의 JAR 파일을 실행 시킵니다. 이전 프로세스를 종료시키고, 신 버전의 프로세스가 서비스되기까지 다운타임이 발생하게 됩니다.
5. after 브랜치는 `deploy.sh` 파일을 실행하게 됩니다. 쉘 스크립트를 통해 구 버전의 JAR 파일은 `old_build` 디렉토리로 옮기고, 신 버전의 애플리케이션을 실행시킵니다. 헬스 체크를 통과하면 Nginx 설정파일을 동적으로 바꿔 로드 밸런싱할 포트를 변경합니다. 헬스 체크가 통과하지 않는다면 롤백 합니다. 마지막으로 구 버전의 애플리케이션을 안정적으로 종료합니다.

# 5️⃣ 무중단 배포 전략 실습
실습에 사용한 deploy.sh 파일은 프로젝트의 script 디렉토리 안에 각 전략별로 작성해두었습니다. 해당 파일을 통해 자세한 내용을 확인할 수 있습니다.
## 🟦🟩 블루그린 전략
![블루그린 전략.png](docs%2F%EB%B8%94%EB%A3%A8%EA%B7%B8%EB%A6%B0%20%EC%A0%84%EB%9E%B5.png)

블루 그린 전략(Blue-Green Deployment)은 두 개의 동일한 환경(블루 환경과 그린 환경)을 사용하여 배포하는 방법입니다. 블루 환경에서 현재 운영 중인 버전을 실행하고, 그린 환경에서 새로운 버전을 배포합니다. 새로운 버전이 준비되면 트래픽을 블루 환경에서 모두 한번에 그린 환경으로 전환하여 배포를 완료합니다.

배포가 완료된 후 기존 구 버전이 있던 환경은 다음 배포를 위해 재사용하거나 제거할 수 있습니다.

### bluegreen_before 브랜치

Blue Green 무중단 배포 전략 도입 전 환경을 구축합니다.
`cd-bluegreen.yml` 파일을 통해 JAR 파일을 업로드하고,  이전 프로세스를 죽이고 JAR 파일을 8081포트에서 실행합니다. 이 과정에서 다운 타임이 발생하게 됩니다.

### bluegreen_after 브랜치

Blue Green 전략 도입하여 무중단 배포를 실행합니다.
`cd-bluegreen.yml` 파일을 통해 deploy.sh 파일을 실행합니다.

### deploy.sh 파일

1. **현재 실행 중인 Spring Boot 애플리케이션 포트 식별**
    - 현재 실행 중인 Spring Boot 애플리케이션의 포트를 찾기 위해 `ss` 명령어를 사용하여 Java 프로세스를 필터링합니다.
2. **새 포트 선택**
    - 현재 실행 중인 포트에 따라 새 애플리케이션을 배포할 포트를 선택합니다. 현재 포트가 8081이면 8082에 배포하고, 반대의 경우 8081에 배포합니다.
3. **기존 JAR 파일 백업**
    - 기존에 실행 중이던 `app-<포트>.jar` 파일을 타임스탬프와 함께 `old_build` 폴더로 이동하여 백업합니다.
4. **새 JAR 파일 배포**
    - 새로 빌드된 JAR 파일을 `app-<새 포트>.jar`로 이름을 변경한 후 그린 포트에서 실행합니다.
5. **헬스 체크**
    - 배포된 새 애플리케이션이 정상적으로 동작하는지 `/actuator/health` 엔드포인트를 통해 확인합니다. 10번의 시도 후에도 응답이 없으면 배포가 실패한 것으로 간주하고 롤백합니다.
6. **헬스 체크 실패 시 롤백**
    - 헬스 체크 실패 시 새로 실행된 프로세스를 종료하고, 새로 배포된 JAR 파일을 `old_build` 폴더로 이동합니다. 이전에 백업된 JAR 파일을 다시 복원하여 원래의 애플리케이션을 복원합니다.
7. **Nginx 설정 업데이트**
    - Nginx 설정 파일에서 `proxy_pass`를 새 포트로 변경하여 새로운 애플리케이션으로 트래픽을 라우팅합니다.
8. **Nginx reload**
    - 변경된 설정이 적용되도록 Nginx를 재로드합니다. 실패 시 배포를 중단합니다.
9. **이전 애플리케이션 종료**
    - 기존 포트에서 실행 중이던 애플리케이션의 프로세스를 안전하게 종료합니다. 종료가 실패하면 강제로 종료합니다.
10. **블루 그린 배포 완료**
    - 배포가 성공적으로 완료되었음을 출력하고, Nginx가 새로운 포트로 트래픽을 라우팅하게 됩니다.
## 🔄️🔄️ 롤링 전략
![롤링전략.png](docs%2F%EB%A1%A4%EB%A7%81%EC%A0%84%EB%9E%B5.png)

롤링 전략(Rolling Deployment)은 애플리케이션의 새 버전을 점진적으로 배포하는 방법입니다. 기존 인스턴스를 하나씩 새로운 버전으로 교체하며, 배포 과정에서 일부 인스턴스는 이전 버전, 나머지 인스턴스는 새 버전이 실행됩니다.

로드 밸런서에 연결되어 있는 서버를 하나 추가 하거나 연결을 끊어서 새로운 버전으로 교체하고 로드밸런서에 다시 연결합니다. 이 과정을 점진적으로 진행하여 최종적으로는 모든 트래픽이 새 버전에 도달할 수 있게 합니다.
이러한 과정을 통해 서비스 중단 없이 애플리케이션을 업데이트할 수 있습니다.

### rolling_before 브랜치

Rolling 무중단 배포 전략 도입 전 환경을 구축합니다.
`cd-rolling.yml` 파일을 통해 JAR 파일을 업로드하고,  이전 프로세스를 죽이고 JAR 파일을 8081포트, 8082포트에서 실행합니다. 이 과정에서 다운 타임이 발생하게 됩니다.

### rolling_after 브랜치

Rolling  전략 도입하여 무중단 배포를 실행합니다.
`cd-rolling.yml` 파일을 통해 deploy.sh 파일을 실행합니다.

### deploy.sh 파일

1. **포트 설정 및 헬스 체크 함수 정의**
    - `PORT1=8081`과 `PORT2=8082`로 두 개의 포트를 설정합니다. `health_check` 함수는 새로 배포된 애플리케이션이 정상적으로 작동하는지 `/actuator/health` 엔드포인트를 사용하여 확인합니다.
2. **타임스탬프 생성**
    - `timestamp=$(date +"%Y%m%d%H%M%S")`를 통해 현재 시간을 타임스탬프 형식으로 저장합니다. 이 값은 JAR 파일을 백업할 때 사용됩니다.
3. **롤백 함수 정의**
    - `rollback()` 함수는 배포가 실패했을 때, 이전에 백업된 JAR 파일을 복원하고 애플리케이션을 다시 실행합니다.
    - 새 버전을 종료하고, 이전 버전을 백업 폴더에서 원래 위치로 복원한 후 다시 실행합니다.
4. **배포 함수 정의**:
    - `deploy()` 함수는 새 버전을 포트에 배포합니다. 먼저 기존 버전의 JAR 파일을 백업하고, 새로 배포된 버전으로 교체한 뒤 애플리케이션을 실행합니다.
    - 배포 후 `health_check` 함수를 통해 헬스 체크를 수행하며, 실패 시 롤백합니다.
5. **포트별 배포 절차**
    - `PORT1=8081`, `PORT2=8082`를 대상으로 순차적으로 배포를 진행합니다. 각각의 포트에 대해 실행 중인 애플리케이션을 종료하고, 새 버전을 배포한 후 헬스 체크를 수행합니다.
6. **Nginx 설정 업데이트**
    - Nginx 설정을 업데이트하여 새로운 포트로 트래픽을 라우팅합니다. `sed` 명령어를 사용하여 `proxy_pass` 설정을 변경하고, Nginx를 재로드합니다.
7. **롤링 배포 완료**
    - 두 포트 모두 성공적으로 배포되면, Nginx 설정 파일을 다시 로드합니다.

## 🐦⛏️ 카나리 전략
![카나리 전략.png](docs%2F%EC%B9%B4%EB%82%98%EB%A6%AC%20%EC%A0%84%EB%9E%B5.png)

카나리 전략(Canary Deployment)은 새로운 버전을 소수의 사용자에게 먼저 배포하여 문제를 식별하는 방법입니다. 초기에는 일부 인스턴스에만 새 버전을 배포하고, 점진적으로 트래픽을 늘려가며 전체 배포를 완료합니다.

카나리 배포의 핵심은 소수의 트래픽으로 새로운 버전에 대한 오류를 조기에 감지하는 것입니다.

### canary_before 브랜치

Canary 무중단 배포 전략 도입 전 환경을 구축합니다.
`cd-canary.yml` 파일을 통해 JAR 파일을 업로드하고,  이전 프로세스를 죽이고 JAR 파일을 8081포트, 8082포트에서 실행합니다. 이 과정에서 다운 타임이 발생하게 됩니다.

### canary_after 브랜치

Canary 전략 도입하여 무중단 배포를 실행합니다.
`cd-canary.yml` 파일을 통해 deploy.sh 파일을 실행합니다.

### deploy.sh 파일

1. **현재 실행 중인 Spring Boot 애플리케이션의 포트 확인**
    - 현재 실행 중인 애플리케이션이 8081 포트에서 동작 중인지 8082 포트에서 동작 중인지 확인하여 `CURRENT_PORT` 변수에 저장합니다.
    - 새 애플리케이션을 배포할 포트(`NEW_PORT`)를 결정합니다. 현재 실행 중인 포트가 8081이라면, 새 애플리케이션은 8082에 배포되고 그 반대라면 8081포트에 배포됩니다.
2. **기존 JAR 파일 백업**
    - 새 애플리케이션이 배포될 포트의 기존 JAR 파일을 `old_build` 디렉토리로 이동시킵니다. 백업 파일명에는 타임스탬프가 추가되어 구분됩니다.
3. **새 애플리케이션 배포**
    - 빌드된 JAR 파일을 `app-$NEW_PORT.jar`로 이름을 변경하고 새 포트에서 실행시킵니다.
4. **헬스 체크**
    - 새 애플리케이션이 제대로 실행되는지 확인하기 위해 /actuator/health 엔드포인트를 사용하여 최대 10회동안 헬스 체크를 반복합니다.
    - 헬스 체크에 실패하면 새 애플리케이션 프로세스를 종료하고, Nginx를 통해 구버전으로 롤백합니다. 이때 새로운 JAR 파일도 백업되고, 이전 JAR 파일로 복원됩니다.
5. **Nginx 설정 업데이트 - 트래픽 30%로 라우팅**
    - Nginx 설정을 업데이트하여 구버전에 70%, 새버전에 30% 트래픽을 라우팅합니다. 설정이 성공적으로 적용되면 Nginx를 재로드합니다.
6. **점진적 트래픽 증가 - 50%, 70%로 업데이트**
    - 일정 시간이 경과할 때마다 새 애플리케이션으로 가는 트래픽 비율을 50%, 70%로 점진적으로 증가시킵니다. 각 단계마다 헬스 체크를 수행하여 새 애플리케이션이 안정적으로 동작하는지 확인합니다.
    - 만약 헬스 체크가 실패하면 Nginx 설정을 원래대로 되돌리고, 새 애플리케이션 프로세스를 종료하여 롤백합니다.
7. **최종 트래픽 100%로 전환**
    - 모든 헬스 체크가 성공하면 Nginx 설정을 업데이트하여 트래픽을 100% 새버전으로 전환합니다.
8. **구버전 프로세스 종료 및 JAR 파일 백업**
    - 구버전 애플리케이션이 동작 중인 포트를 확인한 후, 안전하게 종료합니다.
    - 구버전 JAR 파일을 `old_build` 폴더로 이동하여 타임스탬프와 함께 백업합니다.
9. **카나리 배포 완료**
    - 배포가 성공적으로 완료되었으며, 이제 새로운 버전이 100%의 트래픽을 처리합니다.
