

# Nucleus 설치 절차서

## 1. 환경 준비
- **서버 요구 사항**: Nucleus를 설치할 서버는 최소한의 하드웨어 요구 사항을 충족해야 합니다. CPU, 메모리, 저장 공간을 확인하십시오.
- **운영 체제**: Nucleus는 특정 운영 체제에서만 지원됩니다. 해당 운영 체제를 설치하십시오.

## 2. 포트 설정
- **인바운드 포트**:
  - 8080, 80, 443: 웹 서버
  - 3009, 3019, 3020, 3030, 3100, 3180, 3333, 3400: 기타 서비스 포트

- **아웃바운드 포트**:
  - 80, 443: 기본 아웃바운드 포트

## 3. 환경 변수 설정
- `.env` 파일을 생성하고 다음과 같은 필수 변수를 설정합니다.

```plaintext
# 서버 IP 또는 호스트 이름
SERVER_IP_OR_HOST=<서버_IP_또는_호스트>

# SSL 설정 (SSL을 사용하는 경우)
SSL_INGRESS_HOST=<SSL_호스트>
SSL_INGRESS_PORT=443

# 인스턴스 이름
INSTANCE_NAME=<인스턴스_이름>

# 마스터 비밀번호
MASTER_PASSWORD=<비밀번호>

# 서비스 비밀번호
SERVICE_PASSWORD=<비밀번호>
```

## 4. SSL 설정
- SSL을 사용하는 경우, SSL 인증서와 키를 설정합니다.
```plaintext
SSL_CERT=<SSL_인증서_경로>
SSL_CERT_KEY=<SSL_키_경로>
```

## 5. SSO 설정 (선택 사항)
- SSO 통합이 필요한 경우, SSO 관련 변수를 설정합니다.
```plaintext
SSO_GW_ADDRESS=<SSO_게이트웨이_주소>
USE_SAML_SSO=0  # SAML SSO 사용 여부
```

## 6. 데이터 저장소 설정
- Nucleus 데이터가 저장될 경로를 설정합니다.
```plaintext
DATA_ROOT=<데이터_저장_경로>
```

## 7. Nucleus 설치
- Docker Compose를 사용하여 Nucleus를 설치합니다.
```bash
docker-compose -f nucleus-stack.yml up -d
```

## 8. NGINX 설정 (선택 사항)
- NGINX를 Ingress Router로 사용하는 경우, NGINX 설정 파일을 수정하여 Nucleus 서비스에 대한 프록시를 설정합니다.

```nginx
server {
    listen 80;
    server_name <서버_도메인>;

    location / {
        return 302 https://<서버_도메인>$request_uri;
    }
}

server {
    listen 443 ssl;
    server_name <서버_도메인>;

    ssl_certificate /etc/ssl/certs/<인증서_파일>;
    ssl_certificate_key /etc/ssl/private/<키_파일>;

    location / {
        proxy_pass http://localhost:<포트>;
        proxy_set_header Host $host;
    }
}
```

## 9. 서비스 시작
- Nucleus 서비스가 정상적으로 시작되었는지 확인합니다.
```bash
docker ps
```

## 10. 보안 검토
- 설치 후 보안 설정을 검토하고 필요한 경우 추가적인 보안 조치를 취합니다.

---
