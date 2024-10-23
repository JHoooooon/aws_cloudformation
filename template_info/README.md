# CloudFormation 내부 살펴보기

`cloudformation` 은 몇가지 구성요소로 이루어져있다.
이러한 구성요소는, `JSON` 혹은 `YAML` 포멧으로 작성되며, 무엇을 만들지를 나타낸다

## AWSTemplateFormatVersion

***Not Required***

이는 `CloudFormation` 의 `Template` 작성시 명시할 `Format version` 을 나타낸다.  
`CloudFormation` 역시 `AWS` 에서 제공하는 `API` 이므로, 해당 `API` 의 버전을 타나낸다고 보면된다.

현재까지는 `2010-09-09` 로 고정되어있으며, 추후 `CloudFormation` 버전이 업데이트된다면, 해당 날짜로
고정될수도있다.

그러므로, 꼭 작성하지 않아도 되며, 작성하지 않을시 자동적으로 기본 포멧 버전을 사용한다.

## Description

***Not Required***

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

***Not Required***

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

***Not Required***

| type | 설명 |
| :-- | :-- |
| String | 리터럴 문자열 |
| Number | 정수 또는 부동소수점<br>`Ref` 내장함수를 사용하여, 참조시 파라미터값은 문자열이 된다. |
| List<Number> | 쉼표로 구분된 정수 또는 부동소수점의 배열.<br/>`Ref` 내장 함수를 사용하여, 참조시 파라미터 값은 문자열 배열이 된다. <br/> `"80, 20" 을 지정하면, Ref 는 ["80", "20"] 이 된다.`|
| CommaDelimitedList | 쉼표로 구분된 리터럴 문자열 배열.<br/>총 문자열 수는 총 쉼표수보다 하나 더 많다.<br/>`"test,dev,prod"` 는 `Ref` 내장 함수를 통해 `["test", "dev", "prod"]` 가 된다. |   
| AWS 특정 파라미터 type | `AWS` 관련 파라미터 타입 |
| System Manager 파라미테 type | `Systems Manager Parameter` 타입 참조 |

### Parameters 속성
`Parameters` 에서 제공하는 속성들.

#### AllowedPattern
***Not Required***

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
***Not Required***  

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
***Not Required***  

제약위반신 해당 제약을 설명하는 문자열.
> 예를 들어, 해당하는 타입이 아닌 다른 타입을 작성했던지, 속성에 맞지 않은 값을 지정했을시 나타내는
경고 및 에러 문자열 (에러 처리)

#### Default
***Not Required***  

스택 생성시 지정된 값이 없는 경우에 사용할 기본값.
> 제약 정의시, 이러한 제약을 준수하는 기본값을 설정하는것이 좋다고 한다.

```yml
Parameters:
    DockerImageVersion:
        Type: String
        Default: latest # 아무런 값 지정 없을시 기본값
```

#### Description
***Not Required***  

`Parameter` 를 설명하는 설명구.

#### MaxLength
***Not Required***  

`String` 유형에 해당하며, 문자열의 최대 문자수를 결정하는 정수값

#### MaxValue
***Not Required***  

`Number` 유형에 해당하며, 최대 숫자값을 결정하는 정수값

#### MinLength
***Not Required***  

`String` 유형에 해당하며, 문자열의 최소 문자수를 결정하는 정수값

#### MinValue
***Not Required***  

`Number` 유형에 해당하며, 최소 숫자값을 결정하는 정수값

#### NoEcho
***Not Required***  

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
    Type: "AWS::RDS::DBInstance"  
    Properties:  
      DBName: mydatabase  
      Engine: MySQL  
      MasterUsername: admin  
      MasterUserPassword: !Ref DBPassword  
      DBInstanceClass: db.t2.micro  
      AllocatedStorage: 20  

Parameters:  
  DBPassword:  
    Description: "The database admin account password"  
    Type: "String"  
    NoEcho: true # 비밀번호는 출력되지 않도록 설정  

