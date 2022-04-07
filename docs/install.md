## 1. HyperRegistry 설치
   - HyperRegistry 이미지 설치
   - git (checked version: 1.8.3.1) `ex. yum intall git`
   - podman (checked version: v3.0.1) `or` Docker-ce (checked version 20.10.12)
   - [helm](https://github.com/learncloud/install-helm-v3.0)  (v3.8.0+)
   - [kubectl](https://kubernetes.io/ko/docs/tasks/tools/install-kubectl-linux/) (checked version: v1.19.4)
   - ingress controller


   ### A. 폐쇄망 환경 준비

   1. (외부망 환경에서) HyperRegistry 이미지 및 바이너리 다운로드

      1. git repo 클론
         ```bash
         git clone -b 5.0 https://github.com/learncloud/HyperRegistry-Chart-5.0.git
         
         ```
      
      2. HyperRegistry 이미지 다운로드
         ```bash
         cd HyperRegistry-Chart
         chmod +x download.sh
         ./download.sh ./downloads
         
         #./download.sh <download_dir>
         # docker를 사용할 경우 download.sh내에 podman을 docker로 수정하고 진행해야합니다
         
         ```
        
   ### B. git 설치 & Docker-ce 설치 & helm 설치
      
   - GIT 설치
      ```bash
      yum intall -y git
        
      ```
        
   -  docker-ce 설치
      [docker-ce설치 참고](https://github.com/learncloud/install-registry-docker-ce)
         
          
   - helm 설치
     [Helm 설치 참고](https://github.com/learncloud/install-helm-v3.0)
  
  
  
  
<br><br><br>

## 2. HyperAuth OIDC 연동
   - [OIDC 연동 방법](https://github.com/learncloud/HyperRegistry-Chart-5.0/blob/main/docs/oidc.md)
