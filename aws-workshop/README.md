# Welcome to the Chico AWS Workshop  
Hosted by [Liatrio](http://liatrio.com) in Association with [ChicoStart](http://chicostart.com/)  
  
## Overview  
 - A Brief Talk on AWS Developer Tools
 - Launch a CloudFormation Stack to spin up a CICD pipeline for Java applications using CodeBuild, CodeDeploy, CodePipeline, and EC2
 - Configure Alexa to Provide an Approval Interface for CodePipeline Actions
 
### AWS Developer Tools
 - CodePipeline: Amazon's deployment orchestration tool. Build-test-and deploy using a variety of differnet tools
 - CloudFormation: AWS resource provisioner that allows the automated creation of various Amazon resources and their dependencies
 - AWS Lambda: 'serverles'computing from Amazon. Runs code on an automatically provisioned and destroyed ec2 instance
 - Alexa Skills Kit(ASK): Amazon's SDK to create alexa skills. Allows the mapping of Skill input to a lambda function.

### Setup/Before you begin
- Create an Amazon AWS account if you do not have one already.
- Download the Alexa smartphone app. You will use this to set up the Echo Dot for this excercise.
- Connect your dot to wifi. Follow the instructions provided by the Dot as well as on your phone.

### Launching the CloudFormation Stack  
  
1. Before launching the stack, create a fork of this [sample java application](https://github.com/ebracho/spring-petclinic). It contains specifications on how CodeBuild and CodeDeploy should build and deploy the app.  
  
2. Launch the CloudFormation stack by clicking this button:
[![Launch Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home#/stacks/new?stackName=cloudformation-codepipeline-example&templateURL=https://s3-us-west-2.amazonaws.com/codepipeline-blog/codepipeline-cfn.yml)  
  
3. Fill in the parameters and follow the prompts to complete the launch. For the
GitHub user/repo/branch provide your GitHub username and the master branch of the
fork you created in step 1. If you don't have a GitHub OAuth token, you can generate
one [here](https://github.com/settings/tokens).  
  
4. Watch and wait while CloudFormation provisions all of the resources required
this stack. The process should take roughly 5 minutes.  
  
### Setting up your Alexa Skill

1. Access the [Amazon Developer Panel](https://developer.amazon.com/alexa) sign in to create a new skill. Select the 'Alexa' tab, and follow the 'Get Started' link for the Alexa Skills Kit

2. Click on 'Add a New Skill' to begin creating your vary own skill! The name can be anything you like, but set the invocation name to `code deploy`. Make sure the radio button for `Custom Interaction Model` is selected.

![Entering Skill information](/path/to/img)

3. Add the following snippet to your 'Intent Schema' section.

```

{
  "intents": [
    {
      "intent": "GetQueueIntent"
    },
    {
      "slots": [
        {
          "name": "Choice",
          "type": "AMAZON.NUMBER"
        }
      ],
      "intent": "DeployIntent"
    },
    {
      "slots": [
        {
          "name": "Pin",
          "type": "AMAZON.NUMBER"
        }
      ],
      "intent": "EnterPinIntent"
    },
    {
      "intent": "AMAZON.HelpIntent"
    },
    {
      "intent": "AMAZON.CancelIntent"
    },
    {
      "intent": "AMAZON.StopIntent"
    }
  ]
}
```

4. Populate the `Sampling Utterances` field in orer to get user input through the echo Dot. Paste the following intents:
```
GetQueueIntent do I have any builds queued
GetQueueIntent check my builds
GetQueueIntent check for queued builds
GetQueueIntent do I have builds waiting
GetQueueIntent are any builds ready
GetQueueIntent are there any builds
GetQueueIntent are there any builds queued
DeployIntent deploy {Choice}
DeployIntent send {Choice}
DeployIntent select {Choice}
DeployIntent select option {Choice}
EnterPinIntent confirmation number {Pin}
EnterPinIntent enter pin number {Pin}
EnterPinIntent enter pin {Pin}
EnterPinIntent {Pin}
```

5. Create an IAM role for alexa to use.

6. You are ready to move on to the lambda skill for this project. Go The AWS dashboard and navigate to lambda. It is important you create your lambda function in **US-EAST-1**, which is **N. Virginia**. If you create it in a different region, Alexa won't be able to interact with your function. On the lambda dashboard select, 'Create a lambda function'.

7. Select the `Python 2.7` runtime and the `alexa-skills-kit-color-expert-python` blueprint to get started. Name your function whatever you like and fill out description with somethign relevant. Leaving the rest of the options set to their default values, create the new skill.

7. 
