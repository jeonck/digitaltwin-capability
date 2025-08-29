# Nucleus 설치 절차서

## 1. 환경 준비
- **서버 요구 사항**: Nucleus를 설치할 서버는 최소한의 하드웨어 요구 사항을 충족해야 합니다. CPU, 메모리, 저장 공간을 확인하십시오.
- **운영 체제**: Nucleus는 특정 운영 체제에서만 지원됩니다. 해당 운영 체제를 설치하십시오.

## 2. 포트 설정
### 인바운드 포트
- **웹 서버**: 
  - 80 (HTTP)
  - 443 (HTTPS)
  - 8080 (웹 UI)
- **기타 서비스 포트**:
  - 3009 (API)
  - 3019 (SSL API)
  - 3020 (태깅 서비스)
  - 3030 (대용량 파일 전송 서비스)
  - 3100 (인증 서비스)
  - 3180 (인증 로그인 폼)
  - 3333 (디스커버리 서비스)
  - 3400 (검색 서비스)

### 아웃바운드 포트
- 80 (HTTP)
- 443 (HTTPS)

## 3. 도커 환경 구성
1. 도커 키링 디렉토리 생성:
   ```bash
   sudo mkdir -p /etc/apt/keyrings
   ```

2. 도커 GPG 키 추가:
   ```bash
   sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
   ```

3. 도커 저장소 추가:
   ```bash
   sudo sh -c 'echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list'
   ```

4. 패키지 목록 업데이트:
   ```bash
   sudo apt-get update
   ```

5. 도커 설치:
   ```bash
   VERSION_STRING=<도커_버전>
   sudo apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-compose-plugin
   ```

## 4. 라이선스 포탈에서 패키지 확인
- NVIDIA 라이선스 포탈에서 필요한 패키지를 확인하고 다운로드합니다.
  - [NVIDIA 라이선스 포탈](https://ui.licensing.nvidia.com/software)

## 5. 설치 패키지 다운로드
- 위의 UI에서 Nucleus 설치 패키지를 다운로드합니다.

## 6. 패키지 전송
- 다운로드한 패키지를 VM으로 전송합니다.
  ```bash
  scp *.gz <사용자>@<서버_IP>:/home/<사용자>/
  ```

## 7. 압축 해제
- `/opt/ove` 디렉토리를 만들고 압축을 해제합니다.
  ```bash
  sudo mkdir -p /opt/ove
  sudo tar xzvf <패키지_파일명>.tar.gz -C /opt/ove --strip-components=1
  ```

## 8. 환경 변수 설정
- `.env` 파일을 편집하여 필요한 변수를 설정합니다.
  ```bash
  sudo nano /opt/ove/nucleus-stack.env
  ```

- EULA 및 보안 검토 설정:
  ```plaintext
  ACCEPT_EULA=1
  SECURITY_REVIEWED=1
  ```

- 서버 IP 또는 호스트 이름 설정:
  ```plaintext
  SERVER_IP_OR_HOST=<서버_IP_또는_호스트>
  ```

- Nucleus 비밀번호 설정:
  ```plaintext
  MASTER_PASSWORD=<비밀번호>
  SERVICE_PASSWORD=<비밀번호>
  ```

- 데이터 저장 위치 설정:
  ```plaintext
  DATA_ROOT=/var/lib/omni/nucleus-data
  ```

- 서브넷 설정:
  ```plaintext
  CONTAINER_SUBNET=192.168.2.0/26
  ```

## 9. Nucleus 설치
- Docker Compose를 사용하여 Nucleus를 설치합니다.
  ```bash
  sudo docker compose --env-file /opt/ove/nucleus-stack.env -f /opt/ove/nucleus-stack-no-ssl.yml pull
  sudo docker compose --env-file /opt/ove/nucleus-stack.env -f /opt/ove/nucleus-stack-no-ssl.yml up -d
  ```

## 10. Omniverse Navigator 접속
- 웹 브라우저를 통해 `http://<서버_IP>:8080`으로 접속하여 동작을 확인합니다.

## 11. Nucleus 서버 상태 로그 확인
- Nucleus 서비스의 상태 로그를 확인합니다.
  ```bash
  sudo docker compose --env-file /opt/ove/nucleus-stack.env -f /opt/ove/nucleus-stack-no-ssl.yml logs
  ```

## 12. Nucleus API 토큰 생성
- Nucleus API 토큰을 생성하여 필요한 인증을 수행합니다.

## 13. Nucleus 도구 설치
- Nucleus 도구를 설치합니다.
  ```bash
  sudo docker pull nvcr.io/nvidia/omniverse/nucleus-tools:<버전>
  ```

## 14. Nucleus 파일 목록 보기
- Nucleus에 연결하여 파일 목록을 확인합니다.
  ```bash
  sudo docker run --rm -e "ACCEPT_EULA=Y" $NUC_TOOLS ls -r <서버_IP> /Projects -u <사용자> -p <비밀번호>
  ```

## 15. USD 파일 업로드
- USD 파일을 Nucleus로 업로드합니다.
  ```bash
  sudo docker run --rm -v /home/<사용자>/usd-files:/files -e "ACCEPT_EULA=Y" $NUC_TOOLS upload /files <서버_IP> /Projects/<프로젝트_이름>/ -u <사용자> -p <비밀번호>
  ```

## 16. Kit 기반 앱 개발 후 Nucleus 연결 방법
- Nucleus의 3180 포트로 접속합니다.
  ```plaintext
  http://<서버_IP>:3180
  ```

