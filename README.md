# Project Title

This repository serves as a starting point for teams that want to build a continuous integration and deployment
pipeline for AWS AppSync applications.

## Getting Started

These instructions will get you a copy of the project and show you how to deploy these resources via AWS CloudFormation.

## Create an S3 bucket for your pipeline artifacts

Open the [Amazon S3 console](https://s3.console.aws.amazon.com) and create a new bucket to contain
your pipeline's artifacts (a fancy word for files). Make sure that you enable **versioning** for the
bucket when creating the bucket.

## Create a zip file containing your deployment artifacts

From the root of this repository run the following to create a zip file containing your
application resources.

```
zip -D artifacts.zip artifacts/*
```

This will create an artifact.zip file in the root of this project.

## Upload the artifacts.zip file to S3

Go back to the [Amazon S3 console](https://s3.console.aws.amazon.com) and upload the `artifacts.zip` file that you just created.
If you upload the zip file to a subfolder in the bucket, write down the path because you are going to need it later.

## Deploy your pipeline

Open the [AWS CloudFormation console](https://us-west-2.console.aws.amazon.com/cloudformation/home) and create a new stack
using the `pipeline.yml` template that is included in the repository.

Click "Create Stack" and on the next page select **Upload a template to Amazon S3**. Click **Browse** and select the `pipeline.yml` file in
this repository. On the next page you will need to provide some paramters for your stack. They should look something like this:

![Pipeline Stack Parameters](/images/pipeline-stack-parameters.png)

Click next, and then on the following page, click next again. On the last "review" page, check the checkbox next to "I acknowledge that AWS CloudFormation might create IAM resources." and then click next a final time to create your stack.

## Trigger a build

This pipeline is triggered when you upload a new `artifacts.zip` file to S3. Every time a file at the key you provided when creating the
pipeline stack is updated, a build will trigger.

Go back to the [Amazon S3 console](https://s3.console.aws.amazon.com) and upload the `artifacts.zip` file one more time.

## Watch your build on Code Pipelines

Open the [AWS CodePipelines console](https://console.aws.amazon.com/codesuite/codepipeline/pipelines) and click on your new pipeline.
If you uploaded the new `artifacts.zip` file to S3 in the last step, a build should already have started.

## The pipeline

This pipeline does this following:

1. It creates a new test stack using the CloudFormation templates in your artifacts directory.
2. After the stack completes being created, the pipeline will stop and wait for manual approval.
3. After you grant manual approval, the pipeline will destroy the test stack and move on to the production phase.
4. In the production phase, the pipeline will create a change set that diffs your new stack against the existing stack (if one exists).
5. The pipeline will then stop and wait for you to approve the change set.
6. After approving the changeset, the pipeline will executre the change set and update your production stack.

## Next steps

At this point, you will have a pipeline that implements continuous integration and deployment for an AWS AppSync API. For next steps consider these

1. Add a new source stage that allows you trigger builds when you push code to GitHub, AWS CodeCommit, or some other git hosting service.
2. Add a build phase that uses AWS CodeBuild to build your assets instead of manually zipping your artifacts directory.
3. Use a CodePipeline Invoke action to call a lambda function to run integration tests against your test stack in the test phase.
4. Use a CodePipeline Invoke action that calls a lambda function the diffs the GraphQL schema in your test stack against the GraphQL schema in your production stack to detect breaking changes before executing your change set.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
