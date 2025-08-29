# RHEL 8 NGINX 웹서버 구축 절차

## 1. 저장소(EPEL 또는 공식 NGINX) 추가
### EPEL 저장소 추가 (기본 nginx 패키지 제공)
```bash
sudo dnf install epel-release
```

### 또는 공식 NGINX 저장소 추가
```bash
sudo dnf install -y yum-utils
sudo yum-config-manager --add-repo https://nginx.org/packages/mainline/centos/8/x86_64/
sudo rpm --import https://nginx.org/keys/nginx_signing.key
```

## 2. NGINX 설치
```bash
sudo dnf install nginx
```
설치 후 nginx 버전 확인:
```bash
nginx -v
```

## 3. 서비스 활성화 및 시작
```bash
sudo systemctl enable --now nginx
sudo systemctl status nginx
```

## 4. 방화벽 설정 (HTTP/HTTPS 포트 개방)
```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

## 5. 기본 웹 페이지 접속 확인
웹 브라우저에서 `http://서버_IP/` 접속 후 기본 nginx 환영 페이지 확인

## 6. nginx.conf 주요 설정 파일 이해 및 편집
설정 파일 위치: `/etc/nginx/nginx.conf`

### 대표 내용 예시
```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log notice;
pid /run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    access_log /var/log/nginx/access.log main;
    sendfile on;
    keepalive_timeout 65;
    include /etc/nginx/conf.d/*.conf;
}
```

### 대표 server 블록 예시 (/etc/nginx/conf.d/default.conf 또는 직접 nginx.conf 내)
```nginx
server {
    listen 80;
    server_name example.com;

    root /usr/share/nginx/html;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
}
```

## 7. 설정 문법 검사 및 서비스 재시작
```bash
sudo nginx -t
sudo systemctl restart nginx
```

## 8. 운영 및 관리 주요 명령어
| 동작               | 명령어                             |
|------------------|----------------------------------|
| 서비스 상태 확인      | `sudo systemctl status nginx`     |
| 서비스 시작          | `sudo systemctl start nginx`      |
| 서비스 중지          | `sudo systemctl stop nginx`       |
| 서비스 재시작        | `sudo systemctl restart nginx`     |
| 설정 문법 검사       | `sudo nginx -t`                   |

위 절차대로 세팅하면 RHEL 8에서 nginx 웹서버를 안정적으로 설치 및 운영할 수 있습니다.
