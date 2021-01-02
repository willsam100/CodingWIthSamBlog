---
title: CI/CD tips and tricks with AWS Lambda
date: 2021-01-03 21:00:00 +13:00
categories: [Cloud, AWS Lambda]
tags: [Aws Lambda, ci/cd, Serverless]
description: Tips and tricks on deploying to AWS Lambda
published: false
image: /assets/img/2021-01-02-pipeline.jpg
---

# I break things, have you?

After finishing university, I was very naive about how bad I was at writing bug-free code. I was tasked with updating an iOS app. I had no tester and a teammate that was still in university. Yes, I broke the app. Normally that would be fine (as in push a fix out and move on). I broke payments in the app! The bug was very easy to reproduce and just as easy to fix. The challenge was we had an iOS app in prod not working. The boss was not happy - and fair enough, money was not flowing in. 

This was the first bug that I had experienced. It was not the last. There is a long list of ones to follow. The take-away is not 'be a better developer'. There is research to show that only 80% of bugs/defects will be picked up with the 'best' testing strategy (automated and testers). I still write automated tests and use testers. My expectations and approach have changed though. 

Errors (be it a bug, defect or simply something the customer does not like) is a fact of life. We need to cater for this. This is where DevOps and CI/CD comes in. These tools help us to push to prod quickly and safely. They make it easier to ensure that required features still work before releasing. 

The rest of this post are various tips and tricks that I have found to add levels of automation to an AWS Lambda project. The level of automation required depends on the project. 

# Basic deploy

This is the most basic and one that I covered in my [prevvious post on lambda](/posts/2020/06/13/getting-started-.NET-AWS-lambda). With .NET Core and the global AWS Lambda, a shell script is all that is required. 

```bash
set -e
  
dotnet test test/AWS-NetCoreExample.Tests/AWS-NetCoreExample.Tests.csproj
cd srcAWS-NetCoreExample/
dotnet lambda package
dotnet lambda deploy-function AWS-NetCoreExample --region ap-souttheast-2 -frun dotnetcore3.1
```

> See my [prevvious post on lambda](/posts/2020/06/13/getting-started-.NET-AWS-lambda) full details on setup and IAM access keys. 

This is by the far the fastest to set up. If you only require one environment to be running. If deploying from one machine is fine and handling security IAM access keys locally on the machine is acceptable security. 

