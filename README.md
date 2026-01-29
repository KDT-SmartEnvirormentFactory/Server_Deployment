# Server_Deployment

대제목 : AWS로 서버 구축
[EC2로 구축]
서버 이름 : emqx-server
퍼블릭 IPv4 주소 : 13.239.5.195
인스턴스 유형 : t3.small

[인바운드 보안 규칙]
22     → SSH
1883   → MQTT
8883   → MQTT TLS
18083  → EMQX Dashboard
1880   → Node-RED
8086   → InfluxDB
3000   → Grafana

[명령 프롬프트에서 SSH 접속 명령어]
ssh -i "C:\Users\hooni\Downloads\emqx-key.pem" ubuntu@13.239.5.195
 - 키를 연결
 - ubuntu + 퍼블릭 주소

[그 후에 Ubunut 서버에 설치할 거]
1. Ubuntu 기본 업데이트
2. Docker 설치
3. Docker Compose 설치
4. 작업용 폴더 생성
5. EMQX + Node-RED + InfluxDB + Grafana 한 번에 띄우기 (띄운다 = On해서 사이트로 접속이 가능하게 만들다)

1. Ubuntu 업데이트
```bash
sudo apt update && sudo apt upgrade -y
```

2. Docker 설치
```bash
sudo apt install -y ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

3. Docker 권한 설정
```bash
sudo usermod -aG docker ubuntu

exit
```
다시 접속
```bash
ssh -i "C:\Users\hooni\Downloads\emqx-key.pem" ubuntu@3.104.116.49
```

4. Docker 정상 동작 확인
```bash
docker --version
docker compose version
```

5. 프로젝트 폴더 생성
```bash
mkdir ~/iot-stack
cd ~/iot-stack
```

6. docker-compose.yml 생성
```bash
nano docker-compose.yml
# 하고 아래 전부 붙혀넣으면 됩니댱
version: "3.9"

services:
  emqx:
    image: emqx/emqx:5.5
    container_name: emqx
    restart: always
    ports:
      - "1883:1883"
      - "8883:8883"
      - "18083:18083"
    volumes:
      - emqx-data:/opt/emqx/data
      - emqx-log:/opt/emqx/log

  influxdb:
    image: influxdb:2.7
    container_name: influxdb
    restart: always
    ports:
      - "8086:8086"
    volumes:
      - influxdb-data:/var/lib/influxdb2

  grafana:
    image: grafana/grafana:10.2.3
    container_name: grafana
    restart: always
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana

  nodered:
    image: nodered/node-red:latest
    container_name: node-red
    restart: always
    ports:
      - "1880:1880"
    volumes:
      - nodered-data:/data

volumes:
  emqx-data:
  emqx-log:
  influxdb-data:
  grafana-data:
  nodered-data:
```

7. 모든 서비스를 한 번에 실행하는 명령어
```bash
docker compose up -d
```

8. 실행 확인
```bash
docker ps
```

기본계정
 - EMQX: admin / public
 - Grafana: admin / admin









