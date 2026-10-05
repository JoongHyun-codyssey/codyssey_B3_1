# 실습 증빙 이미지 목록

[README](../README.md)에서 구축 결과를 확인하고, [프로젝트 구성 설명](project-explanation.md)에서 구성 요소와 핵심 증거를 확인할 수 있습니다. 아래 링크는 `docs/screenshots/`에 저장된 원본 이미지를 단계별로 모은 것입니다.

생성·편집 설정 화면은 설정 과정의 기록입니다. 실제 적용 여부는 결과 화면과 실행 결과를 함께 확인합니다. 이 목록은 촬영 당시의 기록이며 현재 AWS 리소스 상태를 조회한 결과는 아닙니다.

## 1. IAM 사용자와 리전

| 단계 | 증빙 | 확인 내용 및 범위 |
| --- | --- | --- |
| 사용자 생성 | [IAM 사용자 목록](screenshots/iam/IAM_user.png) | 별도 IAM 사용자 `aws-mission-IAM`이 표시됩니다. |
| 콘솔 로그인 | [IAM 사용자 로그인](screenshots/iam/IAM_logined.png) | 콘솔 상단에 `aws-mission-IAM`이 표시됩니다. |
| 리전 선택 | [서울 리전 선택](screenshots/iam/IAM_seoul.png) | 서울 `ap-northeast-2` 선택 화면입니다. 정책의 리전 제한 조건을 증명하는 화면은 아닙니다. |
| 정책 연결 | [IAM 사용자와 연결 정책](screenshots/iam/IAM_policy.png) | 고객 인라인 정책 `aws-mission-policy` 1개가 연결된 화면입니다. 이 스크린샷은 정책 연결을 증명하며 실제 정책 요소를 보여주지는 않습니다. |
| 사용자 제공 작업 목록 | [IAM 작업 권한 설명](iam-permissions.md), [IAM Action 목록 JSON](iam-actions.json) | 사용자가 직접 제공한 Action 목록입니다. EC2 실습의 조회, 네트워크 생성·연결·삭제, SG 설정, 키페어, EC2 실행·운영·종료, EBS 삭제와 태그 작업의 용도를 정리했습니다. 스크린샷에서 추출한 목록은 아닙니다. |

IAM 사용자와 정책 연결은 스크린샷에서 확인하고, 구체적인 Action은 사용자 제공 목록을 기준으로 기록했습니다. 목록에는 `ec2:Describe*`가 포함되므로 모든 조회 작업을 개별적으로 최소화한 정책이라고 단정하지 않습니다.

`Effect`, `Resource`, `Condition` 및 다른 권한 부여 경로는 제공되지 않아 허용·거부, 리소스 범위와 전체 권한 범위는 추가 확인 대상입니다. `Condition`은 선택 항목이며 실제 사용 여부는 미확인입니다. 직접 제공된 Action은 EC2 관련 작업이며, 다른 정책을 통한 S3·RDS 관리 권한이나 `AdministratorAccess` 연결 여부까지 보여주지는 않습니다.

## 2. VPC와 Subnet

| 단계 | 증빙 | 확인 내용 및 범위 |
| --- | --- | --- |
| VPC 구성 전 | [VPC 구성 전 목록](screenshots/vpc/VPC_before.png) | 실습 VPC 구성 전 화면입니다. |
| VPC 결과 | [VPC 구성 결과](screenshots/vpc/VPC_after.png) | 실습 VPC 구성 결과 화면입니다. |
| Subnet 구성 전 | [Subnet 구성 전 목록](screenshots/subnet/Subnet_before.png) | 실습 Subnet 구성 전 화면입니다. |
| Subnet 결과 | [Subnet 구성 결과](screenshots/subnet/subnet_after.png) | 실습 Subnet 구성 결과 화면입니다. |
| Subnet 상세 | [Subnet 상세 정보](screenshots/subnet/subnet_info.png) | Subnet의 상세 설정을 확인하는 화면입니다. |

