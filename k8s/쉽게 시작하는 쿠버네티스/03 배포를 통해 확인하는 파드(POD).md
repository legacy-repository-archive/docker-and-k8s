# 배포를 통한 쿠버네티스 체험
## 배포를 통해 확인하는 파드 
 
쿠버네티스의 애플리케이션을 배포한다는 뜻은    
마스터 노드를 기준으로 워커 노드에 애플리케이션(들)을 설치한다고 생각하면 쉽다.     
그리고 **쿠버네티스에서의 애플리케이션(들)의 단위는 Pod 라고 한다.**         
   
쿠버네티스는 컨테이너를 관리하기 위한 솔루션(오케스트레이션)이다.     
즉, Pod(파드)는 **특정한 목적을 수행하기 위한 컨테이너들의 집합(볼륨 포함)** 이다.     

### 파드 생성 
```shell
kubectl run nginx --image=nginx
```
* kubectl : 쿠버네티스 사용을 위한 명령어 
* run : 파드 또는 디플로이 먼트를 생성하기 위한 kubectl 옵션  
* nginx : 파드의 이름
* --image=nginx : 사용할 이미지의 이름 

### 파드 확인 
```shell
kubectl get pods
```

<img width="842" alt="image" src="https://user-images.githubusercontent.com/50267433/171996119-cf3f5cd5-a2b6-49dc-8e60-a2dfe2c167f8.png">

* 모든 파드 및 디플로이먼트의 목록이 나온다.  
* READY 라는 상태가 나오는데 ?/? 가 같으면 생성 완료고 다르면 생성중이다.  

## 파드를 외부에서도 접속하게 하는 서비스 

쿠버네티스 클러스터는 각 노드와 컨테이너를 관리하는 논리적인 개념이며(논리적 네트워크)        
컨테이너들은 특별한 설정 없이는 일반적으로 외부에서의 접근은 불가능하다.(내부에서의 접근은 가능) 
  
Nginx 와 같은 프론트 서버의 역할을 하는 애플리케이션들은 어떻게 사용해야할까?     

1. 게이트웨이를 날린다. -> 보안에 취약해진다.   
2. 서비스 영역(dmz)을 만들고 파드를 연결해서, 서비스를 통해 파드에 접근하도록 한다.   
   정확히 말하면, NodePort 를 이용해서 노드에 접속한 후 노드를 통해 파드를 찾아 접속하는 것이다.     

### 서비스 사용하기(NodePort) 

```shell
kubectl expose pod nginx --type=NodePort --port=80
```
* expose pod { 파드 이름 } : 생성된 파드를 노출시킨다.  
* --type-NodePort : 노드 포트 서비스를 이용한다.   
* --port : 노드 내에서 찾을 Port 를 입력한다.   

```shell
kubectl get service
```

<img width="842" alt="image" src="https://user-images.githubusercontent.com/50267433/171996146-5c063c68-f93a-4cba-b1d0-8c5d18466950.png">

* PORT(S) : `--port번호/실제_외부에_노출된_포트` 
* expose 로 설정했던 port는 노드 내에서 찾을 port이다.    

```shell 
kubectl get node -o wide
```

<img width="1607" alt="image" src="https://user-images.githubusercontent.com/50267433/171996389-bec90f45-ccc7-41ae-82fb-d15549f16a5e.png">

위 결과에서 INTERNAL-IP(private ip) 를 기준으로 포트를 붙이면 pod 에 접근가능하다.  

## 디플로이먼트 
   
만약 파드가 죽으면 어떻게 될까?        
컨테이너는 격리된 환경을 지향하기에, 특별한 설정 없이는 데이터가 전부 날라갈 것이다.       
그리고 이러한 단일 파드는 무중단 서비스를 운영하기에는 리스크가 너무 많다.   

### 파드와 디플로이먼트의 차이 
 
파드는, 특별한 작업을 처리하기 위한 '컨테이너들의 집합'의 **단일 개체**이다.       
디플로이먼트는, 특별한 작업을 처리하고 운영하기 위한 **파드들의 집합**이며 **'컨테이너들의 집합'의 다수 개체**이다.       

### 디플로이먼트 배포 방법 

```
kubectl run
``` 
`kubectl run`은  사실 쿠버네티스 버전1 때 등장한 파드 배포를 위한 명령어이다.      
물론, 쿠버네티스 환경의 파드이기에 디플로이먼트 배포도 지원했지만, 1.8 버전 이후로는 지원을 중단랬다.     

