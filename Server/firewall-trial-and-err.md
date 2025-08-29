🔐 **RHEL 신규 서버 방화벽(firewalld) 구축/운영 체크리스트**

| 구분         | 항목               | 명령어 / 방법                                                       | 주의사항                          |
|--------------|--------------------|--------------------------------------------------------------------|-----------------------------------|
| 기본 원칙    | 보안 정책          | Deny by default → 필요한 서비스만 허용                           | SSH(22) 먼저 확인 후 진행        |
| 설치/활성화  | firewalld 설치     | `sudo dnf install firewalld -y`                                   | 최소 설치 시 필요                 |
|              | 서비스 활성화      | `sudo systemctl enable --now firewalld`                           | --now로 즉시 시작                 |
|              | 상태 확인          | `sudo firewall-cmd --state`                                       | running이어야 함                  |
| 존(zone) 관리 | 기본 존 확인       | `sudo firewall-cmd --get-default-zone`                            | 일반적으로 public                 |
|              | 인터페이스 확인/지정 | `sudo firewall-cmd --get-active-zones`<br>`sudo firewall-cmd --zone=public --change-interface=eth0 --permanent` | 한 인터페이스 = 한 존             |
|              | 적용               | `sudo firewall-cmd --reload`                                      | 변경 후 항상 reload               |
| 서비스/포트 개방 | HTTP/HTTPS        | `sudo firewall-cmd --zone=public --add-service=http --permanent`<br>`sudo firewall-cmd --zone=public --add-service=https --permanent` | 최소 권한 원칙 적용               |
|              | 포트 개방          | `sudo firewall-cmd --zone=public --add-port=3306/tcp --permanent` | TCP/UDP 지정 필수                 |
| 적용 시 안전 절차 | 테스트            | tmux / screen 세션에서 적용 후 다른 세션으로 접속 확인          | SSH 막힘 방지                     |
| 로그/모니터링 | 차단 로그 활성화   | `sudo firewall-cmd --set-log-denied=all`                         | 과도한 로그에 주의                |
|              | 로그 확인          | `journalctl -xe | grep firewalld`                                |                                   |
| 백업/복구    | 설정 백업          | `sudo cp -r /etc/firewalld /backup/firewalld_$(date +%F)`       | 변경 전 항상 백업                 |
|              | 전체 재적용        | `sudo firewall-cmd --complete-reload`                             | 꼬이면 백업에서 복구             |
| 흔한 실수     | --permanent 누락   | 재부팅 후 규칙 사라짐                                            | 항상 --permanent + reload         |
|              | iptables와 동시 실행 | 충돌 발생                                                        | `sudo systemctl stop iptables && sudo systemctl mask iptables` |
|              | 잘못된 존 할당     | 내부망 노출 가능                                                 | --get-active-zones로 재확인      |
|              | 로그 미비          | 원인 파악 불가                                                   | 로그 활성화 필수                  |

✅ **최종 운영 팁**

- "작게 적용 → 테스트 → 영구 반영" 순서 지킬 것
- SSH 접근 가능성 확보 후 설정 변경
- Ansible 등으로 자동화하면 운영 안정성 ↑

---

### 1. firewalld 설치 및 기본 설정

RHEL 신규 설치 시 firewalld는 기본적으로 설치되어 있습니다. 최소 설치 옵션을 선택한 경우 별도로 설치해야 합니다. 아래는 CLI(명령줄 인터페이스)를 통해 진행하는 과정입니다.

#### 단계 1: firewalld 설치 확인 및 설치
서버에 firewalld가 설치되어 있는지 확인:  
```bash
sudo firewall-cmd --version
```
버전 정보가 출력되지 않으면 설치 필요.  
설치:  
```bash
sudo dnf install firewalld  # RHEL 8+에서 dnf 사용, RHEL 7은 yum
```

#### 단계 2: firewalld 서비스 시작 및 활성화
서비스 시작:  
```bash
sudo systemctl start firewalld
```
부팅 시 자동 시작:  
```bash
sudo systemctl enable firewalld
```
상태 확인:  
```bash
sudo systemctl status firewalld  # 또는 sudo firewall-cmd --state (running으로 표시되어야 함)
```

