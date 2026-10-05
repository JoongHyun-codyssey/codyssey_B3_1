# IAM Action 목록과 실습 작업의 관계

사용자가 추가로 제공한 **36개 Action 목록**을 원래 순서, 철자, 와일드카드 그대로 [iam-actions.json](iam-actions.json)에 기록했습니다. 목록의 작업은 EC2와 VPC 기반 인프라의 조회·구성·수정·운영·정리에 해당합니다. 전체 실습은 [README](../README.md), 구성 설명은 [프로젝트 구성 설명](project-explanation.md), 사진은 [증거 목록](evidence-index.md)에서 확인할 수 있습니다.

## 1. 출처와 확인 범위

| 자료 | 출처 | 확인할 수 있는 내용 |
| --- | --- | --- |
| [Action 목록 JSON](iam-actions.json) | 사용자가 대화에서 제공한 목록 | 아래 36개 작업 이름과 `ec2:Describe*` 와일드카드 |
| [IAM 사용자 목록](screenshots/iam/IAM_user.png) | 실습 콘솔 스크린샷 | IAM 사용자 `aws-mission-IAM` 존재 |
| [IAM 사용자 로그인](screenshots/iam/IAM_logined.png) | 실습 콘솔 스크린샷 | 해당 사용자로 콘솔에 접속한 화면 |
| [서울 리전 선택](screenshots/iam/IAM_seoul.png) | 실습 콘솔 스크린샷 | 콘솔에서 서울 `ap-northeast-2`를 선택한 상태 |
| [IAM 사용자와 정책 연결](screenshots/iam/IAM_policy.png) | 실습 콘솔 스크린샷 | 사용자에게 인라인 정책 `aws-mission-policy` 한 개가 연결된 상태 |

`IAM_policy.png`는 정책 연결 화면이며 정책 JSON을 보여주지 않습니다. 따라서 제공된 Action 목록을 이미지에서 읽어 추출한 것으로 설명하지 않습니다. 목록과 연결 화면은 서로 다른 출처의 자료입니다.

[iam-actions.json](iam-actions.json)은 `{"Action": [...]}` 형식의 **Action 조각**입니다. 전체 IAM 정책 문서나 그대로 적용할 수 있는 정책은 아닙니다. `Effect`가 제공되지 않아 `Allow`로 허용하는지 또는 `Deny`로 거부하는지 확인할 수 없고, `Resource`도 제공되지 않아 적용 리소스 범위를 알 수 없습니다. ([Effect 문서](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_effect.html), [Resource 문서](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_resource.html))

`Condition`도 제공되지 않았습니다. `Condition`은 선택 항목이므로 실제 정책에 조건이 있는지 없는지도 아직 확인되지 않았습니다. ([Condition 문서](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition.html))

콘솔에서 서울 리전을 선택한 사실과 정책이 서울 리전으로 권한을 제한하는 사실은 별개입니다. 실제 정책의 조건 내용을 확인하기 전에는 리전 제한을 적용했다고 확정하지 않습니다.

## 2. 작업별 설명

