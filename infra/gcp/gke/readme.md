GKE 노드 스케일 업
--
스케일 아웃 으로 해결할 수 없는 메모리 부족등의 문제가 있을때 머신의 스케일 업이 필요함

GUI에서 템플릿만 변경하면 템플릿이 정상적으로 복제가 되지 않는등 이슈가 많아, 콘솔에서 해결.

- **클러스터에 포함된 노드풀 목록 보기**

`gcloud container node-pools list --cluster <클러스터 명>`

- **새 노드풀 생성 및 클러스터에 추가** 
``` bash
gcloud container node-pools create <생성할 노드풀 이름> \
  --cluster= <클러스터 명> \
  --machine-type=<머신 타입> \
  --num-nodes=<노드풀에 최초 포함할 노드 수>
  ```

> 머신타입 종류 : https://cloud.google.com/compute/docs/machine-types   

- **Pod 가 돌고있는 Node 목록 조회**

`kubectl get pods -o=wide`

위 명령어로 확인해보면 노드풀을 추가해도 Pod는 여전히 기존 노드풀에서 돌고있다.

- **Pod를 새 노드풀로 마이그레이션**

1. 기존 노드풀에 Pod 예약을 차단 

2. 기존 노드풀 제거 

의 순서로 실행을하면 예약 가능한 노드가 새로운 노드풀의 노드밖에 없게 되어 

새로운 노드풀에 파드가 생성됨.

- **노드풀의 Pod 예약 차단**
``` bash
for node in $(kubectl get nodes -l cloud.google.com/gke-nodepool=<노드 풀 명> -o=name); do
  kubectl cordon "$node";
done
```

- **노드풀 제거**

1. 노드풀에서 노드 배출
``` bash
for node in $(kubectl get nodes -l cloud.google.com/gke-nodepool=<노드 풀 명> -o=name); do
  kubectl drain --force --ignore-daemonsets --delete-local-data --grace-period=10 "$node";
done
```  

이제 Pod가 새로운 노드풀에서 돌고있는것을 확인할 수 있음.

2. 노드풀 삭제   
`gcloud container node-pools delete <노드 풀 명> --cluster <클러스터 명>`

-  **확인** 

`gcloud container node-pools list --cluster <클러스터 명>` 
``` bash
NAME          MACHINE_TYPE   DISK_SIZE_GB  NODE_VERSION
노드풀 이름       머신 타입        디스크 크기       노드버전
```
