# AWS 웹 서비스 인프라 설명 자료

![서울 리전의 VPC, Public Subnet, Internet Gateway, Route Table, Security Group 및 Nginx 서버 구성](architecture.png)

이 문서는 프로젝트의 구성 요소와 핵심 검증 결과를 정리한 기술 설명 자료입니다. 실습은 **2026년 10월 5일, 서울 리전 `ap-northeast-2`**에서 진행됐습니다. 아키텍처와 접속 결과는 리소스를 정리하기 전 촬영한 구성을 기준으로 설명합니다.

관련 문서: [전체 구축 결과](../README.md) · [증빙 이미지 목록](evidence-index.md) · [IAM 작업 권한 설명](iam-permissions.md) · [IAM Action 목록](iam-actions.json) · [트러블슈팅 보고서](troubleshooting.md) · [리소스 정리 체크리스트](cleanup-checklist.md)

## 1. 구성 요소와 역할

| 구성 요소 | 역할 | 이번 실습에서 확인한 내용 |
| --- | --- | --- |
| VPC | AWS에서 사용할 논리적으로 분리된 네트워크 범위 | `aws-mission-vpc`, `10.0.0.0/16` |
| Public Subnet | EC2가 배치되는 VPC 내부 주소 범위 | `aws-mission-subnet`, `10.0.1.0/24`, `ap-northeast-2a` |
| Internet Gateway | VPC와 인터넷 간 통신 경로 제공 | 실습 VPC에 `Attached` |
| Route Table | 목적지에 따라 네트워크 경로 결정 | `0.0.0.0/0 → Internet Gateway`, 실습 Subnet에 명시적 연결 |
| Security Group | EC2에 허용할 네트워크 트래픽 결정 | HTTP 80 전체 허용, SSH 22 개인 IPv4 `/32` 제한 |
| EC2 | 운영체제와 Nginx가 실행되는 가상 서버 | `aws-mission-ec2`, `t3.micro`, Ubuntu 26.04 LTS |
| EBS | EC2의 운영체제와 파일을 저장하는 디스크 | 생성 설정 `gp3`, `8 GiB`; 실제 볼륨 ID 증거 없음 |
| Nginx | HTTP 요청을 받아 웹 페이지와 파일 제공 | 버전 `1.28.3`, 기본 페이지와 정적 `/health` 제공 |
| IAM | AWS 콘솔·API로 리소스를 관리하는 주체의 권한 제어 | 사용자·정책 연결은 스크린샷으로 확인, 실습 관리 Action은 사용자 제공 목록에 기록; 전체 권한 범위는 추가 확인 필요 |

## 2. 핵심 검증 증거

아래 표는 구성과 검증 결과를 확인할 수 있는 핵심 증거입니다. 전체 생성 전후 화면은 [증빙 목록](evidence-index.md)에 모아 두었습니다.

| 설명할 내용 | 화면 | 관찰할 부분 |
| --- | --- | --- |
| VPC 구성 | [VPC 생성 결과](screenshots/vpc/VPC_after.png) | 실습 VPC 이름과 `10.0.0.0/16` |
| Subnet 배치 | [Subnet 상세](screenshots/subnet/subnet_info.png) | CIDR, 가용 영역, 소속 VPC |
| 인터넷 경로 | [IGW 연결](screenshots/igw/IGW_after_after_connect.png), [기본 경로](screenshots/rt/rt_connect_igw.png), [Subnet 연결](screenshots/rt/rt_connect_subnet.png) | Attached 상태, 활성 경로, 실제 Subnet 연결 |
| 서버 구성 | [EC2 상세](screenshots/ec2/ec2_after_info.png), [EBS 생성 설정](screenshots/ec2/ec2_setting_volumn.png) | 인스턴스 유형·주소·배치, 생성 시 디스크 설정 |
| 접근 제어 | [EC2에 적용된 보안 그룹](screenshots/ec2/ec2_after_sg.png), [IAM 정책 연결](screenshots/iam/IAM_policy.png), [IAM 작업 권한 설명](iam-permissions.md) | HTTP·SSH 규칙, 관리 주체의 정책 연결, 사용자 제공 EC2 실습 관리 Action |
| SSH와 OS | [SSH 접속](screenshots/ssh-install_nginx_connect/aws-connect-terminal.png), [운영체제 확인](screenshots/ssh-install_nginx_connect/ssh-connect.png) | 로그인 성공, Ubuntu 26.04 LTS |
| Nginx 내부 동작 | [Nginx 실행 상태](screenshots/ssh-install_nginx_connect/ssh-nginx-active.png), [localhost 응답](screenshots/ssh-install_nginx_connect/ssh-web-response.png) | active, `200 OK`, 버전 `1.28.3` |
| 인터넷 아웃바운드 | [외부 HTTPS 응답](screenshots/ssh-install_nginx_connect/ssh-outbound.png) | `https://example.com`, `HTTP/2 200` |
| 외부 브라우저 접속 | [Nginx 페이지](screenshots/ssh-install_nginx_connect/web-connect-public.png) | 퍼블릭 IPv4 주소창과 정상 페이지 |
| 정적 health 파일 | [localhost health 응답](screenshots/ssh-install_nginx_connect/ssh-%3Alocalhost%3Ahealth-ok.png), [EC2발 퍼블릭 IP health 응답](screenshots/ssh-install_nginx_connect/ssh-%3ApublicIP%3Aheatlh-ok.png) | 파일 작성 명령, 요청을 실행한 위치, `200 OK`와 `OK` |
| 장애 원인과 조치 | [localhost 연결 실패](screenshots/troubleshooting/troubleshooting-localhost-no-connect.png), [정지 로그](screenshots/troubleshooting/troubleshooting-service-stop-log.png), [Nginx 재시작 결과](screenshots/troubleshooting/troubleshooting-recovered.png) | 내부 요청 실패, 서비스 정지, active 복구 범위 |
| 실습 후 정리 | [EC2 종료 진행](screenshots/cleanup/ec2-close.png), [EBS 목록](screenshots/cleanup/cleanup-ebs.png), [VPC 삭제](screenshots/cleanup/vpc-delete.png) | 종료 중과 최종 종료 구분, 볼륨 없음, 삭제 완료 |

Subnet 상세 화면에는 생성 초기의 기본 Route Table이 표시됩니다. **최종 아키텍처의 Route Table 연결은 이후 촬영한 [명시적 Subnet 연결 화면](screenshots/rt/rt_connect_subnet.png)을 기준으로 설명합니다.**
