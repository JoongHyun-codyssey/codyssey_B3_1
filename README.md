# AWS 기반 웹 서비스 인프라 구축

> 작성 중인 제출용 초안입니다. `[입력]`을 실제 값으로 수정하고, 스크린샷을 지정된 경로에 추가한 뒤 실제 검증한 항목만 완료 처리합니다. 아래 이미지 경로는 촬영할 증빙의 자리이며, 아직 실제 결과가 첨부된 상태는 아닙니다.

## 1. 프로젝트 소개

AWS에서 VPC 기반의 격리된 네트워크를 구성하고, Public Subnet에 EC2와 Nginx를 배포하여 외부에서 접속 가능한 웹 서비스를 구축하는 실습입니다.

보안 그룹으로 네트워크 접근을 제한하고, IAM 최소권한으로 AWS 리소스 관리 권한을 제한합니다. 통신 또는 권한 오류는 증상과 로그를 근거로 분석하며, 실습 종료 후 생성한 리소스를 정리합니다.

### 실습 정보

| 항목 | 구성 / 기록 |
| --- | --- |
| 작성자 | [입력] |
| 실습 일시 | [입력, KST] |
| 리전 | 서울 `ap-northeast-2` |
| IAM 사용자 또는 Role | [입력, 루트 계정 사용 안 함] |
| VPC / CIDR | [입력] / 예: `10.0.0.0/16` |
| Public Subnet / CIDR | [입력] / 예: `10.0.1.0/24` |
| 가용 영역 | [입력] |
| EC2 인스턴스 ID / 유형 | [입력] / [계정의 프리 티어 적용 여부를 확인한 micro 유형] |
| 운영체제 | [Ubuntu LTS 또는 Amazon Linux의 실제 버전 입력] |
| EBS | [유형 및 크기 입력, 8~10 GiB 수준] |
| 웹 서버 | Nginx [버전 입력] |
| 키페어 이름 | [입력, 개인키 파일은 제출하지 않음] |
| 퍼블릭 IPv4 | [입력] |
| 외부 접속 검증 방식 | A — 브라우저에서 `http://<퍼블릭IP>` 접속 |
| 현재 리소스 상태 | [실습 중 / 정리 완료] |

※ CIDR은 설계 예시입니다. 실제 구성에 맞게 수정합니다. 프리 티어 적용 여부와 사용량은 본인 계정에서 확인하고, 확인 일시 및 내용을 기록합니다: [입력].

## 2. 아키텍처

![VPC, Public Subnet, Internet Gateway, EC2, Security Group 및 외부 요청 흐름](/docs/architecture.png)

> 제출 전 실제 구성에 맞는 `docs/architecture.png`를 추가합니다. PDF로 제출한다면 위 이미지를 `[아키텍처 다이어그램](docs/architecture.pdf)` 링크로 변경합니다.

### 구성 요소의 역할

| 구성 요소 | 역할 및 확인 사항 |
| --- | --- |
| VPC | 서비스에 사용할 논리적으로 격리된 네트워크 범위 |
| Public Subnet | EC2가 배치되는 VPC 내부 IP 대역. 연결된 Route Table에 Internet Gateway 경로 구성 |
| Route Table | Subnet 트래픽의 목적지별 경로 결정. `0.0.0.0/0 → Internet Gateway` 확인 |
| Internet Gateway | VPC와 인터넷 간 통신 경로 제공 |
| EC2 | 퍼블릭 IPv4를 통해 접근하는 Nginx 실행 서버 |
| Security Group | EC2에 허용할 네트워크 트래픽 제한 |
| IAM | AWS API 및 콘솔에서 수행할 리소스 관리 작업의 권한 제한 |

외부 HTTP 요청은 EC2의 퍼블릭 IPv4를 목적지로 Internet Gateway를 통해 들어오며, 보안 그룹에서 TCP 80을 허용하면 EC2의 Nginx가 처리합니다. 응답과 인터넷 아웃바운드 통신에는 Subnet에 연결된 Route Table의 Internet Gateway 경로가 사용됩니다.

다이어그램에는 VPC/Subnet 경계, EC2의 퍼블릭 IPv4, EC2에 연결된 보안 그룹, Route Table과 Subnet 연결, Internet Gateway와 VPC 연결을 표시합니다. HTTP 80 요청 흐름과 개인 IP에서 들어오는 SSH 22 관리 흐름도 구분합니다.

