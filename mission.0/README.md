# kind 클러스터에 gRPC 배포하기

kind 클러스터에 ingress-nginx, gRPC 를 배포하는 가이드입니다.  

먼저, 가이드를 실행하기 위해 작업 경로로 이동합니다:  

```bash
cd $REPO_ROOT/mission.0
```

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

위 매니페스트에는 호스트 포트를 *ingress controller*로 전달하고 *taint tolerations(오염 허용)*을 설정하고 이를 커스텀 레이블이 지정된 노드로 스케줄하기 위한 kind 의 *patch* 가 포함되어 있습니다.  

> [`deploy-ingress.yaml`](./deploy-ingress.yaml) 에 https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml 의 내용이 있습니다.  

<br/>

이제 Ingress가 모두 설정되었습니다.  
다음 명령을 실행하여 진행 중인 요청이 완료될 때까지 기다립니다:  

```bash
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

<br/><br/><br/>

## gRPC 배포하기  

### localhost 에서 https 를 사용하기 위해 `mkcert` 설정하기  

다음 명령을 실행하여 로컬 CA를 생성합니다:  

```bash
mkcert -install
```

다음 명령을 실행하여 지정한 호스트 이름으로 `.pem` 인증서를 생성합니다:  

```bash
mkcert localhost 127.0.0.1
```

명령을 실행하면 `localhost+1.pem`, `localhost+1-key.pem` 파일이 생성됩니다.  

> WSL 을 사용하는 경우 호스트와 WSL 에 각각 mkcert 를 설치하고,  
> CA 인증서를 다른 시스템으로 복사한 후 `mkcert -install` 을 다시 실행해야 합니다.   
> `mkcert -CAROOT` 명령으로 CA 인증서 위치를 알 수 있습니다.  


<br/>

### 인증서로 쿠버네티스 *secret* 만들기  

다음 명령으로 쿠버네티스 *secret*(tls-secret) 를 만듭니다:  

```bash
kubectl create secret tls tls-secret --cert=localhost+1.pem --key=localhost+1-key.pem
```

<br/>

### gRPC 이미지 만들고 발행하기(Build/Publish)  

[`kind-with-registry.sh`](./kind-with-registry.sh) 스크립트 실행하면 kind 클러스터와 Docker Registry 가 생성됩니다.  
[Dockerfile](./Dockerfile)으로 gRPC 이미지를 만들고 Registry 에 발행합니다.    

```bash
docker build --tag localhost:5001/go-grpc-greeter-server .
docker image push localhost:5001/go-grpc-greeter-server
```

<br/>

### gRPC 서버 배포하기  

다음 명령을 실행하여 *Deployment*, *Service*, *Ingress* 를 배포합니다:  

```bash
kubectl apply -f mission.0.resources.yaml
```

<br/><br/><br/>

## 테스트  

모든 구성이 끝났다면 다음 명령으로 테스트를 합니다:  

```bash
grpcurl localhost:443 helloworld.Greeter/SayHello
```

> 만약 `grpcurl` 가 없다면 [fullstorydev/grpcurl](https://github.com/fullstorydev/grpcurl) 를 참고하여 설치합니다.  

<br/>

문제가 없다면, 다음과 같은 메시지가 출력됩니다:  

```bash
{
  "message": "Hello "
}
```

<br/><br/><br/>

## 참고자료  

### TLS 설정하기  

* [로컬 개발에 HTTPS를 사용하는 방법](https://web.dev/i18n/ko/how-to-use-local-https/)

  mkcert 설치하기:  
  ```bash
  sudo apt install mkcert
  sudo apt install libnss3-tools
  ```

  ```bash
  sudo apt install mkcert
  sudo apt install libnss3-tools
  kubectl create secret tls tls-secret --cert=localhost.pem --key=localhost-key.pem
  ```

* [Kubernetes Ingress Explained](https://towardsdatascience.com/kubernetes-ingress-explained-1aeadb30f273)

  ```bash
  mkcert -install
  mkcert localhost 127.0.0.1
  kubectl create secret -n kube-system tls mkcert-tls-secret --cert= 
   <DIRECTORY_CONTAINING_PEM>/_wildcard.myminikube.demo+2.pem --key= 
    <DIRECTORY_CONTAINING_KEY_PEM>/_wildcard.myminikube.demo+2-key.pem
  ```

* [이건 Windows에서 어떻게 셋팅하지?](https://brunch.co.kr/@devapril/49)

  chocolatey 설치하기:  

  ```bash
  # Get-ExecutionPolicy 실행
  $ Get-ExecutionPolicy

  # Restricted라면 AllSigned나 Bypass로 설정
  # ExcutionPolicy를 AllSigned로 설정
  $ Set-ExecutionPolicy AllSigned

  # ExcutionPolicy를 Bypass로 설정
  $ Set-ExecutionPolicy Bypass -Scope Process

  # 설치!
  $ Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

  # 확인
  $ choco
  ```

  mkcert 설치하기:  

  ```bash
  choco install mkcert
  ```

* [mkcert - localhost를 https 환경으로 만들기](https://yung-developer.tistory.com/112)

[Using mkcert for secure localhost development on WSL 2](https://www.haveiplayedbowie.today/blog/posts/secure-localhost-with-mkcert/)

1. Install `mkcert` on Windows - I used [Chocolatey](https://chocolatey.org/) for this
2. Ran `mkcert -install`
3. Install `mkcert` on Ubuntu WSL - for fun I decided to clone the repository and build it from source using [Go](https://golang.org/)
4. Run `mkcert -install` on Ubuntu
5. Back in a Windows PowerShell terminal I ran `mkcert -CAROOT` to find out where the certificates were created on Windows. I copied these.
6. On Ubuntu run `mkcert -CAROOT` and then `explorer.exe` to open File Explorer on Windows at the Ubuntu directory.
7. Paste the certificates from step 5
8. Run `mkcert -CAROOT` again on Ubuntu (I'm not sure if this step is neccessary but it works!)
9. Create certificates on WSL e.g. `mkcert example.com "*.example.com" localhost`
10. Configure nginx or Apache to use the generated certificates

<br/>

### gRPC  

* [NGINX Ingress Controller - gRPC](https://kubernetes.github.io/ingress-nginx/examples/grpc/)  

<br/>

### grpcurl  

* [https://github.com/fullstorydev/grpcurl](https://github.com/fullstorydev/grpcurl)

  ```bash
  go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest
  ```