## 3. Internet Gateway와 Route Table

| 단계 | 증빙 | 확인 내용 및 범위 |
| --- | --- | --- |
| Gateway 구성 전 | [Internet Gateway 구성 전](screenshots/igw/IGW_before.png) | 실습 Internet Gateway 구성 전 화면입니다. |
| Gateway 생성 | [Internet Gateway 연결 전](screenshots/igw/IGW_after_before_connect.png) | Internet Gateway 생성 후 VPC 연결 전 화면입니다. |
| Gateway 연결 | [Internet Gateway 연결 결과](screenshots/igw/IGW_after_after_connect.png) | Internet Gateway와 VPC의 연결 결과 화면입니다. |
| Route Table 구성 전 | [Route Table 구성 전](screenshots/rt/rt_before.png) | 실습 Route Table 구성 전 화면입니다. |
| Route Table 결과 | [Route Table 구성 결과](screenshots/rt/rt_after.png) | 실습 Route Table 구성 결과 화면입니다. |
| 기본 경로 설정 | [Internet Gateway 경로 설정](screenshots/rt/rt_connect_igw.png) | Internet Gateway를 대상으로 하는 라우팅 설정 화면입니다. |
| Subnet 연결 | [Route Table과 Subnet 연결](screenshots/rt/rt_connect_subnet.png) | Route Table과 실습 Subnet의 연결 화면입니다. |

## 4. Security Group

| 단계 | 증빙 | 확인 내용 및 범위 |
| --- | --- | --- |
| 구성 전 | [Security Group 구성 전](screenshots/sg/sg_before.png) | 실습 Security Group 생성 전 목록입니다. |
| 인바운드 생성 설정 | [HTTP·SSH 인바운드 설정](screenshots/sg/sg_setting_inbound.png) | 생성 폼에 TCP 80 → `0.0.0.0/0`, TCP 22 → `180.224.196.41/32`가 입력되어 있습니다. |
| 아웃바운드 편집 설정 | [아웃바운드 설정](screenshots/sg/sg_setting_outbound.png) | 편집 폼에 모든 트래픽 → `0.0.0.0/0`가 표시됩니다. |
| 최종 적용 및 EC2 연결 | [EC2에 적용된 Security Group과 규칙](screenshots/ec2/ec2_after_sg.png) | EC2 `i-054eeabf8b19481d9`에 `aws-mission-sg` (`sg-04bf96b2f2a29e49a`)가 연결되어 있으며 위 인바운드 2개와 전체 IPv4 아웃바운드 규칙을 확인할 수 있습니다. |

`ec2_after_sg.png`가 실제 EC2 연결 및 최종 규칙의 주 증빙입니다. 촬영 당시 표시된 인바운드 규칙에는 전체 포트를 인터넷에 공개하는 규칙이 없고, SSH는 개인 공인 IPv4 1개로 제한되어 있습니다.

## 5. EC2 생성과 배치

| 단계 | 증빙 | 확인 내용 및 범위 |
| --- | --- | --- |
| 구성 전 | [EC2 구성 전](screenshots/ec2/ec2_before.png) | 실습 EC2 생성 전 화면입니다. |
| OS·이미지 선택 | [EC2 OS와 AMI 설정](screenshots/ec2/ec2_setting_os_image.png) | EC2 생성 과정의 OS·이미지 선택 화면입니다. |
| 인스턴스·키페어 선택 | [EC2 인스턴스와 키페어 설정](screenshots/ec2/ec2_setting_instance_keypair.png) | 인스턴스 유형과 키페어 선택 화면입니다. |
| 네트워크 선택 | [EC2 네트워크 설정](screenshots/ec2/ec2_setting_network.png) | VPC·Subnet·보안 그룹 등 생성 시 네트워크 설정 화면입니다. |
| 볼륨 설정 | [EC2 볼륨 생성 설정](screenshots/ec2/ec2_setting_volumn.png) | 생성 시 볼륨 설정 화면입니다. 최종 볼륨 상태나 삭제 여부는 별도 결과 화면으로 확인합니다. |
| 생성 결과 | [EC2 생성 결과](screenshots/ec2/ec2_after.png) | 실습 EC2 생성 결과 화면입니다. |
| 상세 정보 | [EC2 상세 정보](screenshots/ec2/ec2_after_info.png) | 생성된 EC2의 상세 정보 화면입니다. |

