---
title: "AWS LocalStack を試してみた"
emoji: "👏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS", "LocalStack"]
published: false
---

# 前提

- Python3 のインストール
- Docker が使えること(必須では無いけど)

# インストール

pip3 install localstack

# 起動

localstack start -d

# 確認

```
$ localstack status
┌─────────────────┬───────────────────────────────────────────────────────┐
│ Runtime version │ 1.0.2.dev                                             │
│ Docker image    │ tag: latest, id: ce7b7c8539ec, 📆 2022-07-25T16:00:39 │
│ Runtime status  │ ✔ running (name: "localstack_main", IP: 172.17.0.2)   │
└─────────────────┴───────────────────────────────────────────────────────┘
```

```
$ localstack status services
┏━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━┓
┃ Service                  ┃ Status      ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━┩
│ acm                      │ ✔ available │
│ apigateway               │ ✔ available │
│ cloudformation           │ ✔ available │
│ cloudwatch               │ ✔ available │
│ config                   │ ✔ available │
│ dynamodb                 │ ✔ available │
│ dynamodbstreams          │ ✔ available │
│ ec2                      │ ✔ available │
│ es                       │ ✔ available │
│ events                   │ ✔ available │
│ firehose                 │ ✔ available │
│ iam                      │ ✔ available │
│ kinesis                  │ ✔ available │
│ kms                      │ ✔ available │
│ lambda                   │ ✔ available │
│ logs                     │ ✔ available │
│ opensearch               │ ✔ available │
│ redshift                 │ ✔ available │
│ resource-groups          │ ✔ available │
│ resourcegroupstaggingapi │ ✔ available │
│ route53                  │ ✔ available │
│ route53resolver          │ ✔ available │
│ s3                       │ ✔ available │
│ s3control                │ ✔ available │
│ secretsmanager           │ ✔ available │
│ ses                      │ ✔ available │
│ sns                      │ ✔ available │
│ sqs                      │ ✔ available │
│ ssm                      │ ✔ available │
│ stepfunctions            │ ✔ available │
│ sts                      │ ✔ available │
│ support                  │ ✔ available │
│ swf                      │ ✔ available │
└──────────────────────────┴─────────────┘
```

# AWS CLI で使うための準備

AWS LocalStack 上に AWS リソースを構築する方法はいくつか存在する。
https://docs.localstack.cloud/integrations/

ここではとりあえず簡単に触ってみることが目的なので、AWS CLI を使って手軽に試してみることにする。

https://docs.localstack.cloud/integrations/aws-cli/

いつもの AWS CLI でも使えるが、`--endpoint-url`を指定しないといけなかったりが面倒。

なので、LocalStack にアクセスする用の CLI を使うことにする。

https://docs.localstack.cloud/integrations/aws-cli/#localstack-aws-cli-awslocal

インストール

```
$ pip3 install awscli-local
```

# AWS リソースの作成

## S3 バケットの作成

```
$ awslocal s3api create-bucket --bucket demo-localstack
{
    "Location": "/demo-localstack"
}
```

```
$ awslocal s3api list-objects --bucket demo-localstack

$ echo 'Hello, World!' > hello-world.txt

$ awslocal s3api put-object --bucket demo-localstack --key hello-world.txt --body hello-world.txt
{
    "ETag": "\"bea8252ff4e80f41719ea13cdf007273\""
}

$ awslocal s3api list-objects --bucket demo-localstack
{
    "Contents": [
        {
            "Key": "hello-world.txt",
            "LastModified": "2022-07-26T04:56:38+00:00",
            "ETag": "\"bea8252ff4e80f41719ea13cdf007273\"",
            "Size": 14,
            "StorageClass": "STANDARD",
            "Owner": {
                "DisplayName": "webfile",
                "ID": "75aa57f09aa0c8caeab4f8c24e99d10f8e7faeebf76c078efc7c6caea54ba06a"
            }
        }
    ]
}

$ awslocal s3api get-object --bucket demo-localstack --key hello-world.txt out-hello-world.txt
{
    "AcceptRanges": "bytes",
    "LastModified": "2022-07-26T04:56:38+00:00",
    "ContentLength": 14,
    "ETag": "\"bea8252ff4e80f41719ea13cdf007273\"",
    "VersionId": "null",
    "ContentLanguage": "en-US",
    "ContentType": "binary/octet-stream",
    "Metadata": {}
}

$ cat downloaded-hello-world.txt
Hello, World!
```