아래 표는 제공된 Action이 가리키는 작업을 묶은 것입니다. 작업의 기본 역할은 AWS의 [EC2 작업 참조](https://docs.aws.amazon.com/service-authorization/latest/reference/list_ec2.html)를 기준으로 설명합니다. 실제 정책의 `Allow` 문에 포함되고 리소스·조건 및 다른 정책 평가에서 허용될 경우, 해당 실습 관리 작업을 수행할 수 있습니다.

| 작업 묶음 | 제공된 Action | 실습과의 관계 |
| --- | --- | --- |
| 조회 | `ec2:Describe*`, `ec2:GetSecurityGroupsForVpc` | EC2 및 관련 네트워크 리소스의 상태·구성 조회, VPC의 보안 그룹 조회에 해당합니다. `Describe*`는 특정 조회 작업 하나에 제한한 목록이 아니라 `Describe`로 시작하는 EC2 작업을 광범위하게 포함하는 와일드카드입니다. ([Action 문서](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_action.html)) |
| VPC | `ec2:CreateVpc`, `ec2:ModifyVpcAttribute`, `ec2:DeleteVpc` | 실습 네트워크의 VPC 생성, 속성 수정, 정리입니다. |
| Subnet | `ec2:CreateSubnet`, `ec2:ModifySubnetAttribute`, `ec2:DeleteSubnet` | EC2를 배치할 Subnet 생성, 속성 수정, 정리입니다. |
| Internet Gateway | `ec2:CreateInternetGateway`, `ec2:AttachInternetGateway`, `ec2:DetachInternetGateway`, `ec2:DeleteInternetGateway` | Gateway 생성, VPC 연결·분리, 정리입니다. |
| Route Table과 경로 | `ec2:CreateRouteTable`, `ec2:AssociateRouteTable`, `ec2:DisassociateRouteTable`, `ec2:DeleteRouteTable`, `ec2:CreateRoute`, `ec2:ReplaceRoute`, `ec2:DeleteRoute` | Route Table 생성·연결·연결 해제·삭제와 인터넷 기본 경로 등 경로 생성·교체·삭제입니다. |
| Security Group | `ec2:CreateSecurityGroup`, `ec2:DeleteSecurityGroup`, `ec2:AuthorizeSecurityGroupIngress`, `ec2:RevokeSecurityGroupIngress`, `ec2:AuthorizeSecurityGroupEgress`, `ec2:RevokeSecurityGroupEgress`, `ec2:ModifySecurityGroupRules` | 보안 그룹 생성·삭제와 인바운드·아웃바운드 허용 규칙 추가·회수·수정입니다. 실제 HTTP·SSH 허용값은 [EC2 보안 탭](screenshots/ec2/ec2_after_sg.png)으로 확인합니다. |
| Key Pair | `ec2:CreateKeyPair`, `ec2:DeleteKeyPair` | SSH 접속에 사용할 키페어 생성과 정리입니다. |
| EC2 인스턴스 | `ec2:RunInstances`, `ec2:StartInstances`, `ec2:StopInstances`, `ec2:RebootInstances`, `ec2:TerminateInstances` | 인스턴스 생성, 시작·중지·재부팅, 최종 종료입니다. 웹 페이지 방문자의 권한이 아니라 실습 서버를 관리하는 작업에 해당합니다. |
| EBS | `ec2:DeleteVolume` | 필요 없어진 EBS 볼륨 삭제입니다. 제공 목록에 명시적인 `ec2:CreateVolume` 항목은 없습니다. |
| 태그 | `ec2:CreateTags`, `ec2:DeleteTags` | 리소스 식별을 위한 태그 추가·삭제입니다. 이 작업 이름만으로 태그 조건에 따라 권한을 제한했다고 판단하지 않습니다. |

생성·삭제와 인스턴스 시작·중지·재부팅 작업이 포함되어 있어 단순 조회 목록보다 넓은 **실습 인프라 관리 작업 목록**입니다. 이 범위가 실제로 필요한 최소 범위인지는 전체 적용 정책과 실습 작업을 대조해 판단해야 합니다.

## 3. 제공 목록 원문

아래 순서는 사용자 제공 목록과 같습니다. JSON 파일도 동일한 목록을 보존합니다.

```json
{
  "Action": [
    "ec2:Describe*",
    "ec2:GetSecurityGroupsForVpc",
    "ec2:CreateVpc",
    "ec2:ModifyVpcAttribute",
    "ec2:DeleteVpc",
    "ec2:CreateSubnet",
    "ec2:ModifySubnetAttribute",
    "ec2:DeleteSubnet",
    "ec2:CreateInternetGateway",
    "ec2:AttachInternetGateway",
    "ec2:DetachInternetGateway",
    "ec2:DeleteInternetGateway",
    "ec2:CreateRouteTable",
    "ec2:AssociateRouteTable",
    "ec2:DisassociateRouteTable",
    "ec2:DeleteRouteTable",
    "ec2:CreateRoute",
    "ec2:ReplaceRoute",
    "ec2:DeleteRoute",
    "ec2:CreateSecurityGroup",
    "ec2:DeleteSecurityGroup",
    "ec2:AuthorizeSecurityGroupIngress",
    "ec2:RevokeSecurityGroupIngress",
    "ec2:AuthorizeSecurityGroupEgress",
    "ec2:RevokeSecurityGroupEgress",
    "ec2:ModifySecurityGroupRules",
    "ec2:CreateKeyPair",
    "ec2:DeleteKeyPair",
    "ec2:RunInstances",
    "ec2:StartInstances",
    "ec2:StopInstances",
    "ec2:RebootInstances",
    "ec2:TerminateInstances",
    "ec2:DeleteVolume",
    "ec2:CreateTags",
    "ec2:DeleteTags"
  ]
}
```

## 4. 최소권한 검토 범위

제공된 36개 항목은 모두 `ec2:` 작업입니다. 이 **제공 목록에는** `iam:`, `s3:`, `rds:` 등 다른 서비스의 Action이 없습니다. 하지만 이 사실만으로 사용자의 전체 권한에 해당 서비스 접근이나 관리자 권한이 없다고 확정할 수는 없습니다.

전체 최소권한 검토에는 실제 정책의 `Effect`·`Resource`와 조건의 존재 여부 및 내용, 사용자에게 직접 또는 그룹 등을 통해 부여되는 추가 정책을 함께 확인해야 합니다. 실제 허용 범위는 적용 정책들을 합산하고 제한·거부 조건을 평가한 결과에 따라 달라집니다.
