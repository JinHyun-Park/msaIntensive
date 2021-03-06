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
    - https://jdm.kr/blog/208
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
    7. 카프카 토픽 삭제
     ./kafka-topics.sh --zookeeper localhost:2181 --delete --topic DummyTopic
    8. 카프카 토픽 리스트
     ./kafka-topics.sh --list --zookeeper localhost:2181
    9. 카프카가 비정상일 때 
     sudo lsof -i :2181 한뒤  
     kill -9 pid 하고 다시 띄워준다
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
        > `eksctl create cluster --name teamtwohotel --version 1.17 --nodegroup-name standard-workers --node-type t3.medium --nodes 4 --nodes-min 1 --nodes-max 4`
8. Local EKS 클러스터 토큰가져오기 ( CI/CD 할때 필요한건데, 앞에 설정해줘야 할 게 더 있으니 아래 쪽 CI/CD 다시 참고 )
    > `aws eks --region ap-northeast-2 update-kubeconfig --name teamtwohotel`
9. 아마존 컨테이너 레지스트리
    - 아마존 > ecr (elastic container registry) > ecr 레파지터리     : ECR은 각 배포될 이미지 대상과 이름을 맞춰준다
        > `aws ecr create-repository --repository-name teamtwohotel --region ap-northeast-2`  
        > `aws ecr put-image-scanning-configuration --repository-name teamtwohotel --image-scanning-configuration scanOnPush=true --region ap-northeast-2`
10. AWS 컨테이너 레지스트리 로그인
    - aws ecr get-login-password --region (Region-Code) | docker login --username AWS --password-stdin (Account-Id).dkr.ecr.(Region-Code).amazonaws.com
  
  
11. AWS 레지스트리에 도커 이미지 푸시하기  (이건 위에서 한 거랑 좀 겹치는듯)
    - aws ecr create-repository --repository-name (IMAGE_NAME) --region ap-northeast-2
    - docker push (Account-Id).dkr.ecr.ap-northeast-2.amazonaws.com/(IMAGE_NAME):latest
  
  
## 도커 빌드 및 푸시
먼저 mvn package 로 jar 파일을 만들어줘야 함!!
1. 도커 빌드 // 위의 리포지토리 주소 앞에 docker build -t 를 붙이고 뒤에 :v1 . 을 붙여서 각 프로젝트(로컬) 디렉토리에서 실행 
    - docker build -t (ID).dkr.ecr.ap-northeast-2.amazonaws.com/adminxx-game-gateway:v1 .
2. 도커 푸시
    - docker push 690521455231.dkr.ecr.ap-northeast-2.amazonaws.com/admin-customer:v1
    - 오류가 발생한다면 aws ecr get-login-password 가 잘됐는 지 확인

## EKS에 카프카 설치
    > helm 사전에 설치해야함
    > `kubectl --namespace kube-system create sa tiller      # helm 의 설치관리자를 위한 시스템 사용자 생성`
    > `kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller`
    > `helm repo add incubator https://charts.helm.sh/incubator`
    > `helm repo update`
    > `kubectl create ns kafka`
    > `helm install my-kafka --namespace kafka incubator/kafka`
  
  - 카프카 설치 확인
    > kubectl get all -n kafka
  - 카프카 토픽 생성
    > kubectl -n kafka exec my-kafka-0 -- /usr/bin/kafka-topics --zookeeper my-kafka-zookeeper:2181 --topic game --create --partitions 1 --replication-factor 1
  - 토픽 확인
    > kubectl -n kafka exec my-kafka-0 -- /usr/bin/kafka-topics --zookeeper my-kafka-zookeeper:2181 --list
  - 이벤트 발행하기 (producer, 딱히 필요없음)
    > kubectl -n kafka exec -ti my-kafka-0 -- /usr/bin/kafka-console-producer --broker-list my-kafka:9092 --topic teamtwohotel
  - 이벤트 수신하기
    > kubectl -n kafka exec -ti my-kafka-0 -- /usr/bin/kafka-console-consumer --bootstrap-server my-kafka:9092 --topic teamtwohotel --from-beginning

## 컨테이너 만들기
  - Containerizing
    > namespace 만들기 
      - `kubectl create namespace teamtwohotel`  
    > kubectl create deploy order --image=496278789073.dkr.ecr.ap-northeast-1.amazonaws.com/skcc07-order:v1 -n tutorial  (각 이미지별로 다 해줘야함)  
    > kubectl expose deploy order --type=ClusterIP --port=8080 -n tutorial   (상동, port는 모두 8080으로 띄워줘야함!!)  
    > kubectl expose deploy gateway --type=LoadBalancer --port=8080 -n teamtwohotel    ( gateway는 이렇게 해줘야댐 )  
    > kubectl get all -n teamtwohotel  

