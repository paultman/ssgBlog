---
title: "Use CDK to setup AWS for Static Website Hosting"
date: "2021-05-28"
categories: 
  - "tech"
tags: 
  - "aws"
  - "cdk"
  - "codecommit"
  - "s3"
image: "3ceb5-cdk-1.png"
---

This is a continuation from the last post where I used the console to provision and setup the resources needed to host a static website on AWS, complete with SSL certs, CDN resources, code versioning and updates via a simple git push.

Here I will do the same, but I will use Infrastructure as Code (IaC) via the AWS Cloud Development Kit (CDK), which will allow for automated infrastructure creating and updates.

To start, I assume you have used the AWS Command Line Interface or have setup your environment with a IAM user profile already. \[put links from amazon to create/setup/configure IAM user in local environment\].

To start, create a directory for your project, then run the cdk init command:

```
mkdir hello-cdk; cd $_
// first init a project using the "app" template
cdk init app --language typescript
```

that will create the folders and files for a basic CDK project.

\[directory tree screenshot of default project state\]

The program flow starts from the cdk.json file, which runs the typescript file with your project name in the bin directory. There, a CDK App resources is created, which will be the root of all application resources, starting with an Application Stack.

Traditionally you will define your stack in the lib directory. The stack resource follows the normal 3 parameter constructor, scope, physical ID, and a collection of properties. This function is were you will start writing your custom code.

Before we do that, let's add in the AWS resources we will be using at the top of the ts file.

```
import * as Cdk from '@aws-cdk/core';
import * as S3 from '@aws-cdk/aws-s3';
import * as Iam from '@aws-cdk/aws-iam';
import * as CodeCommit from '@aws-cdk/aws-codecommit';
import * as CodeBuild from '@aws-cdk/aws-codebuild';
import * as CodePipeline from '@aws-cdk/aws-codepipeline';
import * as CodePipeline_actions from '@aws-cdk/aws-codepipeline-actions';
import {
CloudFrontWebDistribution,
CloudFrontWebDistributionProps,
OriginAccessIdentity,
} from '@aws-cdk/aws-cloudfront';
```

We will need to add the files to our workspace as well, so back to the command line and run this:

```
npm install aws-cdk @aws-cdk/core @aws-cdk/aws-s3 @aws-cdk/aws-iam @aws-cdk/aws-codecommit @aws-cdk/aws-codebuild @aws-cdk/aws-codepipeline @aws-cdk/aws-codepipeline-actions @aws-cdk/pipelines
```

You'll notice that I installed the aws-cdk. This is because I'll prefer to use the local version of the CDK which I have control over the version rather than the globally installed CDK cli command. One thing to look out for in v1 of the cdk is that all your assets use the same version, including the CDK cli itself. To use the local version, prefix it with "npx." You'll see me do it on my examples. Btw, in v2 of the CDK, that's going to go away as all the assets will be distributed in one package so they will all have the same version.

With those packages installed and using Visual Studio Code, you will get code complete for the imported objects.

Inside our stack, we will start with setting up the Code Commit repo we will be using:

```
  const repo = new CodeCommit.Repository(this, 'WorkshopRepo', {
      repositoryName: `${domainName}Repo`
    });
```

Next, we setup our S3 bucket:

```
    //setup web storage bucket
    const websiteBucket = new S3.Bucket(this, 'websiteBucket', {
      bucketName: `${subDomain}.${domainName}.${domainExtension}`,
      websiteIndexDocument: 'index.html',
      websiteErrorDocument: 'error.html',
      publicReadAccess: true,
      removalPolicy: Cdk.RemovalPolicy.DESTROY, // NOT recommended for production code
    });
    new Cdk.CfnOutput(this, 'Bucket', { value: websiteBucket.bucketName });
```

Our Codebuild setup with a generic buildSpec (you can also use a buildspec.yml file as well did in the previous walkthough).

```
    const staticSiteBuild = new CodeBuild.PipelineProject(this, 'CdkBuild', {
      buildSpec: CodeBuild.BuildSpec.fromObject({
        version: '0.2',
        phases: {
          install: {
            "runtime-versions": { nodejs: 12 },
            commands: 'npm build',
          },
          build: {
            commands: [
              'npm install',
              'npm run build',
            ],
          },
        },
        artifacts: {
          'base-directory': 'dist',
          files: ['**/*']
        },
      }),
      environment: {
        buildImage: CodeBuild.LinuxBuildImage.STANDARD_5_0,
      },
    });
```

Now for the pipeline itself:

```
    const source = new CodePipeline.Artifact();
    const ssCompiled = new CodePipeline.Artifact('ssCompiled');

    new CodePipeline.Pipeline(this, 'Pipeline', {
      stages: [
        {
          stageName: 'Source',
          actions: [
            new CodePipeline_actions.CodeCommitSourceAction({
              actionName: 'CodeCommit_Source',
              branch: 'master',
              repository: repo,
              output: source,
            }),
          ],
        },
        {
          stageName: 'Build',
          actions: [
            new CodePipeline_actions.CodeBuildAction({
              actionName: 'CDK_Build',
              project: staticSiteBuild,
              input: source,
              outputs: [ssCompiled],
            }),
          ],
        },
        {
          stageName: 'Deploy',
          actions: [
            new CodePipeline_actions.S3DeployAction({
              actionName: 'copy-files',
              bucket: websiteBucket,
              input: ssCompiled,
              runOrder: 1,
              // @ts-ignore
              accessControl: 'public-read'
            })
          ],
        },
      ],
    });
```

Last, we setup the CloudFront details. First we update our bucket to only allow access from CloudFront:

```
    // setup OAI so that access to storage (S3) is limited to requests though cloudfront
    const cloudFrontOAI = new OriginAccessIdentity(this, 'OAI', {
      comment: `OAI for ${domainName} website.`,
    });

    // add IAM roles for Cloudfront only access to S3
    const cloudfrontS3Access = new Iam.PolicyStatement();
    cloudfrontS3Access.addActions('s3:GetBucket*');
    cloudfrontS3Access.addActions('s3:GetObject*');
    cloudfrontS3Access.addActions('s3:List*');
    cloudfrontS3Access.addResources(websiteBucket.bucketArn);
    cloudfrontS3Access.addResources(`${websiteBucket.bucketArn}/*`);
    cloudfrontS3Access.addCanonicalUserPrincipal(
      cloudFrontOAI.cloudFrontOriginAccessIdentityS3CanonicalUserId
    );
    websiteBucket.addToResourcePolicy(cloudfrontS3Access);
```

Next we setup the configuration for CloudFront

```
    const cloudFrontDistProps: CloudFrontWebDistributionProps = {
      originConfigs: [
        {
          s3OriginSource: {
            s3BucketSource: websiteBucket,
            originAccessIdentity: cloudFrontOAI,
          },
          behaviors: [{ isDefaultBehavior: true }],
        },
      ],
    };
```

Last, we instantiate CloudFront based on the properties we just defined:

```
    const cloudfrontDist = new CloudFrontWebDistribution(
      this,
      `${domainName}-cfd`,
      cloudFrontDistProps
    );
```
