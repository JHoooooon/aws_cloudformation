# 💻 Continuous Interation and Deployment

`Continuous Delivery` 는 실제 `Sinario` 에 대해 `Application` 을 테스트하고 원활하게 배포하여 최종 사용자가 `Page` 를 새로 고치면 변경 사항을 알수 있도록 하는데 사용된다.

`Infrastructure as Code` (`IaC`) 에서는 `Infra` 와 `Resource` 를 컴퓨터 애플리케이션으로 취급한다.

`Template` 에 대한 단위 테스트를 수행할수는 없지만, 적절한 `CD Pipeline` 을 실행하는 방법에 대한 다양한 옵션이 존재한다.

이번장에서는 `Application` 에 `Template` 를 포함하는 방법과, `Stack` 을 `Test` 하는 방법, `CI/CD` 에서 `CloudFormation` 을 적용하는 방법을 알아본다.

## 🗒️ 목차

✳️ [Application 에 Template 포함하기](#-application-에-template-포함하기)

✳️ [Running smoke tests on your stack](#-running-smoke-tests-on-your-stack)

## 📌 Application 에 Template 포함하기

`Stack Template` 는 여러 계층을 나누어, 만들수 있다.

`Network 및 보안` 관련 `Template`, `FrontEnd` 관련 `Template`, `Backend/Middleware` 관련 `Template` `DB` 관련 `Template` 등등... 이러한 다양한 계층을 나누어 생성가능하다.

이를 `CD` 에 적용하기 위해서는 `VCS` 를 사용하여, `Template` 를 저장할수 있어야 한다.

이러한 `CD` 를 구성할때, `Workflow` 를 어떻게 구성할지에 따라 많은것들이 달라진다.

인프라와 운영을 담당하는 별도의 운영 또는 클라우드팀이 잇는 경우, 모든 `Template` 를 단일 저장소에 보관하는것이 좋다고 말한다.
 그러면, 개발자가 인프라에 대한 원하는 변경사항이나 새버전의 `Application` 을 제공한다.

구성흐름은 다음과 같다

1️⃣ 개발자가 앱의 새로운 버전을 게시

2️⃣ 개발자는 운영팀에 새로운 버전 혹은 변경사항을 제공

3️⃣ 운영팀은 `parameter` 또는 `template` 을 변경

4️⃣ 운영팀은 `Stack` 에 변경사항을 배포

`DevOps` 문화는 개발자와 운영간의 협업에 관한 것이므로 흐름을 혼합하고 양측이 인프라에 대한 책임을 맡는것이 현명하다.

따라서, `Application` 저장소에 `Template` 를 포함하는것이 좋다.
><br/>
>
>**💬 `Template` 을 `Application` 의 구성중 하나로 간주한다는 말이다.**
>
>`Template` 는 `Application` 의 설정이나 구성요소를 포함한 것으로, `Application` 이 실행될때 필요한 모든 구성을 사전에 정의해 놓은 것이다.
>
>이는 개발과정에서 일관된 설정과 환경을 유지하고, 배포와 유지보수를 간편하게 만들어준다.
>
>`Template` 를 `Application` 저장소에 포함시키면, `Application` 개발 및 운영 과정에서 더욱 효율적이고 예측 가능한 환경을 유지할수 있다.
><br/>

앱 구성파일을 포함한 애플리케이션에 대한 저장소는 다음과 같다.

```sh
my_app/
|----.git/
|----config
|   |----test
|   |    |----config.ini
|   |----prod
|        |----config.ini
|----src/
|    |---- # your source code here
|----tests/
|    |---- # your tests here
|----cloudformation/
     |----template.yaml
```

이러한 이유로, `Developer` 는 `AWS CloudFormation` 을 사용하는 방법을 알아야 한다고 말한다.

><br/>
> ⚠️ 물론, 전체 `Stack` 에 영향을 끼치는 `Resource` 같은 경우, 별도의 저장소에 보관하여 처리하는것이 좋다.
><br/><br/>

## 📌 Running smoke tests on your stack

`Smoke tests` 는 ***소프트웨어의 핵심 기능들이 제대로 작동하는지를 검증하는 테스트 방법이다.***

`Application` 의 초기 단계나 중요한 변경사항을 배포하기전에 수행된다.
이 테스는 ***`Smoke` 라는 단어처럼 초반에는 조금 연기만 나오는 정도의 빠른 테스트*** 를 뜻한다.

즉, ***빠르게 핵심 기능들이 예상대로 동작하는지 확인하는 목적***을 가진다.
이는 `release` 전 혹은 `release` 후에도 많이 사용되는 `test` 방법이다.

`Stack` 상의 `resource` 중 하나라도 만들수 없는경우, 이전 상태로 전부 되돌린다.

> 💬 이러한 기능을 `rollback` 이라 하지!

보통 이렇게 `resource` 생성에 실패하면 `Error` 를 보고하고, 우리는 확인가능하다.

하지만, 일부 `resource` 같은경우, 보고되지 않는 경우도 있다고 한다.

> 😠 뭥미?!!? :angry:

다음의 예시는 `EC2 AutoScaling Group` 에 대한 `Smoke Tests` 이다.  

> ✏️ **`ELB` 로 `ASG`(`Auto Scalining Group`) 을 프로비저닝한다.**
>
> `ASG` 는 `Instance` 생성시 필요한 설정과 명령을 미리 정의할수 있도록, `LaunchConfigurationName` 혹은 `LaunchTemplate` 를 사용하여 구성한다.
>
>`LaunchConfigurationName` 혹은 `LaunchTemplate` 을 정의하면 되는데, `2` 설정 같이 적용은 못하고, 둘중 하나의 설정만 정의해야한다.
>
> 🔗 [AWS::AutoScaling::AutoScalingGroup](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/aws-resource-autoscaling-autoscalinggroup.html) 을 보면, 내용이 나와있다.
>
> `ASG` 은 상태검사 실패시, 실패한 `Instance` 를 `LunchConfiguration` 을 통해 만들어진 새 `Instance` 와 교체한다.
>
> 이러한 과정은 다음과 같을것이다.
>
> 1️⃣ `LunchConfiguration` 에는 `EC2 Instance` 를 만드는 동안 실행할 명령어 집합이 있을수 있다.
>
> 2️⃣ `ELB` 가 `Instance` 상태를 주기적으로 확인하며,, 상태검사가 실패할경우 문제가있음을 알아챈다.
>
> 3️⃣ 이때, `ASG` 는 문제 발생 `Instance` 를 종료하고, `LaunchConfiguration` 을 참조하여, 새로운 `Instance` 를 만들고 실패한 `Instance` 와 교체한다.
>
>> 이때, `LaunchConfiguration` 의 명령어 집합을 실행하여, `web server software` 를 설치한다.
>
><br/>

다음은 예시 `Template` 이다.

```yml
AWSTemplateFormatVersion: "2010-09-09"
Parameters:
  AmiId:
    Type: 'AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>'
    Default: '/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2'
  VpcId:
    Type: AWS::EC2::VPC::Id
    Description: Use it to provide default vpc
  SubnetIds:
    Type: List<AWS::EC2::Subnet::Id>
    Description: Use it to provide a list of subnetids

Resources:

  Lc:
    Type: "AWS::AutoScaling::LaunchConfiguration"
    Properties:
      ImageId: !Ref AmiId
      InstanceType: "t2.micro"
      UserData: # can you find a mistake in this UserData?
        Fn::Base64: |
          #!/bin/bash
          yum -y install yolo
          systemctl start yolo
      SecurityGroups:
        - !Ref Sg

  Sg:
    Type: "AWS::EC2::SecurityGroup"
    Properties:
      GroupDescription: "not secure sg"
      SecurityGroupIngress:
        - IpProtocol: "-1"
          CidrIp: "0.0.0.0/0"
      VpcId: !Ref VpcId

  Asg:
    Type: "AWS::AutoScaling::AutoScalingGroup"
    Properties:
      MaxSize: "1"
      MinSize: "1"
      LaunchConfigurationName: !Ref Lc
      AvailabilityZones: !GetAZs
      HealthCheckGracePeriod: 30
      HealthCheckType: "ELB"
      TargetGroupARNs:
        - !Ref Tg

  Tg:
    Type: "AWS::ElasticLoadBalancingV2::TargetGroup"
    Properties:
      Port: 80
      Protocol: "HTTP"
      VpcId: !Ref VpcId
      UnhealthyThresholdCount: 5

  Elb:
    Type: "AWS::ElasticLoadBalancingV2::LoadBalancer"
    Properties:
      Type: "application"
      Scheme: "internet-facing"
      SecurityGroups:
        - !Ref Sg
      Subnets: !Ref SubnetIds

  Listener:
    Type: "AWS::ElasticLoadBalancingV2::Listener"
    Properties:
      Port: 80
      Protocol: "HTTP"
      LoadBalancerArn: !Ref Elb
      DefaultActions:
        - Type: "forward"
          TargetGroupArn: !Ref Tg

Outputs:
  Dns:
    Value: !GetAtt Elb.DNSName
```

이 `Template` 를 배포하면 `CloudFormation` 에서 `Stack` 생성이 완료되었다고 알려준다.

```sh
$ aws cloudformation deploy \
                     --template-file broken_asg.yaml \
                     --stack-name broken \
                     --parameter-overrides VpcId=vpc-12345678\
                     SubnetIds=subnet-123,subnet-456,subnet-789,subnet-012,subnet-345,subnet-678

Waiting for changeset to be created..
Waiting for stack create/update to complete
Successfully created/updated stack - broken
```

그러나, `CloudFormation` 에서 `ELB` 의 `URL` 을 통해 `curl` 로 접근하려고 하면 다음처럼 `503` 에러를 나타낸다.

```sh
$ aws cloudformation describe-stacks \
      --stack-name broken \
      --query 'Stacks[0].Outputs[0].OutputValue' \
      --output text

broken-Elb-157OHF7S5UYIC-1875871321.us-east-1.elb.amazonaws.com

$ curl broken-Elb-157OHF7S5UYIC-1875871321.us-east-1.elb.amazonaws.com
<html>
<head><title>503 Service Temporarily Unavailable</title></head>
<body bgcolor="white">
<center><h1>503 Service Temporarily Unavailable</h1></center>
</body>
</html>
```

이는, `AWS::AutoScaling::LaunchConfiguration` 의 `UserData` 에 `Web Server Software` 를 설치하지 않았기 때문에 발생하는 일이다.

> 💬 `yolo` 는 `Web Server Software` 가 아니라, `Web Server` 의 역할을 못하기에 이러한 에러가 나온것이다.
>
> 웹 서버역할을 하기 위해서는 `Nginx` 혹은 `Apach` 같은 소프트웨어를 설치해야 한다..

이로인해 `ASG` 는 `503` 을 내뱉으므로, `Instance` 가 제대로 동작하지 않는다는것으로 인식하고, 지속적으로 새로 생성한다.

> ➿ ***이로인해, 서비스가 시작되지 않아 무한 생성/삭제 과정이 시작된다.***

이는 `CloudFormation` 에서 인식하고 처리해야 하는것 아니냐고 반문할수 있지만, `CloudFormation` 은 오직 `Stack` 의 `Resource` 를 생성하는것 까지만 처리한다.

이후 생성된 `Resource` 의 동작까지는 어떻게 처리하지 못한다.

👼 ***즉, `CloudFormation` 은 이러한 부분까지는 전혀 인식 못한다는것이다.***

> 책에서는 `ASG` 를 사용하면 이문제를 쉽게 해결가능하다고 하는데 `6장` 에서 `cnf-lint` 를 이용한 `EC2` 인스턴스의 구성 관리에 대해 살펴보겠다고 한다...
>
> 여기서는 안다룰것이므로 `pass`

`Template` 를 `Test` 하는데 있어 또다른 문제는 `CloudFormation` 에 일반적인 프로그래밍 언어와 같은 단위 테스트 기능이 없다는것이다.

`Test` 하는 유일한 방법은 `Template` 에서 `Stack` 을 `Provisioning` 하고 `Stack` 과 해당하는 `Resource` 에 대한 `Test` 를 실행하는것이다.

다음은 `Python` 을 사용하여 `CloudFormation` 의 `Stack` 을 가져와 `Test` 하는 코드이다.

이 코드는 다음과 같은 과정으로 이루어진다.

1️⃣ `Stack` 이름을 입력한다.

2️⃣ `CloudFormation` `API` 를 가져와, `describe_stacks` 를 사용하여 `Stack` 내용을 가져온다. 만약, 해당하는 `Stack` 이 없다면,  `This stack does not exist or region is incorrect` 를 출력하고 비정상 종료한다.

3️⃣ `describe_stacks` `outout` 에서 `DNS` 를 가져온다.

4️⃣ 해당 `DNS` 로 `HTTP` 요청을 보낸다. `200 OK` 이면 `Test successed` 를 출력하고 정상종료, 아니면 `Test did not succeed` 를 출력하고 비정상 종료한다.

```py
from time import sleep
import boto3
import botocore
import requests
import sys

try:
    stack = sys.argv[1]
except IndexError:
    print('Please provide a stack name')
    sys.exit(1)

cfn_client = boto3.client('cloudformation')

try:
    stack = cfn_client.describe_stacks(StackName=stack)
except botocore.exceptions.ClientError:
    print('This stack does not exist or region is incorrect')
    sys.exit(1)

elb_dns = stack['Stacks'][0]['Outputs'][0]['OutputValue']
for _ in range(0, 2):
    resp = requests.get(f"http://{elb_dns}")
    if resp.status_code == 200:
        print("Test succeeded")
        sys.exit(0)
    sleep(5)

print(f"Result of test: {resp.content}")
print(f"HTTP Response code: {resp.status_code}")
print("Test did not succeed")
sys.exit(1)
```

이는, 앞의 `cli` 로 `curl` 을 통해, `HTTP` 요청을 보낸 코드를 `python` 방식으로 풀어낸것이다.

이는 앞의 `cli` 와 똑같은 `result` 값을 받아오며, `503` 에러와 함께 `Test did not succeed` 를 출력한다.

이를 통해 배포된 `Stack` 에 문제가 있음을 확인했으니, 문제가된 부분을 수정해야 한다.

지금 같은경우 `web server` 를 설치하지 않아 생긴문제이니, `web server` 를 설치하고, 실행시킨다.

```yml
// working_asg.yaml

# Parameters section...
Resources:
  Lc:
    Type: "AWS::AutoScaling::LaunchConfiguration"
    Properties:
      ImageId: !Ref AmiId
      InstanceType: "t2.micro"
      UserData:
        Fn::Base64: |
          #!/bin/bash
          amazon-linux-extras install -y epel
          yum -y install nginx
          systemctl start nginx
      SecurityGroups:
        - !Ref Sg
# the rest of the template...
```

```sh
$ python asg_test.py brokenTest 
succeeded
```

이를 통해 제대로 처리된것을 볼수 있다.
책에서는 다음처럼 설명한다.

> 📝 ***"무엇을 빌드하는지에 따라 배포하는 모든 어플리케이션이나 인프라에 대해 새 테스트 사례를 만들어야 한다."***

### VPC resource smoke test

이는 `VPC` `resource` 가 제대로 작동하는지 알기 위한 테스트방법이다.

> 💬 [앞에서](#-running-smoke-tests-on-your-stack) 말했지만, ***`CloudFormation` 은 구성이 잘 되었는지를 확인하지, `Resource` 가 제대로 동작하는지는 확인하지 않는다.***

책에서 다음과 같은 가정을 한다.

:test_tube: `Private Subnet` 이 `Private route table` (`NAT GW` 를 통해 `Traffic` 을 라우팅하는 테이블) 과 연결되어있는지 확인하고 싶다.

이를 테스트하기 위해서는 다음을 수행한다.

1️⃣ `Subnet resource` 에 `Tag` 를 추가한다.

```yml
# Public Subnet tag 는 다음과 같다.
Tags:
  - Key: "Private"
    Value: "False"

# Private Subnet tag 는 다음과 같다.
Tags:
  - Key: "Private"
    Value: "True"
```

2️⃣ `Script` 는 `Stack` 의 출력이 아닌 `resource` 를 수집한다.<br/><br/> `Resource` 를 `AWS::EC2::Subnet` `Type` 으로 `filtering` 하고, `Tag` 값을 `Private: "True"` 로 만든다.

3️⃣ `Private route table` 의 `ID` 를 얻고, `Private Subnet` 이 `Private route table` `ID` 와 연결되어있는지 확인한다.

```python
import boto3
import botocore
import sys

matches = []

try:
    stack = sys.argv[1]
except IndexError:
    print('Please provide a stack name')
    sys.exit(1)

cfn_client = boto3.client('cloudformation')

try:
    resources = cfn_client.describe_stack_resources(StackName=stack)
except botocore.exceptions.ClientError:
    print('This stack does not exist or region is incorrect')
    sys.exit(1)

subnets_in_stack = []
for resource in resources['StackResources']:
    # (LogicalResourceId) 논리적 `ID` 는 해당 `Resource` 가 어떠한 `Resource` 인지
    # 쉽게 알려주기 위한 문자열 값이다.

    # (PysicalResourceId) 물리적 `ID` 는 `AWS` 에서 생성되면서 만들어진 `ID` 값이다.

    # ---- Stack 안의 PrivateRouteTable 의 ID 값을 할당하는 코드 ------
    # resource 의 논리적 `ID` 값이 `PrivateRouteTable` 인지 확인
    if resource['LogicalResourceId'] == 'PrivateRouteTable':
        # private_route_table 에 물리적 `ID` 값 할당
        private_route_table = resource['PhysicalResourceId']

    # ---- Stack 안의 PrivateSubnet 의 ID 값을 할당 코드 ------
    # ResourceType 이 `AWS::EC2::Subnet` 인지 확인
    if resource['ResourceType'] == 'AWS::EC2::Subnet':
        # subnets_in_stack 에 `AWS::EC2::Subnet` `resource` 의 `ID` 값
        # 추가
        subnets_in_stack.append(resource['PhysicalResourceId'])

# `ec2 api` 를 가져옴 
ec2_client = boto3.client('ec2')
subnets_to_check = []
# 추가한 모든 `subnet` 을 순회
for subnet in subnets_in_stack:
    # 해당 subnet id 의 내용을 가져옴 
    resp = ec2_client.describe_subnets(SubnetIds=[subnet])
    # 가져온 subnet 의 tags 를 순회
    for tag in resp['Subnets'][0]['Tags']:
        # tag.Key 가 Private 이고 tag.Value 가 True 인지 확인
        if tag['Key'] == 'Private' and tag['Value'] == 'True':
            # 맞다면, subnets_to_check 리스트에 subnet 추가
            subnets_to_check.append(subnet)

# ec2 에 private_route_table pysical value 값에 해당하는 route_table 내용을 가져옴
route_table = ec2_client.describe_route_tables(RouteTableIds=[private_route_table])
private_subnets = []
# `route_table` 에 연관(Associations) 되어있는 `Association` 을 가진 리스트를
# 순회하여, `Association Dict` 를 가져옴 
for assoc in route_table['RouteTables'][0]['Associations']:
    # assoc(Association Dict) 에서 연관된 `SubnetId` 를 `private_subnets`
    # 리스트에 추가
    private_subnets.append(assoc['SubnetId'])

# subnets_to_check 를 순회
for subnet in subnets_to_check:
    # subnet 이 private_subnets 에 속하지 않는다면,
    if subnet not in private_subnets:
        # matches 에 subnet 추가
        matches.append(subnet)

# matches 가 있다면, checks_to_subnet 의 PrivateSubnet 이 한개라도, 
# PrivateRoutingTable 에 연결되어 있지 않다는것을 의미: 비정상 종료
if matches:
    print('One or more private subnets are not associated with proper route table!')
    print(f"Non-compliant subnets: {matches}")
    sys.exit(1)

print('All subnets are compliant!')
exit(0)
```

위는 `Route table` 에 연관된 `Private Subnet` 에 대한 `smoke test` 코드이다.

이를 통해 `VPC` 내부의 `Private Subnet` 이 `Private Route Table` 에 연결되어있는지 빠르게 `Test` 가능해졌다.

## CloudFormation Stack release 관리를 위한 모범사례

`release` 관리에 대한 모범사례를 다음과 같다.

### ✳️ `VCS` 를 사용

`VCS` 는 협업, 변경 내역, `source code` 의 저장, `build system` 과의 통합등 여러 이점이 있다.

### ✳️ 종속성을 쉽게 관리

앞의 [Smoke Test](#-running-smoke-tests-on-your-stack) 에서 보았듯이, `template` 에 대한 여러 `script` 를 작성했다.

이러한 `Script` 에는 외부 종속성이 있으므로, 이를 쉽게 설치하고 해결할수 있는지 확인해야 한다.

### ✳️ `Code base` 를 깨끗하게 유지

`CloudFormation` 레파지토리를 제대로 구성했는지 확인해야 한다.
이는, 회사 `convention` 에 맞도록 구성하되, 구성폴더가 제대로 이루어져 있는지를 확인하고 유지해야 한다는 말이다.

### ✳️ 적절한 Branch Model 을 선택

이는 `git` 을 사용하면서, `CI/CD` 시스템을 통합할수 있는 `Branch` 모델을 선택하는것이 좋다고 한다.

이는 상용 및 개발 단계로 나누어 `prod` / `dev` / `test` ... 식으로 여러 `Branch` 를 생성하여 `Workflow` 를 분할할수 있다.

이는 `Jira` 와 같은 `project management tool` 을 사용하는경우 `Commit message` 에 `Issue Key` 를 추가할수 있다.

```sh
git commit -m "PROJ-1234 add extra subnets for databases"
```

`Issue Key` 를 포함하면 `Codebase` 를 깔끔하게 유지하고 개발 프로세스를 투명하게 유지하는데 도움이 된다.

### ✳️ 코드 검토 필수

이는 `Push` 를 하지말고, `Pull Request` 를 사용하여, 변경사항을 확인하고 처리할수 있는 방식을 고수하라는 말이다.

이는 관리자가 `Pull Request` 의 내용을 확인하고, 불필요한 코드 및 잘못된 코드가 없는지 확인하여 더 신뢰성있고, 믿을수 있는 코드 통합이 가능하다.

### ✳️ 테스트 검증 수행

`Stack` 변경 사항을 배포하기 전에, 항상 `linting` 및 `template` 검증을 수행한다.

`CloudFormation` 에 대한 단위 테스트를 수행할수 없지만, 오류를 최소한으로 하는것이 좋기 때문이다.

### ✳️ CloudFormation IAM 역할을 사용

`Stack` 을 배포할때, 자체 `IAM` 권한을 사용한다.
이는 `AWS` 에 최소한 관리자 엑세스 정책이 있어야 한다는 말이ㅏㄷ.
이는 안전하지 않다.

많은 사람이 협업하게 될수 있으므로, 각 `template` 개발자에게 `AWS` 에 대한 관리자 엑세스 권한을 제공하지 않으며, `CloudFormation IAM` 역할을 사용하여 해당 권한을 부여해야한다.

### ✳️️ 올바른 `CI/CD` 도구를 사용

`Stack` 작업(Valication, Lining, Deploy)는 기본 명령이다.
그러므로, `CI/CD` 시스템이 이를 수행할수 있는지 확인하는것이 중요하다

이는 가격 / 보안(`AWS` 에 접근하므로) 에 대해 생각해볼 필요가 있다.

이러한 `CI/CD` 관련 도구는 `github`, `gitlab`, `jenkins`, `AWS CodeBuild`, `AWS CodePipeline` 등 많으므로, 적절한 도구를 사용해야한다.

여기서는, `AWS CodePipeline` 을 사용할 예정이다.

## CloudFormation 및 CodePipeline 을 사용

`pipeline` 은 소프트웨어 개발측면에서 다음의 단계로 구성된다.

1️⃣ **Pre build**: `source code` 관리에서 `code` 를 가져와 종속성을 설치

2️⃣️ **Build**: `build` 를 실행하고, `artifact` 를 검색

3️⃣ **Test**: `build` 에 대해 `test` 모음을 실행

4️⃣ **Packge**: `build artfact` 를 `artifact` 저장소에 저장

5️⃣ **Deploy**: `envorinment` 에 `new verion` 을 `deploy`

이 단계중 하나라도 실패하면, `build` 시스템은 오류가 발생한다.
책에서는 위의 단계들을 다음처럼 실행한다.

1️⃣ **Pre build**: `AWS CodeCommit` 에 `code 및 template` 를 저장하고, `AWS CodeBuild` 를 `build system` 으로 사용

`CodeCommit` 에서 `code` 를 `pull` 하여 다음의 종속성 패키지를 설치

- cfn-lint
- awscli
- boto3

2️⃣ **Build**: `CloudFormation` 은 `compile` 할 필요가 없으므로, 유효성검사 만 실행

```sh
aws cloudformation validate-template \
  --template-body ...
```

3️⃣ **Test**: `cfn-lint` 를 사용하여 `CloudFormation` `linter` 검사,
실패하지 않으면, `smoke test` 실행

4️⃣ **Package**: `artifact`(산출물은 `cloudformation` 이 될것이다.) 를 `Aratifact repository` 에 저장

`AWS` 에서 `AWS::CodeCommit::Repository` 의 `TemplateBucket` 으로 `S3` 가 `Artifact Repository` 역할을 한다.  

5️⃣ **Deploy**: `S3` 에 `Package` 되어 저장된 `Template` 를 사용하여 `Production` `Deploy` 를 진행한다.

> **💬 흐름은 이러한 방식으로 진행된다.**
>
> 나머지 내용은 읽고 넘어간다.
>
> `github action` 으로 `CI/CD` 를 구현할것이므로 내용만 알아두는것이 좋을듯 싶다.
