# This is a sample to demonstrate the bug
https://github.com/matt-filion/serverless-external-s3-event/issues/66

# Steps to reproduce:
## 1- clone the project
clone the project using the following command:
```commandLine
git clone https://github.com/alexdorand/existingS3Example.git
```

## 2- install the plugins
```commandLine
npm install
```

you can also run (but not necessary) the following commands:
```commandLine
npm install serverless-plugin-existing-s3
npm install serverless-pseudo-parameters
```


## 3- deploy

Run the following commands to deploy to the server:

```commandLine
sls deploy
```

followed by

```commandLine
sls s3deploy
```


# Problem
I am trying to create resources dynamically, one which is S3 buckets. And I want one these buckets to act as a trigger for my lambda. When trying to dynamically set existingS3 bucket name it fails to execute and add the S3 event. 

## Sample code provided:
https://github.com/alexdorand/existingS3Example

## facts
this is how my serverless.yml (not the entire thing)

```

service: d365-integ # NOTE: update this with your service name

custom:
  global:
    bucketName:
      Fn::Join:
      - ''
      - - 'existings3-bug-'
        - ${self:provider.stage}
        - Ref: AWS::AccountId

provider:
  name: aws
  runtime: java8

# you can overwrite defaults here
  stage: dev
  region: us-east-1

# you can add statements to the Lambda function's IAM Role here
  iamRoleStatements:
    - Effect: "Allow"
      Action:
      - "s3:*"
      - "s3:PutObject"
      - "s3:DeleteObject"
      - "s3:GetObjectTagging"
      - "s3:PutObjectTaggings"
      - "s3:PutBucketNotification"
      Resource: "arn:aws:s3:::*"

# you can define service wide environment variables here
#  environment:
#    variable1: value1

# you can add packaging information here
package:
  artifact: target/hello-dev.jar

functions:
  hello:
    handler: com.serverless.Handler
    events:
    - existingS3:
        bucket: ${self:custom.global.bucketName}
        events:
        - s3:ObjectCreated:*    

resources:
  Resources:
    VendorBucketName:
      Type: AWS::S3::Bucket
      Properties:
        BucketName: ${self:custom.global.bucketName}

plugins:
  - serverless-plugin-existing-s3
  - serverless-pseudo-parameters
```

When I run the sls s3Deploy I get the following error:

```
 C02TN4JWG8WN:existingS3Example aldor$ sls s3deploy
Serverless: Load command config
Serverless: Load command config:credentials
Serverless: Load command create
Serverless: Load command install
Serverless: Load command package
Serverless: Load command deploy
Serverless: Load command deploy:function
Serverless: Load command deploy:list
Serverless: Load command deploy:list:functions
Serverless: Load command invoke
Serverless: Load command invoke:local
Serverless: Load command info
Serverless: Load command logs
Serverless: Load command login
Serverless: Load command logout
Serverless: Load command metrics
Serverless: Load command print
Serverless: Load command remove
Serverless: Load command rollback
Serverless: Load command rollback:function
Serverless: Load command slstats
Serverless: Load command plugin
Serverless: Load command plugin
Serverless: Load command plugin:install
Serverless: Load command plugin
Serverless: Load command plugin:uninstall
Serverless: Load command plugin
Serverless: Load command plugin:list
Serverless: Load command plugin
Serverless: Load command plugin:search
Serverless: Load command config
Serverless: Load command config:credentials
Serverless: Load command rollback
Serverless: Load command rollback:function
Serverless: Load command s3deploy
Serverless: Invoke s3deploy
Serverless: beforeFunctions --> building ... 
Serverless: beforeFunctions <-- Complete, built 1 events.
Serverless: functions --> prepare to be executed by s3 buckets ... 
 
  Type Error ---------------------------------------------
 
  bucketName.replace is not a function
 
     For debugging logs, run again after setting the "SLS_DEBUG=*" environment variable.
 
  Stack Trace --------------------------------------------
 
TypeError: bucketName.replace is not a function
    at LambdaPermissions.getId (/Users/aldor/Projects/alex-bug/existingS3Example/node_modules/serverless-plugin-existing-s3/Permissions.js:10:54)
    at LambdaPermissions.createPolicy (/Users/aldor/Projects/alex-bug/existingS3Example/node_modules/serverless-plugin-existing-s3/Permissions.js:20:25)
    at results.map.result (/Users/aldor/Projects/alex-bug/existingS3Example/node_modules/serverless-plugin-existing-s3/index.js:94:41)
    at Array.map (<anonymous>)
    at Promise.all.then.results (/Users/aldor/Projects/alex-bug/existingS3Example/node_modules/serverless-plugin-existing-s3/index.js:70:33)
    at <anonymous>
From previous event:
    at PluginManager.invoke (/usr/local/lib/node_modules/serverless/lib/classes/PluginManager.js:390:22)
    at PluginManager.run (/usr/local/lib/node_modules/serverless/lib/classes/PluginManager.js:421:17)
    at variables.populateService.then.then (/usr/local/lib/node_modules/serverless/lib/Serverless.js:157:33)
    at runCallback (timers.js:810:20)
    at tryOnImmediate (timers.js:768:5)
    at processImmediate [as _immediateCallback] (timers.js:745:5)
From previous event:
    at Serverless.run (/usr/local/lib/node_modules/serverless/lib/Serverless.js:144:8)
    at serverless.init.then (/usr/local/lib/node_modules/serverless/bin/serverless:43:50)
    at <anonymous>
 
  Get Support --------------------------------------------
     Docs:          docs.serverless.com
     Bugs:          github.com/serverless/serverless/issues
     Issues:        forum.serverless.com
 
  Your Environment Information -----------------------------
     OS:                     darwin
     Node Version:           8.11.3
     Serverless Version:     1.32.0
```
### it works when
If I change the bucket name to string value like:
```
functions:
  hello:
    handler: com.serverless.Handler
    events:
    - existingS3:
        bucket: "existings3-bug-dev-1234567890"
        events:
        - s3:ObjectCreated:*    
```

it works. But when I pass ${self:custom.global.bucketName} it doesn't work