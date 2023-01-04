# 참고자료  

가이드를 작성할 때 참고했던 자료들입니다.  

<br/>

## TLS  

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

## gRPC  

### Common  

* [gRPC | Introduction to gRPC](https://grpc.io/docs/what-is-grpc/introduction/)  
* [gRPC | Basics tutorial](https://grpc.io/docs/languages/go/basics/)  
* [Building a gRPC Server in Go](https://sahansera.dev/building-grpc-server-go/)  
* [grpc/grpc-go](https://github.com/grpc/grpc-go)  
* [NGINX Ingress Controller - gRPC](https://kubernetes.github.io/ingress-nginx/examples/grpc/)  

<br/>

### grpcurl  

* [https://github.com/fullstorydev/grpcurl](https://github.com/fullstorydev/grpcurl)

  ```bash
  go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest
  ```
