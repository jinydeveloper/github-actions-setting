시나리오 4개
- 간단한 시나리오에서 시작해서 요구사항을 추가
- 각 시나리오의 깃헙액션 워크플로우
-- 단순한 구성 ~ 복잡한 구성

CICD
- 구성
1. CI(테스트)
2. CD(배포)

CI(Continuous Integration)
- 공유 저장소 내의 코드 변경사항에 대해 자동으로 테스트 실행
=> 문제를 조기에 발견하고, 더 안정적으로 개발


CD(Continuous Delivery or Deployment)

CD(Continuous Delivery)
- 코드 변경사항을 배포 환경에 배포 준비 상태로 만드는 것
- 실제 배포는 수동

CD(Continuous Deployment)
- 코드 변경사항이 배포 환경에 자동으로 배포

예시 
- 소셜 미이디어 애플리케이션 개발
-- 개발자 A: 메시지 기능
-- 개발자 B: 이미지 업로드 기능

1. CI 프로세스
- 각 개발자의 작업 독립적으로 검증됨
- 코드 변경사항 자동으로 테스트
2. CD 프로세스
- CI 프로세스에서 이상없다면, 배포
사용자에게 안정적인 서비스 지속적으로 제공

CICD를 통해 얻을 수 있는 것
- 검증된 소프트웨어 (CI)를 배포 환경에 안정적, 효과적으로 전달(CD)

CICD 실행 시점
구성
CI : PR Open, Synchronize
PR Merge
- 코드 변경사항이 처음 메인 코드베이스로 병합되도록 제안되는 순간
- 기능 추가나 버그 수정과 같은 변경사항을 만들고 이를 메인 코드베이스에 통합하고자 할 때

이 시점에 CI 프로세스를 시작하는 이유?
- 변경사항이 기존 코드와 문제없이 잘 통합되는 지
- 새로운 버그가 발생하지 않는 지

Synchronize
- 이미 열려 있는 PR에 추가 커밋이 생길 때 발생
추가적인 변경사항이 기존 PR에 추가되면
- 이 변경사항도 기존 코드와 잘 통합되는 지
- 새로운 문제 발생시키지 않는 지 검증

이 시점에 CI 프로세스를 시작하는 이유? 
- 최초의 PR(PR Open된 시점)이 안정적이라도 추가된 변경사항에서 문제 발생하는 지 확인

=> CI 프로세스가 없다면 버그나 오류가 그대로 반영될 위험

CD : PR Merge
- 코드 변경사항이 메인 코드베이스로 통합되는 순간

이 시점에 CI 프로세스를 시작하는 이유? 
- 안정적이고, 검증된 코드를 실제 배포 환경에 신속하게 반영
- 새로운 기능이나 수정사항 제공 

CD 프로세스가 없다면 검증된 코드가 사용자에게 도달하는 데 시간이 지체됨

=> github Actions을 활용해서 CICD프로세스가 언제 실행되도록 할 지
- PR Request

CICD
- 구성
1. CI(테스트)
- 유닛 테스트, 통합 테스트, 리그레션 테스트
- 코드 컨벤션, 보안 취약점 검사
- etc

2. CD(배포)
- CI프로세스에서 검증된 안정적인 코드를 제공하는 단계



---------------------------------- 
쿠버네티스를 통한 CICD 구축에 필요한 개념
Docker
- 컨테이너 기술을 사용해서 애플리케이션과 그 환경을 이미지라는 형태로 패키징하는 플랫폼
- 어떤 환경에서든 동일한 실행 보장

Kubernetes
- 컨테이너화된 애플리케이션 배포, 확장, 관리 플랫폼

Helm 
- 쿠버네티스 패키지 매니저
- 쿠버네티스 애플리케이션 설정, 배포 관리 간편화

CD 프로세스
1. 이미지 생성(docker)
2. 배포(helm + kubernetes)

AWS 
- 아마존이 제공하는 클라우드 서비스
- 손쉽게 인프라 구축, 확장 및 실행 가능

AWS ECR
- Docker 이미지를 쉽게 저장, 관리
- 관리형 컨테이너 레지스트리 서비스

AWS EKS
- 관리형 쿠버네티스 서비스
- 쿠버네티스의 고가용성, 신뢰성, 확장성 등을 클라우드 환경에서 제공

ECR & EKS
- ECR을 사용해서, CD프로세스에서 생성된 도커 이미지를 관리 및 사용
- 이미지를 바탕으로 EKS 배포

정리
1. 이미지 생성(docker + AWS ECR)
2. 배포(helm + AWS EKS)

---------------------------
시나리오 1
- 배포하는 application : React 기반 my-app

가정 : 개발 환경에만 배포하는 CICD 구성

branch
- dev : 개발 환경

요구사항
1. 특정 path에 대해서만 실행(my-app)
2. dev branch로 PR이 생성&동기화 될 때 테스트 작업 실행(CI)
3. PR이 dev branch로 머지되면, 이미지 빌드하고 개발 환경에 배포(CD)
4. 배포 성공 여부 슬랙으로 전송


워크플로우 구성

1. 트리거 구성
- github event : pull Request > PR이 생성 & 동기화(CI), 머지되면(CD)
- path filter : my-app > my-app이라는 디렉토리가 변경될 때
- branch filter : dev > 개발 환경으로 사용되는 dev branch에만


2. 잡 구성
- test(CI)
-- cache action 사용
-- npm build

- imgage-build(CD)
-- aws action
-- aws ecr action
-- docker

- deploy(CD)
-- aws action
-- kubectl action
-- helm action
-- slack action

------------------------------
AWS 적용하기

1. 각 시나리오마다 aws 환경 구성은 동일
2. 시나리오에 따라 달라지는 점
- ecr 레포지토리 생성

[작업과정]
- Github OIDC 설정
- AWS IAM Role 생성
-- github actions
-- cloud 9
- AWS ECR 생성