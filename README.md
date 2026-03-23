# High Availability WordPress Infrastructure

> Vagrant + VirtualBox 기반 고가용성 WordPress 인프라 구축 프로젝트

단일 서버 환경의 구조적 한계(하드웨어 장애, 네트워크 취약성, 트래픽 급증)를 극복하기 위해 **7대의 VM**으로 구성된 분산 인프라를 설계하고 구현했습니다.  
웹서버 이중화, DB 복제, 로드밸런싱, 네트워크 스토리지를 조합하여 **장애 발생 시에도 서비스가 중단되지 않는 환경**을 목표로 합니다.

---

## Architecture

<img width="884" height="574" alt="image" src="https://github.com/user-attachments/assets/85e02cfc-b7ce-434c-8db1-27836b11a782" />

---

## Server Inventory

| Hostname | IP | Role | Spec |
|----------|-----|------|------|
| `lb01` | 192.168.56.10 / .57.10 | HAProxy 로드밸런서 | 1 vCPU, 1GB |
| `web01` | 192.168.57.11 / .58.11 | Apache + WordPress | 2 vCPU, 1GB |
| `web02` | 192.168.57.12 / .58.12 | Apache + WordPress | 2 vCPU, 1GB |
| `nfs01` | 192.168.57.20 | NFS (소스코드 공유) | 1 vCPU, 1GB |
| `db01` | 192.168.58.13 | MySQL Primary | 2 vCPU, 1GB |
| `db02` | 192.168.58.14 | MySQL Secondary | 2 vCPU, 1GB |
| `storage01` | 192.168.58.20 | iSCSI Target | 1 vCPU, 1GB |

---

## Network Design

세 개의 Private Network를 계층별로 분리하여 보안성과 가용성을 동시에 확보했습니다.

| Subnet | 용도 | 통과 트래픽 |
|--------|------|-----------|
| `192.168.56.0/24` | 외부 접속용 | Client ↔ Load Balancer |
| `192.168.57.0/24` | 웹/NFS 서비스 | LB ↔ Web ↔ NFS |
| `192.168.58.0/24` | DB/스토리지 | Web ↔ DB ↔ iSCSI |

---

## Tech Stack

| 구분 | 기술 |
|------|------|
| 가상화 | VirtualBox 7.0+ / Vagrant 2.3+ |
| OS | Rocky Linux 9 |
| 웹서버 | Apache HTTP Server 2.4+ |
| CMS | WordPress 6.8.2 |
| DB | MySQL 8.0 (Primary-Secondary Replication) |
| 로드밸런서 | HAProxy 2.4+ (Round Robin) |
| 파일 공유 | NFS v4 |
| 블록 스토리지 | iSCSI (CHAP 1-way 인증) |

---

## Quick Start

### Prerequisites

- [VirtualBox](https://www.virtualbox.org/) 7.0+
- [Vagrant](https://www.vagrantup.com/) 2.3+

### VM 생성

```bash
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>

# 전체 VM 한번에 생성
vagrant up

# 개별 VM 접속
vagrant ssh lb01
vagrant ssh web01
# ...
```

> ⚠️ **Note**: `vagrant up`으로 VM만 생성됩니다. 각 서버의 소프트웨어 설치 및 설정은 아래 구성 가이드를 참고하여 수동으로 진행합니다.

### 수동 구성 순서

VM 생성 후, 아래 순서로 각 서버를 설정합니다.

1. **storage01** — iSCSI Target 구성 (targetcli, 백스토어, ACL, CHAP 인증)
2. **db01, db02** — iSCSI Initiator 연결 → 파일시스템 마운트 → MySQL 설치 및 Replication 설정
3. **nfs01** — NFS 서버 구성 → WordPress 다운로드 및 설정
4. **web01, web02** — Apache + PHP 설치 → NFS 마운트 → SELinux 설정
5. **lb01** — HAProxy 설치 및 백엔드 서버 등록

---

## Verification

### 기능 테스트

- **WordPress 접속**: `http://192.168.56.10` (LB 경유), `http://192.168.57.11` / `.12` (직접 접속)
- **로드밸런싱**: 두 웹서버의 `access_log`에 요청이 교대로 기록되는지 확인
- **DB 복제**: db02에서 `SHOW SLAVE STATUS\G` → `Slave_IO_Running: Yes`, `Slave_SQL_Running: Yes`

### 장애 대응 테스트

```bash
# web01의 Apache를 중지해도 서비스가 유지되는지 확인
vagrant ssh web01 -c "sudo systemctl stop httpd"

# 브라우저에서 http://192.168.56.10 접속 → 정상 동작 (web02로 자동 우회)
```

---

## Troubleshooting

프로젝트 진행 중 겪은 주요 이슈와 해결 과정입니다.

### iSCSI CHAP 인증 실패

- **증상**: db01에서 iSCSI Target 로그인 시 `non-retryable iSCSI login failure`
- **원인**: TPG의 `authentication` 속성이 비활성화(`no-auth`) 상태
- **해결**: `set attribute authentication=1`로 TPG 인증 활성화 후 CHAP 계정 재설정

### MySQL 복제 동기화 누락

- **증상**: db02에서 `USE WP;` 실행 시 `Unknown database 'WP'`
- **원인**: 복제 시작 시점 이전에 생성된 DB는 자동으로 동기화되지 않음
- **해결**: db02에서 수동으로 WP DB 생성 후 SQL Thread 재시작, 이후 정상 복제 확인

---

## Project Structure

```
.
├── Vagrantfile          # 7개 VM 정의 (네트워크, 리소스 설정)
└── README.md
```

---

## 향후 개선 계획

- DB 자동 Failover (MHA 또는 Group Replication)
- Monitoring 대시보드 구축 (Prometheus + Grafana)
- SSL/TLS 적용 (HTTPS)
- 컨테이너 기반 전환 (Docker / Kubernetes)
- Ansible 등을 활용한 프로비저닝 자동화
