# nginx-poly-layers

These manual is written in Korean. We wish you understand kindly.  
こちらの説明書は韓国語で作成させていただきました。ご了承お願いします。

[Contact to Jason](wjsuk@ideatec.co.kr) *EN, JP OK*


## 개요  

내부에 Nginx alpine 컨테이너를 4개소 품은 rockylinux:8.7 기반 Nginx 컨테이너입니다.  

컨테이너 내에 Nginx를 컴파일 설치하였으며, 해당 Nginx 엔진은 내장된 컨테이너 4대로 로드밸런싱 및 리버스 프록싱을 실시하기 위해 구동됩니다.  
리버스 프록싱은 라운드로빈 규칙을 따라 이뤄집니다.  

## 설치되어 있는 것

1. Nginx (컴파일 설치)
2. Docker, Docker compose
3. 컨테이너 속 컨테이너를 빌드하기 위한 yaml 파일 두 개: `/docker-compose.yaml` 및 `/docker-build-cp.yaml`이 마련되어 있습니다.  
4. `/docker-build-cp.yaml` 파일이 nginx 이미지를 변형해 빌드하기 위해 마련한 도커 파일 4점: `/Dockerfile.0[0-3]` 으로 하나씩 마련되어 있습니다. 컨테이너 내에서 개별 컨테이너를 하나씩 이미지로 빌드하시는 것도 가능합니다. 
5. 그 외 앞 파일들을 설치하기 위한 의존성 패키지 다수: `dnf`를 통해 설치하였습니다.  

## 실행 방법  

### 파워셸에서

※사전에 Docker Desktop을 설치해 두셔야 합니다.  

```Powershell
docker run --name ${CONTAINER_NAME} -p ${HOST_PORT}:80 -dt --privileged ghcr.io/wjsuk/nginx-poly-layers /usr/bin/dockerd  
docker exec ${CONTAINER_NAME} /usr/local/nginx/sbin/nginx 
## 단순 컨테이너 속 컨테이너 구현을 확인하고 싶다면  
docker exec -d ${CONTAINER_NAME} docker-compose -p ${PROJECT_NAME} up
## 컨테이너 속 컨테이너를 로드밸런싱 및 리버스 프록싱을 받아 접근하는 것을 확인하고 싶다면  
docker exec -d ${CONTAINER_NAME} docker-compose -p ${PROJECT_NAME} -f docker-build-cp.yaml up   
```

사용하시는 분께서 직접 정해서 쓰실 수 있는 변수 부분은 다음과 같습니다.
- CONTAINER_NAME: 컨테이너 이름으로 쓰실 이름을 정해서 채워 적습니다.
- HOST_PORT: 이 컨테이너의 로드밸런서를 받을 호스트 인스턴스(즉 사용하고 계신 PC)의 tcp 포트를 적습니다. 80, 443 포트 등은 웹 서버에서 상용하는 포트이므로, 충돌을 방지하기 위해 20000-65535 범위 내의 숫자를 정해 적습니다.  
- PROJECT_NAME: 컨테이너 속 컨테이너를 일괄 실행시킬 때 필요한 구분자입니다. 컨테이너는 이 `${PROJECT_NAME}`-`${yaml 파일 서비스 문단에서 정의한 컨테이너 이름}`-`일련번호(※기본적으로 '1')` 순서대로 이름을 부여받습니다.  

### Docker Desktop에서

해당 환경에서는 컨테이너 Run시 실행오류를 막을 수 있는 옵션을 설정 불가하므로 파워셸을 이용 부탁드립니다. 😥  
하지만 파워셸 이용시 생성결과 확인은 가능하십니다.  

실제 접속 결과 확인은 `http://localhost:${HOST_PORT}` 에서 가능합니다.  
※`docker-compose` 명령으로 인용한 yaml파일이 `docker-build-cp.yaml`인 경우, 불러오는 HTML 파일의 첫 주어가 `Asashio-the nameship-`·`Oshio`·`Michishio`·`Arashio` 순서대로 바뀝니다.  

## 혹시 컨테이너 속 컨테이너의 상태를 직접 확인하고 싶으시다면

### 파워셸에서
파워셸의 경우 다음과 같이 입력합니다.  
보여주는 결과값은 두 명령 모두 대체로 동일하므로 원하시는 명령을 골라 쓰시면 됩니다.  

```Shell
docker exec ${CONTAINER_NAME} docker ps
```
```Shell
docker exec ${CONTAINER_NAME} docker-compose -p ${PROJECT_NAME} ps
```

### Docker Desktop에서
Docker Desktop에서는 `Containers`>`${CONTAINER_NAME}`>`Terminal` 에 들어가셔서 다음과 같이 확인이 가능합니다.  
보여주는 결과값은 두 명령 모두 대체로 동일하므로 원하시는 명령을 골라 쓰시면 됩니다.  

```Shell
docker ps
```
```Shell
docker-compose -p ${PROJECT_NAME} ps
```  