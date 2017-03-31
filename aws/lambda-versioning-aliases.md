# AWS Lambda Versioning and Aliases

## Overview

### Versioning
* AWS는 기본적으로 Lambda Function에 대해 versioning 기능을 제공하기 때문에 Lambda function의 여러 버전을 발행할 수 있음  
따라서, Development, beta, 그리고 production 등의 다양한 버전으로 작업이 가능함

* 각각의  Lambda function version은  unique한 ARN(Amazon Resource Name)을 가짐

* Version을 발행하고 나면 **코드 + 환경변수**가 snapshot으로 관리 되고, 수정 불가함

### Aliases
* 개념적으로 특정 Lambda function version를 가리키는 Pointer라고 볼 수 있음

* Alias도 하나의 resource이기 때문에 각각의 alias는 unique한 ARN을 가짐

* Version과 달리 변경 가능하기 때문에 다른 version을 가리키도록 수정할 수 있음

### 예제
- **helloworld** lambda function  
- version $LATEST, version 1, 2
- alias DEV, BETA, PROD




![](https://pbs.twimg.com/media/CRMFJ1jWcAA13Tg.png:large)
## Versioning

### Creating a Lambda Function (the $LATEST version)

* **Qualified ARN** - version 정보가 접미사로 붙은 ARN


```

arn:aws:lambda:aws-region:acct-id:function:helloworld:$LATEST
```  
* **Unqualified ARN** - version 정보가 없는 ARN으로 alias를 붙일 수 없음


```

arn:aws:lambda:aws-region:acct-id:function:helloworld
```

아무런 version도 발행하지 않으면, 유일하게 $LATEST 버전만 있고, 이 버전은 qualified 또는 unqualified ARN 둘 다 사용 가능함  


### Publishing a Lambda Function Version

Version을 발행하면 AWS Lambda는 $LATEST버전의 코드와 설정 정보 등에 대한 snapshot을 생성함  
발행된 version은 immutable이기 때문에 코드를 변경하거나, 설정을 정보를 변경할 수 없음

#### 발행 방법 두 가지
* **Publish a version explicitly** - PublishVersion API를 사용하여  $LATEST 버전 코드에 명시적으로 버전을 발행함
* **Publish a version at the time you create or update a Lambda function** - CreateFunction 또는 UpdateFunctionCode 요청을 할 때 publish 옵션으로 버전을 발행함

```javascript
{
    "CodeSize": 287,
    "Description": "test function."
    "FunctionArn": "arn:aws:lambda:aws-region:acct-id:function:helloworld",
    "FunctionName": "helloworld",
    "Handler": "helloworld.handler",
    "LastModified": "2015-07-16T00:34:31.322+0000",
    "MemorySize": 128,
    "Role": "arn:aws:iam::acct-id:role/lambda_basic_execution",
    "Runtime": "nodejs6.10",
    "Timeout": 3,
    "CodeSHA256": "OjRFuuHKizEE8tHFIMsI+iHR6BPAfJ5S0rW31Mh6jKg=",
    "Version": "1" 
}
```

helloworld Lambda function의 version 1에 대한 ARN  

```

arn:aws:lambda:aws-region:acct-id:function:helloworld:1
```

## Alias

Lambda function의 특정 버전에 대한 포인터

* **Lambda function의 버전 업이나 roll back이 용이함**


* **Event source에서의 mapping이 간단해짐**


### Alias ARNs Example
```

arn:aws:lambda:aws-region:acct-id:function:helloworld:PROD
arn:aws:lambda:aws-region:acct-id:function:helloworld:BETA
arn:aws:lambda:aws-region:acct-id:function:helloworld:DEV
```



## API Gateway stage variable 
API의 배포 단계(deployment stage)와 관련한 configuration attributes등을 정의한 name-value 쌍

* API method의 request integration에서 stage variable을 request에 통합시켜서 전송할 수 있음

* Lambda function에서는 event객체를 통해 API Gateway에서 전송한 stage variable을 참조하여 사용할 수 있음



### Example
![](https://github.com/hwangssi/TIL/blob/master/aws/CI_STAGE.png?raw=true)

