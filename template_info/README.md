# CloudFormation 내부 살펴보기

`cloudformation` 은 몇가지 구성요소로 이루어져있다.
이러한 구성요소는, `JSON` 혹은 `YAML` 포멧으로 작성되며, 무엇을 만들지를 나타낸다

이는 다음의 주제가 존재한다.

## 목록

- [AWSTemplateFormatVersion](#awstemplateformatversion)
- [Description](#description)
- [Metadata](#metadata)
- [Parameters](#parameters)
- [Mappings](#mappings)
- [Resources](#resources)
- [이미 존재하는 Stack 참조](#이미-존재하는-stack-참조)
- [AWS pseudo parameters](#aws-pseudo-parameters)

## AWSTemplateFormatVersion

**_Not Required_**

이는 `CloudFormation` 의 `Template` 작성시 명시할 `Format version` 을 나타낸다.  
`CloudFormation` 역시 `AWS` 에서 제공하는 `API` 이므로, 해당 `API` 의 버전을 타나낸다고 보면된다.

현재까지는 `2010-09-09` 로 고정되어있으며, 추후 `CloudFormation` 버전이 업데이트된다면, 해당 날짜로
고정될수도있다.

그러므로, 꼭 작성하지 않아도 되며, 작성하지 않을시 자동적으로 기본 포멧 버전을 사용한다.

## Description

**_Not Required_**

`CloudFormation` 의 `Description` 은 `Template` 에대한 설명을 위한 주석과 같다고 보면된다.
이 `Template` 의 목적이 무엇이고, 어떻게 사용하면 되는지에 대한 설명이다.

```yaml
# cloudformation template

AWSTemplateFormatVersion: 2010-09-09
# yaml 에서 > 는 멀티라인 문자열을 작성가능하다.
Description: >
    네트워크 템플릿으로 다음으로 구성되어짐:
    - VPC
    - IGW
    - Subnets
    - RouteTables
```

## Metadata

메타데이터 섹션은 추가적인 구성 기능을 제공한다.
다음처럼 사용가능한데, 이는 나중에 살펴보도록 한다.

- `AWS::CloudFormation::Init`
- `AWS::CloudFormation::Interface`
- `AWS::CloudFormation::Designer`

## Parameters

**_Not Required_**

`Template` 를 재사용가능하도록 만들어진 `Stack Variables` 이다.
`Stack` 생성시 지정한 `Parameters` 의 값을 저장하여, 해당 `Stack` 에서 사용되는  
`Template` 에서 `Parameters` 를 참조할수있다.

이는 다음과 같다.

```yml
Parameters:
    InstanceType:
        Type: String

Resources:
    Ec2Instance:
        Type: AWS::EC2::Instance
        Properties:
            # ec2::instance properties...
            InstanceType: !Ref InstanceType
```

이렇게 하면 `Stack` 생성시 `Parameters` 인 `InstanceType` 의 값을 `String` 타입으로 작성해야 하며,  
작성된 값을 `Ec2Instance` 에서 참조해 사용한다.

`Stack` 생성시 사용자 요구에 따라 동적으로 생성가능하도록 구성할수 있다.
이는 꽤나 다양하게 설정가능한데, 다음의 타입과 속성을 제공한다.

### Parameters 타입

**_Not Required_**

| type                         | 설명                                                                                                                                                                           |
| :--------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| String                       | 리터럴 문자열                                                                                                                                                                  |
| Number                       | 정수 또는 부동소수점<br/>`Ref` 내장함수를 사용하여, 참조시 파라미터값은 문자열이 된다.                                                                                         |
| List\<Number\>               | 쉼표로 구분된 정수 또는 부동소수점의 배열.<br/>`Ref` 내장 함수를 사용하여, 참조시 파라미터 값은 문자열 배열이 된다. <br/> `"80, 20" 을 지정하면, Ref 는 ["80", "20"] 이 된다.` |
| CommaDelimitedList           | 쉼표로 구분된 리터럴 문자열 배열.<br/>총 문자열 수는 총 쉼표수보다 하나 더 많다.<br/>`"test,dev,prod"` 는 `Ref` 내장 함수를 통해 `["test", "dev", "prod"]` 가 된다.            |
| AWS 특정 파라미터 type       | `AWS` 관련 파라미터 타입                                                                                                                                                       |
| System Manager 파라미테 type | `Systems Manager Parameter` 타입 참조                                                                                                                                          |

### Parameters 속성

`Parameters` 에서 제공하는 속성들.

#### AllowedPattern

**_Not Required_**

`String` 또는 `CommaDelimitedList` 유형에 허용할 패턴 정규식.  
`String` 타입에 적용할 경우 패턴이 값과 일치해야 한다.
`CommaDelimitedList` 타입에 적용시 각 값과 패턴이 일치해야 한다.

```yml
Parameters:
    SubnetCidr:
        Type: String
        AllowedPattern: '((\d{1,3})\.){3}\d{1,3}/\d{1,2}'
```

#### AllowedValues

**_Not Required_**

`Parameter` 에 허용되는 값 목록.  
`String` 타입에 적용할 경우 `Parameter` 값은 허용된 값중 하나여야 한다.  
`CommaDelimitedList` 타입에 적용할 경우 `Parameter` 배열의 각 원소값은 허용된 값중 하나여야 한다.

```yml
Parameters:
    Environment:
        Type: String
        AllowedValues: [dev, test, prod]
```

값은, `dev`, `test`, `prod` 중 하나만 허용한다.

#### ConstraintDescription

**_Not Required_**

제약위반신 해당 제약을 설명하는 문자열.

> 예를 들어, 해당하는 타입이 아닌 다른 타입을 작성했던지, 속성에 맞지 않은 값을 지정했을시 나타내는
> 경고 및 에러 문자열 (에러 처리)

#### Default

**_Not Required_**

스택 생성시 지정된 값이 없는 경우에 사용할 기본값.

> 제약 정의시, 이러한 제약을 준수하는 기본값을 설정하는것이 좋다고 한다.

```yml
Parameters:
    DockerImageVersion:
        Type: String
        Default: latest # 아무런 값 지정 없을시 기본값
```

#### "Description"

**_Not Required_**

`Parameter` 를 설명하는 설명구.

#### MaxLength

**_Not Required_**

`String` 유형에 해당하며, 문자열의 최대 문자수를 결정하는 정수값

#### MaxValue

**_Not Required_**

`Number` 유형에 해당하며, 최대 숫자값을 결정하는 정수값

#### MinLength

**_Not Required_**

`String` 유형에 해당하며, 문자열의 최소 문자수를 결정하는 정수값

#### MinValue

**_Not Required_**

`Number` 유형에 해당하며, 최소 숫자값을 결정하는 정수값

#### NoEcho

**_Not Required_**

콘솔, 명령줄 도구 또는 `API` 에 표시되지 않도록 `Parameter` 값을 마스킹 처리할지 여부.
`NoEcho` 를 `true` 로 설정할 경우, 지정된 위치에 저장된 정보를 제외한다.

> `NoEcho` 는 속성을 사용해도 다음에 저장되는 정보는 마스킹되지 않는다.
>
> - `Metadata` 템플릿 섹션: `CloudFormation` 은 `Metadata` 정보를 변환, 수정, 삭제하지 않는다.
> - `Outputs` 템플릿 섹션: `Outputs` 는 로그파일, `API` 응답, 실행중 반환된 응답, 보안그룹 및 정책에 포함될수 있으므로, `NoEcho` 로 보호되지 않아야 한다.

```yml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
    MyDBInstance:
        Type: 'AWS::RDS::DBInstance'
        Properties:
            DBName: mydatabase
            Engine: MySQL
            MasterUsername: admin
            MasterUserPassword: !Ref DBPassword
            DBInstanceClass: db.t2.micro
            AllocatedStorage: 20

Parameters:
    DBPassword:
        Description: 'The database admin account password'
        Type: 'String'
        NoEcho: true # 비밀번호는 출력되지 않도록 설정

Outputs:
    DatabaseConnectionString:
        Description: 'The connection string for the RDS database'
        Value:
            !Join [':', ['mysql://admin', !Ref MyDBInstance.Endpoint.Address]]
        Export:
            Name: 'DBConnectionString'
```

이 같은경우, `DBPassword` `Parameter` 는 `CloudFormation` 의 `cli` 혹은, 콘솔상에 마스킹되어 노출되지 않는다.

## Mappings

**_Not Required_**

`Mappings` 는 `Parameters` 와 비슷하지만, 사전에 정의된 형식으로 제공된다.

> `Parameters` 는 `Stack` 생성시 동적으로 할당한다.

이렇게 미리 작성된 변수를 사용하기 위한 역할로써 제공된다.

### Fn::FindInMap

`Fn::Ref` 함수를 사용하여 값을 얻는 `Parameters` 와 달리,  
`Fn::FindInMap` 함수를 사용하여 값을 얻을수 있다.

```yml
Mappings:
  RegionMap:
    us-east-1:
      HVM64: ami-0ff8a91507f77f867
      HVMG2: ami-0a584ac55a7631c0c
    us-west-1:
      HVM64: ami-0bdb828fd58c52235
      HVMG2: ami-066ee5fd4a9ef77f1
    eu-west-1:
      HVM64: ami-047bb4163c506cd98
      HVMG2: ami-0a7c483d527806435
    ap-northeast-1:
      HVM64: ami-06cd52961ce9f0d85
      HVMG2: ami-053cdd503598e4a9d
    ap-southeast-1:
      HVM64: ami-08569b978cc4dfa10
      HVMG2: ami-0be9df32ae9f92309
Resources:
  Ec2Instance:
    Type: "AWS::EC2::Instance"
    Properties:
      ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", HVM64]
```

```json
    // json 에서는 다음처럼 선언한다.
    "Properties": {
        "ImageId": {
            "Fn::FindInMap": [
                "RegionMap",
                {
                    "Ref": "AWS::Region"
                },
                "HVM64"
            ]
        }
    }

```

`Fn::FindInMap` 은 `MapName`, `TopLevelKey`, `SecondLevelKey` 로 해당하는 값을 찾아 할당한다.
`Mappings` 는 변경되지 않는 상수값을 지정하는 형식이라고 보면 된다.

## Conditions

**_Not Required_**

어떠한 조건에 의해 `Template` 의 `Resource` 생성여부를 제어하는데 사용된다.  
`Conditions` 는 `true` 혹은 `false` 값을 받는데,  
이를 통해 해당 `resource` 를 사용할지 안할지를 결정할수 있다.

> `Conditions` 에서 `Boolean` 값을 받아 처리한다고 해서, 수동으로 `true` 나 `false` 를 선언하지  
> 않아야 한다.
>
> 반드시, [Fn::Equals](#fnequals), [Fn::If](#fnif), [Fn::Not](#fnnot), [Fn::And](#fnand), [Fn:Or](#fnor), 같은 조건을 정의하는 함수를 통해 비교하여 만들어지 값이어야 한다.

```yml
AWSTemplateFormatVersion: '2010-09-09'
Parameters:
  Env:
    Default: dev
    Description: Define the environment (dev, test or prod)
    Type: String
    AllowedValues: [dev, test, prod]
Conditions:
  IsProd: !Equals [!Ref Env, 'prod']
Resources:
  Bucket:
    Type: "AWS::S3::Bucket"
    Condition: IsProd
```

위의 예시에서 `IsProd` 는 `Equals` 함수를 사용하여, `!Ref Env` 값과 `'prod'` 값이 같으면,  
`ture` 가 되며, `Bucket` 이 생성된다.

그렇지 않다면, `false` 가 되어 `Budket` 은 생성되지 않는다.

### Fn::Equals

첫번째 값이, 두번째 값과 같은지 확인 맞으면 `true`, 아니면 `false`

```yml
Conditions:
  IsProd: !Equals [!Ref Env, 'prod']
```

### Fn::If

첫번째 값이 `true` 이면, 두번째값, 아니면 세번째값

```yml
Properties:
    ImageId:
        Type: String
        Description: Define image ID

    Env:
        Type: String
        Default: dev
        AllowedValues: [prod, dev, test]
        Description: Define the environment

Conditions:
    IsProd: !Equals [!Ref Env, 'prod']

Resources:
    WebLt:
        Type: AWS:EC2:LaunchTemplate
        Properties:
            LaunchTemplateName: web
            LaunchTemplateData:
                ImageId: !Ref ImageId
                InstanceType: !If [!Ref IsProd, 'm5.large', 't3.micro']
```

### Fn::Not

들어간 값이 `true` 면 `false` 로 부정.
`false` 면 `true` 로 반환

```yml
Conditions:
    IsNotProd: !Not [!Equals [!Ref Env, 'prod']]
```

위는 `Fn::Equals` 의 부정을 나타낸다.

### Fn::Or

주어진 조건중 하나라도 `true` 이면, `true` 아니면, `false`

```yml
Properties:
    Env:
        Type: String
        Default: dev
        AllowedValues: [prod, dev, test]
        Description: Define the environment

Conditions:
    IsDevOrTest: !Or
        - !Equals [!Ref Env, 'dev']
        - !Equals [!Ref Env, 'test']
        # Env 가 dev 혹은 test 이면 true
        # 아니면 false
```

### Fn::And

주어진 조건중 하나라도 `false` 이면 `false`, 전부다 `true` 이면 `true`

```yml
Properties:
    Env:
        Type: String
        Default: dev
        AllowedValues: [prod, dev, test]
        Description: Define the environment
    Region:
        Type: String
        Default: ap-northeast-2
        AllowedValues: [ap-northeast-1, ap-northeast-2, ap-southeast-1]

Conditions:
    IsProdAndUseRegion: !And
        - !Equals [!Ref Env, 'prod']
        - !Equals [!Ref Region, 'ap-northeast-2']
    # Env 가 prod 이고, Region 이 ap-northeast-2 이면 true
    # 아니면 false
```

## Transform

`CloudFormation` 에서 특정 `Macro` 를 실행하려고할때 선언된다.
이는 특수한 부분이므로, 추후 설명한다.

## Resources

**_Required_**

이는, `CloudFormation` 에서 유일하게 반드시 작성되어야 하는 섹션이다.
`Resources` 섹션은 프로비저닝하고자 하는 `resource` 를 제공한다.

[CloudFormation 지원 리소스](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html) 에서 `CloudFormation` 에서 제공하는 `resource` 가 무엇인지 알려준다

> 양이 많다..

### Resources 의 Attributes

여기서는 `Resources` 속서에 대한 개요만 살펴본다.

- **Type**: `Resource` 의 `type` <br/> `ec2 subnet`, `ec2 instance`, `dynamodb table` `lambda` 등등 많다.

- **Properties**: 해당 `Resource` 타입의 설정인자값

- **DependsOn**: 직접 종속성을 추가한다.

- [CreationPolicy](#): `Stack` 에서 해당하는 `resource` 가 정상적으로 생성되었음을 확인하여,  
     이 조건이 충족될때까지 기다리도록 한다.  
     즉, **_`resource` 의 `완료 신호`(`success signal`)_** 을 기다린다.

- [DeletionPolicy](#deletionpolicy): `Stack` 에서 해당하는 `resource` 를 삭제할때 `CloudFormation` 이 `Resource` 를  
     어떻게 처리할지 지시하는데 사용된다.  
     예를들어, 삭제시 **_데이터 손실을 방지_** 하고자, 삭제전에 `resource` 를 백업하기 위해  
     **_삭제 동작을 변경_** 할수있다.

- [UpdatePolicy](#): `Stack` 에서 해당하는 `resource` 를 업데이트 할때 `CloudFormation` 이 `Resource` 를  
     어떻게 처리할지 지시하는데 사용된다.  
     이는 **_리소스를 업데이트하는 과정을 제어하여, 리소스가 중단되지 않고도 안정적으로 업데이트_** 되도록 한다.

- **UpdateReplacePolicy**: `Stack` 에서 해당하는 `resource` 를 새로운 `resource` 로 교체할때  
     `CloudFormation` 이 `Resource` 를 어떻게 처리할지 지시한다.  
     업데이트중 **_리소스가 삭제되지 않거나, 삭제 되더라도 스냅샷을 생성하도록 하여 데이터 손실을 방지_** 한다.

#### DeletionPolicy

`Stack` 삭제시, 중요한 `Resource` 가 실수로 삭제되어서는 안된다.
`EnableTerminationProtection` 을 사용하여, `RDS` 및 `EC2` 의 삭제를 보호할수 있다.

```yml
Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      EnableTerminationProtection: true
      ...
```

이 같은경우 `Properties.EnableTerminationProtection` 을 통해, `Termination` 되지 않도록 보호하는기능이다.

하지만, `CloudFormation` 에 삭제 정책을 사용하여 처리도 가능하다.
이때 사용하는것이 `DeletionPolicy` 이다.

이는 다음의 `3` 가지 옵션이 존재한다.

- [Retain](#retain): 리소스를 삭제하지 않고 보존.
- [Snapshot](#sanpshot): 리소스 삭제전에 스냅샷 생성.
- [Delete](#delete): 기본값으로 리소스를 삭제

##### Retain

만약 `Stack` 삭제시 `EC2` 를 삭제하고 싶지 않다면 다음처럼도 가능하다.

```yml
Resources:
    MyEC2Instance:
        Type: AWS::EC2::Instance
        DeletionPolicy: Retain
        Properties: ...
```

> 물론 `EnableTerminationProtection` 과 `DeletionPolicy: Retain` 과는 목적자체가 다르다.
> `EnableTerminationProtection` 은 `EC2` 인스턴스 자체를 종료하지 못하도록 활성화 시킨것으로
> 사용자가 콘솔및 cli 를 통해서도 종료를 못하며, 종료하기 위해서는 명시적으로 해제해야 한다.
>
> 반면, `DeletionPolicy: Retain` 은 `CloudFormation` 을 통핸 `Stack` 에서 삭제시에만 해당 `Resource` 삭제를
> 보호한다.

##### Sanpshot

테스트 단계가 끝나면 필요 없는 테스트 `Stack` 을 만들었지만 동일한 데이터 구조를 다시 만들고 싶지 않을수 있다.
프로덕션 환경으로 옮기고 싶은 데이터가 있을수 있으므로 `DB Dump` 가 필요할수 있다.

이때 다음처럼 `Snapshot` 을 사용할수 있다.

```yml
Resources:
    VeryImportantDb:
        Type: AWS::RDS::Instance
        DeletionPolicy: Snapshot
        Properties:
        # Here, you set properties for your RDS instance.
```

이렇게 하면, `Stack` 삭제시 `CloudFormation` 은 `RDS` 에 `instance` 의 최종 `Snapshot` 을 찍은다음
삭제하도록 한다.

`DeletionPolicy` 의 `Snapshot` 은 모든 `Resource` 가 아닌 사용가능한 `Resource` 들이 존재한다.

- `AWS::EC2::Vloume`
- `AWS::ElasticCache::CacheCluster`
- `AWS::ElasticCache::ReplicationGroup`
- `AWS::Neptune::DBCluster`
- `AWS::RDS::DBCluster`
- `AWS::RDS::DBInstance`
- `AWS::RDS::Cluster`

##### Delete

`Delete` 는 간단하다. `Stack` 삭제시 해당 `Resource` 도 같이 삭제된다.
이는 기본값이다.

### Outputs

`Outputs` 는 `Stack` 이 생성된후 `Stack` 에서 내보내는 값이다.
이렇게 내보내진 값은, `Stack` 에서 생성된 리소스를 다른 `Stack` 에서 참조하여 사용도 가능하다.

이러한 방식은 `shared stack` 이라고 하는데, `Application` 별 `Stack` 이 있는 대규모 `workload` 에 대한
모범 사례라고 한다.

> `Outputs` 는 `read` 권한이 있는 모든 사람이 엑세스할수 있으므로, 이렇게 노출하는 것을 위험하다.  
> 반드시, 권한있는 사용자만 `stack` 에 접근할수 있도록 해야 한다.

## 이미 존재하는 `Stack` 참조

기존에 존재하는 `Stack` `Resource` 를 관리하는 공유 책임 모델(`Shared responsibility models`)이 적용된 대규모 환경에서,
여러 `Stack` 상의 `Resource` 나 `Resource Attributes` 를 공유해야 하는 상황이 발생한다.

`Network Stack` 을 생성해서 `Resource` 를 만들었다면, 이러한 `VPC` 및, `Subnet` 의 `Attributes` 를 다른 `Stack`에서
참조해야 하는 경우가 발생한다.

> `EC2`, `AutoSacaling`, `LoadBalencer`, `DB Cluster` 배포 시 `VPC ID` 를 지정해야 한다.

해당 `Subnet` 의 `ID` 를 매개변수로 지정한다음, `Fn::Ref` 를 사용하여 `Resource` `Attributes` 를
`Mapping` 할수 있지만, 이는 불필요한 작업을 많이 요구한다.

이러한 불필요한 작업은 `Outputs` 의 `Exports` 와 `Fn::ImportValue` 를 사용하여 회피가능하다.

`Fn::ImportValue` 는 `Outputs.Exports` 를 통해 내보내진 값에서, 필요한 속성을 조회하고 `Resource` 에
필요한 항목을 작성한다.

다음을 보자.

```yml
# core.yamlParameters:
# ...
Resources:
  Vpc:
    Type: "AWS::EC2::VPC"
    Properties:
      # ...
  PublicSubnet1:
    Type: "AWS::EC2::Subnet"
    Properties:
      # ...
      # The rest of our resources...

Outputs:
  VpcId:
      Value: !Ref Vpc
      Export:
        Name: Vpc
  PublicSubnet1Id:
    Value: !Ref PublicSubnet1
    Export:
      Name: PublicSubnet1Id
  PublicSubnet2Id:
    Value: !Ref PublicSubnet2
    Export:
      Name: PublicSubnet2Id
  PublicSubnet3Id
    Value: !Ref PublicSubnet3
    Export:
      Name: PublicSubnet3Id
    # And so on
```

이는 `core Stack` 이다.
여기에서보면, `Outputs` 에서 `Export` 를 통해 내보낼 값의 이름을 지정하고,
`Value` 에 각 `Ref` 로 참조한 값을 할당한다.

> `Fn::Ref` 로 참조한 `Resource` 는 `Resource` 의 `ID` 값을 반환한다.

이렇게 만들어진 `core Stack` 에서 내보낸진 값을 `webtier Stack` 에서 참조한다.

```yml
# webtier.yamlResources:
WebTierAsg:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
        # Some properties..
        VpcZoneIdentifier:
            - !ImportValue PublicSubnet1Id
            - !ImportValue PublicSubnet2Id
            - !ImportValue PublicSubnet3Id
    # And so on...
```

책에서는 이부분을 `Refectoring` 하는 방법을 소개한다.
이는 다음처럼 `Fn::Join` 과 `Fn::Split` 을 사용하여 `Refectoring` 한다.

```yml
# core.yamlOutputs

Outputs:
    VpcId:
        Value: !Ref Vpc
        Export:
            Name: Vpc
    PublicSubnet1Id:
        Value: !Ref PublicSubnet1
        Export:
            Name: PublicSubnet1Id
    PublicSubnet2Id:
        Value: !Ref PublicSubnet2
        Export:
            Name: PublicSubnet2Id
    PublicSubnet3Id:
        Value: !Ref PublicSubnet3
        Export:
            Name: PublicSubnet3Id
    PublicSubnetIds:
        Value:
            !Join [
                ',',
                !Ref PublicSubnet1Id,
                !Ref PublicSubnet2Id,
                !Ref PublicSubnet3Id,
            ]
        Export:
            Name: PublicSubnetIds
    # And so on
```

```yml
# webtier.yamlResources:
Resources:
    WebTierAsg:
        Type: AWS::AutoScaling::AutoScalingGroup
        Properties:
            # Some properties...
            VpcZoneIdentifier: !Split [',', !importValue PublicSubnetIds]
        # And so on...
```

이렇게 하면, 더 쉽게 관리할수 있도록 할수 있다.
출력/내보내기는 조건부 함수도 지원한다.

다음은 데이터베이스 자격증명을 내보내는 예시이다.

```yml
# database.yamlOutputs:
DbCredentials:
    Value: !If [ProdEnv, GeneratedDatabaseSecret, HardcodedDatabaseSecret]
    Export:
        Name: DbCredentials
```

이 경우, `Value` 값은, `ProdEnv` 가 `true` 일 경우 `GeneratedDatabaseSecret` 이 되고,
그렇지 않은 경우 `HardcodedDatabaseSecret` 이 된다.

이러한 코드는 다음처럼 정리할수도 있다.

```yml
DbCredentials:
    Value:
        Fn::If:
            - ProdEnv
            - GeneratedDatabaseSecret
            - HardcodedDatabaseSecret
    Export:
        Name: DbCredentials
```

이 경우 어떠한 코드가 더 깔끔한지에 따라 선택적으로 사용가능하다.

## AWS pseudo parameters

이 매개변수들은 `AWS` 에서 제공하는 특벌한 매개변수들이다.
다음은 `CloudFormaion` 에서 제공되는 `peseudo parameters` 이다.

- [AWS::AccountId](#awsaccountid)
- [AWS::NotificationARNs](#awsnotificationarns)
- [AWS::NoValue](#awsnovalue)
- [AWS::Region](#awsregion)
- [AWS::StackId](#awsstackid)
- [AWS::StackName](#awsstackname)
- [AWS::URLSuffix](#awsurlsuffix)
- [AWS::Partition](#awspartition)

> [peseudo parameters](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/pseudo-parameter-reference.html) 에서 확인가능하다.

### AWS::AccountId

`IAM principal` 즉 주체를 `AccountId` 로 사용하는 경우에 사용한다.
동시에, `AWS` 계정 `Id` 를 노출하는 것은 보안상 위험하다.

운영환경에서는 이러한 종류의 데이터를 참조할때, `AWS` 에서 제공하는
`peseudo parameter` 인 `AWS::AccountId` 를 통해 참조한다.

```yml
Resources:
    MyIamRole:
        Type: AWS::IAM::Role
        Properties:
            AssumeRolePolicyDocument:
                Version: 2012-10-17
                Statement:
                    - Sid: AllowRoleAssume
                      Effect: Allow
                      Action:
                        - sts:AssumRole
                      Principal:
                        AWS: !Ref AWS::AccountId
```

이를 통해 `AWS::Role` 생성시, 적용되는 신뢰기반 정책의 `principal` 은
`AWS::AccountId` 로 계정내의 모든 사용자 및 서비스에게 정책을 부여한다.

다음은, 계정내의 모든 사용자를 지정하는 예시이다.

```yml
Parameters:
    awsUsername:
        Type: String
        Default: testUser
        Description: AWS User name parameter
Resources:
    MyIamRole:
        Type: AWS::IAM::Role
        Properties:
            AssumeRolePolicyDocument:
                Version: 2012-10-17
                Statement:
                    - Sid: AllowRoleAssume
                      Effect: Allow
                      Action:
                        - sts:AssumRole
                      Principal:
                        AWS: !Sub 
                            - "arn:aws:iam::${AWS::AccountId}:user/${awsUsername}"
```

> `Fn::Sub` 함수는 문자열내의 변수를 치환하는 함수이다.

위 같은경우, `Principal` 이 `iam` 계정에 속하는, 특정 `user` 가 된다.

### AWS::NotificationARNs

현재 `Stack` 에 대한 `Notification` `ARN` 목록을 반환한다.
> 이는 `Stack` 과 연결된 `Amazon SNS` 에 대한 `ARN` 목록이다.

이러한 `Notification` 은 `Stack 생성`, `Update`, `Delete` 같은 이벤트 발생시
이를 모니터링하고 `alarm` 을 받기 위해 연결된 `SNS Topic` 의 `ARNs` 이다.

> 이렇게 `Notification` 을 사용하는 이유는, `Stack` 이벤트에 대한 알림을 받아, 보고서를 작성하고나, 연속된 다른 동작을 처리할수 있도록 하기 위함이다.

해당 목록에서 단일 `ARN` 을 가져오려면 `Fn::Select` 를 사용하여 가져온다.

```yml
myASGrpOne:
  Type: AWS::AutoScaling::AutoScalingGroup
  Version: '2009-05-15'
  Properties:
    AvailabilityZones:
    - "us-east-1a"
    LaunchConfigurationName:
      Ref: MyLaunchConfiguration
    MinSize: '0'
    MaxSize: '0'
    NotificationConfigurations:
    - TopicARN:
        Fn::Select:
        - '0'
        - Ref: AWS::NotificationARNs
      NotificationTypes:
      - autoscaling:EC2_INSTANCE_LAUNCH
      - autoscaling:EC2_INSTANCE_LAUNCH_ERROR
```

이는 `AWS::NOtificationARNs` 목록중 첫번째 `arn` 이 해당 리소스를 모니터링한다.

`autoscaling` 시 `EC2 Instance` 가 `Launch` 될때와 `Launch Error` 가 발생할때를 포착하여 알림을 보낸다.

### AWS::NoValue

`AWS::NoValue` 는 `null` 가 동일하다.
이는 조건함수와 같이 사용하지 않는한, 사용용도는 많지 않다.

다음의 예시는 `AWS::NoValue` 를 사용하여, `snapshot` 생성하는지 여부를
`Fn::If` 를 사용하여 확인한다.

`true` 라면, `DBSnapshotName` `Parameter` 값을 사용하고,
그렇지 않다면 `AWS::NoValue` 를 사용한다.

`AWS::NoValue` 가 될시에, `DBSnapshotIdentifier` 는 작동되지 않는다.

```yml
Properties:
    DBSnapshotName:
        Type: String
        Description: DB snapshot name
        Default: ""
    
Conditions:
    UserDBSanpshot: !Not
        - !Equals [!Ref DBSnapshotName]
        - ""

Resources:
    MyDB:
        Type: AWS::RDS::DBInstance
        Properties:
            AllocatedStorage: "5"
            DBInstanceClass: "db.t3.micro"
            Engine: MySQL
            EngineVersion: "5.7"
            DBSnapshotIdentifier:
                Fn::If:
                    - UserDBSnapshot
                    - !Ref DBSnapshotName
                    - !Ref AWS::NoValue
```

### AWS::Region

`Resource` 사용시 `AWS::Region` 을 지정할수 있다.
`AWS` 에서 `Region` 을 하드코딩하여 작성할때, `cfn lint` 는 `Error` 를
내뿜는다.

이는 다른 리전에 동일한 `Stack` 을 배포할때 문제가 될수 있으므로, 다중 리전 배포로 확장시 `peseudo parameter` 를 사용할것을 권장한다.

> 그외에도 `AWS::Region` 을 사용할것을 계속 권장한다.
> 실수가 많았나보다..

다음은 `CloudWatch` 로그를 `log driver` 로 사용할때, `log groups` 와 `AWS` 의
`Region` 을 지정해주어야 한다.

```yml
Resources:
    EcsTaskDefinition:
        Type: AWS::ECS::TaskDefinition
        Properties:
            # some properties
            ContainerDefinition:
                - Name: myContainer
                  # some container properties
                  LogConfiguration:
                    LogDriver: awslogs
                    Options:
                        awslogs-group: mylogggroup
                        awslogs-region: !Ref AWS::Region
                        awslogs-stream-prefix: ""
```

`CloudFormation` 은 `Stack` `Region` 에 `CloudWatch` 로그 그룹을 생성한다.

### AWS::StackId

`aws coloudformation create-stack` 명령을 내릴때, 지정된 `Stack ID` 를
반환한다.

> 예시)
>
> - arn:aws:cloudformation:us-west-2:123456789012:stack/teststack/51af3dc0-da77-11e4-872e-1234567db123.

스택의 고유한 `ID` 이므로, 특정 스택과 연관된 리소스나 속성에서 `Stack ID` 를
참조할수 있도록 한다.

```yml
Resources:  
  MyBucket:  
    Type: AWS::S3::Bucket  
    Properties:  
      BucketName: !Sub "my-bucket-${AWS::AccountId}-${AWS::Region}"  
      Tags:  
        - Key: StackId  
          Value: !Ref AWS::StackId
```

이는, `Tag` 를 사용하여, 어떠한 `Stack` 에서 생성된 `resource` 인지, `StackId` 명시할때 많이 사용한다.

### AWS::StackName

`aws cloudformation create-stack` 명령으로 지정된 `Stack name` 을 반환한다.

> 예시)
> `Stack` 이름이 `teststack` 이라면,
>
> - teststack

`AWS::StackName` 은 [리소스태킹](#리소스-태깅), [리소스 이름지정](#리소스-이름지정), [로그그룹](#로그그룹), [IAM 역할 생성](#iam-역할-정책에-스택이름-포함) 등에 많이 사용된다.

#### 리소스 이름지정

다음은, `resource` 이름지정의 예시이다.

```yml
Properties:
    BucketName:
        Type: String
        Description: Bucket Name
        Default: test

Resources:
    MyBucket:
        Type: AWS::S3::Bucket
        Properties:
            BucketName: !Sub "${AWS::StackName}-${!Ref BucketName}"
```

`Bucket` 의 이름은 고유해야 하므로, 이렇게 하면 고유한 `BucketName` 을
지정할수 있다.

#### 리소스 태깅

여러 `Stack` 에서 동일한 유형의 `Resource` 를 만들때, `Resource` 에 `Tag` 를 걸어 `Stack name` 을 포함시킨다.

이렇게 하면, `Tag` 를 통해 관리 및 추적을 용이하게 할수 있다.

```yml
Resources:
    MyInstance:
        Type: AWS::EC2::Instance
        Properties:
            InstanceType: t2.micro
            Tags:
                - Key: "StackName"
                  Value: !Ref AWS::StackName
```

#### 로그그룹

`CloudWatch` 에 로그그룹을 생성할때, `resource` 에 대한 고유 식별자로 `StackName` 을 사용할수 있다.

```yml
Resources:  
  MyLogGroup:  
    Type: AWS::Logs::LogGroup  
    Properties:  
      LogGroupName: !Sub "/aws/lambda/${AWS::StackName}-log-group"
```

이렇게 어떤 `Stack` 의 로그 그룹인지 분별 가능하다.

#### IAM 역할 정책에 스택이름 포함

`IAM Role` 생성시, `StackName` 을 포함하여, 어떤 `Stack` 의 역할인지 분별가능하게 만든다.

```yml
Resources:  
  MyIamRole:  
    Type: AWS::IAM::Role  
    Properties:  
      RoleName: !Sub "${AWS::StackName}-role"  
      AssumeRolePolicyDocument:  
        Version: '2012-10-17'  
        Statement:  
          - Effect: Allow  
            Principal:  
              Service: lambda.amazonaws.com  
            Action: sts:AssumeRole
```

### AWS::URLSuffix

`Domain` 의 접미사(`Suffix`) 를 반환한다.
접미사는 일반적으로  `amazonaws.com` 이지만, `Region` 에 따라 다를수 있다.

중국, `AWS GovCloud` 와 같은 특주지역에 애플리케이션 배포시 고유한 `URL` 접미사를 사용해야 한다.

> 중국 베이징은 `amazonaws.com.cn` 이라 한다.

`AWS::URLSuffix` 를 사용하는 경우는, `IAM` 역할같은 서비스에 대한 `URL` 접미사가 필요할때이다.

```yml
Resources:  EcsTaskExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: 2012-10-17
        Statement:
          - Sid: AllowAssumeRole
            Effect: Allow
            Principal:
              Service: !Sub "ecs-tasks.${AWS::URLSuffix}"
            Action: sts:AssumeRole
      # Some role properties...
```

위의 예시는 `ecs` 클러스터에서 컨테이너를 관리하고 배포하는데 사용되는 서비스이다.

이러한 서비스에 `Role` 이 부여되는 주체가 특정 서비스임을 볼수 있다.
이 서비스의 후미 접미사는, `amazonaws.com` 일수있지만, 특수한 지역일 경우 다른 접미사가 붙을수 있다.

이러한 부분을 고려하여, 서비스 주소의 접미사를 사용할시 `AWS::URLSuffix` 를 사용할것을 권장한다.

> `cfn lint` 에서도 `error` 를 뿜뿜한다.

### AWS::Partition

`ARN` 작성시, 다음처럼 작성한다.

`arn:aws:service:region:account_id:...`

여기서, `aws` 부분은 `partition` 을 말한다.
이를 `namespace` 로 이해하면 편하다.

이렇나 `partition` 은 앞의 [AWS::URLSuffix](#awsurlsuffix) 와 같이, 중국 리전 혹은 `GovCloud Region` 같은 경우, 전혀 다른 `partition` 을 사용할수 있다.

> 중국은, `aws-cn` 이고, `GovCloud Region` 은 `aws-us-gov` 처럼 `partition` 을 작성해야 한다.

이러한 상황이 있을수 있으므로, 작성시 `AWS::Partition` 으로 명시하기를 권장한다.

## Parameter Store 와 Secrets Manager

`JSON` 으로 관리되는 `Parameters`([Parameters](#parameters) 혹은 [Mappings](#mappings)) 만 살펴보았다.

보통은 `VCS`(`Version Conrol System`) 에서 `Parameter` 를 유지하는것이
`IaaC` 의 일반적인 관행이지만, 이로인한 추가적인 복잡성이 발생한다.

> `push` 시 `Github action` 에 의해 자동적으로 `aws stack` 생성을 실행하며, 이때, `secret` 으로 `Parameters` 에들어간 값을 지정하여 삽입가능하다.

만약 `VCS` 에 `Parameters` 를 저장한다면, 보안에 예민한 `data` 를 살펴보고 돌봐야만 한다.

> 예를 들어, `password` 혹은 `credentials` 같은 `data` 를 말한다.

또한, `cli` 를 통해 전달한다면, 타이핑시 실수나 오타가 발생하지 않도록 해야한다.

이러한 부분을 쉽게 관리하기 위해, `AWS` 는 `Parameters` 를 관리하기 위한 `AWS SSM`(`AWS System Manager`)`Parameter Store` 와 `Secret Manager` 를 제공한다.

이러한 `Parameter Store` 와 `Secret Manager` 는 버전 관리를 지원하며, 실수가 발생한 경우 `Parameter` 를 이전 `Version` 으로 되돌리고 `Stack` 을 다시 배포할수 있다.

또한, `CloudFormation` 개발을 담당하지 않는 개발자에게 `Instance type` 을 변경하거나 `AutoScaling` 그룹의 크기를 늘리는 등 `Stack` 에 약간의 변경을 적용할수 있는 기능을 제공할수도 있다.

```yml
// core.yamlResources:
Resources:
# Some resources...
  WebAsgMinSize:
    Type: "AWS::SSM::Parameter"
    Properties:
      Type: String
      Name: "/my-app/web/asg-min-size"
      Value: "1"
  WebAsgMaxSize:
    Type: "AWS::SSM::Parameter"
    Properties:
      Type: String
      Name: "/my-app/web/asg-max-size"
      Value: "1"
  WebAsgDesiredSize:
    Type: "AWS::SSM::Parameter"
    Properties:
      Type: String
      Name: "/my-app/web/asg-desired-size"
      Value: "1"
# The rest...
```

그리고 `SSM` 매개변수를 출력해본다.

```yml
Outputs:
    WebAsgMinSize:
        Value: !Ref WebAsgMinSize
        Export:
            Name: WebAsgMinSize
    WebAsgMaxSize:
        Value: !Ref WebAsgMaxSize
        Export:
            Name: WebAsgMaxSize
    WebAsgDesiredSize:
        Value: !Ref WebAsgDesiredSize
        Export:
            Name: WebAsgDesiredSize
```

이를 통해 `Stack` 이 사용할 세가지 매개변수를 만들었다.
다음은 `WebTire Stack` 에서 이를 참조한다.

이렇게 참조하는 방식은 총 2가지가 있다.

1. [`Parameters` 에서 참조](#parameters-에서-참조)

2. [Resource 선언 참조](#resource-선언-참조)

### Parameters 에서 참조

[Parameters](#parameters) 섹션에서 `SSM Parameter` 를 참조하면,
`Stack` 생성시 사용자가 값을 입력하거나, 다른 `Stack` 이나 `SSM Parameter Store` 에서 값을 가져와서 참조할수 있다.

`SSM Parameter Store` 에 저장된 값을 가져오면, 사용자가 직접 입력할 필요없이 해당 값을 참조할수 있다.

이를 참조하기 위해서는 `Parameter` `Type` 을 `AWS::SSM:Parameters::Value<Type>` 으로 지정하며, `Defalut` 값을 사용하여 `SSM Parameter Store` 에 저장된 `Parameter` 이름을 작성해준다.

```yml
Parameters:  
  WebTierDesierdSizeParameter:  
    Type: AWS::SSM::Parameter::Value<String>  
    Default: "/my-app/web/asg-desired-size"  
    Description: "SSM Parameter Store에서 가져온 웹 티어의 기본 작업 수"
```

> `AWS::SSM:Parameters::Value<String>` 는 `SSM Parameter Store` 에서 가져온값이 `String` 임을 알려준다n.
>
> `Default` 의 값인 `"/my-app/web/desired-size"` 는 `SSM Parameter Store` 에 저장된 `Parameter` 의 이름이다.
>
> 즉, `SSM Parameter Store` 의 `/my-app/web/desired-size` 라는 이름을 가진 `Parameter` 를 가져오며, `Type` 은 `String` 임을 알수있다.

이렇게 작성하면, `Stack` 실행시 `WebTireDesSizeParameter` 는 `/my-app/web/desired-size` `Parameter` 의 값을 자동으로 참조하고, `Resource` 선언시 `Parameter` 참조를 통해 이 값을 사용할수있다.

### Resource 선언 참조

앞의 [Parameters 에서 참조](#parameters-에서-참조) 하는 방식은 번거로울수 있다.
`javascript` 에서 `template literal` 이 나오기전에, 일일히 `+` 연산자를 사용해 문자열을 조합했다...

하지만 `javascript` 에서 `template literal` 방식이 나온후, 문자열에 `변수` 값을 넣어 바로 처리할수 있어, 개발방식이 굉장히 편해졌다.

이처럼, `SSM Parameter Store` 를 매번 `Parameters` 로 생성한후 참조하는 방식은 불편하다.

그래서, 마치 `template literal` 처럼, `Resource` 에서 선언시 `SSM Parameter Store` 에서 `Parameter` 를 직접 참조할수 있는 방식을 제공한다.

```yml
Resources:  
  WebTierAsg:  
    Type: AWS::AutoScaling::AutoScalingGroup  
    Properties:  
        DesiredCapacity: !Sub 
            - "{{resolve:ssm:${parameter}:1}}"
            # Output 에서 가져온 WebTireDesiredSize 
            - parameter: !importValue WebTireDesiredSize
        MaxSize: !Sub
            - "{{resolve:ssm:${parameter}:1}}"
            # Output 에서 가져온 WebTireMaxSize 
            - parameter: !importValue WebTireMaxSize
        MinSize: !Sub
            - "{{resolve:ssm:${parameter}:1}}"
            # Output 에서 가져온 WebTireMinSize 
            - parameter: !importValue WebTireMinSize
```

> `Jinja` 서식을 사용하여 `Resource` 속성에 전달하는 방식으로 처리한다.
> 이러한 직접 참조 방식을 동적참조라 하며 다음의 형식을 가진다.
>
> `"{{resolve:service-name:reference-key:version}}"`
>
> - `resolve:ssm` `block` 은 `SSM Parameter Store` 에서 값을 가져오는 동적참조임을 뜻한다.
> - `reference-Key` `block` 은 `SSM Parameter Store` 에 저장된 참조 `key` 이다.
> - `version` `block` 은 `SSM Parameter Store` 에 저장된 `Parameter` 의 `Version` 이다.
>
>> [Parameter Store 와 Secret Manager](#parameter-store-와-secrets-manager) 에서 `Version Contorl Service` 도 같이 지원한다고 했다. 이는 `Version Control` 을 위한 버전 명이다.

### SSM Parameter Store 제한사항

`SSM Parameter Store` 에 최신 `version` 의 `Parameter` 를 참조할수 없다.
`Parameter` 를 변경하는 경우에도 `resolve` `block` 에서 해당 버전을 변경해야 한다.

> 여기서 말하는 `version` 은 앞에 말한 `resolve:ssm:reference-key:version` 에서 사용되는 `version block` 이다.
>
> 만약, [Parameters 에서 참조](#parameters-에서-참조) 방식으로 참조하면, 기본적으로 처음 정의된 버전(`1`) 을 참조하며, 변경된 버전을 자동으로 참조안한다.
>
> 반면, [Resource 선언 참조](#resource-선언-참조) 같은 경우, `version` `block` 에 꼭 `version` 을 명시해야 한다.
---
> :anger: 이거 약간 애매하다. [Parameter Store에서 파라미터 버전으로 작업](https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/sysman-paramstore-versions.html) 에서는 항상 최신 파라미터 버전을 사용한다고 말한다.
>
> 그런데 일단 책에서는 `Parameter version` 에 대해서 최신버전을 가져오지 못한다고 말하고 있다..
>
> 이는 나중에 실험해보고, 조금더 작성해 보아야 겠다.
> 일단, `version` 명을 명시할수 있는 [Resource 선언 참조](#resource-선언-참조) 로 작업하는것으로!
>
>> 참고로 [Parameters 에서 참조](#parameters-에서-참조) 는 `version` 을 명시할수 없다고 한다..
---

다음은, `Secret Manager` 를 통해 `DB password` 를 작성하는 예시이다.
이는 노출할 필요가 없으니, `Parameter Store` 를 사용하지 않는다.

```yml
# database.yamlResources:
Resources:
    DbPwSecret:
        Type: "AWS::SecretsManager::Secret"
        Properties:
            GenerateSecretString:
                GenerateStringKey: "DbPassword"
    RdsInstance:
        Type: AWS::RDS::DBInstance
        Properties:
            # Some RDS Instance properties...
            MasterUserPassword: !Sub 
                - "{{resolve:secretsmanager:${DbPwSecret}::SecretString: DbPassword}}"
```

여기에서 사용된 `GenerateSecretString` 은 자동적으로 `password` 를 생성하며, `GetnerteStringKey` 는 생성할 `password` 를 참조하는 `key` 이다.

여기에서 `resolve` 방식이 약간 다른데, 이는 다음과 같다.

`"{{resolve:secretsmanager:secret-id::service-attribute:key}}"`

👌 먼저  `:` 와 `::` 이 뜻하는 바를 보도록 한다.

- `:` 은 `secret` 값의 내부에서 특정 `key` 를 참조할때 사용
- `::` 는 `service` 와 `attribute` 를 구분하는 역할

👌 각 `block` 이 뜻하는바를 설명한다.

1 `resolve:secretsmanager block` 은 `secrets manager` 리소스를 동적참조한다는 선언이다.

2 `:secret-id block` 은 `secerts manager` 에서 참조할 `resource id` 를 뜻한다.(seserts manager 의 arn 이라고 보면된다.)

3 `::service-attribute block` 은 참조할 `secerts` 의 속성을 지정한다.  
이는 `SecretString` 과 `SecretBinary` 로 구분할수 있다.

:bulb: `SecretString`: `secret` 값이 `string` 형태로 저장되어있을때 사용
:bulb: `SecretBinary`: `secret` 값이 `binary` 형태로 저장되어있을때 사용

4 `:key block` 은 `secrets` 값 내부에서 특정 `key` 를 참조할때 사용한다.  
`SecretString` 은 특정 `JSON key` 를 의미한다.  
`secerts` 값이 `JSON` 형식으로 저장되었을때, 해당 `JSON` 내의 특정 항목을 참조하기 위해서 사용된다.

`SSM Parameter Store` 와는 다르게 `Secrets Manager` 는 버전을 지정하지 않고, 참조하려는 실제 `key` 를 지정한다.

이를 통해 어떠한 방식으로, `resource` 를 참조하는지 알수 있다.
