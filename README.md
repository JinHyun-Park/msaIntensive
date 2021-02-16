# msaIntensive MSA 구성을 위한 내용 정리

## 이론
1. SAGA 패턴 정리
[SAGA 정리 블로그]  
    [https://jjeongil.tistory.com/1100]  
    [https://cla9.tistory.com/22]

## 환경 구성 세팅 (Mac)
1. 도커 설치
2. 카프카 설치
    - https://dev-jj.tistory.com/entry/MAC-Kafka-%EB%A7%A5%EC%97%90-Kafka-%EC%84%A4%EC%B9%98-%ED%95%98%EA%B8%B0-Docker-homebrew-Apache
    1. 경로 이동 /Users/jinhyeonbak/intensive/kafka_2.12-2.3.0/bin
    2. 주키퍼 실행  
     ./zookeeper-server-start.sh ../config/zookeeper.properties &
    3. 카프카 broker 실행  
     ./zookeeper-server-start.sh ../config/server.properties &
    4. 카프카 topic 만들기  
     ./kafka-topic.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic booking
    5. 카프카 producer 실행  
     ./kafka-console-poducer.sh --broker-list localhost:9092 --topic booking
    6. 카프카 consumer 실행  
     ./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic booking --from-beginning
3. httpie 설치
4. aws cli 설치
    - https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/install-cliv2-mac.html
5. eksctl 설치
    - https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/getting-started-eksctl.html
6. IAM 생성
    - https://www.44bits.io/ko/post/publishing_and_managing_aws_user_access_key
    
## Spring 세팅
[spring.io](start.spring.io)  
Dependencies : JPA, H2(java embeded DB), data rest(Rest Repositories)
