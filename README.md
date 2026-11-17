# Target IP Guard

Ubuntu Server에서 서비스 포트에 접근할 수 있는 출발지 IP를 관리하는 CLI 방화벽 도구입니다. nftables를 사용하며 IPv4와 IPv6를 지원합니다.

## 다운로드

- [저장소 ZIP 다운로드](https://github.com/shmasd/linuxcliprotecttool/archive/refs/heads/main.zip)
- [저장소 보기](https://github.com/shmasd/linuxcliprotecttool)

ZIP을 서버에서 다운로드하고 압축을 푼 뒤 설치하세요. Git을 사용한다면 `git clone https://github.com/shmasd/linuxcliprotecttool.git`으로 받을 수 있습니다.

## 설치와 첫 실행

```bash
cd linuxcliprotecttool-main
sudo ./install.sh
```

설치 스크립트가 Ubuntu의 `nftables` 패키지와 CLI, systemd 유닛을 설치합니다. **설치만으로 방화벽 규칙을 적용하거나 부팅 서비스를 켜지는 않습니다.** SSH 연결을 유지할 수 있도록 관리자 IP를 먼저 허용 목록에 넣고 실제 SSH 포트를 설정한 다음 규칙을 적용하세요.

```bash
# 아래 주소를 실제 관리자 공인 IP 또는 신뢰할 CIDR로 바꾸세요.
sudo target-ip-guard allow add 198.51.100.24/32

# 서버에서 사용하는 SSH 포트를 포함해 보호할 TCP/UDP 포트를 지정합니다.
sudo target-ip-guard ports --tcp 22,443 --udp 53

# 메뉴를 열어 상태와 규칙을 확인합니다.
sudo target-ip-guard

# 새 SSH 연결이 되는지 확인한 뒤 재부팅 시 자동 적용을 활성화합니다.
sudo systemctl enable target-ip-guard.service
```

`allow add`와 `ports`는 저장한 설정을 즉시 적용합니다. SSH 포트나 관리자 IP를 잘못 지정하면 새 SSH 연결이 차단될 수 있습니다. 현재 접속을 유지한 채 다른 터미널에서 새 연결을 확인한 다음 종료하세요. 클라우드 보안 그룹이나 네트워크 ACL도 별도로 확인해야 합니다.

## 메뉴 화면

```text
╭──────────────────────────────────────────────────────╮
│  TARGET IP GUARD                         Ubuntu CLI  │
├──────────────────────────────────────────────────────┤
│  보호 포트  TCP 22,443            UDP 53              │
│  허용 네트워크  1                                    │
├──────────────────────────────────────────────────────┤
│  1  허용 IP/CIDR 추가        2  허용 IP/CIDR 삭제     │
│  3  보호 포트 설정           4  방화벽 상태 보기      │
│  5  차단 접속 기록           6  관리자 조치 기록      │
│  7  방화벽 규칙 다시 적용    0  종료                  │
╰──────────────────────────────────────────────────────╯
선택 >
```

인자 없이 `sudo target-ip-guard`를 실행하면 메뉴가 열립니다. `sudo target-ip-guard menu`로도 실행할 수 있습니다. 포트 설정을 바꿀 때는 적용 전에 확인을 한 번 더 받습니다.

## 명령어

```text
target-ip-guard                         # 메뉴 열기
target-ip-guard allow add IP_OR_CIDR    # IP/CIDR 허용
target-ip-guard allow remove IP_OR_CIDR # 허용 취소
target-ip-guard allow list              # 허용 목록 보기
target-ip-guard ports --tcp 22,443 [--udp 53]
target-ip-guard apply                   # 저장된 규칙 재적용
target-ip-guard status                  # 상태와 nftables 규칙 보기
target-ip-guard logs [--lines 50]       # 차단 접속 기록
target-ip-guard actions [--lines 50]    # 관리자 변경 기록
```

모든 관리 명령은 `sudo`로 실행합니다. `ports`는 기존 TCP와 UDP 목록을 모두 교체하므로 적용할 전체 포트를 입력하세요. 허용 주소와 CIDR은 IPv4 및 IPv6 형식을 지원합니다.

## 기록과 동작 범위

- 지정된 서비스 포트에서 허용 목록에 없는 패킷은 커널 journal에 기록한 뒤 차단합니다. 로그는 규칙별 분당 6건으로 제한됩니다.
- 차단 기록은 `sudo target-ip-guard logs`에서 확인합니다. 커널 로그에 출발지 주소와 목적지 포트가 표시됩니다.
- 허용 목록이나 포트 변경은 `/var/log/target-ip-guard/actions.jsonl`에 JSONL로 기록됩니다.
- 도구는 `inet target_ip_guard` nftables 테이블만 관리합니다. 그 외 포트와 방화벽 테이블은 일반 방화벽 규칙에 맡깁니다.
- 이 도구는 호스트 전체 방화벽이나 클라우드 네트워크 ACL을 대체하지 않습니다.

## 요구 사항

- Ubuntu Server
- Python 3
- nftables (설치 스크립트가 Ubuntu 패키지로 설치)

Python 외부 패키지는 필요하지 않습니다.

## 라이선스

MIT. 자세한 내용은 [LICENSE](LICENSE)를 확인하세요.
