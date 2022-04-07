# HyperRegistry-Chart
This is helm repository for HyperReigstry

## Prerequisite Installation
- [HyperRegistry-Chart 설치](https://github.com/learncloud/HyperRegistry-Chart-5.0/blob/main/docs/install.md#폐쇄망에서-설치를-위한-환경-준비하기)
- [HyperAuth OIDC 연동](https://github.com/learncloud/HyperRegistry-Chart-5.0/blob/main/docs/oidc.md)
- [레지스트리 생성 및 설정하기](https://github.com/learncloud/HyperRegistry-Chart-5.0/blob/main/README.md#%EB%A0%88%EC%A7%80%EC%8A%A4%ED%8A%B8%EB%A6%AC-%EC%83%9D%EC%84%B1-%EB%B0%8F-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0)

## Optional Installation

- [P2P Preheat 연동](https://github.com/learncloud/HyperRegistry-Chart-5.0/blob/main/docs/kraken.md) 
- [외부 Registry Replication 연동](https://github.com/learncloud/HyperRegistry-Chart-5.0/blob/main/docs/replication.md)
- [외부 HA DB 구성](https://github.com/learncloud/HyperRegistry-Chart-5.0/blob/main/docs/postgres.md)
- [외부 HA REDIS 구성](https://github.com/learncloud/HyperRegistry-Chart-5.0/blob/main/docs/redis.md)
- [이미지 서명 가이드 (Download pptx)](https://tmaxcloud-ck1-2.s3.ap-northeast-2.amazonaws.com/%EC%9D%B4%EB%AF%B8%EC%A7%80+%EC%84%9C%EB%AA%85.pptx)

## HyperRegistry-Chart 설치
## Prerequisite

- git (checked version: 1.8.3.1) `ex. yum intall git`
- podman (checked version: v3.0.1) `or` Docker-ce (checked version 20.10.12)
- [helm](https://helm.sh/docs/intro/install/) **홈페이지에설치방식있음** (v3.8.0+)
- [kubectl](https://kubernetes.io/ko/docs/tasks/tools/install-kubectl-linux/) (checked version: v1.19.4)
- ingress controller

## Installation

### 1. 폐쇄망 환경 준비

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
      
      ```
      
   3. [Helm 클라이언트 다운로드](https://github.com/learncloud/install-helm-v3.0/) 

## 공통 설치 진행
### 레지스트리 생성 및 설정하기
1. 포털 로그인 (https://core.hr.domain)    
   `ex) core.hr.192.168.178.82.nip.io`

3. 프로젝트 생성하기
   1. Projects > New Project 클릭
   2. 이름 입력 후 OK
   
4. 프로젝트 환경설정
   1. Proejcts > [생성한 레지스트리] > Configuration
   2. (필요 여부에 따라) 다음 항목 체크
      - Public (check)
      - Enable content trust (non-check)
      - Prevent vulnerable images from running (non-check)
      - Automatically scan image on push (non-check)
   3. Save
   
5. (Optional) 레지스트리 사용자 생성 및 등록
   1. Administration > Users > New User
   2. 유저 생성
   3. Project > [생성한 유저가 사용할 프로젝트] > Members > + User
   4. [생성한 유저 이름] 입력 및 권한 설정 후 OK

### 신뢰하는 서버로 등록하기
**[NOTE] Harbor로부터 이미지를 pull받을 모든 노드에 적용**
1. 인증서(ca.crt) 다운로드
   - Proejcts > Repositories클릭 > Registry Certificate

2. (모든 노드에) 인증서 등록
   - 다운로드한 인증서(ca.crt)를 신뢰하는 인증서 경로에 복사 및 등록
      - CentOS
        ```bash
        scp C:\Users\jace\Desktop\ca.crt root@192.168.178.17:/etc/pki/ca-trust/source/anchors
        update-ca-trust
        
        ```
        
      - Ubuntu
        ```bash
        scp C:\Users\jace\Desktop\ca.crt root@192.168.178.17:/usr/local/share/ca-certificates
        update-ca-certificates
        
        ```
        
3. 컨테이너 런타임 재기동
   - Docker
     ```bash
     systemctl restart docker
     
     ```
     
   - CRI-O
     ```bash
     systemctl restart crio
     
     ```

### 이미지 푸시하기
```bash
docker login [harbor_domain]/[project이름] # docker login core.hr.192.168.178.82.nip.io/test-project # 로그인은 admin/admin이 기본임 -- docker에 로그인 계정으로 하는게 아님
docker pull {imgge} # docker pull nginx
docker tag [to_push_image] [harbor_domain]/[project]/[repository]:[tag] # docker tag nginx:latest core.hr.192.168.178.82.nip.io/test-project/nginx:latest
docker push [harbor_domain]/[project]/[repository]:[tag] # docker push core.hr.192.168.178.82.nip.io/test-project/nginx:latest
#Hyper registry 업로드 완료스크린샷은 바탕화면에

```