## 3. 기능 요구사항별 구현 및 스크린샷

모든 스크린샷은 실제 실습 결과를 촬영합니다. 리전·대상 리소스·관련 설정이나 명령 결과가 식별되도록 촬영하며, 비밀번호·액세스 키·개인키는 포함하지 않습니다. 하나의 화면으로 증명이 부족하면 같은 항목에 이미지를 추가합니다.

### 3.1. 네트워크 구성

**구성 내용**

- VPC 1개와 Public Subnet 1개 생성
- Internet Gateway를 해당 VPC에 연결
- Public Subnet에 연결된 Route Table에 `0.0.0.0/0 → Internet Gateway` 설정
- EC2에서 외부 HTTPS 요청 수행

| 확인 항목 | 실제 값 / 결과 |
| --- | --- |
| VPC ID / CIDR | [입력] |
| Subnet ID / CIDR / VPC ID | [입력] |
| Internet Gateway ID / 연결된 VPC | [입력] |
| Route Table ID / 연결된 Subnet | [입력] |
| 기본 경로의 대상 / 상태 | [Internet Gateway ID / 실제 상태 입력] |
| 인터넷 아웃바운드 요청 | [HTTP 상태 코드 및 검증 일시 입력] |

**증빙 ① VPC와 Subnet 구성**

VPC의 CIDR, Subnet의 CIDR 및 소속 VPC를 확인할 수 있는 화면입니다.

![VPC 구성 결과](docs/screenshots/01-vpc.png)

![Public Subnet 구성 결과](docs/screenshots/02-subnet.png)

**증빙 ② Internet Gateway 연결 및 라우팅**

Internet Gateway의 연결 상태와 대상 VPC, Route Table의 기본 경로 및 Subnet 연결을 확인합니다. 기본 경로가 다른 Subnet의 Route Table에 설정된 것은 아닌지도 확인합니다.

![Internet Gateway 연결 결과](docs/screenshots/03-internet-gateway.png)

![Route Table 기본 경로](docs/screenshots/04-route-table.png)

![Route Table과 Public Subnet 연결](docs/screenshots/05-subnet-association.png)

**증빙 ③ EC2의 인터넷 아웃바운드 통신**

EC2에 SSH로 접속한 터미널에서 실행합니다.

```bash
curl -I --connect-timeout 10 https://example.com
```

기대 결과: 외부 서버의 정상 HTTP 응답 수신. 아래에 실제 명령과 응답이 보이는 화면을 첨부합니다.

![EC2에서 외부 HTTPS 요청 결과](docs/screenshots/06-outbound.png)

### 3.2. 컴퓨트 및 웹 서버 배포

**구성 내용**

- Public Subnet에 EC2 1대 생성 및 퍼블릭 IPv4 할당
- 개인 키페어를 이용한 SSH 접속
- Nginx 설치 및 실행
- EC2 내부에서 `http://localhost`의 200 응답 확인

| 확인 항목 | 실제 값 / 결과 |
| --- | --- |
| EC2 ID / 인스턴스 유형 | [입력] |
| VPC / Subnet ID | [입력] |
| 퍼블릭 IPv4 | [입력] |
| AMI / OS 버전 | [입력] |
| EBS 볼륨 ID / 크기 | [입력] |
| SSH 접속 결과 | [입력] |
| Nginx 실행 상태 | [입력] |
| localhost HTTP 상태 코드 | [입력] |

**증빙 ① EC2 배치 및 스토리지 설정**

실행 상태, 인스턴스 유형, VPC/Subnet ID, 퍼블릭 IPv4가 보이는 상세 화면과 EBS 용량을 첨부합니다.

![EC2 상세 정보](docs/screenshots/07-ec2.png)

![EBS 용량 및 연결 정보](docs/screenshots/08-ebs.png)

**증빙 ② SSH 접속**

다음 명령은 로컬 PC에서 실행합니다. `<...>`는 실제 값으로 치환합니다. 기본 사용자명은 선택한 AMI에서 확인합니다(일반적인 예: Ubuntu는 `ubuntu`, Amazon Linux는 `ec2-user`).

```bash
chmod 400 <키파일.pem>
ssh -i <키파일.pem> <사용자명>@<퍼블릭IP>
```

