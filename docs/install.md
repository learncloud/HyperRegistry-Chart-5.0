# 설치 가이드

## Prerequisite

- git (checked version: 1.8.3.1) `ex. yum intall git`
- podman (checked version: v3.0.1) `or` Docker-ce (checked version 20.10.12)
- [helm](https://helm.sh/docs/intro/install/) **홈페이지에설치방식있음** (v3.8.0+)
- [kubectl](https://kubernetes.io/ko/docs/tasks/tools/install-kubectl-linux/) (checked version: v1.19.4)
- ingress controller

## Installation

### 1.HyperRegistry 설치 

1. (외부망 환경에서) HyperRegistry 이미지 및 바이너리 다운로드

   1. git repo Clone
      ```bash
      git clone -b 5.0 https://github.com/learncloud/HyperRegistry-Chart-5.0.git
      
      ```
      
   2. HyperRegistry 이미지 다운로드
      ```bash
      cd HyperRegistry-Chart
      chmod +x download.sh
      ./download.sh <download_dir> # ./download.sh ./downloads
      
      ```
      
   3. [Helm 클라이언트 다운로드](https://github.com/learncloud/install-helm-v3.0/) 
      
      
