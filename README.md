# Learning gRPC  

![](docs/images/grpc-icon-color.small.png)  

<br/><br/><br/>

## 전제조건 

* Linux or [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install)(Windows)  
* Docker or [Docker Desktop](https://www.docker.com/products/docker-desktop/)(Windows)  
* [`kubectl`](https://kubernetes.io/docs/tasks/tools/)  
* [`Helm`](https://helm.sh/)  
* [`Kind`](https://kind.sigs.k8s.io/)  

<br/>

### localhost 에서 https 를 사용하기 위해 `mkcert` 설정하기  

이 과정은 localhost 에서 https 를 사용하기 위한 설정입니다.  

일반적인 TLS 인증서를 설정하려면 [kubernetes/ingress-nginx](https://github.com/kubernetes/ingress-nginx)의 [PREREQUISITES tls-certificates](https://github.com/kubernetes/ingress-nginx/blob/main/docs/examples/PREREQUISITES.md#tls-certificates) 과정을 참고하세요.  

[`mkcert`](https://github.com/FiloSottile/mkcert)는 로컬에서 신뢰할 수 있는 개발 인증서를 만드는 도구입니다.  
시스템 루트 저장소에 로컬 CA를 자동으로 생성 및 설치하고 로컬에서 신뢰할 수 있는 인증서를 생성합니다.  

![](./docs/images/sticker-transparent.png)

다음 명령을 실행하여 로컬 CA를 생성합니다:  

```bash
mkcert -install
```

다음 명령을 실행하여 지정한 호스트 이름으로 `.pem` 인증서를 생성합니다:  

```bash
mkcert localhost 127.0.0.1
```

다음 명령으로 쿠버네티스에서 사용할 *secret*(tls-secret) 를 만듭니다:  

```bash
kubectl create secret tls tls-secret --cert=localhost+1.pem --key=localhost+1-key.pem
```

<br/><br/><br/>

## 미션  

### 0. [kind 클러스터에 gRPC 배포하기](mission.0/README.md)  

kind 클러스터에 ingress-nginx, gRPC 를 배포하는 가이드입니다.  

<br/>

### 1. [gRPC 배우기](mission.1/README.md)  

...