## DynamoDB Table の作成

https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WorkingWithTables.Basics.html

```
$ awslocal dynamodb create-table --attribute-definitions 'AttributeName=id,AttributeType=S' --table-name 'users' --key-schema 'AttributeName=id,KeyType=HASH' --provisioned-throughput 'ReadCapacityUnits=10,WriteCapacityUnits=5'
{
    "TableDescription": {
        "AttributeDefinitions": [
            {
                "AttributeName": "id",
                "AttributeType": "S"
            }
        ],
        "TableName": "users",
        "KeySchema": [
            {
                "AttributeName": "id",
                "KeyType": "HASH"
            }
        ],
        "TableStatus": "ACTIVE",
        "CreationDateTime": "2022-07-26T14:24:11.607000+09:00",
        "ProvisionedThroughput": {
            "ReadCapacityUnits": 10,
            "WriteCapacityUnits": 5
        },
        "TableSizeBytes": 0,
        "ItemCount": 0,
        "TableArn": "arn:aws:dynamodb:us-east-1:000000000000:table/users",
        "TableId": "59435772-c6ba-4819-87d2-ca1ef487b705"
    }
}

$ awslocal dynamodb scan --table-name users
{
    "Items": [],
    "Count": 0,
    "ScannedCount": 0,
    "ConsumedCapacity": null
}

$ awslocal dynamodb put-item --table-name users --item '{"id": {"S": "1"}, "name": {"S": "TaroTanaka"}}'

$ awslocal dynamodb scan --table-name users
{
    "Items": [
        {
            "name": {
                "S": "TaroTanaka"
            },
            "id": {
                "S": "1"
            }
        }
    ],
    "Count": 1,
    "ScannedCount": 1,
    "ConsumedCapacity": null
}
```

## Lambda 用の IAM role 作成

https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/lambda-intro-execution-role.html

```
$ awslocal iam create-role --role-name demo-localstack-lambda-ex --assume-role-policy-document '{"Version": "2012-10-17","Statement": [{ "Effect": "Allow", "Principal": {"Service": "lambda.amazonaws.com"}, "Action": "sts:AssumeRole"}]}'
{
    "Role": {
        "Path": "/",
        "RoleName": "demo-localstack-lambda-ex",
        "RoleId": "jiw8lhc4je201i0y2vgw",
        "Arn": "arn:aws:iam::000000000000:role/demo-localstack-lambda-ex",
        "CreateDate": "2022-07-26T05:50:17.360000+00:00",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "lambda.amazonaws.com"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        },
        "MaxSessionDuration": 3600
    }
}
```

```
$ awslocal iam list-policies
        {
            "PolicyName": "AWSLambdaBasicExecutionRole",
            "PolicyId": "AR8Q5I0B6F695V1JSJQUM",
            "Arn": "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole",
            "Path": "/service-role/",
            "DefaultVersionId": "v1",
            "AttachmentCount": 0,
            "CreateDate": "2015-04-09T15:03:43+00:00",
            "UpdateDate": "2015-04-09T15:03:43+00:00"
        },
        {
            "PolicyName": "AWSLambdaDynamoDBExecutionRole",
            "PolicyId": "AN5Y43VEKO7885XHT3EBX",
            "Arn": "arn:aws:iam::aws:policy/service-role/AWSLambdaDynamoDBExecutionRole",
            "Path": "/service-role/",
            "DefaultVersionId": "v1",
            "AttachmentCount": 0,
            "CreateDate": "2015-04-09T15:09:29+00:00",
            "UpdateDate": "2015-04-09T15:09:29+00:00"
        },

	=====

        {
            "PolicyName": "AmazonS3ObjectLambdaExecutionRolePolicy",
            "PolicyId": "AN8UUKCOCIFP9O45NWXYD",
            "Arn": "arn:aws:iam::aws:policy/service-role/AmazonS3ObjectLambdaExecutionRolePolicy",
            "Path": "/service-role/",
            "DefaultVersionId": "v1",
            "AttachmentCount": 0,
            "CreateDate": "2021-08-18T10:07:41+00:00",
            "UpdateDate": "2021-08-18T10:07:41+00:00"
        },

```

