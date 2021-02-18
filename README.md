msaIntensive MSA 구성을 위한 내용 정리
=============================

## 이론
1. SAGA 패턴 정리
[SAGA 정리 블로그]  
    [https://jjeongil.tistory.com/1100]  
    [https://cla9.tistory.com/22]

## 환경 구성 세팅 (Mac) --> 아래 참고 사이트에서 더 자세한 기술도 있으니 참고
1. 도커 설치
    - https://whitepaek.tistory.com/38
    - 위에 가면 도커 관련 명령어들도 있음
2. 카프카 설치
    - https://dev-jj.tistory.com/entry/MAC-Kafka-%EB%A7%A5%EC%97%90-Kafka-%EC%84%A4%EC%B9%98-%ED%95%98%EA%B8%B0-Docker-homebrew-Apache
    1. 경로 이동 /Users/jinhyeonbak/intensive/kafka_2.12-2.3.0/bin
    2. 주키퍼 실행  
     ./zookeeper-server-start.sh ../config/zookeeper.properties &
    3. 카프카 broker 실행  
     ./kafka-server-start.sh ../config/server.properties
    ( 4 ~ 5 는 건너뛰어도 됨 )
    4. 카프카 topic 만들기  
     ./kafka-topic.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic teamtwohotel
    5. 카프카 producer 실행  
     ./kafka-console-poducer.sh --broker-list localhost:9092 --topic teamtwohotel
    6. 카프카 consumer 실행  
     ./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic teamtwohotel --from-beginning
3. httpie 설치
4. aws cli 설치
    - https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/install-cliv2-mac.html
    - aws configure 로 액세스 ID 등 입력
5. eksctl 설치
    - https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/getting-started-eksctl.html
6. IAM 생성
    - https://www.44bits.io/ko/post/publishing_and_managing_aws_user_access_key
7. eksctl 생성 ( 시간이 좀 걸림 )
    - 클러스터 생성
        > `eksctl create cluster --name admin-eks --version 1.17 --nodegroup-name standard-workers --node-type t3.medium --nodes 4 --nodes-min 1 --nodes-max 4`
8. Local EKS 클러스터 토큰가져오기
    > `aws eks --region ap-northeast-2 update-kubeconfig --name admin-eks`
9. 아마존 컨테이너 레지스트리
    - 아마존 > ecr (elastic container registry) > ecr 레파지터리     : ECR은 각 배포될 이미지 대상과 이름을 맞춰준다
        > `aws ecr create-repository --repository-name admin-eks --region ap-northeast-2`  
        > `aws ecr put-image-scanning-configuration --repository-name admin-eks --image-scanning-configuration scanOnPush=true --region ap-northeast-2`
10. AWS 컨테이너 레지스트리 로그인
    - aws ecr get-login-password --region (Region-Code) | docker login --username AWS --password-stdin (Account-Id).dkr.ecr.(Region-Code).amazonaws.com
  
  
11. AWS 레지스트리에 도커 이미지 푸시하기  (이건 위에서 한 거랑 좀 겹치는듯)
    - aws ecr create-repository --repository-name (IMAGE_NAME) --region ap-northeast-2
    - docker push (Account-Id).dkr.ecr.ap-northeast-2.amazonaws.com/(IMAGE_NAME):latest
  
  
## 도커 빌드 및 푸시
1. 도커 빌드 // 위의 리포지토리 주소 앞에 docker build -t 를 붙이고 뒤에 :v1 . 을 붙여서 각 프로젝트(로컬) 디렉토리에서 실행 
    - docker build -t (ID).dkr.ecr.ap-northeast-2.amazonaws.com/adminxx-game-gateway:v1 .
2. 도커 푸시
    - docker push 690521455231.dkr.ecr.ap-northeast-2.amazonaws.com/admin-customer:v1

## Spring 세팅
[spring.io](start.spring.io)  
Dependencies : JPA, H2(java embeded DB), data rest(Rest Repositories)


# 용어 기타 등
- hystrix : 원격 시스템이나 호출하는 구간을 격리해 관리, 모니터링 해주는 라이브러리

# 참고 사이트
- https://workflowy.com/s/msa/27a0ioMCzlpV04Ib#/62a296734f02 (각종 설치 및 실행 명령어)