## 6. SSH 접속과 Nginx 배포

| 단계 | 증빙 | 확인 내용 및 범위 |
| --- | --- | --- |
| SSH 접속 과정 | [로컬 SSH 명령과 로그인 배너](screenshots/ssh-install_nginx_connect/aws-connect-terminal.png) | 로컬 PC에서 실행한 SSH 명령과 서버 로그인 배너를 확인하는 화면입니다. |
| 접속 후 셸 | [SSH 접속 후 EC2 셸](screenshots/ssh-install_nginx_connect/aws-connect-terminal2.png) | SSH로 접속한 뒤의 EC2 셸 화면입니다. |
| 키파일 권한 설정 | [키파일 chmod 실행](screenshots/ssh-install_nginx_connect/aws-file-chmod.png) | SSH 개인키 파일의 권한 설정 기록입니다. |
| SSH 접속 | [SSH 접속 결과](screenshots/ssh-install_nginx_connect/ssh-connect.png) | SSH를 통한 서버 접속 기록입니다. |
| Nginx 설치 | [Nginx 설치 결과](screenshots/ssh-install_nginx_connect/ssh-install-nginx.png) | 서버에서 수행한 Nginx 설치 기록입니다. |
| 서비스 상태 | [Nginx 실행 상태](screenshots/ssh-install_nginx_connect/ssh-nginx-active.png) | Nginx 서비스 상태를 확인한 기록입니다. |
| 외부 통신 | [EC2 인터넷 아웃바운드 요청](screenshots/ssh-install_nginx_connect/ssh-outbound.png) | EC2에서 수행한 외부 요청의 결과 기록입니다. |
| HTTP 응답 | [서버 HTTP 응답](screenshots/ssh-install_nginx_connect/ssh-web-response.png) | 웹 서버의 HTTP 응답 확인 기록입니다. |

## 7. 브라우저와 health 응답 검증

| 단계 | 증빙 | 확인 내용 및 범위 |
| --- | --- | --- |
| 외부 브라우저 접속 | [퍼블릭 IP의 웹 페이지](screenshots/ssh-install_nginx_connect/web-connect-public.png) | 퍼블릭 IP로 웹 페이지에 접속한 브라우저 화면입니다. |
| 내부 health 요청 | [localhost health 응답](screenshots/ssh-install_nginx_connect/ssh-%3Alocalhost%3Ahealth-ok.png) | 서버 터미널에서 localhost의 health 응답을 확인한 기록입니다. |
| 퍼블릭 IP health 요청 | [터미널의 퍼블릭 IP health 응답](screenshots/ssh-install_nginx_connect/ssh-%3ApublicIP%3Aheatlh-ok.png) | 서버 터미널에서 퍼블릭 IP를 대상으로 요청한 기록입니다. 요청 대상이 퍼블릭 IP라는 사실만으로 외부 PC에서 수행한 검증으로 분류하지 않습니다. |
| 다운로드 파일 확인 | [다운로드한 health 파일의 OK 내용](screenshots/ssh-install_nginx_connect/web-connect-%3Ahealth-ok_check.png) | 다운로드한 health 파일을 열어 `OK` 내용을 확인한 보조 증거입니다. 요청 URL과 HTTP 상태 코드는 표시되지 않습니다. |

## 8. 장애와 복구