#### 단계 3: 기본 구성 확인
기본 존 확인:  
```bash
sudo firewall-cmd --get-default-zone  # 보통 public
```
활성 존 목록:  
```bash
sudo firewall-cmd --get-active-zones
```
인터페이스 할당: 네트워크 인터페이스(예: eth0)를 존에 할당.  
```bash
sudo firewall-cmd --zone=public --change-interface=eth0 --permanent
```
재로드:  
```bash
sudo firewall-cmd --reload
```
이 단계에서 기본적으로 모든 인바운드 트래픽은 차단되며, SSH(포트 22)만 허용되는 경우가 많습니다. 신규 서버의 경우, 최소 권한 원칙(least privilege)을 적용해 불필요한 포트를 열지 않도록 합니다.

---

### 2. 상세 설정: 존, 서비스, 포트, 규칙 관리

firewalld는 존(zone) 기반으로 규칙을 관리합니다. 각 존은 신뢰 수준에 따라 트래픽을 분류하며, public(기본, 외부용), internal(내부 네트워크), dmz(노출 서버) 등이 있습니다.

#### 존 관리
존 목록 확인:  
```bash
sudo firewall-cmd --get-zones
```
새 존 생성:  
```bash
sudo firewall-cmd --new-zone=custom --permanent
```
인터페이스나 소스 IP를 존에 추가:  
```bash
sudo firewall-cmd --zone=internal --add-source=192.168.1.0/24 --permanent
```

#### 서비스 및 포트 허용
사용 가능한 서비스 목록:  
```bash
sudo firewall-cmd --get-services  # (http, https, ssh 등)
```
서비스 추가: 웹 서버 예시.  
```bash
sudo firewall-cmd --zone=public --add-service=http --permanent
sudo firewall-cmd --zone=public --add-service=https --permanent
```
포트 직접 추가:  
```bash
sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent  # TCP/UDP 지정 필수
```
규칙 제거:  
```bash
sudo firewall-cmd --zone=public --remove-service=http --permanent
```

#### 고급 규칙: 리치 룰(Rich Rules)
복잡한 규칙을 위해 사용. 예: 특정 IP만 허용.  
```bash
sudo firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.100" port protocol="tcp" port="3306" accept' --permanent
```

#### 정책 및 masquerading
아웃바운드 정책: 기본적으로 허용되지만, 정책으로 제한 가능.  
```bash
sudo firewall-cmd --new-policy=outbound --permanent  # RHEL 8+에서 정책 객체 사용
```
NAT/Masquerading: 내부 네트워크 숨김.  
```bash
sudo firewall-cmd --zone=external --add-masquerade --permanent
```

#### 로그 및 모니터링
로그 활성화:  
```bash
sudo firewall-cmd --set-log-denied=all  # 거부된 트래픽 로그
```
로그 확인:  
```bash
/var/log/firewalld.log  # 또는 journalctl
```
GUI 도구(firewall-config)나 웹 콘솔(cockpit)도 사용할 수 있지만, 신규 서버에서는 CLI가 안정적입니다.

---

### 3. 흔히 실수하는 사례

firewalld 설정 시 초보자가 자주 저지르는 실수가 있으며, 이는 보안 취약점이나 서비스 중단을 초래합니다. 아래는 실제 사례 기반으로 정리한 주요 실수입니다.

- **실수 1: --permanent 옵션 누락**  
  문제: `firewall-cmd --add-service=http`처럼 영구 옵션 없이 규칙 추가 시, 재부팅 후 사라짐.  
  영향: 서버 재시작 후 서비스 접근 불가, 다운타임 발생.  
  예방: 항상 --permanent 사용 후 --reload.

- **실수 2: iptables와 firewalld 충돌**  
  문제: 기존 iptables 규칙이 남아 있거나, 동시에 실행 시 충돌 (예: Virtualmin 등에서 발생).  
  영향: 방화벽 규칙이 예상대로 작동하지 않음.  
  예방: iptables 서비스 중지 및 마스킹:  
  ```bash
  sudo systemctl stop iptables
  sudo systemctl mask iptables
  ```

- **실수 3: 잘못된 존 할당 또는 다중 존 혼란**  
  문제: 인터페이스를 잘못된 존에 할당하거나, 여러 존이 중복 적용됨. 예: public 존에 내부 IP 허용.  
  영향: 보안 홀(불필요한 포트 노출) 또는 접근 차단.  
  예방: `--get-active-zones`로 확인하고, 하나의 인터페이스당 하나의 존만 할당.

- **실수 4: 재부팅 후 설정 유지 실패**  
  문제: 구성 파일 업데이트 중 오류나 firewalld 재시작 실패로 기본 구성으로 복귀.  
  영향: 모든 규칙 초기화, 보안 취약.  
  예방: 변경 후 `--complete-reload` 사용하고, 백업:  
  ```bash
  sudo cp -r /etc/firewalld /backup
  ```