## CI/CD
- 환병변수 준비
  > AWS_ACCOUNT_ID
  > KUBE URL : EKS -> 클러스터 -> 구성 "세부정보"의 "API 엔드포인트 URL"
  > CodeBuild 와 EKS 연결
```
1. eks-admin-service-account.yaml 파일 생성하여 sa 생성
apiVersion: v1
kind: ServiceAccount
metadata:
  name: eks-admin
  namespace: kube-system
  
2. kubectl apply -f eks-admin-service-account.yaml
혹은, 바로 적용도 가능함
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: eks-admin
  namespace: kube-system
EOF

3. eks-admin-cluster-role-binding.yaml 파일 생성하여 롤바인딩
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: eks-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: eks-admin
  namespace: kube-system
  
4. kubectl apply -f eks-admin-cluster-role-binding.yaml
혹은, 바로 적용도 가능함
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: eks-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: eks-admin
  namespace: kube-system
EOF


만들어진 eks-admin SA 의 토큰 가져오기
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep eks-admin | awk '{print $1}')
```

  > KUBE TOKEN 가져오기  
    : 
    
  > Code build와 ECR 연결 정책 설정
    : code build -> 빌드 프로젝트 생성
    ![codebuild1](https://user-images.githubusercontent.com/17754849/108522319-0e35a600-7310-11eb-8d63-f32cf0651e0a.png)  
    ![codebuild2](https://user-images.githubusercontent.com/17754849/108524004-ed6e5000-7311-11eb-831d-e6fca77ab59e.png)  
    ![codebuild3](https://user-images.githubusercontent.com/17754849/108524571-843b0c80-7312-11eb-968a-9d14b182afb8.png)  
    그 뒤는 다음의 url로 설명 대체 https://jootc.com/p/201905122828  
    그리고 다시 뒷 내용은 "3. CICD-Pipeline_AWS_v2" pdf 자료 39페이지부터 (이미지가 많은 관계로, buildspec.yml은 복사하기)
  >  > 환경 변수 (아직 정상 동작 안해서 맞는 지는 모름)
      ![env](https://user-images.githubusercontent.com/17754849/108544309-a1c7a080-7329-11eb-9e2f-702697073c45.png)
  > \\_\\_아마 위 내용만 하고 진행하면 AccessDeniedException 발생할텐데 role 추가해줘야함\\_\\_
  ```
  { "Action": [
        "ecr:BatchCheckLayerAvailability",
        "ecr:CompleteLayerUpload",
        "ecr:GetAuthorizationToken",
        "ecr:InitiateLayerUpload",
        "ecr:PutImage",
        "ecr:UploadLayerPart"
      ],
      "Resource": "*",
      "Effect": "Allow"
}

  ```
      
  > Codebuild cache 적용 : CICD PDF p.45, S3 만들고 설정해야 함  
  > ~~buildspec.yml에 `aws eks --region $AWS_DEFAULT_REGION update-kubeconfig --name $_EKS` 이거 넣어줘야 하는데 권한 에러 날 경우~~
  >  > ~~https://stackoverflow.com/questions/56011492/accessdeniedexception-creating-eks-cluster-user-is-not-authorized-to-perform~~
  > 상세 내용은 buildspec.yml과 코브빌드의 환경변수 확인하면 됨


# AutoScale


# 서비스 메시 - istio, 키알리, 예거, siege
> istio
> `curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.7.1 TARGET_ARCH=x86_64 sh -`  
> `cd istio-1.7.1`  
> `export PATH=$PWD/bin:$PATH`  
> `istioctl install --set profile=demo`  
> 설치 확인 `kubectl get all -n istio-system`  
> istio injection  
>  > ~~`kubectl label ns teamtwohotel istio-injection=enabled`~~ (굳이 안해도 최신버전은 잡힘)  
>  > kubectl label ns default istio-injection- (잘못 설정 시 삭제 방법)  
> istio 모니터링 툴 설치  
>  > kubectl apply -f samples/addons  
>  > 모니터링 툴 설정  
>  >  > Monitoring Server - Kiali  
>  >  >  > 기본 ServiceType 변경 : ClusterIP를 LoadBalancer 로  
>  >  >  >  > `kubectl edit svc kiali -n istio-system`  
>  >  >  >  > `:%s/ClusterIP/LoadBalancer/g`  
>  >  >  >  > `:wq!`  
>  >  >  > 모니터링 시스템(kiali) 접속 : EXTERNAL-IP:20001 (admin/admin)  
>  >  > Tracing Server - Jaeger  
>  >  >  > 기본 ServiceType 변경 : ClusterIP를 LoadBalancer 로  
>  >  >  >  > `kubectl edit svc tracing -n istio-system`  
>  >  >  >  > `:%s/ClusterIP/LoadBalancer/g`  
>  >  >  >  > `:wq!`  
>  >  >  > 분산추적 시스템(tracing) 접속 : EXTERNAL-IP:80  

# httpie, siege 툴 설치 (kube에 설치하는거)
1. httpie 설치
```
cat <<EOF | kubectl apply -f -
apiVersion: "v1"
kind: "Pod"
metadata: 
  name: httpie
  namespace: tutorial
  labels: 
    name: httpie
spec: 
  containers: 
    - name: httpie
      image: clue/httpie
      command:
        - sleep
        - "360000"
EOF
```
  > kubectl exec -it httpie bin/bash   

2. seige 설치 (로컬)
  > Mac : `brew install siege`
  > Window : `https://trend21c.tistory.com/1438`


# Configmap  (4.Container_Orchestration(Docker & Kubernetes)_AWS_v2 2.pdf p.139)
- configmap 생성  
  > kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2
- configmap 정보 가져오기  
  > kubectl get configmaps my-config -o yaml  

- 파일로부터 configmap 생성 (configmap.yml 생성)
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: customer1
data:
  TEXT1: Customer1_Company
  TEXT2: Welcomes You
  COMPANY: Customer1 Company Technology Pct. Ltd.
```
  > kubectl create -f configmap.yml
- ![configmap](https://user-images.githubusercontent.com/17754849/108792601-fd618a80-75c4-11eb-9386-3d8785979309.png)

# Secret
- 시크릿 생성
```
kubectl create secret generic my-password --from-literal=password=mysqlpassword --namespace teamtwohotel
```
  > ![secret](https://user-images.githubusercontent.com/17754849/108868200-4f43f800-7639-11eb-8915-1999a695b85b.png)
- 시크릿 확인
```
kubectl get secret my-password -o yaml
```
  > ![확인](https://user-images.githubusercontent.com/17754849/108868606-b8c40680-7639-11eb-8296-dfc2ad9cb4e0.png)
- 시크릿 buildspec.yml
  > ![소스](https://user-images.githubusercontent.com/17754849/108870840-dd20e280-763b-11eb-8e55-bfc9dc70e9e0.png)
- 시크릿 자바 출력
  > ![결과출력](https://user-images.githubusercontent.com/17754849/108871144-30933080-763c-11eb-8e76-453348bb7ec0.png)

## hystrix 세팅
https://jeong-pro.tistory.com/183 참고

## pod에 siege 설치
```
kubectl run siege --image=apexacme/siege-nginx -n recruit

접속
kubectl exec -it siege -- /bin/bash

```

## pod 여러 개 띄우게 replica 설정
- kubectl scale deploy qualification --replicas=1

## 떠 있는 deploy의 이미지 변경
- kubectl set image deploy qualification qualification=496278789073.dkr.ecr.ap-northeast-1.amazonaws.com/recruit-qualification:v2

## Auto scaling HPA 설정
- kubectl autoscale deployment payment --cpu-percent=50 --min=1 --max=10 cpu-percent=50 -n teamtwohotel

## Spring 세팅 ( 소스 내려받아서 하는 경우 안해도 됨 )
[spring.io](start.spring.io)  
Dependencies : JPA, H2(java embeded DB), data rest(Rest Repositories)


# 용어 기타 등
- hystrix : 원격 시스템이나 호출하는 구간을 격리해 관리, 모니터링 해주는 라이브러리

# 참고 사이트
- https://workflowy.com/s/msa/27a0ioMCzlpV04Ib#/62a296734f02 (각종 설치 및 실행 명령어)

# kubectl 명령어
- kubectl delete deploy gateway -n teamtwohotel
- kubectl delete service gateway -n teamtwohotel
- kubectl exec -it -n teamtwohotel gateway-77c6dd9f89-wkzm5 -- /bin/sh
- kubectl config set-context --current --namespace=teamtwohotel   (네임 스페이스 저장)
