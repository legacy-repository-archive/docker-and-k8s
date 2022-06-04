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




  




