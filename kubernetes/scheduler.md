# k8s Scheduling
k8s에서 scheduling은 클러스터 내에서 파드(Pod)를 적절한 노드(Node)에 할당하는 과정을 의미  
스케줄러는 파드의 요구사항과 노드의 상태를 고려하여 최적의 배치를 결정  

## kube-scheduler
kube-scheduler는 쿠버네티스 control plane의 핵심 컴포넌트 중 하나로, 파드를 노드에 할당하는 역할을 담당
새로 생성되었지만 노드에 스케줄링되지 않는 파드를 감지하고, 스케줄링 정책에 따라 적절한 노드를 선택하여 파드를 할당  

기본적으로 12단계를 거침  
그 중 파드를 실행시킬 최적의 노드를 선택할 때 Filtering과 Scoring 단계를 거침

![filtering](/kubernetes/img/img_1.png)

### Filtering
파드의 요구사항을 충족하지 못하는 노드를 걸러내는 단계
pod를 스케줄링 할 수 있는 노드 집합 탐색  

Filter 단계에서 기본적으로 활성화된 플러그인은 `NodeAffinity`부터 `NodePort` 까지 13 단계가 있으며,  
사용자가 제어할 수 있는 부분은 `NodeAffinity`, `NodeUnschedulable`까지 총 6개 단계가 있음  

나머지 7개 단계에서는 노드의 실질적인 상태를 확인하여 파드를 스케줄링 할 수 있는지 확인을 함  

### Scoring
Filtering 단계가 끝난 후ㅜ 파드를 스케줄링 할 수 있는 노드집합이 결정되고,  
이 노드집합이 비어있지 않다면 해당 노드집합 중 파드를 실행시킬 최적의 노드를 찾기 위해 순위를 매김  

마찬가지로 Scoring 단계에서 기본적으로 활성화된 플러그인은 `NodeAffinity`부터 `VolumeBinding`까지 8단계가 있으며,  
사용자가 제어할 수 있는 부분은 `NodeAffinity`, `TaintToleration`까지 4단계이며  
나머지는 노드의 실질적인 상태를 확인하며 노드 우선순위를 매김

### 스케줄링 제어 방식
|플러그인|	내용|	Filtering|	Scoring|
|---|---|---|---|
|NodeAffinity|	NodeSelector와 NodeAffinity에 대한 스케줄링을 구현한다.|	O|	O|
|InterPodAffinity|	InterPod Affinity / AntiAffinity에 대한 스케줄링을 구현한다.|	O|	O|
|PodTopologySpread|	PodTopologySpread에 대한 스케줄링을 구현한다.|	O|	O|
|TaintToleration|	Taint와 Toleration에 대한 스케줄링을 구현한다.|	O|	O|
|NodeName|	Pod 스펙에 명시되어있는 nodeName을 확인한다.|	O|	X|
|NodeUnschedulable|	Node 스펙 중 unschedulable이 true로 설정되어있는 노드를 필터링한다.|	O|	X|

6가지 방식에서 언급된 방식으로 파드를 특정 노드에 스케줄링 할 지 결정할 수 있다.

### NodeName
![nodeName](/kubernetes/img/img_2.png)  
  
`nodeName` 방식은 가장 직관적이고 간단한 방식으로 파드를 특정 노드에 바로 배치되는 방식을 말함  
Scheduler 에서는 NodeName 플러그인에서 필터링 기능을 구현  
`nodeName` 은 파드 스펙의 필드(spec.nodeName)이고, 파드 생성 시점에서 이 값이 비어있다면,  
Filtering, Scoring 단계를 거쳐 나온 최적의 노드 이름으로 할당시켜줌  
`nodeName` 값이 지정되어 있다면, 해당 이름의 노드로 파드를 실행  
  
파드에 `nodeName`을 지정하는 방법은 파드의 `spec.nodeName`을 지정한 후 적용  
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nodename-test
  name: nodename-test
spec:
  containers:
  - image: nginx
    name: nodename-test
  nodeName: k8s-wn1
```
파드를 생성한 후 `-o wide` 옵션으로 파드가 실행되는 노드를 확인 가능  

### 제약사항
- nodeName과 일치하는 노드가 없다면 파드는 실행되지 않는다. 경우에 따라 파드를 자동으로 삭제
- nodeName과 일치하는 노드에 파드에 정의한 resource.requests 만큼의 리소스가 없다면 파드 실패하고 OOM 또는 OOC 발생

### nodeSelector
![nodeselector](/kubernetes/img/img_3.png)

`nodeName`을 제외하면 `nodeSelector` 방식이 파드를 특정 노드로 스케줄링하는 가장 간편한 방법
NodeAffinity 플러그인에서 nodeSelector에 대한 필터링 기능 구현
![nodeSelector2](/kubernetes/img/img_4.png)

출처  
https://beer1.tistory.com/56