현재는 아래 2가지 명령어를 통해 pod 와 deployment 를 배포할 수 있다.  
 
* kubectl create : 컨테이너 이미지를 통한 배포   
* kubectl apply : 컨테이너 파일(yml)을 통한 배포     

### 디플로이먼트 배포하기 

```shell
kubectl create deployment deploy-nginx --image=nginx
```

![image](https://user-images.githubusercontent.com/50267433/171996844-bb445be9-16ce-4c31-acd7-c55927511723.png)

* kubectl create { pod / deployment } : run 과 다르게 어느 형태로 배포할지 선언해야한다.   


<img width="1526" alt="image" src="https://user-images.githubusercontent.com/50267433/171996891-e5312df5-4f16-4f2a-b524-27ba5f8f1fa5.png">
  
* `kubectl get pods`를 했을시 정상적으로 배포되었음을 확인할 수 있다.    
* NAME 뒤에 특별한 문구들이 들어갔는데 '해쉬'를 이용한 고유 이름을 주었다고 생각하면 된다.  
    * 디플로이먼트는 여러 파드의 단위이기에 각각의 파드들을 식별할 수 있도록 해시값을 넣어준 것이다.    
  
`kubectl get pods` 를 통해 알 수 있듯이 현재 하나의 파드만 배포가 된 상태이다.           
다시 생각해보면 디플로이먼트는 파드의 집합이라고 했지만, 하나의 파드만 생성된 것이 이상하다.  
 
사실 이는 정상적이며.    
`kubectl apply`의 파일을 통해서 배포하면 한번에 디플로이내의 여러 파드를 배포할 수 있지만,    
`kubectl create` 를 통해 배포하면 마스터에만 배포되고 이후 `replicaSet` 을 이용해야한다.        

### 워커 노드에 배포하기   

```shell
kubectl scale deployment deploy-nginx --replicas=3
```

* kubectl scale deployment { 파드(디플로이) 이름 } : 스케입 아웃할 디플로이먼트 선택 
* --replicas=? : 스케일 아웃할 개수 선택  

<img width="1526" alt="image" src="https://user-images.githubusercontent.com/50267433/171997034-43add8c1-9052-485d-a8f8-f4f66bc677c8.png">

* 결과를 보면 알 수 있듯이, 여러 파드들이 생성되었고 각각의 해시값을 이름에 포함하고 있다.   

## 외부로 노출하는 더 좋은 방법인 로드 밸런서 

### 디플로이먼트 외부로 노출하기 

```shell
kubectl expose deployment deploy-nginx --type=NodePort --port=80
> service/deploy-nginx exposed
```

<img width="1526" alt="image" src="https://user-images.githubusercontent.com/50267433/171997271-7c37cb87-68b9-4653-b185-b7ab855a97ed.png">

* pod 를 노출했던 것과 동일하게 디플로이먼트를 노출시킨다(pod -> deployment)  
  
단, 이전처럼 Inernal IP 를 찾아서 IP와 포트를 공개할 필요가 있을까?         
이 같은 문제를 해결하고자 LoadBalancer를 이용해 **공용 IP 를 제공해주는 방법이 있다.**      
이 같은 방법은 특정 노드가 죽었어도 영향을 받지 않으며 보안에 비교적 강하다는 장점이 있다.   

네이티브 쿠버네티스 방식에서는 주로 `MetaLB`를 통해서 로드밸런서를 적용시킨다.     


* 필자의 환경에서는 현재 MatLB와 강의에서 사용된 도커 이미지를 사용할 수 없으므로 이부분은 생략한다.  


```shell
kubectl expose deployment chk-hn --type=LoadBalancer --prot=80
kubectl get services

> NAME    TYPE    CLUSTER-IP    EXTERNAL-IP    PORT(S)    AGE
 chk-hn LoadBalancer ...          공용 IP      80/외부 포트   ..     
```
* 결과와 같이 공용 IP가 나오는 것을 확인할 수 있다.  

## 배포한 것들 삭제하기 
  
**expose 삭제하기**
```shell
kubectl delete service nginx
kubectl delete service deploy-nginx
```

**디플로이먼트 삭제하기**
```shell
kubectl delete deployment deploy-nginx
```

**파드 삭제하기**
```shell
kubectl delete pod nginx
```