```
$ awslocal iam attach-role-policy --role-name demo-localstack-lambda-ex --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

$ awslocal iam attach-role-policy --role-name demo-localstack-lambda-ex --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaDynamoDBExecutionRole

$ awslocal iam attach-role-policy --role-name demo-localstack-lambda-ex --policy-arn arn:aws:iam::aws:policy/service-role/AmazonS3ObjectLambdaExecutionRolePolicy
```

```
$ awslocal iam list-attached-role-policies --role-name demo-localstack-lambda-ex
{
    "AttachedPolicies": [
        {
            "PolicyName": "AWSLambdaBasicExecutionRole",
            "PolicyArn": "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
        },
        {
            "PolicyName": "AWSLambdaDynamoDBExecutionRole",
            "PolicyArn": "arn:aws:iam::aws:policy/service-role/AWSLambdaDynamoDBExecutionRole"
        },
        {
            "PolicyName": "AmazonS3ObjectLambdaExecutionRolePolicy",
            "PolicyArn": "arn:aws:iam::aws:policy/service-role/AmazonS3ObjectLambdaExecutionRolePolicy"
        }
    ]
}
```

## Lambda Function の作成

(今から実行します~)

```
$ cd /path/to/demo-aws-localstack/lambda/
$ cd create-user/ && zip -r lambda-create-user.zip . && cd ../
$ cd get-user/ && zip -r lambda-get-user.zip . && cd ../
$ awslocal lambda create-function \
    --function-name demo-aws-localstack-lambda-create-user \
    --runtime nodejs16.x \
    --handler 'index.handler' \
    --role arn:aws:iam::000000000000:role/demo-localstack-lambda-ex \
    --package-type Zip \
    --zip-file fileb://create-user/lambda-create-user.zip
{
    "FunctionName": "demo-aws-localstack-lambda-create-user",
    "FunctionArn": "arn:aws:lambda:us-east-1:000000000000:function:demo-aws-localstack-lambda-create-user",
    "Runtime": "nodejs16.x",
    "Role": "arn:aws:iam::000000000000:role/demo-localstack-lambda-ex",
    "Handler": "index.handler",
    "CodeSize": 581,
    "Description": "",
    "Timeout": 3,
    "LastModified": "2022-07-26T07:11:48.408+0000",
    "CodeSha256": "12X7Gu97ZNxdXapIIo/SFes74GoDQqP0RPXsxfwTzEQ=",
    "Version": "$LATEST",
    "VpcConfig": {},
    "TracingConfig": {
        "Mode": "PassThrough"
    },
    "RevisionId": "9ddce21a-8769-4b5d-8eae-08e038f842c8",
    "State": "Active",
    "LastUpdateStatus": "Successful",
    "PackageType": "Zip",
    "Architectures": [
        "x86_64"
    ]
}
$ awslocal lambda create-function \
    --function-name demo-aws-localstack-lambda-get-user \
    --runtime nodejs16.x \
    --handler 'index.handler' \
    --role arn:aws:iam::000000000000:role/demo-localstack-lambda-ex \
    --package-type Zip \
    --zip-file fileb://get-user/lambda-get-user.zip
{
    "FunctionName": "demo-aws-localstack-lambda-get-user",
    "FunctionArn": "arn:aws:lambda:us-east-1:000000000000:function:demo-aws-localstack-lambda-get-user",
    "Runtime": "nodejs16.x",
    "Role": "arn:aws:iam::000000000000:role/demo-localstack-lambda-ex",
    "Handler": "index.handler",
    "CodeSize": 576,
    "Description": "",
    "Timeout": 3,
    "LastModified": "2022-07-26T07:12:03.455+0000",
    "CodeSha256": "Nd/Cprd/Z6VyrxfV+LhcifKo6878fKEe3amGlv+E508=",
    "Version": "$LATEST",
    "VpcConfig": {},
    "TracingConfig": {
        "Mode": "PassThrough"
    },
    "RevisionId": "9e3d0a6e-4134-472c-be6b-d2afca58885c",
    "State": "Active",
    "LastUpdateStatus": "Successful",
    "PackageType": "Zip",
    "Architectures": [
        "x86_64"
    ]
}
```

