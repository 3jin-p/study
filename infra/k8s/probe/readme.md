Kubernetes 무중단 배포
--

### Kubernetes 배포전략 

쿠버네티스는 기본적으로 3가지 배포전략을 지원한다.  
1. RollingUpdate
2. Blue/Green Update
3. Canary Update 

---
### 각 배포 전략별 쿠버네티스에서의 진행방식
-  **RollingUpdate**
![RollingUpdate](rolling.png)
K8s 의 RollingUpdate는 다음과 같이 진행된다.  
1. ReplicaSet 을 복제
2. 복제한 ReplicatSet 에서 동시생성 설정한 갯수만큼의 Pod를 생성
3. 생성한 Pod의 HealthCheck 가 통과하면, 그 갯수만큼 기존의 Pod를 중지
4. 2번부터 모든 파드가 새로운것으로 교체될때까지 반복

RS 를 추가적으로 생성해야하는 자원외에는 추가자원이 필요하지 않고 설정이 단순하다는 장점이 있지만  
구버전과 신버전이 공존하는 시간이 있다는 점과 롤백시 역순을 거쳐야해 롤백이 오래걸린다는 단점이 있다.

- **Blue/Green Update**
![Blue/Green](bluegreen.png)
K8s 의 Blue/Green 배포는 labelSelector 를 이용하여 진행된다.
1. 기존 버전은 Blue label 로 생성되어있다.
2. Deployment 설정을 변경해 신규 버전을 Green label로 추가로 생성한다.
3. Inress Service의 labelSelector를 Blue에서 Green 으로 변경한다. 

모든 파드를 생성해놓은 후, 로드밸런서를 연결해둔 Ingress 를 통해 트래픽만 한번에 교체하므로,  
버전이 공존하여 발생하는 문제가 없고 Ingress의 라벨만 교체하면 롤백역시 단순하다는 장점이 있다.  
하지만 클러스터의 리소스가 두배로 든다는 단점이 있다. 


- **Canary 배포**
![Canary](canary.png)
Canary 배포 역시 label 을 이용한다.
1.배포에 Service 가 바라볼 label 이외에 버전을 명시할 label 을 추가한다.
2.Service가 바라보는 label 을 포함하고 버전만 변경한 Deployment 를 추가한다.
3.추가한 Deployment의 replica 수를 점점 조절해나간다.
4.일정 비율이상이 넘을때 까지 오류가 발견되지 않는다면, 새로운 버전으로 완전히 교체한다.

신규 버전에 문제가 생겼을때, 모든 사용자에게 해당 오류를 노출시킬수 있는 위험부담이 최소화된다.
하지만 배포에 많은 시간을 들여야하며, 결국 특정사용자는 오류를 경험해야한다는 단점이 있다. 
---
# 각 배포별 설정 

