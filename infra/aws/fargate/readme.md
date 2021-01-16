AWS Fargate
======

### Fargate? 

Fargate 란 AWS Container 기반의 서비스에서 제공하는 완전관리형 서버리스 컴퓨팅.

> **Serverless Computing Service?**   
> Cloud 서비스 제공 업체에서 동적으로 자원을 할당하여 주고, 사용량만큼만 요금을 지불하는 컴퓨팅 서비스  
> 컴퓨팅 Node 자원에 프로비저닝과 관리비용이 준다는 장점이 있다.

### with EKS
EKS 에서는 FargateProfile 이라는 오브젝트 안에 설정한 label 값이 일치하는 파드가 생성되면, 컴퓨팅 엔진을 올려준다.
Fargate 는 Pod와 1대1 로 매핑되어 하나의 Fargate 노드에는 하나의 Pod 만 올라갈수있다. 

### 장단점

- 장점
1. 자동으로 노드의 성능을 조절해주기 때문에 오토스케일러를 세팅할 필요가 없다.

2. Pod 실행 시간단위로 비용청구가 된다.

3. Fargate 구조상 Pod 간 기본 커널, CPU 리소스, 메모리 리소스 또는 탄력적 네트워크 인터페이스를 공유하지 않아 VM 수준의 격리가 가능하다

- 단점

1. Statefule 한 워크로드가 사용불가능하다  → 세션 정보를 서버가 아닌 외부에 저장하도록 설계 혹은 JWT 이용

2. 특수 권한이 있는 DaemonSet 등의 Pod가 이용불가능하다 → DaemonSet 처럼 이용하고 싶은 컨테이너가 있다면 k8s SideCar 패턴으로 지원하여야한다. (EFK 스택을 sidecar 패턴으로 구현할 예정이다.)

3. 로드밸런서의 사용이 제한적이다. ALB or ELB + Ingress 를 사용하여야한다

 