![SSH 접속 성공 화면](docs/screenshots/09-ssh.png)

**증빙 ③ Nginx 실행 및 내부 HTTP 응답**

EC2 내부에서 실행합니다.

```bash
nginx -v
systemctl is-active nginx
curl -i --connect-timeout 10 http://localhost
```

기대 결과: Nginx 버전, `active`, `HTTP/1.1 200 OK` 및 응답 본문 확인.

![Nginx 실행 상태와 localhost 200 응답](docs/screenshots/10-nginx-localhost.png)

### 3.3. 접근 제어 — Security Group

**인바운드 설정**

| 용도 | 프로토콜 | 포트 | 허용 소스 |
| --- | --- | --- | --- |
| 웹 서비스 | TCP | 80 | `0.0.0.0/0` |
| SSH 관리 접속 | TCP | 22 | `[개인 공인 IPv4]/32` 또는 과제에서 지정한 IP 대역 |

- 보안 그룹 ID: [입력]
- 보안 그룹이 연결된 EC2 ID: [입력]
- SSH 허용 소스의 실제 값: [입력]
- 아웃바운드 규칙 및 설정 이유: [입력]
- 전체 포트 `0–65535`를 `0.0.0.0/0`에 허용하는 인바운드 규칙 존재 여부: [없음 확인 후 입력]

**증빙 ① 전체 인바운드 규칙**

HTTP 80의 전체 허용과 SSH 22의 소스 제한을 확인합니다. 불필요한 규칙이 없는지 확인할 수 있도록 전체 규칙을 촬영합니다.

![보안 그룹 인바운드 규칙](docs/screenshots/11-sg-inbound.png)

**증빙 ② EC2 연결 및 아웃바운드 규칙**

규칙을 설정한 보안 그룹이 실제 EC2에 연결되어 있는지 확인합니다. EC2에 여러 그룹이 연결되어 있다면 모든 그룹의 허용 규칙을 함께 확인합니다.

![EC2에 연결된 보안 그룹](docs/screenshots/12-sg-association.png)

![보안 그룹 아웃바운드 규칙](docs/screenshots/13-sg-outbound.png)

### 3.4. IAM 최소권한

**적용 원칙**

- 별도 IAM 사용자 또는 Role로 실습 수행
- EC2/VPC/Security Group 등의 생성·조회·태그·연결·수정·정리에 필요한 작업만 허용
- 필요에 따라 리전, 리소스, 태그 조건으로 권한 범위 제한
- 실습과 무관한 S3/RDS 등 서비스 권한 부여하지 않음
- `AdministratorAccess` 부여하지 않음

| 확인 항목 | 실제 설정 |
| --- | --- |
| IAM 사용자 또는 Role 이름 | [입력] |
| 연결한 정책 이름 | [입력] |
| 허용한 작업과 이유 | [실제 Action과 실습 용도 입력] |
| 리소스 / 조건 제한 | [실제 Resource 및 Condition 입력] |
| `Resource: "*"`가 필요한 작업과 이유 | [사용했다면 입력, 없으면 해당 없음] |
| 사용자 직접 정책·그룹 정책 또는 Role 정책 검토 | [입력] |
| 관리자 및 무관 서비스 권한 부재 확인 | [입력] |

**실제 적용 정책**

아래 영역에 실제 적용한 IAM 정책 JSON을 붙여 넣습니다. 정책이 여러 개라면 모두 기록합니다.

```text
[실제 적용한 정책 JSON 입력]
```

**증빙 ① 사용 주체와 연결 정책**

IAM 사용자 또는 Role 이름과 연결된 정책을 촬영합니다. 사용자에게 그룹을 통해 부여된 권한이 있으면 그룹 정책도 함께 첨부합니다.

![IAM 사용자 또는 Role 및 연결 정책](docs/screenshots/14-iam-principal.png)

**증빙 ② 정책의 허용 범위**

정책의 Action, Resource, Condition을 읽을 수 있도록 촬영합니다. 정책 이름만으로는 최소권한 적용을 입증할 수 없으므로 정책 내용도 첨부합니다.

![IAM 정책 내용](docs/screenshots/15-iam-policy.png)

Security Group은 서버에 도달하는 네트워크 트래픽을 제어하고, IAM은 AWS 리소스를 조회·생성·삭제하는 주체의 권한을 제어합니다.

