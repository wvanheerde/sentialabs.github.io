---
layout: post
title: Github CodeBuild Integration
banner: /assets/posts/2017-05-09-Github-CodeBuild-Integration/aws-codebuild.png
github: svdgraaf/github-codebuild-webhook
author: svdgraaf
---

As you might know, AWS CodeBuild is a service by AWS which can run your integration test or builds for you. It can be triggered by CodePipeline to deliver artifacts, and you can use CodeDeploy to deploy those artifacts to your servers.

Unfortunately, it's currently not possible to connect CodeBuild directly to Github. Which is where this project comes in. This project creates an endpoint for your github repository webhook, which is called every time you create/update a Pull Request.

The nice thing about having your builds run in AWS CodeBuild, is that everyting is completely serverless. Everything also runs within your own AWS account, so you don't have to setup any additional billing, etc. On top of that, your AWS CodeBuild project builds are free for the first month (based on the free tier)! And ofcourse you get IAM control out of the box.

You can find all the source code here: [https://github.com/svdgraaf/github-codebuild-webhook](https://github.com/svdgraaf/github-codebuild-webhook)

Quick launch 🚀
---------------
If you just want to launch the service yourself, you can use this magic button which will setup everything for you in your AWS account through the magic of CloudFormation:

[![Launch Awesomeness](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks/new?stackName=serverless-build-trigger&templateURL=https://s3-eu-west-1.amazonaws.com/github-webhook-artifacts-eu-west-1/serverless/github-webhook/trigger/1494319871068-2017-05-09T08%3A51%3A11.068Z/compiled-cloudformation-template.json)

Architecture
------------
![Flow](https://raw.githubusercontent.com/svdgraaf/github-codebuild-webhook/master/architecture.png)

How does this work?
-------------------
When you deploy the [CloudFormation template](https://s3-eu-west-1.amazonaws.com/github-webhook-artifacts-eu-west-1/serverless/github-webhook/trigger/1494342303416-2017-05-09T15%3A05%3A03.416Z/compiled-cloudformation-template.json), a couple of things happen. First, some arbitrairy resources are created (IAM roles, Loggroups, etc). The Lambda functions are created, then a Step Function is created which points to the Lambda functions. An ApiGateway is created which is connected to the Step Function. And lastly a Custom Resource is created, which calls another Lambda function, which installs the webhook on the repository that you configured.

When you create a PR on your repository, a notification is send to the ApiGateway endpoint and the Lambda Step Function is triggered. This will trigger the start of your CodeBuild project, and sets a status of `pending` on your specific commit.

While the build is running, the Lambda Step Function will check the status of your build every X seconds.

When the status of the build changes to `done` or `failed`, the Github api is called and the PR status will be updated accordingly.

Setup steps
-----------
Use the steps below to launch the stack directly into your AWS account. You can setup as many stacks as you want, as the stack is currently connected to 1 CodeBuild project.

1. First, we'll need to setup an [AWS CodeBuild](https://eu-west-1.console.aws.amazon.com/codebuild/home) project. Create a new project in the AWS console, and be sure to add a [`buildspec.yml`](http://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html) file to your project with some steps. Here's an [example](https://github.com/svdgraaf/webhook-test/blob/foobar/buildspec.yml).
2. Create a github api token in your account here, so that the stack is allowed to use your account: [https://github.com/settings/tokens/new](https://github.com/settings/tokens/new). You can ofcourse choose to setup a seperate account for this.
3. Deploy the stack:

   [![Launch Awesomeness](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks/new?stackName=serverless-build-trigger&templateURL=https://s3-eu-west-1.amazonaws.com/github-webhook-artifacts-eu-west-1/serverless/github-webhook/trigger/1494331984949-2017-05-09T12%3A13%3A04.949Z/compiled-cloudformation-template.json)

	(or use `sls deploy`).

4. Create a Pull Request on your project, and see the magic be invoked 😎
5. ...
6. Profit!

Example video (nay, gif!)
-------------------------
In the example video below, a PR is created, and a build runs, which fails. Then, a new commit is pushed, a new build is started, and it succeeds! When you click on the 'details' link of the PR status, it will take you to the CodeBuild build log.

![AWS Codebuild Triggered after PR update](https://github.com/svdgraaf/github-codebuild-webhook/blob/master/example.gif?raw=true)
