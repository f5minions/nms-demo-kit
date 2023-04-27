알림: 현재 시점 기준으로 ARM 프로세스를 지원하는 것은 NIM 모듈 뿐이며, 다른 모듈(ADM, ACM, SM)은 ARM 프로세스를 지원하지 않습니다.

## 이 문서는 UDF랩 환경을 기준으로 하기 때문에 명령 부분이 조금 다를 수 있습니다.

## 원본
이 repository는 훌륭한 [Fabrizio](https://github.com/fabriziofiorucci) at https://github.com/nginxinc/NGINX-Demos/tree/master/nginx-nms-docker를 복제하여 작성된 내용 입니다.

## 요약

이 데모 킷은 NGINX 관리 모듈을 쉽게 사용하고 활용할 수 있으며, 관리 모듈의 다양한 기능(API Connectivity Manager - ACM, NGINX Instance Manager - NIM, Security Monitoring - SM)과 NGINX Plus의 로드밸런서 및 API 게이트웨이 그리고 K8s Ingress Controller를 모두 함께 사용할 수 있습니다.

이 데모에서는 하나의 LB를 프론트엔드에 배치하고 이를 통해 내부의 K8s 클러스터에 구성된 API 게이트웨이로 트래픽을 전달하여 최종적으로 API 엔드포인트를 엑세스할 수 있습니다.

![alt text](assets/demo-setup-part-1.png)

## 사전 준비사항: 
- docker-compose를 desktop에 설치
- [NGINX 공식웹사이트](https://www.nginx.com/pricing/)에서 NGINX Plus 및 NGINX Management Suite에 대한 Trial 라이센스를 요청하여 준비


## 시작하기
0. **UDF 랩 환경 접속하기
UDF 랩 시작 후 Ubuntu 머신의 엑세스 > VSCODE를 통해 vscode 앱에 엑세스 합니다.
![alt text](assets/access_vscode.png)

VSCODE 앱을 실행 후 터미널을 실행
![alt text](assets/launch_terminal_in_vscode.png)



1. **NMS 시작**
```
# 아래 주소를 이용해서 github 리파지토리를 복제
git clone https://github.com/f5minions/nms-demo-kit

cd nms-demo-kit

# 스크립트 파일에 실행 권한을 부여
chmod 755 ./scripts/*

# NGINX Plus trial 라이센스를 다운받은 후 nginx-repo.crt와 nginx-repo.key 파일을 nginx-plus 폴더에 복사
cp nginx-repo.* nginx-plus/

# NMS (NGINX Instance Manager, API Connectivity Manager) 컨테이너 이미지를 빌드
# 최신 릴리스 버전을 기준으로 컨테이터 이미지 빌드 예시
sudo ./scripts/buildNMS.sh -t nginx-nms -i -C nginx-plus/nginx-repo.crt -K nginx-plus/nginx-repo.key -A -W

# 또는 원하는 버전을 명시하여 컨테이너 이미지를 빌드
# Dockerfile을 수정을 통해 원하는 이미지 버전 패키지를 명시하면 됩니다
# sudo ./scripts/buildNMS.sh -t nginx-nms:2.6 -C nginx-plus/nginx-repo.crt -K nginx-plus/nginx-repo.key -A -W

# NMS 컨테이너 배포하기
# 만약 MacOS 환경에서 진행을 한다면 5000번 포트는 AirPlay Sharing 모듈에 의해 이미 예약되어 있어 사용이 불가능하기 때문에 다른 포트를 명시에서 사용해야 합니다. 
# 포트의 변경은 docker-compose.yaml 파일에서 예시와 같이 변경을 할 수 있습니다. 예) "6000:5000"

sudo docker compose -f docker-compose.yaml up -d
```

2. **배포 후 NMS GUI 접속**

NMS 배포 후 (배포 및 구동에 약간의 시간이 소요됨) UDF의 Ubuntu 머신의 HTTP-443 엑세스를 클릭하면 GUI 화면에서 admin/admin 계정 정보로 로그인을 할 수 있습니다. 초기 docker-compose.yaml 파일에서 admin:admin으로 비밀번호를 설정하지 않고 임의의 비밀번호를 사용하였다면 해당 계정 및 비밀번호를 통해서 로그인을 할 수 있습니다. 

![alt text](assets/access_nms_vis_http_443.png)
![alt text](assets/nms_prelogin_landing_page.png)
![alt text](assets/nms_login_prompt.png)


NMS 화면 로그인 후 Settings(설정) 메뉴를 클릭하여 라이센스를 업로드하여 NMS를 활성화 합니다.
![alt text](assets/nms-license.png)

업로드 후 브라우저의 새로고침 버튼을 통해 새로고침을 하면 정상적으로 NMS 메뉴가 보입니다.

3. **NGINX Plus 시작하기**
```
# nginx-agent가 포함된 NGINX Plus 이미지 빌드
# NMS의 IP 주소를 입력 합니다. localhost 또는 127.0.0.1 주소를 사용하지 않고 시스템(또는 Labtop)의 IP 주소를 입력 합니다. 
sudo ./scripts/buildNPlusWithAgent.sh -t npluswithagent -n https://10.1.1.6

# nginx-agent가 포함된 NGINX Plus(ACM Dev Portal용) 이미지를 빌드
# NMS의 IP 주소를 입력 합니다. localhost 또는 127.0.0.1 주소를 사용하지 않고 시스템(또는 Labtop)의 IP 주소를 입력 합니다. 
sudo ./scripts/buildNPlusWithAgent.sh -t npluswithagent:devportal -D -n https://10.1.1.6

# docker-compose.yaml에서 nginx-lb, nginx-gw, httpbin-app, acm.nginx-devportal에 Uncomment nginx-lb, nginx-gw, httpbin-app, acm.nginx-devportal에 설정된 주석을 제거하고 docker-compose를 다시 실행하면 업데이트된 내용이 반영 됩니다.
sudo docker compose -f docker-compose.yaml up -d
```
아래의 화면과 같은 수의 컨테이너가 실행 중이어야 정상 입니다
![alt text](assets/running-containers-new.png)

NMS Instance Manager 대시보드에서 새로고침을 했을 때 아래의 화면과 같이 인스턴스가 정상적으로 표시가 되어야 합니다
![alt text](assets/nim-managed-instances.png)


4. **NIM을 통해 Loadbalancer의 구성하기**

NGINX LB에 대한 NGINX API 사용 활성화

- Instance Manager의 Instances 매뉴에서 nginx-lb 인스턴스를 클릭하고, edit config 메뉴를 클릭하여 새파일 /etc/nginx/conf.d/nplusapi.conf 파일을 추가 합니다. repo misc/nplusapi.conf파일에서 해당 내용을 복사할 수 있습니다. 
![alt text](assets/edit-nginx-plus-api-conf.png)

- 기존 default.conf 설정을 repo의 misc/lb.conf 파일 설정으로 변경 합니다.
![alt text](assets/edit-nginx-plus-default-conf.png)

- 설정이 모두 완료되면 publish를 클릭하여 nginx-lb 인스턴스에 업데이트를 합니다.

LB 설정 테스트
```
curl -I http://localhost/
curl -I http://localhost/
```

Backend 서버의 IP 주소를 반환하도록 NGINX 로드밸런서를 설정했기 때문에, 여러 Backend 서버(여기서는 Backend에 구성한 NGINX 게이트웨이)가 있으며, NGINX 로드밸런서는 각각의 서버로 라운드로빈으로 분산하고 각각의 IP주소를 반환 합니다. Backend 컨테이너 서버의 IP 주소는 아래와 표시된 것과 다를 수 있습니다.

![alt text](assets/lb-curl-testing.png)


1. **ACM을 통해 NGINX API Gateway 설정**

팁!: 5번째 단계에서 스크립트를 사용하여 ACM 구성을 자동화하도록 선택할 수 있습니다. 하지만 ACM 워크플로에 익숙해지기 위해 아래 단계를 수동으로 실행하는 것을 권장 합니다. 
```
#sh misc/end2end_deploy.sh
#sh misc/end2end_delete.sh
```

API Gateway 클러스터 활성화

- API Connectivity Manager에서 Infrastructure에서 Workspace와 Environment를 생성하고 Environment 메뉴에서 API Gateway Cluster를 추가 합니다. API Gateway Cluster의 이름은 받드시 Instance Group의 이름과 동일하게 설정을 해야 정상적으로 연결 설정이 됩니다. 이 예제에서는 "gwcluster"라는 이름으로 생성을 하였습니다. 

![alt text](assets/infra-workspace.png)
![alt text](assets/infra-env.png)

- 위 설정이 완료되었다면, API Gateway 메뉴에서 Instance 부분에 2개의 nginx-gw 인스턴스가 연결된 것을 확인할 수 있습니다.
![alt text](assets/infra-creation-complete.png)

선택사항:
nginx-gw 게이트웨이를 추가로 생성하여 spin-up을 할 경우 추가되는 nginx-gw 인스턴스는 자동으로 Instance Group으로 연결되고 해당 그룹에 자동으로 설정이 추가 됩니다. 이 부분은 nginx-gw replicas를 변경해서 확인할 수 있습니다. (아래 예시 참조)
```
# docker-compose 파일의 nginx-gw 부분에서 replicas를 3으로 변경 후 docker-compose 재실행 
sudo docker compose -f docker-compose.yaml up -d
```

- API Connectivity Manager의 "Services" 메뉴에서 서비스에 해당하는 Workspace를 생성하여 해당 Workspace에 API Proxy를 배포 합니다.
```    
Name: <anything>
Service Target Hostname: httpbin-app
Use an OpenAPI spec: Yes
Upload repo misc/httpbin-oas.yaml
Gateway Proxy Hostname: Select the gateway proxy that you have created
```

![alt text](assets/service-workspace.png)
![alt text](assets/service-upload-api-spec.png)
![alt text](assets/service-api-proxy.png)


- In Advanced Configuration, inisde Ingress section change the following fields and then Click Save and Publish
```
Append Rule: None
Strip Base Path and Version before proxying the request: Yes
```
![alt text](assets/service-edit-ingress.png)


Test the ACM setting
```
#You may refer to the misc/httpbin-oas.yml for other possible httpbin endpoints
curl -v localhost/get
curl -v localhost/headers
```

You may try to configure different policies in ACM and see how it work.

6. **Configure Dev Portal using ACM**

Enable Dev Portal Cluster
- In the previously created API Connectivity Manager->Infrastructure->Workspace->Environment. Inside the environment, Create Developer Portal Cluster.
The Name of the Dev Portal Clusters must be same with the instance group name you specify for the acm.nginx-devportal, in this case "devportal".

![alt text](assets/create-devportal-cluster.png)

- In the previously created API Connectivity Manager->Services-API Proxies, Tick "Also publish API to developer portal"

![alt text](assets/publish-devportal.png)

- In UDF, you create additional Access method ubuntu server for port 90 and click access.

![alt text](assets/udf-access-http90.png)

![alt text](assets/sample-devportal.png)

## Bonus
Instead of using NGINX Plus as LB, you may use NGINX App Protect (NAP) as LB + WAF to protect the API endpoints.
At the time of this writing, it is possible to manage NAP policies via NMS in VM setup but not in docker container. Hence, we will just manually create NAP policies in NGINX NAP instance and then manage the nginx.conf via NMS.

![alt text](assets/demo-setup-part-2.png)

```
#Build NGINX App Protect image with nginx-agent
#Specify NMS IP address (your laptop IP address), DO NOT use localhost or 127.0.0.1
sudo ./scripts/buildNAPWithAgent.sh -t napwithagent -n https://10.1.1.6

#Uncomment nginx-nap section in docker-compose.yaml section
sudo docker compose -f docker-compose.yaml up -d
```
NGINX NAP takes slightly more time to start up, wait for it.

Configure NAP in Instance Manager

- In Instance Manager section, Instances tab, click on nginx-nap and Edit Config, replace nginx.conf with repo misc/nap_lb.conf
![alt text](assets/edit-nap-conf.png)

- Click Publish

```
#Send traffic to NAP
curl localhost:83/get

#Violation
curl "localhost:83/get?<script>"
curl "localhost:83/get?username=1'%20or%20'1'%20=%20'1'))%20LIMIT%201/*&amp;password=foo"
```

After generating some violations, head over to NMS Security Monitoring dashboard to view the report.
![alt text](assets/nms-security-monitoring.png)

