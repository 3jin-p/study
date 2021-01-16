AWS-EKS with Fargate 
------
- AWS Fargate? 
[AWS Fargate 간단 설명](https://github.com/3jin-p/study/tree/master/infra/aws/fargate)  

## 사전준비  

- AWS CLI 설치 
버전 1.18.163 이상 또는 버전 2.0.59  OR AWS-CLI-2  
`curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg" sudo installer -pkg AWSCLIV2.pkg -target /`  
- AWS 자격증명 구성  
```bash
$ aws configure
AWS Access Key ID [None]: <>
AWS Secret Access Key [None]: <>
Default region name [None]: <>
Default output format [None]: <>
```
- EKSCTL 설치  
1. Homebrew가 설치되어 있지 않다면 Homebrew 설치   
`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"`   
2. Weaveworks Homebrew tap 설치 및 eksctl 설치 
`brew tap weaveworks/tap && brew install weaveworks/tap/eksctl`  

 eksctl 을 설치하면 kubectl 은 자동으로 설치되며 사전준비 끝!
  
## 서버구성

1.클러스터 생성  
--
- 클러스터생성  
``` bash 
eksctl create cluster \ 
--name <cluster-name> \ 
--version <1.18> \ 
--region <cluster-region> \ 
--fargate
```  
으로 클러스터를 생성하면 Fargate Profile 과 함께 설정한 값들로 클러스터가 생성된다.


- 확인
``` bash
eksctl get cluster
eksctl get fargateprofile --cluster <cluster-name>  
```


처음 클러스터를 생성하면 default 네임스페이스에 대한 fargateprofile 만 존재한다.   
원하는 레이블과 네임스페이스에 대한 fargateprofile 의 생성이 필요하다.  
 
``` bash
eksctl create fargateprofile  \
--cluster <cluster-name> \
--name <profile-name> \
--namespace <name-space for fargateprofile> \
--lables <“key1=value1, key2=value2”>
```
  
  
  
2.어플리케이션 배포   
--
- Namespace 생성   
`kubectl create namespace <namespace 명>`

별도의 Namespace 를 두면 관리가 용이하다.

- 배포생성 

``` bash
kubectl create deployment <deployment name>
--namespace=<namespace> 
--image=<image path:tag>
```

를 입력하면 배포가 생성된다.  

`kubectl edit deployment -n <namespace> <deployment name>`

을 실행하여 fargateprofile에 맞는 레이블을 세팅해주면 fargate 인스턴스에 파드가 올라온 것을 확인 할 수 있다.

- 확인  
`kubectl get po -n <namespace>`  

3.파드 노출 (서비스 생성)
--
- 서비스 생성
``` bash
kubectl expose deployment <deployment name> \
-n <namespace> \
--type=NodePort \
--port=<serviceport> \
--target-port=<pod port>
```
OR
``` bash
kubectl create svc <service type> <service name> \
--namespace=<namespace> \
--tcp=<port>:<targetPort>
```
>Fargate를 이용할시 serviceType 은 NodePort만 허용된다.

port 는 서비스로 접근할 포트, targetPort는 해당 포트로 접근했을 시 연결되는 컨테이너의 포트이다.

- 확인  
`kubectl -n <namespace> describe service <service-name>`


이제 클러스터 내부에서는 서비스로 인해 생성된 해당 NodePort 로 파드에 접근할 수 있다.

4.서비스를 외부에 노출 (인그레스 생성)
--
이제 클러스터 외부에서 파드에 접근할 수 있도록 해야한다.

외부에서 접근할때는 로드밸런서를 타도록 설정.

>로드밸런서로는 ALB를 사용한다.

AWS의 ELB를 연결하기 위해서, 클러스터의 자격증명이 필요하다.

---
1. 클러스터의 자격증명  
- IAM OIDC 공급자를 생성하여 클러스터와 연결
``` bash
eksctl utils associate-iam-oidc-provider \ 
--region <region-code> \ 
--cluster <cluster name> \ 
--approve
```

- 로드 밸런서 컨트롤러에 IAM AWS를 호출할 수 있도록 하는 정책 다운로드 

`curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json`


- 다운로드 받은 파일로 정책 생성
``` bash
aws iam create-policy \
    --policy-name <policy-name> \
    --policy-document file://iam-policy.json
```
- 사용할 로드 밸런서 컨트롤러에 대한 서비스 어카운트 생성 및 정책 연결
``` bash
eksctl create iamserviceaccount \
  --cluster=<cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=arn:aws:iam::<AWS_ACCOUNT_ID>:policy/<policy-name> \
  --override-existing-serviceaccounts \
  --approve
```

ALB를 위한 Ingress Controller 설치
---
2.AWS LoadBalancer Controller 설치  
``` bash
kubectl apply \
-k "github.com/aws/eks-charts/stable/aws-load-balancer-controller//crds?ref=master"
```
```
helm repo add eks https://aws.github.io/eks-charts

helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller \ 
--set clusterName=<cluster-name> \ 
--set serviceAccount.create=false \ 
--set serviceAccount.name=aws-load-balancer-controller \ 
-n kube-system
```
- 설치확인

`kubectl get deployment -n kube-system aws-load-balancer-controller`


3.설치한 로드밸런서 컨트롤러가 관리할 Ingress 생성
``` bash
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip 
    kubernetes.io/ingress.class: <위에서 설정한 alb 명>
  creationTimestamp: "2020-12-22T08:19:05Z"
  finalizers:
  - ingress.k8s.aws/resources
  labels:
    app: <>
  name: <인그레스명>
  namespace: <namespace>
  resourceVersion: "613797"
  selfLink: /apis/extensions/v1beta1/namespaces/kubetest/ingresses/testingress
  uid: 9cd097fb-33fc-4ca1-81f9-fa2c93e68446
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: <인그레스가 연결할 서비스 명>
          servicePort: <서비스의 포트>
        path: /* (해당 백엔드로 연결될 uri)
        pathType: ImplementationSpecific
```  
- Ingress 생성   
`kubectl apply -f <yaml 파일명>`

- 설명  
annotation과 backend 이하 설정값만 유의해준다.  
kubernetes.io/ingress.class: 로드밸런서 컨트롤러가 인그레스에 연결되도록 판단하는 값.  
 alb.ingress.kubernetes.io/target-type: ip  여러 속성이 있지만 fargate에서는 ip 만 사용이 가능.

- 확인   
`kubectl get ingress/<인그레스 명> -n <네임스페이스>`  
- 출력  
``` bash  
NAME           CLASS    HOSTS   ADDRESS                                                                   PORTS   AGE
인그레스   <none>   *       k8s-game2048-ingress2-xxxxxxxxxx-yyyyyyyyyy.us-west-2.elb.amazonaws.com   80      2m32s
```
이런식으로 주소가 할당된 것을 확인할 수 있다.  
 이제 해당 주소로 접속하면 파드에 떠있는 컨테이너의 어플리케이션으로 연결된다. 

제대로 되지 않았다면

`kubectl logs -n kube-system   deployment.apps/aws-load-balancer-controller`

으로 로드밸런서 컨트롤러의 로그를 확인할 수 있다.  

**NEXT STEP**  
[k8s 무중단배포](https://github.com/3jin-p/study/tree/master/infra/k8s/deploy)   
[EFKstack 로깅시스템 구성](https://github.com/3jin-p/infra-aws-eks/edit/master/efkStack)

