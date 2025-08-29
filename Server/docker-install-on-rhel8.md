📌 RHEL 8 / CentOS 8 기준 Docker 설치 절차 (최신 방식)

1. **기존 Podman / Buildah 등 충돌 패키지 제거**  
   RHEL 8 기본 컨테이너 툴(Podman, Buildah 등)과 Docker가 충돌할 수 있으므로 우선 제거합니다.  
   ```bash
   sudo dnf remove -y podman buildah
   ```

2. **패키지 및 유틸리티 업데이트**  
   ```bash
   sudo dnf update -y
   sudo dnf install -y yum-utils device-mapper-persistent-data lvm2
   ```
   - `yum-utils`: `yum-config-manager` 같은 명령어 제공
   - `device-mapper-persistent-data`, `lvm2`: Docker 스토리지 드라이버(requirement) 관련

3. **Docker CE 공식 저장소 추가**  
   ```bash
   sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
   ```
   ⚠️ RHEL 8도 CentOS 리포지토리를 사용합니다. 공식적으로 Docker는 Red Hat 계열에 대해 CentOS 저장소를 제공하기 때문입니다.  
   저장소 활성화 확인:  
   ```bash
   sudo dnf repolist
   ```

4. **Docker CE 및 관련 패키지 설치**  
   ```bash
   sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```
   - `docker-ce`: Docker 엔진
   - `docker-ce-cli`: Docker CLI
   - `containerd.io`: 컨테이너 런타임
   - `docker-buildx-plugin`, `docker-compose-plugin`: 최신 Docker에서 빌드/Compose 기능  
   ✅ `docker-compose`는 별도 바이너리가 아니라 플러그인으로 제공되며 최신 방식은 `docker compose` (하이픈 없음) 명령어로 사용합니다.

5. **Docker 서비스 시작 및 부팅 시 자동 실행 설정**  
   ```bash
   sudo systemctl enable --now docker
   ```  
   상태 확인:  
   ```bash
   sudo systemctl status docker
   ```

6. **일반 사용자 권한 부여 (선택)**  
   root 없이 Docker 실행하려면:  
   ```bash
   sudo usermod -aG docker $USER
   ```  
   ➡️ 적용을 위해 로그아웃/재로그인 필요

7. **설치 확인**  
   ```bash
   docker --version
   docker compose version
   docker run hello-world
   ```  
   정상 동작하면 "Hello from Docker!" 메시지를 확인할 수 있습니다.