For an more in-depth take on SAM, checkout the great course by Mark Hatch on Pluraalsight [
Deploying Serverless Applications in AWS Using the Serverless Application Model](https://app.pluralsight.com/library/courses/aws-deploying-serverless-applications-application-model/table-of-contents)

# AWS SAM 

The next level in automation is to define the lambda in code. We could use CloudFormation but that is hard. AWS figured this out too. They created the Serverless Application Model or SAM. It is a superset of CloudFormation. The benefits are that it is really easy to declare a lambda function using SAM. What's better, is that a default SAM template is included in some of the default `dotnet new` AWS templates. 


I tried this out, by running `dotnet new serverless.AspNetCoreWebAPI  -n mySamLambda` which created an ASP.Net Core web app ready for lambda. The SAM template can be found at `mySamLambda/src/mySamLambda/severless.template`. By default, it is defined in JSON. YAML is also supported

> `json2yaml` is a VS Code extension that allows for a quick conversion between JSON and YAML. 

Here is what the serverless template looks like in YAML:

    AWSTemplateFormatVersion: '2010-09-09'
    Transform: 'AWS::Serverless-2016-10-31'
    Description: An AWS Serverless Application that uses the ASP.NET Core framework running in Amazon Lambda.
    Parameters: {}
    Conditions: {}
    Resources:
        AspNetCoreFunction:
            Type: 'AWS::Serverless::Function'
            Properties:
                Handler: 'mySamLambda::mySamLambda.LambdaEntryPoint::FunctionHandlerAsync'
                Runtime: dotnetcore3.1
                CodeUri: ''
                MemorySize: 256
                Timeout: 30
                Role: null
                Policies:
                    - AWSLambdaFullAccess
                Events:
                    ProxyResource:
                        Type: Api
                        Properties:
                            Path: '/{proxy+}'
                            Method: ANY
                    RootResource:
                        Type: Api
                        Properties:
                            Path: /
                            Method: ANY
    Outputs:
        ApiURL:
            Description: API endpoint URL for Prod environment
            Value:
                'Fn::Sub': 'https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/'

What's really nice is that this includes a bit more than the lambda function. The template includes the required setup for the API Gateway as well. This is a nice benefit as I always tend to get something wrong using the console to setup Lambda and API Gateway. 

The SAM template also comes with a few handy CLI tools that we can use to deploy this. When we first run the command it will ask for some defaults. To you the tool you will need to have an IAM Access key install on your machine. Instructions on how how to install the CLI tool are available on a post by [AWS here](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html). 

After the SAM CLI tool is installed, a deploy can be done: 

    sam deploy -g -t serverless.template

There are a few prompts with appropriate defaults. Once completed (and assuming the IAM access key has appropriate permissions), the deploy will complete successfully. A URL will be posted when complete where the lambda can be accessed!

To push updates to the lambda without the guided prompts (the defaults should be saved to a config file) run the same command without `-g`

    sam deploy -g -t serverless.template

This is a really good next step in automating. I'll be using this anytime I need to try out a POC. 

One major downside is that this requires a single developers' machine. If your working for a company or a team this is ideal. Let's consider more automation. 

# Create a CI/CD pipeline with AWS CodePipeline

The key to DevOps and CI/CD is to remove single points of failure. AWS CodePipeline addresses a few limitations with the previous approach:
- The code was not built consistently built from lastest off the main branch
- The code was not built with a consistent .NET SDK 
- The code required a single dev machine to do a deploy 

AWS CodePipeline (along with AWS CodeBuild and CodeDeploy) addresses these challenges. 
To understand some of the AWS concepts, AWS re:Invent has a great video on this: [CI/CD for serverless applications (SVS336-R1)](https://www.youtube.com/watch?v=jUXiOPTX9S4). 

The AWS Console is quite good for this. I created the following actions via the console: 
- Source: pull from GitHub
- Build: using CodeBuild complete a build via a BuildSpect.yml
    - More details on this file below
- Deploy: Using Cloudformation and our SAM template

There are a few key parts here that are challenging. The first is the BuildSpec.yml file. Here is an example to make this easy: 

    version: 0.2

    phases:
    install:
        commands: 
            - dotnet tool install -g Amazon.Lambda.Tools

    build:
        commands:
        - dotnet test
        - cd src/mySamLambda/
        - dotnet lambda package --configuration Release --output-package "mySamLambda.zip"
        - cd ../../
        - aws s3 cp mySamLambda.zip s3://${S3_BUCKET}/release/

> This approach has a limitation of overwriting the lambda package, making traceability from running code to source code a bit hard. 
> We'll address that in the next section of automation. 

For full details on CodeBuild and the BuildSpec.yml file see the [AWS Docs](https://docs.aws.amazon.com/codebuild/latest/userguide/concepts.html). 

Setting up the CodePipeline is really easy - follow the defaults. For CodeBuild be sure to select an appropriate machine image. Also, be sure to set up the appropriate inputs and file paths. Most of the defaults are fine. For CodeDeploy, the simplest solution is to select CloudFormattion to create/update. Follow the steps using the path to the SAM template. 

> For CodeBuild be sure to use the lastest default machine image. This supports .NET Core 3.1. 
> At the time of writing the default machine images do not support .NET 5. Use a docker image if .NET 5 is required. 

This provided a simple CI/CD pipeline. The deployment has been automated. 

We can go further on our level of automation. There may be a need to tie the version in prod back to a specific build. The next section will address that. 

# CodePipeline via CloudFormation

The final stage of automation is to automate the pipeline. This means that all configuration is clearly documented in code. It allows the greatest level of control. The entire pipeline can be torn down and rebuilt. Changes across the pipeline are also stored in source control. 

As part of declaring the pipeline via CloudFormation, I'll also show how we can address the issue of code traceability. Simply put, I achieved with the following steps:
- Add the source commit SHA to the lambda zip filename
- pass commit SHA and the filename to the deploy stage
- deploy with the filename, set the commit as an environment variable

## Cloudformation declaration

This can all be seen as an example CloudFormation script. Here is the core subset for a CodePipeline in CloudFormation:

    # Omitted: KmsKey, IAM Roles etc
    # Inputs required: ProjectName, FullRepoId, GitHubBranch


    # An S3 bucket is needed to store the lambda zip file
    S3ArtifactBucket:
        Type: AWS::S3::Bucket
        DeletionPolicy: Delete
        Properties:
        BucketName: !Sub "${ProjectName}-pipeline"
        VersioningConfiguration:
            Status: Enabled

    # This creates a GitHub connection. The Console will still need to be used to configure this. 
    GithubConnection:
        Type: 'AWS::CodeStarConnections::Connection'
        Properties:
        ConnectionName: !Sub "${ProjectName}-connection"

    # The declaration of the how to build the project
    CodeBuild:
        Type: AWS::CodeBuild::Project
        DependsOn: [S3ArtifactBucket]
        Properties:
        Artifacts:
            Type: CODEPIPELINE
        Environment:
            ComputeType: BUILD_GENERAL1_SMALL
            EnvironmentVariables:
            - Name: S3_BUCKET
            Value: !Ref S3ArtifactBucket
            - Name: ProjectName
            Value: !Ref ProjectName
            Image: !Ref CodeBuildImage
            Type: LINUX_CONTAINER
            PrivilegedMode: true
        Name: !Sub "${ProjectName}-codebuild-build"
        ServiceRole: !Ref CodeBuildRole
        EncryptionKey: !Ref KMSKey
        Source:
            Type: CODEPIPELINE
        Tags:
            - Key: app-name
            Value: !Ref ProjectName
        TimeoutInMinutes: 5

    CodePipeline:
        Type: AWS::CodePipeline::Pipeline
        DependsOn: [CodeBuild, GithubConnection]
        Properties:
        Name: !Sub "${ProjectName}-pipeline"
        RoleArn: !Ref CodePipelineRole
        RestartExecutionOnUpdate: true
        Stages:
        - Name: Source
            Actions:
            - InputArtifacts: []
                ActionTypeId:
                Version: '1'
                Owner: AWS
                Category: Source
                Provider: CodeStarSourceConnection
                OutputArtifacts:
                - Name: !Sub "${ProjectName}-SourceArtifact"
                RunOrder: 1
                Configuration:
                ConnectionArn: !Ref GithubConnection
                FullRepositoryId: !Ref FullRepoId
                BranchName: !Ref GitHubBranch
                OutputArtifactFormat: "CODE_ZIP"
                Name: get-source-code
                RunOrder: 1
        - Name: Build
            Actions:
            - Name: build-from-source
            InputArtifacts:
            - Name: !Sub "${ProjectName}-SourceArtifact"
            OutputArtifacts:
            - Name: !Sub "${ProjectName}-BuildArtifact"
            ActionTypeId:
                Category: Build
                Owner: AWS
                Version: "1"
                Provider: CodeBuild
            Configuration:
                ProjectName: !Ref CodeBuild
            RunOrder: 1

        - Name: DeployStaging
            Actions:
            - Name: create-changeset
            InputArtifacts:
            - Name: !Sub "${ProjectName}-BuildArtifact"
            - Name: !Sub "${ProjectName}-SourceArtifact"
            OutputArtifacts: []
            ActionTypeId:
                Category: Deploy
                Owner: AWS
                Version: "1"
                Provider: CloudFormation
            Configuration:
                StackName: !Sub "${ProjectName}-app-stack-staging"
                ActionMode: CHANGE_SET_REPLACE
                ChangeSetName: app-changeset-dev
                Capabilities: CAPABILITY_NAMED_IAM
                TemplatePath: !Sub "${ProjectName}-SourceArtifact::${SAMOutputFile}"
                # TemplateConfiguration allows for a template file which can be used for many enviornments and better configuration 
                TemplateConfiguration: !Sub "${ProjectName}-BuildArtifact::ci/params/staging.json"
                RoleArn: !Sub "arn:aws:iam::${AWS::AccountId}:role/${ProjectName}-cloudformation-role"
            RunOrder: 1
            - Name: execute-changeset
            InputArtifacts: []
            OutputArtifacts: []
            ActionTypeId:
                Category: Deploy
                Owner: AWS
                Version: "1"
                Provider: CloudFormation
            Configuration:
                StackName: !Sub "${ProjectName}-app-stack-staging"
                ActionMode: CHANGE_SET_EXECUTE
                ChangeSetName: app-changeset-staging
                RoleArn: !Sub "arn:aws:iam::${AWS::AccountId}:role/${ProjectName}-cloudformation-role"
            RunOrder: 2

        ArtifactStore:
            Type: S3
            Location: !Ref S3ArtifactBucket
            EncryptionKey:
            Id: !Ref KMSKey
            Type: KMS


## Breakdown of the template

This appears to be quite a lot. Let's break it down from top to bottom. 
- Create an S3 bucket to hold the zip of the compiled code. 
- Define how to build the code using CodeBuild. 
    - This will default to using `BuildSpec.yml` in the root of the project. 
    - This is also how we can change to using a docker image with .NET 5 if required. 
- Define the pipeline 
    - The main section is the `stages` section
    - Define a source input using a GitHub connection
        - The connection requires a one-time setup via the Console
    - Define the build stage and link to our CodeBuild already defined. 
        - This also defined two outputs, source and build artifacts
        - The build artifacts are required to pass the code commit SHA. 
    - Define the CloudFormation deploy 
        - In this case, it is declared with two actions, a change-set and an execute changeset. 
        - The CloudFormation file (our SAM template) is parameterized to accept the file path/code commit SHA
        - The code commit SHA is passed in via a params file that is updated from the code build stage

## Update BuildSpec.yml with versioning/code commit SHA

The pipeline above also requires some changes to the `BuildSpec.yml` file. Here are the important additions/changes

    # Header is the same as above

    # Get the code commit SHA
    pre_build:
        commands:
        - COMMIT_COMMIT_ID=`echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7`

    # Update build commands to use the SHA in the file name
    build:
        commands:
        - dotnet test
        - cd src/mySamLambda/
        - dotnet lambda package --configuration Release --output-package "mySamLambda-${COMMIT_COMMIT_ID}.zip"
        - cd ../../
        - mv "src/mySamLambda/mySamLambda-${COMMIT_COMMIT_ID}.zip" .
        - echo Uploading mySamLambda-${COMMIT_COMMIT_ID}.zip to s3://${S3_BUCKET}/release/
        - aws s3 cp mySamLambda-${COMMIT_COMMIT_ID}.zip s3://${S3_BUCKET}/release/

    # Cloudformation/SAM template is now parameterized. Update parameters file with the required values. 
    # This can be extended to handle many environments. 
    post_build:
        commands:
        # set the CodeCommitSHA for each enviornment
        - sed -i.bak 's/\$CodeCommitSHA\$/'${COMMIT_COMMIT_ID}'/g' ci/params/staging.json

        # set the S3 bucket for each enviornment
        - sed -i.bak 's/\$S3Bucket\$/'${S3_BUCKET}'/g' ci/params/staging.json

        # set the ProjectName ie name of file pathfor each enviornment
        - sed -i.bak 's/\$ProjectName\$/'${ProjectName}'/g' ci/params/staging.json

    # output the params file to be used and input the Cloudformation/SAM template. 
    artifacts:
    files:
        - ci/params/staging.json

Here is an example of the params file: 

    {
        "Parameters": {
            "CodeCommitSHA": "$VERSION$",
            "S3Bucket": "$S3Bucket$",
            "ProjectName": "$ProjectName$"
        }
    }

## Update SAM template with params

To complete the sequence, here is how the SAM template is updated to accept the parameter filename: 


    AWSTemplateFormatVersion: '2010-09-09'
    Transform: AWS::Serverless-2016-10-31
    Description: An AWS Serverless Application that uses the ASP.NET Core framework running
    in Amazon Lambda.

    Parameters: 
    S3Bucket:
        Type: String
    CodeCommitSHA:
        Type: String
        Description: The version of the docker image to release
    ProjectName:
        Type: String
        Description: Name of the application.

    Conditions: {}

    Resources:

    # Ommited some resources for brevity

    AspNetCoreFunction:
        Type: AWS::Serverless::Function
        Properties:
        CodeUri: 
            Bucket: !Ref 'S3Bucket'
            Key: !Join 
            - ''
            - - 'release/'
                - !Ref 'ProjectName'
                - '-'
                - !Ref 'CodeCommitSHA'  
                - '.zip'

    # Ommited rest of file for brevity


## For more configuraiton

For more configuration AWS some good examples: 
- [Continuous delivery with CodePipeline](https://docs.aws.amazon.com/AWSCloudFormation/latest/-UserGuide/continuous-delivery-codepipeline.html)
- [Create a pipeline with AWS CloudFormation](https://docs.aws.amazon.com/codepipeline/latest/userguide/tutorials-cloudformation.html)
- [CodePipeline-Nested-CFN](https://github.com/aws-samples/codepipeline-nested-cfn)


# Strech Goal: Notifications

CodePipeline also supports notificaiton. This is very easy to setup via the Console. Notifcations use the AWS SNS service. This means that TXT, EMail or IM (Slack) can be used. 
I found the slack intergration to be the most useful. 
There are options to configure when notifcaitons are sent. I opted to be notified every time the pipeline started, and each time an action failed. This allows me to know the pipeline is running as well as when something has gone wrong. 


# Wrap up

Each project needs a different level of automation. The final section takes a lot of time to setup. It will also require quite a bit of configuration for your needs - don't underestimate those file path changes required!



Applying a CI/CD practices to a project helps us avoid the silly mistakes, and ship more ~~busines value~~ cool shit!