Outputs:  
  DatabaseConnectionString:  
    Description: "The connection string for the RDS database"  
    Value: !Join [ ":", [ "mysql://admin", !Ref MyDBInstance.Endpoint.Address ] ]  
    Export:  
      Name: "DBConnectionString"
```

이 같은경우, `DBPassword` `Parameter` 는 `CloudFormation` 의 `cli` 혹은, 콘솔상에 마스킹되어 노출되지 않는다. 


## Mappings

***Not Required***

`Mappings` 는 `Parameters` 와 비슷하지만, 사전에 정의된 형식으로 제공된다.

> `Parameters` 는  `Stack` 생성시 동적으로 할당한다.

이렇게 미리 작성된 변수를 사용하기 위한 역할로써 제공된다.

### Mappings intrinsic functions

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

***Not Required***

어떠한 조건에 의해 `Template` 의 `Resource` 생성여부를 제어하는데 사용된다.  
`Conditions` 는 `true` 혹은 `false` 값을 받는데,  
이를 통해 해당 `resource` 를 사용할지 안할지를 결정할수 있다.

> `Conditions` 에서 `Boolean` 값을 받아 처리한다고 해서, 수동으로 `true` 나 `false` 를 선언하지  
않아야 한다.
>
> 반드시, `Fn::Equals` `Fn::If` `Fn::Not` `Fn::And` `Fn:Or` 같은 조건을 정의하는 함수를 통해  
비교하여 만들어지 값이어야 한다.

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

## Transform

`CloudFormation` 에서 특정 `Macro` 를 실행하려고할때 선언된다.
이는 특수한 부분이므로, 추후 설명한다.

## Resources

***Required***

이는, `CloudFormation` 에서 유일하게 반드시 작성되어야 하는 섹션이다.
`Resources` 섹션은 프로비저닝하고자 하는 `resource` 를 제공한다.

[CloudFormation 지원 리소스](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html) 에서  `CloudFormation` 에서 제공하는 `resource` 가 무엇인지 알려준다
> 양이 많다..

### Resources 의 Attributes

여기서는 `Resources` 속서에 대한 개요만 살펴본다.

- **Type**: `Resource` 의 `type` <br/> `ec2 subnet`, `ec2 instance`, `dynamodb table` `lambda` 등등 많다.

- **Properties**: 해당 `Resource` 타입의 설정인자값

- **DependsOn**: 직접 종속성을 추가한다.

- **CreationPolicy**: `Stack` 에서 해당하는 `resource` 가 정상적으로 생성되었음을 확인하여,  
이 조건이 충족될때까지 기다리도록 한다.  
즉, ***`resource` 의 `완료 신호`(`success signal`)*** 을 기다린다. 

- **DeletionPolicy**: `Stack` 에서 해당하는 `resource` 를 삭제할때 `CloudFormation` 이 `Resource` 를  
어떻게 처리할지 지시하는데 사용된다.  
예를들어, 삭제시 ***데이터 손실을 방지*** 하고자, 삭제전에 `resource` 를 백업하기 위해  
***삭제 동작을 변경***  할수있다.

- **UpdatePolicy**: `Stack` 에서 해당하는 `resource` 를 업데이트 할때 `CloudFormation` 이 `Resource` 를  
어떻게 처리할지 지시하는데 사용된다.  
이는 ***리소스를 업데이트하는 과정을 제어하여, 리소스가 중단되지 않고도 안정적으로 업데이트*** 되도록 한다.

- **UpdateReplacePolicy**: `Stack` 에서 해당하는 `resource` 를 새로운 `resource` 로 교체할때  
`CloudFormation` 이 `Resource` 를 어떻게 처리할지 지시한다.  
업데이트중 ***리소스가 삭제되지 않거나, 삭제 되더라도 스냅샷을 생성하도록 하여 데이터 손실을 방지*** 한다.

### Outputs 