実際に呼び出してみる

```
$ awslocal lambda invoke --function-name demo-aws-localstack-lambda-create-user outfile
$ cat outfile
"Hello! This is create-user function!"

$ awslocal lambda invoke --function-name demo-aws-localstack-lambda-get-user outfile
$ cat outfile
"Hello! This is get-user function!"
```

## API Gateway の作成

```
$ awslocal apigateway create-rest-api --name demo-aws-localstack-apigateway
{
    "id": "9aeiawjhah",
    "name": "demo-aws-localstack-apigateway",
    "createdDate": "2022-07-26T15:45:01+09:00",
    "version": "V1",
    "binaryMediaTypes": [],
    "apiKeySource": "HEADER",
    "endpointConfiguration": {
        "types": [
            "EDGE"
        ]
    },
    "tags": {},
    "disableExecuteApiEndpoint": false
}
```

ルートリソースの id を確認する

```
$ awslocal apigateway get-resources --rest-api-id 9aeiawjhah
{
    "items": [
        {
            "id": "wf5zl0ssso",
            "path": "/"
        }
    ]
}
```

リソースの追加

```
$ awslocal apigateway create-resource \
  --rest-api-id 9aeiawjhah \
  --parent-id wf5zl0ssso \
  --path-part 'users'
{
    "id": "wz1ixldzj4",
    "parentId": "wf5zl0ssso",
    "pathPart": "users",
    "path": "/users"
}

$ awslocal apigateway create-resource \
  --rest-api-id 9aeiawjhah \
  --parent-id  wz1ixldzj4 \
  --path-part '{userId}'
{
    "id": "apz8njkd50",
    "parentId": "wz1ixldzj4",
    "pathPart": "{userId}",
    "path": "/users/{userId}"
}
```

```
$ awslocal apigateway get-resources --rest-api-id 9aeiawjhah
{
    "items": [
        {
            "id": "wf5zl0ssso",
            "path": "/"
        },
        {
            "id": "wz1ixldzj4",
            "parentId": "wf5zl0ssso",
            "pathPart": "users",
            "path": "/users"
        },
        {
            "id": "apz8njkd50",
            "parentId": "wz1ixldzj4",
            "pathPart": "{userId}",
            "path": "/users/{userId}"
        }
    ]
}
```

メソッドの追加

```
$ awslocal apigateway put-method \
    --rest-api-id 9aeiawjhah \
    --resource-id wz1ixldzj4 \
    --http-method POST \
    --authorization-type 'NONE'
{
    "httpMethod": "POST",
    "authorizationType": "NONE",
    "apiKeyRequired": false,
    "methodResponses": {}
}

$ awslocal apigateway put-method \
    --rest-api-id 9aeiawjhah \
    --resource-id apz8njkd50 \
    --http-method GET \
    --authorization-type 'NONE'
{
    "httpMethod": "GET",
    "authorizationType": "NONE",
    "apiKeyRequired": false,
    "methodResponses": {}
}
```

インテグレーション作成

--integration-http-method POST にしないといけないっぽい
https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html

```
$ awslocal apigateway put-integration \
    --rest-api-id 9aeiawjhah \
    --resource-id wz1ixldzj4 \
    --http-method POST \
    --type AWS \
    --integration-http-method POST \
    --uri 'arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:000000000000:function:demo-aws-localstack-lambda-create-user/invocations'
{
    "type": "AWS",
    "httpMethod": "POST",
    "uri": "arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:000000000000:function:demo-aws-localstack-lambda-create-user/invocations",
    "requestParameters": {},
    "cacheNamespace": "wz1ixldzj4",
    "cacheKeyParameters": []
}

$ awslocal apigateway put-integration \
    --rest-api-id 9aeiawjhah \
    --resource-id apz8njkd50 \
    --http-method GET \
    --type AWS \
    --integration-http-method POST \
    --uri 'arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:000000000000:function:demo-aws-localstack-lambda-get-user/invocations'
{
    "type": "AWS",
    "httpMethod": "POST",
    "uri": "arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:000000000000:function:demo-aws-localstack-lambda-get-user/invocations",
    "requestParameters": {},
    "cacheNamespace": "apz8njkd50",
    "cacheKeyParameters": []
}
```

