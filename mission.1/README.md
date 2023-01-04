# gRPC 튜토리얼 배포하기  

kind 클러스터에 [gRPC - Basics tutorial](https://grpc.io/docs/languages/go/basics/) 를 배포하는 가이드입니다.

먼저, 가이드를 실행하기 위해 작업 경로로 이동합니다:  

```bash
cd $REPO_ROOT/mission.1
```

<br/><br/><br/>

## 개요  

[gRPC Go Basics tutorial](https://grpc.io/docs/languages/go/basics/) 을 쿠버네티스에 배포하는 가이드입니다.  

튜토리얼의 예제는 클라이언트가 라우트에 있는 기능에 대한 정보를 얻고, 라우트 요약을 만들고, 트래픽 업데이트와 같은 라우트 정보를 서버 및 다른 클라이언트와 교환할 수 있는 간단한 라우트 매핑 앱입니다.  

서비스에 대한 내용은 가이드를 참조하세요.  


<br/><br/><br/>

## 설정하기  

시작하기 전에 클라이언트 및 서버 인터페이스 코드를 생성하기 위한 도구를 설치해야 합니다.  
설치하지 않은 경우 [빠른 시작](https://grpc.io/docs/languages/go/quickstart/) 의 [전제 조건](https://grpc.io/docs/languages/go/quickstart/#prerequisites) 섹션의 설치 지침을 참조하세요.  

<br/><br/><br/>

## 예제 코드 가져오기  

예제 코드는 [grpc-go](https://github.com/grpc/grpc-go) 저장소에 있습니다.  

[저장소를 zip 파일로 다운로드](https://github.com/grpc/grpc-go/archive/v1.50.0.zip)하여 압축을 풀거나 저장소를 복제합니다:  

```bash
git clone -b v1.50.0 --depth 1 https://github.com/grpc/grpc-go
```

<br/><br/><br/>

## 이미지 빌드/게시하기  

### client 와 server 코드 생성하기  

```bash
RUN protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    routeguide/route-guide.proto
```

<br/>

### 도커 이미지 빌드/게시하기  

```bash
docker build -t localhost:5001/route-guide ./grpc-go/ -f Dockerfile
docker image push localhost:5001/route-guide 
```

> * 이미지 디버깅하기  
>   `docker run -it --rm --name route_guide localhost:5001/route-guide` 

<br/><br/><br/>

## kind 클러스터 준비하기  

### kind 클러스터와 Docker Registry 만들기  

[`kind-with-registry.sh`](./kind-with-registry.sh) 스크립트 실행하여 kind 클러스터와 Docker Registry 를 만듭니다:  

```bash
../common/kind-with-registry.sh grpc
```

<br/>

### ingress NGINX 배포하기  

*kind* 는 다음 *Ingress controller* 를 지원합니다.  
* *Contour*
* *Ingress Kong*
* *Ingress NGINX*  

이 글에서는 *Ingress NGINX controller* 를 사용하겠습니다.  

다음 명령을 실행하여 ingress-nginx controller 를 배포합니다:  

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

<br/>

### 인증서로 쿠버네티스 *secret* 만들기  

다음 명령으로 쿠버네티스 *secret*(tls-secret) 를 만듭니다:  

```bash
kubectl create secret tls tls-secret --cert=grpc-go/examples/data/x509/server_cert.pem --key=grpc-go/examples/data/x509/server_key.pem
kubectl create secret tls tls-secret --cert=localhost+1.pem --key=localhost+1-key.pem
```

<br/><br/><br/>

## gRPC 배포하기  

다음 명령을 실행하여 *Deployment* 를 배포합니다:  

```bash
kubectl apply -f mission.1.resources.yaml
```

<br/><br/><br/>

```bash
go run client/client.go -tls=true -addr=localhost:443
```

<br/><br/><br/>

## [참고자료](../references/README.md)  

가이드를 작성할 때 참고했던 자료들입니다.  