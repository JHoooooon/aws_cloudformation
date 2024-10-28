# 🏗️ Validation, Linitng and Deploying the Stack

`Stack` 배포는 하지 않고 [template_info](../template_info/) 에서 `Template` 에 대한 각 섹션과 활용방법에 대해서 살펴보았다.

여기에서는 `Resource` 를 어떻게 `provisioning` 하고, `validation` 및 `template lining` 을 하는지 설명한다.

## 🗒️ 목차

📌 [Template Validation](#pushpin-template-validation)

📌 [Using a lint for best practices on templates](#pushpin-using-a-lint-for-best-practices-on-templates)

📌 [Provisioning our stack](#pushpin-provisioning-our-stack)

📌 [Handling errors](#pushpin-handling-errors)

📌 [Working with drift](#-working-with-drift)

📌 [추가 읽기](#-추가-읽기)

## :pushpin: Template Validation

`aws cloudformation create-stack` 혹은 `aws cloudfromation update-stack` 을 실행할때마다, `CloudFormation` 은 먼저 `template` 의 유효성을 검사한다.

유효한 `JSON` 혹은 `YAML` 파일인지 구문검사하고, 순환종속성(`curcular dependencies`) 등의 문제가 있는지 확인하는 작업이다.

`Template Validation` 은 `Stack` 을 `deploy` 전에 필요한 단계이지만, `Validation` 이 성공해도 다음과 같은 이유로 중단될수 있다.

- `Resource` 에 `required` 속성이 없는 경우
- `Resource property name` 오타 및 오류
- `Not Exists Resource property value`

`Template` 를 검증하는 명령어도 존재하는데 다음과 같다.

```sh
> aws cloudformation validate-template \
--template-body file://path_to_your_template
```

`Template` 에 오류가 발생하면 다음과 같은 에러가 나온다.

```sh
An error occurred (ValidationError) when calling the ValidateTemplate operation: Template format error: Unrecognized resource types: [AWS::EC2::DoesNotExist]
```

책에 있는 내용인데, `Template format error` 가 발생했으며, 인식되지 않은 `resource`  유형에 대한 에러가 나온다.

그리고 에러에 대한 `resource`  가 `AWS::EC2::DoesNotExist` 라고 명시해준다.
만약 `Fn:Select` `instric function` 를 존재하지 않는 `Fn::NoSelect` 함수로 변경한다면 에러가 날것이다.

```yml
An error occurred (ValidationError) when calling the ValidateTemplate operation: Template format error: YAML not well-formed. (line 31, column 18)
```

좋지 않은 포맷이라며, `line 31 에 cloumn 18` 에서 에러가 났다고 알려준다.
아마도, `Fn:NoSelect` 구문이 작성된 라인과 컬럼이다.

이렇게, 스택 배포전에 `Validation` 을 통한 검증을 먼저 한후 배포하면 더 빠르게 `Error` 를 확인할수 있다.

## :pushpin: Using a lint for best practices on templates

[Template Validation](#pushpin-template-validation) 에서 `Validation` 을 통해 더 빠르게 `Error` 를 확인할수 있다고 했다.

하지만, `Template` 작성시 바로바로 확인할수 있다면 더 빠르게 확인하고 처리할수 있다.

이를 위한 작업이 `linter` 이다.
`linter` 는 정적 소스 코드 분석기로, 명백한 프로그래밍 오류와 버그, 스타일 및 비관용적 문제를 표시해준다.

이를 통해, 규칙있고 일관된 포멧으로 오류없는 `Template` 작성을 도와준다.

### :eyes: cfn-lint 사용

`cfn-lint` 는 `python` 으로 제작된 `linter` 이다.
당연, `python` 이 설치되어있어야 한다.

```sh
> pip install cfn-lint;

```

이를 통해 `cfn-lint` 설치가 가능하며, 다음처럼 `validation` 가능하다.

```sh
> cfn-lint core.yml
```

이에 대한 오류가 있을시 `linter` 가 즉시 알려준다.

### :eyes: 규칙 무시

`cfn-lint` 는 `linter` 규칙을 준수하지만, 특정 규칙을 무시하도록 할수도있다.
이는 `convention` 에 따라 달라질수 있으므로, 사용할수 있는 방식이다.

```sh
> cfn-lint core.yaml --igrno-checks 'W2001';
```

이는 `W2001` 이라는 규칙을 무시하라는 명령이다.
혹은 `Metadata` 에 추가가능하다.

```yml
AWSTemplateFormatVersion: 2010-09-09
Metadata:
  cfn-lint:
    ignore-checks:
      - W2001
Transform: AWS::LanguageExtensions
Prameters:
  # and so on...
Mappings:
  # and so on...
Resources:
  # and so on...
```

### :eyes: 사용자 정의 규칙 만들기

`cfn-lint` 는 사용자 정의 규칙 생성이 가능하다.

책에서 만들고자 하는 규칙은, `AWS::RDS::DBInstance`, `AWS::RDS::DBCluster` 에 `DeletionPolicy` 가 있으며, `Retain` 혹은 `Snapshot` 이면 통과하고, 그렇지 않고 `Deletion` 이거나 정의가 되어있지 않으면 `Warning` 을 내뿜는 규칙이다.

`custom_rules` 폴더에 `rdsdelitionpolicy.py` 를 생성한다.

>**⚠️ `cfn-lint` 클래스 이름과 파일명이 같아야 한다.**
>
> 👍예를 들어, `myrule.py` 라면, `class MyRule(CloudFormationLinterRule)` 이어야 한다.
>
> 👎`my_rule.py` 같은 경우, `class MyRule(CloudFormationLinterRule)` 로 선언되면 작동되지 않는다.

규칙을 지정하면, 이를 알수있는 규칙번호를 지정해야 한다.
이는 [cfn-lint](https://pypi.org/project/cfn-lint/0.2.1/)  의 `Rule IDs Categories` 부분을 보면, `Reserved for users rules` 은 `(E|W)9xxx` 에 작성하라고 정해져 있다.

그러므로, 여기서 만들 규칙은, `Warning` 이므로 `W9xxx` 대의 규칙 번호를 만들면 된다.

여기서는 `W9001` 로 생성한다.

사용자 정의 규칙은 `CloudFormationLintRule` 클래스 객체를 상속받으므로, `pip` 을 통해 설치해주어야 한다.

```sh
> pip install virtualenv;
> virtualenv venv;

(venv) > pip install cfnlint;
```

> 여기서 설치된 [cfnlint](https://pypi.org/project/cfnlint/) 는 [cfn-lint](https://pypi.org/project/cfn-lint/) 와는 다르다.
>
> 👉 [cfnlint](https://pypi.org/project/cfnlint/) 패키지로써의 `AWSCloudFormation linter` 이며, 이를 사용하여 사용자 정의 및 확장이 가능하다.  
>
> 👉 [cfn-lint](https://pypi.org/project/cfn-lint/) 는 `yaml/json` 템플릿에 대한 [AWS CloudFormation resource 제공 스키마](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/resource-type-schemas.html) 와 추가 확인을 하는 검증도구이다.
>
> 즉, `Package` 로써 사용해야 하므로, [cfnlint](https://pypi.org/project/cfnlint/) 을 설치해야 한다.

이제 해당 규칙을 작성한다.

[cfn-lint/docs/getting_strated/rules.md](https://github.com/aws-cloudformation/cfn-lint/blob/main/docs/getting_started/rules.md) 를 보면, 어떻게 `Role` 을 생성해주는지 자세하게 나와있다.

> 다음은 책에서 나온 예시코드이다.

```py
#rdsdeletionpolicy.py

# CloudFormationLintRule 을 상속받은 RdsDeletioNPolicy
class RdsDeletionPolicy(CloudFormationLintRule):
    # Rule Number 설정
    id = 'W9001'

    # rule 에 대한 짧은 설명구문
    shortdesc = 'Check RDS Deletion Policy'

    # rule 에 대한 자세한 설명구문
    description = 'This rule checks DeletionPolicy on RDS resources to be Snapshot or Retain'

    # Rule 을 적용할 resource 의 list
    RDS_RESOURCES = [
        'AWS::RDS::DBInstance',
        'AWS::RDS::DBCluster'
    ]

    # Lint 가 실행될시, match 가 실행되어, matches 에 저장하고,
    # 리턴한다.
    def match(self, cfn):
        matches = []

        # cfn.get_resources 를 사용하여, 해당하는 
        # resources dictionary 를 가져온다.
        resources = cfn.get_resources(self.RDS_RESSOURCES)

        # resources.items() 를 사용하여, key 와 value 값을 가져온다.
        # key = resource_name
        # value = resource
        for resource_name, resource in resources.items():

          # resource 에서 DeletionPolicy 를 가져온다
          deletion_policy = resource.get('DeletionPolicy')

          # RuleMatch class 에 전달할 path list 작성
          path = ['Resources', resource_name]

          # deletionPolicy 가 없다면,
          if deletionPolicy is None:
            # 전달할 message 작성
            message = f'Resource {resource_name} does not have DeletionPolicy!'

            # matches list 에 RuleMatch class 의 객체를 전달
            matches.append(RuleMatch(path, message))

          # deletionPolicy 가 `Snapshot` 혹은 `Retain` 이 아니라면,
          elif deletionPolicy not in ('Snapshot', 'Retain'):
            # 전달한 message 작성
            message = f'Resource {resource_name} does not have Deletion Policy set to Snapshot or Retain!'

            # matches list 에 RuleMatch class 의 객체를 전달
            matches.append(RuleMatch(path, message))

        # return 되고, list 에 값이 있으면, `W9001` 발생
        return matches
```

이로써, `DeletionPolicy` 가 없거나, 값이 `Delete` 라면, 경고 메시지를 뿜뿜한다.

> :warning: **`cfn-lint` 에서 `Condition` 을 사용한 `Deletion Policy` 를 사용시 주의해야 한다.**
>
> `cfn-lint` 는 `Condition in DeletionPolicy` 를 사용하는경우, 제대로 평가하지 않는다. 이는 평가시점이 다르기 때문이다.
>
> ***`cfn-lint` 는 `as-is`(그대로) 평가하는 방식을 사용하여, 템플릿에 대한 조건을 고려하지 않고, 작성된 조건 자체만을 보고 평가하기 때문이라 한다.***
>
> 간단한 예시를 보자.

```yml
Parameters:  
  EnvironmentType:  
    Type: String  
    Default: "test"  
    AllowedValues:  
      - prod  
      - test  
    Description: "Choose the environment type, either prod or test."  

Conditions:  
  IsProdEnvironment: !Equals [!Ref EnvironmentType, "prod"]  

Resources:  
  MyS3Bucket:  
    Type: AWS::S3::Bucket  
    DeletionPolicy: !If  
      - IsProdEnvironment  
      - Retain    # 프로덕션 환경에서는 S3 버킷을 보존  
      - Delete    # 테스트 환경에서는 S3 버킷을 삭제  

Outputs:  
  BucketName:  
    Value: !Ref MyS3Bucket  
    Description: "Name of the S3 bucket"
```

> 이 같은경우, `IsProdEnvironment` 를 사용하여, `DeletionPolicy` 를 평가해야만 한다.
>
> 하지만, `cfn-lint` 는 ***있는 그대로 `!If` 구문 자체를 가져오고, 잘못된 구문으로 인식한다.***
>
> `DeletionPolicy` 에서 평가되어 `Retain` 혹은 `Delete` 가 아닌

```yml
 !If  
      - IsProdEnvironment  
      - Retain
      - Delete
```

> 로 평가한다는 말이다.
>
> 이는 당연 잘못된 구문으로 인식된다.
> 이는, `Condition` 을 사용시 발생할수 있는 문제점으로 기억해주어야 한다.

이제 `cfn-lint` 를 실행시 `-a` 에 `custum-rules` 폴더를 포함하고 실행하면 제대로 실행된다.

```sh
(venv)> deactivate$ cfn-lint database_failing.yaml -a custom_rules

W9001 Resource TestDatabase does not have Deletion Policy!
database_failing.yaml:42:3
W9001 Resource ProdDatabase does not have Deletion Policy set to Snapshot or Retain!
database_failing.yaml:65:3
```

### 👀 정책 기반 린터 사용

`cfn-lint` 는 `CloudFormation linter` 였다면, `cloudformation-guard` 는 특정 규칙을 준수하는지 확인하는 정책기반 `linter` 이다

> ❓ 정책기반검증이란?
> ***정책언어를 사용하여 규칙을 정의***하고 템플릿이 이 규칙을 따르는지 검사한다.
>
> 굳이 비교하자면,
> `cfn-gaurd` 는 `eslint` 와 같은 역할이라고 보면되며, `cfn-lint` 는 `javascript compiler` 로 보면된다.

[cloudformation-guard/cfn-guard](https://github.com/aws-cloudformation/cloudformation-guard/tree/Guard1.0/cfn-guard#writing-rules) 에 자세한 내용이 나온다.

> 해당부분은 간단하게 훑고 따로 더 자세한 내용을 훑어봐야 겠다.

## :pushpin: Provisioning our stack

`AWS cli` 를 통해서 `CloudFormation` `API` 로 `Stack` 에 대한 처리가 가능하다.

`CloudFormation` 은 `Stack` 을 생성할수 있다

```sh
> aws cloudformation create-stack --stack-name ... --template-body file://...
```

`CloudFormation` 은 `Stack` 을 업데이트 할수 있다

```sh
> aws cloudformation update-stack --stack-name ... --template-body file://...
```

이 두가진 `Cli` 명령어는 `ListStacks` `API` 메서드를  호출하여, `Stack` 이 존재하는지 여부를 먼저 확인한다.

이렇게 하는 이유는 `create-stack`, `update-stack` 명령만으로, 새 `Stack` 을 `Provisioning` 하는지 아니면 `update` 하는지 알수 없기 때문이다.

> 이는 초창기 `CloudFormaion` 의 `API` 지원시 문제되었던 점이라고 말한다.

이러한 문제를 해결하기 위해 `create-stack` 및 `update-stack` 시 `ListStacks` 명령을 실행하여 `Stack` 의 존재 유무를 확인하고 처리한다.

생성 작업중에 `Stack` 이 존재하는 경우 다음과 같은 에러가 나온다.

```sh
An error occurred (AlreadyExistsException) when calling the CreateStack operation: Stack [...] already exists
```

다음은, 존재하지 않은 `Stack` 을 업데이트하려고 할때 발생하는 에러이다.

```sh
An error occurred (ValidationError) when calling the UpdateStack operation: Stack [foo] does not exist
```

즉 `Provisioning` 하기전에, 항상 `Template` 를 `Validation` 하여 `Template` 가 사용가능한지 확인한다는 사실을 알수 있다.

하지만 다음의 몇가지 문제가 존재할수 있다.

---

1️⃣ 이러한 방식은 `CI/CD` 파이프라인에서 문제가 발생할 여지가 있다.

`CI/CD` 시, `create-stack` 혹은 `update-stack` 명령을 같이 사용하기 어렵다.

새로운 `Stack` 을 `Provisioning` 하는지, 아니면, 기존 `Stack` 을 업데이트 한느지 알수없으므로, `pipline` 을 통한 처리시, `list-stacks` 를 실행하여 `Stack` 이 있는지 없는지에 따라 실행할 작업을 나누어주어야 한다.

다른 방법은, `Stack` 을 먼저 수동으로 만들고, `CI/CD` 파이프라인에서 `update-stack` 을 실행시키는것이다.

2️⃣ `update-stack` 은 `Stack` 마다 별도로 실행해야 한다.

`update-stack` 은 `Stack` 단위로, 내부의 `Resources` 를 원자적으로 업데이트한다.

> ❗ **`CloudFormation` 의 업데이트방식은 총 3가지로 나누어질수 있다.**
>
> ✳️ **교체 가능한 리소스**: 새로운 리소스를 생성하고 기존 리소스를 교체  
>
> ✳️ **수정 가능한 리소스**: 속성을 변경하는 방식으로 업데이트 (교체없이 업데이트)
>
> ✳️ **파괴후 재생성되는 리소스**: 기존 리소스를 삭제한후 새롭게 리소스를 생성

이러한 특성으로 중요한 `resource` 를 세분화 하지 않으면, 어느시점에 삭제할지 예측이 어렵다.

그렇기에, 여러 `Stack` 업데이트를 위해 `resources` 를 한번에 묶어 `update` 하는것이 아니라, 각 변경 요청은 독립적으로 처리되므로, **작고 세분화된 업데이트가 권장** 된다.

---

이러한 2가지 이유로 인해, 코드 검토에 많은 시간이 할애되며, 개발 속도가 느려질수 있음을 말한다.

이러한 부분을 해결하기 위해 `Change sets` 라는 기능을 제공한다.

### 👀 Deploying stacks using change sets

이 기능은 ***`Stack` 에 적용되는 변경 사항의 집합***이다.

적용할 변경사항이 바람직한지 여부에 따라 `Change sets` 를 만들고, 검토하고, 실행하거나 삭제할수 있다.

```sh
> aws cloudformation create-stack \      
      --stack-name core \
      --template-body file://core_partial.yaml \
      --parameters file://testing.json
```

이렇게 생성하고 나면, `AWS Console` 의 `Core Stack` 에서 `Change sets` 탭을 통해 변경된 집합이 있는지 확인가능하다.

다음은 `change-set` 을 생성하는 `cli` 이다.

```sh
$ aws cloudformation create-change-set \      
      --stack-name core \ # 적용할 stack 이름
      --change-set-name our-change-set \ # 생성할 change set 이름
      --template-body file://core_full.yaml \ # stack 에 적용할 template file
      --parameters file://testing.json \ # stack parameters 에 적용할 parameters file
      --capabilities CAPABILITY_IAM # `IAM`역할, 정책 생성, 수정하는 권한을 요구할경우 이를 승인하기 위해 `CAPABILITY_IAM` 권한 지정

{    
  "Id": «arn:aws:cloudformation:REGION:ACCT_ID:changeSet/our-change-set/bd04aeb3-386b-44d7-a25c-5fe626c24aed»,
  "StackId": "arn:aws:cloudformation:REGION:ACCT_ID:stack/core/00697420-1123-11ea-9d40-02433c861a1c"
}
```

그리고 변경 사항검토를 위해 다음의 명령어로 확인가능하다.

```sh
> aws cloudformation describe-change-set \
      --change-set-name arn:aws:cloudformation:REGION:ACCT_ID:changeSet/our-change-set/bd04aeb3-386b-44d7-a25c-5fe626c24aed

      {    
        "Changes": [
          {
            "Type": "Resource",
            "ResourceChange": {
                "Action": "Add",
                "LogicalResourceId": "AdminRole",
                "ResourceType": "AWS::IAM::Role",
                "Scope": [],
                "Details": []
            }
          },
          {
            "Type": "Resource",
            "ResourceChange": {
                "Action": "Add",
                "LogicalResourceId": "DevRole",
                "ResourceType": "AWS::IAM::Role",
                "Scope": [],
                "Details": []
            }
          },
      # The rest of the output...
      ]
    }
    
```

이를 보면, `Resource` 타입을 가지며, `ResourceChange` 의 `Action` 은 `"Add"` 인것을 볼 수 있다.

이는 새로운 `resource` 를 추가했기에 `"Add"` 로 표시되며, `resource` 를 제거하면, `"Remove"` 로, 수정하면 `"Modify"` 로 표시된다.

이는, `AWS Console` 에서 해당하는 `Stack` 의 `Change sets` 탭에서도 확인가능하다.

변경사항을 검토후에, 만든 `create-set` 을 배포하기만 하면된다.

```sh
> aws cloudformation execute-change-set \
    --change-set-name our-change-set \ # 변경적용할 change-set 의 이름
    --stack-name core # 변경적용할 stack 이름
```

조금더 간단하게 배포하기 위해 `deploy` 명령어가 등장한다.

```sh
> aws cloudformation deploy --stack-name foo --template-file bar
```

`Stack` 생성은 `2` 가지로 나누어지는데, `deploy` 는 해당 `stack` 이 있으면, 새롭게 만들고, 없으면 `update` 한다.

이로인해 더 쉽게 `Stack` 생성이 가능하다.
이는 `create-stack` 및 `update-stack` 과 몇가지 차이점이 존재한다.

1. `change-sets` 를 사용하여 `Stack Provisioning` 을 실행한다.

2. `--template-body` 대신 `--template-file` 인수를 사용한다.

3. `--parameter-file` 을 지원하지 않는다. 대신 `--parameter-overrides` 로 인수에 매개변수를 지정해야 한다.

4. 단순히 응답으로 `Stack ID` 를 반환하는 것이 아니라, 스택이 생성되거나 실패할 때까지 기다린다 (CI/CD 파이프라인에 CloudFormation 명령을 포함할 때 유용하다 ).

만약, `core stack` 을 `deploy` 로 `provisioning` 한다면 다음과 같다.

```sh
> aws cloudformation deploy \
                     --stack-name core \
                     --template-file core_full.yaml \
                     --capabilities CAPABILITY_IAM \
                     --parameter-overrides \
                     VpcCidr="10.1.0.0/16" \
                     Environment="test"
```

이는, `core stack` 이 이미 존재하는경우, 자동적으로 `update` 를 위한 `create-sets` 가 생성된다. (`--change-set-name` 옵션이 없다면, `UUID` 로 저장)

그리고, `create-set` 을 실행시켜 `core stack` 에 적용하는 과정을 `deploy` 라는 짧은 명령어로 사용가능하다.

## :pushpin: Handling errors

`Error` 가 발생하면, `CloudFormation` 의 `Stack` `deploy` 는 이전 상태로 되돌리는 `rollback` 으로 응한다.

이것이 `production infra` 를 관리하는 적절한 방법이기는 하지만, 다양한 문제가 발생할수 있다.

`Termination Protection`(종료 방지 기능) 과 함께 `resource` 를 생성했는데, 해당 `resource` 생성에 `failed` 하면, `CloudFormation` 에서 `cleanup` 할수 없다.

예를들어, `LoadBalancer` 에서 `Termination Protection` 을 `active` 하는 `WebTier` `Stack` 을 생성한다고 해보자.

```yml
# webtier_failing.yaml

Resources:
    WebTierLoadBalancer:
        Type: AWS::ElasticLoadBalancingV2::LoadBalancer
        Properties:
            Type: application
            LoadBalancerAttributes:
                - Key: deletion_protection.enabled
                  Value: true
        # rest of properties
```

이와 동시에, `failed` 하는 `WebInstance` 보안그룹을 같이 생성했다.

```yml
# webtier_failing.yaml

Resources:
    WebTierLoadBalancer:
      # ...resource properties
    WebInstanceSg:
      Type: AWS::EC2::SecurityGroup
      DependsOn: WebTierLoadBalancer
      Properties:
          GroupDescription: WebTier Instance SG
          # SG inbound 설정
          SecurityGroupIngress:
              - IpProtocol: notexist # will fail

                # WebTierLbSg 에서오는 트래픽으로 제한
                SourceSecurityGroupId: !Ref WebTierLbSg
                # 80 port 로 inbound
                FromPort: 80
                # 80 port 로 outbound
                ToPort: 80
          # the rest of properties....
```

여기에서는 `IpProtocol` 은 [ec2-securitygroup-ingress-ipprotocol](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-securitygroup-ingress.html#cfn-ec2-securitygroup-ingress-ipprotocol) 에서 정해진 `tcp`, `udp`, `icmp`, `icmpv6` 혹은 [iana Protocol Numbers](https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml) 를 사용해야 한다.

하지만, 존재하지 않은 `notexist` 를 사용하여, `failed` 된다.
하지만, `DependsOn` 을 보면, `WebInstanceSg` 가 명시적으로, `WebTierLoadBalancer` 를 의존하고있다.

의존하는 `WebTierLoadBalancer` 는 `deletion_protection.enable` 로 삭제방지기능이 활성화된 상태이다.

이는 `failed` 로 인한 `rollback` 이 되어, 모든 `resource` 가 삭제되어야 하는데, `CloudFormation` 은 `deletion_protection.enable` 로 인해 `WebTierLoadBalancer` 를 삭제하지 못한다.

이로인해, `failed` 된 `resource` 를 수정하여 다시 `stack` 을 생성하더라도,
`stack` 업데이트도 안되고 `stack` 을 삭제할수도 없다.

> 💢 **`Stack` 은 이러한 상황에서 삭제되지 않는다.**
>
> `WebInstance` 는 `Stack` 생성시 삭제되지만, `WebTierLoadBalancer` 는 삭제되지 않고 남아있기에, `Stack` 또한 그대로 존재한다.
>
> 즉 다시 `Stack` 을 생성하면, 같은 `Stack` 을 다시 생성한다는 의미가 되므로 `Stack` 생성이 실패한다.

이는 수동으로, `WebTierLoadBalancer` 의 `termination_protection` 기능을 제거하고 `Stack` 을 삭제하거나, 그대로 유지해야 한다.

보통은 이러한 상황을 원치 않는다.
`Stack` 으로 모든 `resource` 를 `controll` 하기 원한다.

이럴때, `--diable-rollback` 옵션을 사용하면, `Stack` 이 `failed` 상태에서 `rollback` 되지 않고, 그대로 유지된다.

> ❗ **`--diable-rollback` 은 `rollback` 을 하지 않고 `Stack` 의 `resource` 는 그대로 남아있게 된다.**
>
> `rollback` 되지 않고 `resource` 가 그대로 있을경우,
>
>***앞의 문제처럼 같은 `Stack` 이 여러개 생기는 문제를 해결할 뿐만 아니라, 기존의 문제되지 않은 `resource` 역시 그대로 유지하면서 문제되는 `resource` 만 처리***해주면된다.
>
> 이로인해 ***`rollback` 되어 삭제된 `resource` 때문에 `Stack` 을 삭제하고, 다시 `Stack` 을 재생성하는것*** 보다 훨씬 효율적으로 처리할수 있게 된다.

이는 `create-sets` 에서 시도할때만 유용하며, 처음 `Stack` 을 만들어도, `Stack` 이 그대로 남아있어 오류를 분석하고 필요한 수정을 하게 도와준다.

하지만, ***`Stack` 업데이트시 사용하지 말라고 한다***
`rollback` 을 비활성화 할경우 `infra` 상태가 예기치 않게 불안정해질수 있기 때문이다.

## 📌 Working with drift

👼 사실 책을 읽으면서 가장 애매한 내용으로 느껴진다.

✳️ `Drift` 는 ***`CloudFormation` 이 자동으로 처리하는 방식으로 인해, 기존의 예상 내용과 다른 방식으로 내용이 적용된다.*** 라는 늬앙스로 느껴진다.

일단 여기서 말하는 `Drift` 가 발생하는 상황은 `IAM` 에서 한정되어 설명한다.

다음은 설정된 `IAM Role` 을 생성하는 `Resource` 이다.

```yml
# iam_role.yamlAWSTemplateFormatVersion: "2010-09-09"
Description: "This is a dummy role"
Resources:
  IamRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          - Sid: AllowAssumeRole
            Effect: Allow
            Principal:
              AWS:
                - !Ref "AWS::AccountId"
            Action: «sts:AssumeRole»
      ManagedPolicyArns:
        - "arn:aws:iam::aws:policy/AmazonEC2FullAccess"
        - "arn:aws:iam::aws:policy/AWSCloudformationFullAccess"
Outputs:
  IamRole:
    Value: !GetAtt IamRole.Arn
```

여기에서 보면 `!Ref AWS::AccountId` 를 통해, `Principal` 을 지정해준것을 볼수 있다.

여기서 의도하는 목적은 `AWS 계정내의 모든 사용자` 를 주체로 하는 의도인듯하다.
하지만, 이는 의도대로 동작안한다. :confused:

```sh
> aws cloudformation deploy \
                     --template-file core_full.yaml \
                     --stack-name core \
                     --parameter-overrides \
                     VpcCidr=10.1.0.0/16 \
                     Environment=test \
                     --capabilities CAPABILITY_IAM;

# stack 생성이후 drift 를 감지하는 cli 이다.
> aws cloudformation detect-stack-resource-drift \
                     --stack-name core \                      --logical-resource-id DevRole;
```

이는 다음처럼 출력된다

```json
// drift 를 감지하여 출력된 json
{  
    "StackResourceDrift": {  
        "StackId": "arn:aws:cloudformation:REGION:ACCT_ID:stack/core/32861920-16a7-11ea-bd3e-064a8d32fc9e",  
        "LogicalResourceId": "DevRole",  
        "PhysicalResourceId": "core-DevRole-G5L6EMSFH153",  
        "ResourceType": "AWS::IAM::Role",  
        "ExpectedProperties": "...",  
        "ActualProperties": "...",  
        "PropertyDifferences": [  
            {  
                "PropertyPath": "/AssumeRolePolicyDocument/Statement/0/Principal/AWS",  
                "ExpectedValue": "ACCT_ID",  
                "ActualValue": "arn:aws:iam::ACCT_ID:root",  
                "DifferenceType": "NOT_EQUAL"  
            }  
        ],  
        "StackResourceDriftStatus": "MODIFIED",  
        "Timestamp": "..."  
    }  
}
```

이기에서 중요한 부분은 `AcutalValue` 부분이다.
이는, `!Ref AWS::AccountId` 값인 `account 계정` 의 `ID` 값이 아니라, `arn` 형식(`"arn:aws:iam::ACCT_ID:root"`)으로 변경된것을 볼수 있다.

> `ACCT_ID` 는 `account 계정` 의 `ID` 값을 의미한다.

이는, `CloudFormation` 이 `Stack` 을 다시 생성하면서, `root` 계정의 `URL` 로 다시 변경한다.

> 이는 `IAM` 의 특성으로, `Principal` 로 `AccountId` 를 제공하면 이를 `root` 사용자로 간주한다.
>
> 이로 인해 `"arn:aws:iam::ACCT_ID:root"` 로 변경해서 적용한 것이다.

이는 원래 의도했던 동작이 아니므로 `Drift` 되었다.
이를 해결하기 위해 책에서는 `Principal` 을 아예 제거한다.

```yml
# core_drift.yaml  
DevRole:  
    Type: "AWS::IAM::Role"  
    Properties:  
      # some properties...  
      ManagedPolicyArns:  
        - !If [ ProdEnv, "arn:aws:iam::aws:policy/ReadOnlyAccess", "arn:aws:iam::aws:policy/AdministratorAccess"]  
        - "arn:aws:iam::aws:policy/AmazonEC2FullAccess"
```

그리고 다시 배포한다.

```sh
$ aws cloudformation deploy \
                     --template-file core_drift.yaml \
                     --stack-name core \
                     --parameter-overrides \
                     VpcCidr=10.1.0.0/16 \
                     Environment=test \
                     --capabilities CAPABILITY_IAM
```

이 경우, 드리프트가 사라지고, `AccountID` 가 `Account 계정 ID` 그대로 사용되는것을 볼수 있다.

> **👼 혼란스럽네?...**
>
> 원리가 애매하다...
>
> `IAM` 에서 `Principal` 로 `AccountID` 를 주면, 자동적으로 `arn:aws:iam:ACC_ID:root` 로 된다는건 이해했다.
>
> 그리고 `IAM` 에서 `Principal` 을 주지 않으면, 모든 접근을 불허하는것으로 알고 있다.
>
> 그런데, `CloudFormation` 에서는 주체를 제외하면 `AccountID` 로 새로 써주네? 😅
>
> 일단, `CloudFormation` 에서 적용되는 규칙이 약간 다르다는점을 인지하자...
> 이것도 뭔가 규칙이 있겠지?

---

## 📌 추가 읽기

✳️ [CloudFormation Linter](https://github.com/aws-cloudformation/cfn-lint)

✳️ [CloudFormation Guard](https://github.com/aws-cloudformation/cloudformation-guard)

✳️ [Resource Import in the Stack](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/resource-import.html)

✳️ [Working with drift](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/using-cfn-stack-drift.html)