| 단계 | 증빙 | 확인 내용 및 범위 |
| --- | --- | --- |
| 접속 실패 | [웹 접속 실패 화면](screenshots/troubleshooting/troubleshooting-failure.png) | 장애 증상을 확인한 화면입니다. |
| 서버 내부 확인 | [localhost 접속 실패](screenshots/troubleshooting/troubleshooting-localhost-no-connect.png) | 서버 내부의 localhost 요청 실패 기록입니다. |
| 서비스 상태 확인 | [Nginx 비활성 상태](screenshots/troubleshooting/troubleshooting-nginx-inactive.png) | Nginx 서비스의 비활성 상태를 확인한 기록입니다. |
| 서비스 로그 확인 | [서비스 중지 관련 로그](screenshots/troubleshooting/troubleshooting-service-stop-log.png) | 서비스 중지와 관련된 로그 기록입니다. |
| 서비스 상태 복구 | [Nginx 실행 상태 복구](screenshots/troubleshooting/troubleshooting-recovered.png) | 조치 후 Nginx의 `active` 상태를 확인한 화면입니다. HTTP 재검증 결과는 이 화면에 없으며 별도 복구 후 HTTP 요청 사진도 없습니다. |

## 9. 리소스 정리

| 단계 | 증빙 | 확인 내용 및 범위 |
| --- | --- | --- |
| EC2 종료 진행 | [EC2 종료 중 화면](screenshots/cleanup/ec2-close.png) | 실습 EC2가 종료 중인 화면입니다. 최종 `terminated` 상태까지 확인한 증빙은 아닙니다. |
| EBS 정리 | [EBS 정리 결과](screenshots/cleanup/cleanup-ebs.png) | EBS 볼륨 정리 결과를 확인하는 화면입니다. |
| Security Group 삭제 | [Security Group 삭제 결과](screenshots/cleanup/sg-delete.png) | 실습 Security Group 삭제 관련 화면입니다. |
| Internet Gateway 삭제 | [Internet Gateway 삭제 결과](screenshots/cleanup/igw-delete.png) | 실습 Internet Gateway 삭제 관련 화면입니다. |
| Route Table 삭제 | [Route Table 삭제 결과](screenshots/cleanup/rt-delete.png) | 실습 Route Table 삭제 관련 화면입니다. |
| Subnet 삭제 | [Subnet 삭제 결과](screenshots/cleanup/subnet-delete.png) | 실습 Subnet 삭제 관련 화면입니다. |
| VPC 삭제 | [VPC 삭제 결과](screenshots/cleanup/vpc-delete.png) | 실습 VPC 삭제 관련 화면입니다. |

정리 상태는 실습 리소스 ID와 삭제 후 화면을 대조하여 판단합니다. 이 목록에는 Elastic IP, NAT Gateway, 로드 밸런서, RDS 및 IAM 정책·키페어 정리의 개별 증빙이 없습니다. 비용 화면만으로 모든 리소스의 삭제 완료를 판단하지 않습니다.

## 10. 비용과 크레딧

| 단계 | 증빙 | 확인 내용 및 범위 |
| --- | --- | --- |
| 2026년 10월 예상 청구서 | [10월 예상 청구서와 무료 플랜 안내](screenshots/billing/billing-credits.png) | 새로 고침 시각 `2026-10-05 18:24:18 KST`, 예상 합계 `USD 0.00`, 결제 상태 대기 중입니다. 실제 내용은 청구서이며 파일명은 `billing-credits.png`입니다. |
| 크레딧 잔액 | [크레딧 잔액 US$120.00](screenshots/billing/billing-october.png) | 총 잔액·예상 잔액 `US$120.00`, 사용·예상 사용 `US$0.00`입니다. AWS Free Tier `$100`과 EC2 실습 크레딧 `$20`이 표시됩니다. 실제 내용은 크레딧이며 파일명은 `billing-october.png`입니다. |

두 파일은 파일명과 화면 내용이 서로 반대이므로 실제 화면 내용에 맞춰 캡션을 달았습니다. 크레딧 화면에는 예상 금액이 약 24시간마다 갱신된다고 표시됩니다. 예상 청구서의 `USD 0.00`은 촬영 당시의 비용 기록이며 최종 확정 비용이나 리소스 삭제 완료를 증명하지 않습니다.