- **실수 5: 로그 설정 미비 또는 과도한 규칙**  
  문제: 로그를 활성화하지 않아 차단 원인 파악 어려움, 또는 너무 많은 규칙으로 성능 저하.  
  영향: 디버깅 지연, CPU 부하 증가.  
  예방: 최소 규칙 유지, `journalctl`로 모니터링.

기타: 포트 프로토콜(TCP/UDP) 지정 누락, 테스트 후 영구 적용 잊음 등.

---

안정적인 설정을 위한 최선의 관행  
RHEL 신규 서버에서 firewalld를 설정할 때는 'deny by default' 원칙을 기억하세요. 변경 시 테스트 환경에서 검증하세요. 자동화 스크립트나 Ansible을 활용하면 실수를 줄일 수 있습니다. 문제가 발생하면 Red Hat 지원이나 공식 문서를 참조하세요. 이러한 접근으로 서버의 보안과 가용성을 동시에 확보할 수 있습니다.
[1] https://www.techtarget.com/searchdatacenter/tip/A-few-ways-to-configure-Linux-firewalld
[2] https://www.servermon.kr/m/board.html?code=servermon_board2&page=7&type=v&board_cate=&num1=999664&num2=00000&number=318&lock=N&srsltid=AfmBOooCa4L5Q3wHVUKFUhRQ--CZ9v3B30zOn7xD9i0gq9S8PtouaEH5
[3] https://meyouus.tistory.com/221
[4] https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/using-and-configuring-firewalld_configuring-and-managing-networking
[5] https://gogumawang.tistory.com/40
[6] https://forums.rockylinux.org/t/firewalld-best-practices-for-zone-based-configuration/19079
[7] https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/7/html/security_guide/bh-stopping_firewalld
[8] https://www.reddit.com/r/redhat/comments/fke9sj/question_need_help_with_firewalld_issue/?tl=ko
[9] https://serverfault.com/questions/674874/is-there-a-way-to-run-just-save-with-firewalld-in-rhel7
[10] https://lesstif.atlassian.net/wiki/spaces/WS/pages/891093653/firewalld
[11] https://linuxconfig.org/how-to-stop-start-and-disable-enable-firewall-on-redhat-7-linux-system
[12] https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/9/html-single/configuring_firewalls_and_packet_filters/index
[13] https://tmjb.tistory.com/37
[14] https://wlsvud84.tistory.com/entry/Centos7RHEL-%EB%A6%AC%EB%88%85%EC%8A%A4-%EB%B0%A9%ED%99%94%EB%B2%BD-%EC%84%A4%EC%A0%95-%ED%8F%AC%ED%8A%B8-%EC%97%B4%EA%B8%B0-%ED%94%84%ED%86%A0-%ED%99%95%EC%9D%B8
[15] https://jong-bae.tistory.com/22
[16] https://serverfault.com/questions/1159080/firewalld-adding-a-source-ip-blocks-ssh-different-behaviour-between-versions
[17] https://bamdule.tistory.com/63
[18] https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/8/html/system_design_guide/using-and-configuring-firewalld_system-design-guide
[19] https://www.youstable.com/ko/blog/fix-firewalld-on-linux/
[20] https://heni.tistory.com/24
[21] https://www.reddit.com/r/ansible/comments/g8o88s/firewalld_enforcement_of_infrastructure_as_code/
[22] https://servermon.tistory.com/578
[23] https://93it-serverengineer.co.kr/2
[24] https://yongeekd01.tistory.com/50
[25] https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/using-and-configuring-firewalld_configuring-and-managing-networking
[26] https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/security_guide/bh-stopping_firewalld
[27] https://www.youstable.com/blog/fix-firewalld-on-linux/
[28] https://www.tecmint.com/configure-firewalld-rhel-rocky-almalinux/
[29] https://www.cyberciti.biz/faq/configure-set-up-a-firewall-using-firewalld-on-rhel-8/
[30] https://www.lisenet.com/2016/firewalld-rich-and-direct-rules-setup-rhel-7-server-as-a-router/
[31] https://forum.virtualmin.com/t/firewall-iptables-and-firewalld-conflict/58278
[32] https://www.reddit.com/r/redhat/comments/fke9sj/question_need_help_with_firewalld_issue/
[33] https://serverfault.com/questions/696182/debugging-iptables-and-common-firewall-pitfalls
[34] https://www.redhat.com/en/blog/automating-firewall-configuration-rhel-system-roles
