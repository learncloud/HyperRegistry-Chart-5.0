# HyperRegistry-Chart
This is helm repository for HyperReigstry

## Installation
- [폐쇄망 환경 설치준비](https://github.com/learncloud/HyperRegistry-Chart-5.0/blob/main/docs/install.md#폐쇄망에서-설치를-위한-환경-준비하기)
- [HyperAuth OIDC 연동](https://github.com/learncloud/HyperRegistry-Chart-5.0/blob/main/docs/oidc.md)
- [P2P Preheat 연동](https://github.com/learncloud/HyperRegistry-Chart-5.0/blob/main/docs/kraken.md) **설치안해도됨 즉시 Usage부터 진행**
- [외부 Registry Replication 연동](https://github.com/learncloud/HyperRegistry-Chart-5.0/blob/main/docs/replication.md)
- [외부 HA DB 구성](https://github.com/learncloud/HyperRegistry-Chart-5.0/blob/main/docs/postgres.md)
- [외부 HA REDIS 구성](https://github.com/learncloud/HyperRegistry-Chart-5.0/blob/main/docs/redis.md)
- [이미지 서명 가이드 (Download pptx)](https://tmaxcloud-ck1-2.s3.ap-northeast-2.amazonaws.com/%EC%9D%B4%EB%AF%B8%EC%A7%80+%EC%84%9C%EB%AA%85.pptx)

## Usage
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
   1. Proejcts > [임의의 레지스트리] > Registry Certificate 클릭 **(이거 Projects -> Repositories -> REGISTRY CERTIFICATE 다운 클릭을 잘못쓴듯)**
2. (모든 노드에) 인증서 등록
   1. 다운로드한 인증서(ca.crt)를 신뢰하는 인증서 경로에 복사 및 등록
      - CentOS
        ```bash
        cp ca.crt /etc/pki/ca-trust/source/anchors
        update-ca-trust
        ```
      - Ubuntu
        ```bash
        cp ca.crt /usr/local/share/ca-certificates
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
