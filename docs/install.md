# 설치 가이드

## Prerequisite

- git (checked version: 1.8.3.1)
- podman (checked version: v3.0.1) `or` Docker-ce (checked version 20.10.12)
- [helm](https://helm.sh/docs/intro/install/) (v3.8.0+)
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
      ./download.sh <download_dir> # ./download.sh ./downloads
      ```
   3. [Helm 클라이언트 다운로드](https://github.com/learncloud/install-helm-v3.0/) 
      

2. git repo 전체(다운로드한 이미지 포함)를 내부망으로 복사

3. 패키지 및 유틸리티 설치
   
   ```bash
   cd <copied_repo>
   sudo cp <helm_binary> /usr/local/bin
   sudo yum -y install git podman
   ```

4. 폐쇄망 환경에 레지스트리 설치 [참조](https://github.com/tmax-cloud/install-registry/tree/5.0)

5. 설치한 레지스트리에 이미지 업로드

   ```bash
   chmod +x ./upload.sh
   ./upload.sh <download_dir> <registry> # ./update.sh ./downloads 192.168.178.17:5000
   
   ```

### 2. 설치

1. values.yaml 수정
   
   - 이미지 레지스트리 소스 설정
     
      ```bash
      sed 's/__REPO__/<registry>/' ./values.yaml.tpl > ./values.yaml
      ```
        > public망에 접근가능한 환경에 설치할 경우 registry에 `tmaxcloudck` 입력

   - 인증서 설정 (별도로 생성한 서버 인증서를 사용할 경우)

       1. 인증서와 키를 담은 시크릿 생성
      ```bash
      kubectl create secret tls <core_secret_name> --cert=path/to/cert/core_cert_file --key=path/to/key/core_key_file
      kubectl create secret tls <notary_secret_name> --cert=path/to/cert/notary_cert_file --key=path/to/key/notary_key_file
      ```

       2. values.yaml 설정
       - **expose.tls.certSource**: `secret`
       - **expose.tls.secret.secretName**: `<core_secret_name>`
       - **expose.tls.secret.notarySecretName**: `<notary_secret_name>`
   
   - 도메인 설정

       1. Ingress를 사용할 경우
          1. 자동으로 자체 발급한 인증서를 사용할 경우 
             - **expose.type**: `ingress`
             - **expose.ingress.hosts.core**: `core.hr.<ingress_external_IP>.nip.io`
             - **expose.ingress.hosts.notray**: `notary.hr.<ingress_external_IP>.nip.io`
             - **expose.ingress.annotations**: `kubernetes.io/ingress.class: "nginx-shd"`
             - **externalURL**: `https://core.hr.<ingress_external_IP>.nip.io`
           
          2. 외부에서 발행한 인증서를 사용할 경우
             - **expose.type**: `ingress`
             - **expose.ingress.hosts.core**: `<core의 도메인이름> ex) core.hyperregistry.hypercloud.com`
             - **expose.ingress.hosts.notray**: `<notary의 도메인이름> ex) notary.hyperregistry.hypercloud.com`
             - **expose.ingress.annotations**: `kubernetes.io/ingress.class: "nginx-shd"`
             - **externalURL**: `https://<core의 도메인이름> ex) https://core.hyperregistry.hypercloud.com`
           
       2. LoadBalancer를 사용할 경우
          - **expose.type**: `loadBalancer`
          - **expose.tls.auto.commonName**: `loadBalancer의 externalIP`
          - **externalURL**: `https://loadBalancer의 externalIP`

       3. NodePort를 사용할 경우
          - **expose.type**: `nodePort`
          - **expose.tls.auto.commonName**: `cluster노드 IP 중 택일`
          - **externalURL**: `https://<expose.tls.commonName>:<expose.nodePort.ports.https.nodePort>`

   - 로그 레벨 설정
     - **loglevel**: `debug`, `info`, `warning`, `error`, `fatal` 중 1

   - 레지스트리 스토리지 용량 설정
     - **persistence.persistentVolumeClaim.registry.size**: `500Gi` (as big as your needs)
     
   - (Optional) [Database HA](https://github.com/tmax-cloud/HyperRegistry-Chart/blob/5.0/docs/postgres.md) 구성 시
     - **database.type** : `external`
     - **database.external**: 

   - (Optional) [Redis HA](https://github.com/tmax-cloud/HyperRegistry-Chart/blob/5.0/docs/redis.md) 구성 시
     - **redis.type** : `external`
     - **redis.external**: 
       - addr : pod의 주소:26379 (ex. 10.244.166.186:26379,10.244.33.190:26379,10.244.135.46:26379)
       - sentinelMasterSet: "mymaster"
       - password: kubectl get secret --namespace default redis-ha -o jsonpath="{.data.redis-password}" | base64 --decode 의 값

   - (Optional) [Kraken](https://github.com/tmax-cloud/HyperRegistry-Chart/blob/5.0/docs/kraken.md) 구성 시
     - **registry.notifications.url**: `http://proxy.kraken.172.22.11.2.nip.io/registry/notification`

2. 스토리지클래스 설정

   ```bash
   kubectl get storageclass
   NAME                    PROVISIONER                       RECLAIMPOLICY      VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE              AGE
   rook-cephfs (default)   rook-ceph.cephfs.csi.ceph.com     DELETE             Immediate           true                   1d
   ```

   cf. 기본 스토리지클래스가 없을 경우 새로 지정하기

   ```bash
   kubectl patch storageclass rook-cephfs -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
   ```

3. 배포

   ```bash
   helm install <release_name> . -n <namespace> --create-namespace
   ```

## Trouble-Shoting

- x509: certificate signed by unknown authority

  1.  crio 재시작

      ```bash
      systemctl restart crio
      ```
