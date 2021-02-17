msaIntensive MSA 구성을 위한 내용 정리
=============================

## 이론
1. SAGA 패턴 정리
[SAGA 정리 블로그]  
    [https://jjeongil.tistory.com/1100]  
    [https://cla9.tistory.com/22]

## 환경 구성 세팅 (Mac)
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
    - 클러스터 생성 명령어
        > `eksctl create cluster --name admin-eks --version 1.17 --nodegroup-name standard-workers --node-type t3.medium --nodes 4 --nodes-min 1 --nodes-max 4`
8. Local EKS 클러스터 접속정보 설정
    > `aws eks --region ap-northeast-2 update-kubeconfig --name admin-sk-Cluster`
## Spring 세팅
[spring.io](start.spring.io)  
Dependencies : JPA, H2(java embeded DB), data rest(Rest Repositories)
