# AWS Lambda를 이용한 EC2 인스턴스 제어
진행에 앞서 요구되는 권한 목록
### IAM

`iam:CreateRole`: Lambda가 사용할 역할을 만들 권한.

`iam:AttachRolePolicy`: 역할에 권한 정책을 붙일 권한.

`iam:PutRolePolicy`: (인라인 정책 사용 시) 역할에 정책을 넣을 권한.

`iam:PassRole`: 만든 역할을 Lambda 서비스에 넘겨줄 권한.

`iam:ListPolicies, iam:GetRole, iam:ListRoles`: 콘솔에서 정책과 역할을 조회할 권한.

### Lambda

`lambda:CreateFunction`: 함수 생성.

`lambda:UpdateFunctionCode, lambda:UpdateFunctionConfiguration`: 코드 수정 및 설정 변경.

`lambda:GetFunction`: 함수 조회.

`lambda:AddPermission`: EventBridge가 Lambda를 실행할 수 있도록 트리거 권한 부여.

### EventBridge Schedule

`scheduler:CreateSchedule`

`scheduler:UpdateSchedule`

`scheduler:DeleteSchedule`

`scheduler:ListSchedules`

`scheduler:GetSchedule`

추가

`access-analyzer:ValidatePolicy` : inline 정책에서 다시 action 선택창으로 넘어가기 위함.

## 진행 순서
### IAM 역할 생성
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances"
      ],
      "Resource": 해당 ec2
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": 해당 ec2
    }
  ]
}
```
Lambda에서 사용할 역할을 생성하고 위 정책을 넣어줌  

### Lambda 함수 생성
```python
import boto3
region = '리전명'
ec2 = boto3.client('ec2', region_name=region)
instances = ['인스턴스Ids']
def lambda_handler(event, context):
    # EventBridge에서 전달받은 'action' 파라미터 확인
    action = event.get('action')
    if action == 'start':
        ec2.start_instances(InstanceIds=instances)
        print(f"Starting instances: {instances}")
    elif action == 'stop':
        ec2.stop_instances(InstanceIds=instances)
        print(f"Stopping instances: {instances}")
    else:
        print("No valid action specified.")
```
위 코드를 Lambda 함수로 생성  

### EventBridge 스케줄 생성
- Lambda 함수를 주기적으로 실행하기 위해 EventBridge 스케줄 생성
- 예: 매일 오전 9시에 EC2 시작, 오후 6시에 EC2
- EventBridge에서 스케줄을 생성하고, 대상(Target)으로 Lambda 함수를 지정
- 'Input' 필드에 JSON 형식으로 `{"action": "start"}` 또는 `{"action": "stop"}` 입력
- 스케줄 생성 후 Lambda 함수가 주기적으로 실행되어 EC2 인스턴스를 제어하게 됨
- 예:  
  - 오전 9시 스케줄: `{"action": "start"}`
  - 오후 6시 스케줄: `{"action": "stop"}`

### 테스트
- EventBridge 스케줄이 정상적으로 작동하는지 확인
- Lambda 함수의 로그를 CloudWatch에서 확인하여 EC2 인스턴스가 정상적으로 시작/중지되는지 검증