### 3.5. 외부 접속 검증

**선택 방식: A — 브라우저 접속**

| 항목 | 검증 기록 |
| --- | --- |
| 접속 URL | `http://[실제 퍼블릭IP]` |
| 검증 일시 | [입력, KST] |
| 접속 환경 | [로컬 PC 브라우저 등 입력] |
| 기대 결과 | Nginx 기본 페이지 또는 직접 작성한 웹 페이지 정상 표시 |
| 실제 결과 | [입력] |

EC2 내부의 `localhost`가 아닌, 외부 PC의 브라우저에서 퍼블릭 IPv4로 접속합니다. 주소창의 URL과 정상 표시된 페이지가 함께 보이도록 촬영합니다.

![외부 브라우저에서 웹 서비스 접속 결과](docs/screenshots/16-external-access.png)

> B 방식을 선택한다면 이 절을 `GET http://<퍼블릭IP>/health` 검증으로 변경합니다. `/health`가 실제로 200과 고정 응답을 반환하도록 구현한 뒤 외부 PC에서 `curl -i http://<퍼블릭IP>/health`를 실행하고 명령·상태 코드·본문을 촬영합니다. A 방식만 제출한다면 `/health` 구현은 필수가 아닙니다.

### 3.6. 운영 안정성 및 리소스 정리

생성 시점부터 리소스 ID를 기록하고, 접속 증빙을 확보한 뒤 실습 리소스를 정리합니다. 아래 표는 계획이며, 실제 확인 전에는 완료로 표시하지 않습니다.

| 정리 대상 | 리소스 ID / 미생성 여부 | 완료 기준 | 확인 일시 / 상태 | 증빙 |
| --- | --- | --- | --- | --- |
| EC2 | [입력] | `terminated` 확인 | [입력] | 17 |
| EBS | [입력] | 실습 볼륨 및 잔여 미사용 볼륨 삭제 확인 | [입력] | 18 |
| Elastic IP | [입력] | 할당했다면 Release, 미생성이면 할당 없음 확인 | [입력] | 19 |
| NAT Gateway | [입력] | 생성했다면 삭제, 미생성이면 없음 확인 | [입력] | [해당 시 추가] |
| ELB/ALB | [입력] | 생성했다면 삭제, 미생성이면 없음 확인 | [입력] | [해당 시 추가] |
| RDS | [입력] | 생성했다면 삭제 및 관련 잔여 리소스 확인 | [입력] | [해당 시 추가] |
| Internet Gateway | [입력] | VPC에서 분리 후 삭제 | [입력] | 20 |
| Subnet / 사용자 생성 Route Table / Security Group | [입력] | 의존 관계 해제 후 실습 리소스 삭제 | [입력] | [해당 화면 추가] |
| VPC | [입력] | 실습 VPC 삭제 | [입력] | 21 |
| 기타 실습 생성 리소스 | [키페어, IAM 정책 등 입력] | 사용 종료 및 정리 여부 기록 | [입력] | [해당 시 추가] |

EC2는 단순 `stopped` 상태가 아니라 `terminated` 상태를 확인합니다. 종료 후 EBS 삭제 여부를 별도로 확인하고, 할당한 Elastic IP가 있다면 해제합니다. 이후 네트워크 리소스의 의존 관계를 해제하고 VPC를 삭제합니다. 기존의 다른 프로젝트 리소스는 삭제하지 않습니다.

**정리 결과 스크린샷**

삭제 전 기록한 리소스 ID와 삭제 후 목록을 대조할 수 있도록 촬영합니다. 목록의 검색 필터와 리전을 확인하여 잘못된 필터 때문에 리소스가 없는 것처럼 보이지 않도록 합니다.

![EC2 종료 상태](docs/screenshots/17-cleanup-ec2.png)

![EBS 볼륨 정리 결과](docs/screenshots/18-cleanup-ebs.png)

![Elastic IP 해제 또는 미할당 확인](docs/screenshots/19-cleanup-eip.png)

![Internet Gateway 삭제 결과](docs/screenshots/20-cleanup-igw.png)

![VPC 삭제 결과](docs/screenshots/21-cleanup-vpc.png)

상세 정리 기록: [리소스 정리 체크리스트](docs/cleanup-checklist.md)