API Gateway -> Client の method response の作成

```
$ awslocal apigateway put-method-response \
    --rest-api-id 9aeiawjhah \
    --resource-id wz1ixldzj4 \
    --http-method POST \
    --status-code 200
{
    "statusCode": "200"
}

$ awslocal apigateway put-method-response \
    --rest-api-id 9aeiawjhah \
    --resource-id apz8njkd50 \
    --http-method GET \
    --status-code 200
{
    "statusCode": "200"
}
```

Lambda->APIGateway の integration response 作成

```
$ awslocal apigateway put-integration-response \
    --rest-api-id 9aeiawjhah \
    --resource-id wz1ixldzj4 \
    --http-method POST \
    --status-code 200
{
    "statusCode": "200",
    "responseTemplates": {}
}

$ awslocal apigateway put-integration-response \
    --rest-api-id 9aeiawjhah \
    --resource-id apz8njkd50 \
    --http-method GET \
    --status-code 200
{
    "statusCode": "200",
    "responseTemplates": {}
}
```

API Gateway から呼び出せるよう、Lambda の権限変更

```
$ awslocal lambda add-permission \
    --function-name demo-aws-localstack-lambda-create-user \
    --statement-id cbe306e4-5ac0-465d-9914-ec03a2a4e528 \
    --action 'lambda:InvokeFunction' \
    --principal 'arn:aws:execute-api:ap-northeast-1:000000000000:9aeiawjhah/*/POST/users'
{
    "Statement": "{\"Sid\": \"cbe306e4-5ac0-465d-9914-ec03a2a4e528\", \"Effect\": \"Allow\", \"Action\": \"lambda:InvokeFunction\", \"Resource\": \"arn:aws:lambda:us-east-1:000000000000:function:demo-aws-localstack-lambda-create-user\", \"Principal\": {\"Service\": \"arn:aws:execute-api:ap-northeast-1:000000000000:9aeiawjhah/*/POST/users\"}}"
}

$ awslocal lambda add-permission \
    --function-name demo-aws-localstack-lambda-get-user \
    --statement-id 2309fca4-d618-41a7-8e40-d9de49da4caa \
    --action 'lambda:InvokeFunction' \
    --principal 'arn:aws:execute-api:ap-northeast-1:000000000000:9aeiawjhah/*/GET/users/{userId}'
{
    "Statement": "{\"Sid\": \"2309fca4-d618-41a7-8e40-d9de49da4caa\", \"Effect\": \"Allow\", \"Action\": \"lambda:InvokeFunction\", \"Resource\": \"arn:aws:lambda:us-east-1:000000000000:function:demo-aws-localstack-lambda-get-user\", \"Principal\": {\"Service\": \"arn:aws:execute-api:ap-northeast-1:000000000000:9aeiawjhah/*/GET/users/{userId}\"}}"
}
```

## 動作検証

```
$ awslocal apigateway test-invoke-method \
    --rest-api-id 9aeiawjhah \
    --resource-id wz1ixldzj4 \
    --http-method POST
{
    "status": 200,
    "body": "Hello! This is create-user function!",
    "headers": {
        "Content-Length": "36"
    }
}

$ awslocal apigateway test-invoke-method \
    --rest-api-id 9aeiawjhah \
    --resource-id apz8njkd50 \
    --http-method GET
{
    "status": 200,
    "body": "Hello! This is get-user function!",
    "headers": {
        "Content-Length": "33"
    }
}
```

# TODO

- [ ] Implement Lambda function, using S3 and DynamoDB.
- [ ] Deploy API Gateway.