Billing 확인 일시 및 내용: [입력]. 비용 표시에는 지연이 있을 수 있으므로 Billing 화면만으로 삭제 완료를 판단하지 않고 각 리소스 상태를 함께 확인합니다.

리소스 정리 완료 후에는 위 서비스 URL로 접속할 수 없습니다. 외부 접속 성공 여부는 정리 전 촬영한 증빙과 검증 일시를 기준으로 확인합니다.

## 4. 트러블슈팅

상세 보고서: [트러블슈팅 보고서](docs/troubleshooting.md)

최소 1건의 실제 발생 또는 통제된 재현 사례를 작성합니다. 원인 확인을 위해 불필요한 포트를 전체 공개하지 않습니다.

| 단계 | 기록 내용 |
| --- | --- |
| 증상 | [요청 URL, 발생 시각, timeout/403/연결 거부 등 실제 증상] |
| 가설 | [증상의 원인으로 의심한 설정 또는 권한] |
| 검증 | [확인한 설정, 명령, 로그와 가설 판단 근거] |
| 조치 | [실제 수정한 항목과 변경 전후 값] |
| 결과 | [동일 요청 재시도 결과와 검증 시각] |
| 재발 방지 | [설정 점검 항목, 문서화 등 구체적인 예방 조치] |

오류 및 해결 후 화면을 각각 첨부합니다. 서버 요청 로그가 없다면 요청이 서버에 도달하지 않았을 가능성을 검토하되, 로그 부재만으로 원인을 확정하지 않습니다.

![트러블슈팅 오류 재현](docs/screenshots/22-troubleshooting-before.png)

![트러블슈팅 검증 및 해결 결과](docs/screenshots/23-troubleshooting-after.png)

## 5. 제출 파일 구성

| 경로 | 내용 |
| --- | --- |
| `README.md` | 실습 구성, 요구사항별 결과, 접속 정보 및 증빙 |
| `docs/architecture.png` 또는 `docs/architecture.pdf` | 실제 구축한 아키텍처 다이어그램 1장 |
| `docs/troubleshooting.md` 또는 `docs/troubleshooting.pdf` | 최소 1건의 오류 재현·분석·해결 보고서 |
| `docs/cleanup-checklist.md` | 리소스별 정리 상태 및 근거 |
| `docs/screenshots/` | 각 요구사항의 실제 검증 스크린샷 |

> 위 파일들은 제출할 산출물 목록입니다. 이 README 초안 외의 파일은 별도로 작성·촬영해야 합니다. 스크린샷 번호는 정리용이며, 필수 촬영 장수를 의미하지 않습니다. 필요한 정보가 명확하게 보이면 한 화면을 여러 요구사항의 증빙으로 활용할 수 있습니다.

## 6. 최종 제출 점검

- [ ] `[입력]` 및 예시 값을 실제 값으로 수정했다.
- [ ] 서울 리전에서 VPC 1개, Public Subnet 1개, EC2 1대를 구성했다.
- [ ] Internet Gateway 연결, Route Table 기본 경로, Subnet 연결을 확인했다.
- [ ] EC2에서 인터넷 아웃바운드 통신을 확인했다.
- [ ] SSH 접속, Nginx 실행 및 localhost의 200 응답을 확인했다.
- [ ] HTTP 80은 전체 공개, SSH 22는 개인 IP 또는 지정 대역으로 제한했다.
- [ ] 불필요한 전체 포트 공개 규칙이 없다.
- [ ] 루트 계정과 AdministratorAccess를 사용하지 않고 IAM 최소권한을 적용했다.
- [ ] 외부 접속 방식, 실제 URL, 검증 일시와 스크린샷을 기록했다.
- [ ] 각 기능 요구사항의 스크린샷을 추가하고 이미지 경로를 확인했다.
- [ ] 아키텍처 다이어그램을 PNG 또는 PDF로 제출했다.
- [ ] 트러블슈팅 보고서에 증상 → 가설 → 검증 → 조치 → 결과 → 재발 방지를 기록했다.
- [ ] EC2, EBS, Elastic IP, Internet Gateway, VPC 및 추가 생성 리소스를 정리했다.
- [ ] 리소스 정리 체크리스트에 완료 근거를 남겼다.
- [ ] 비밀번호·액세스 키·개인키 파일이 제출물에 포함되지 않았다.
