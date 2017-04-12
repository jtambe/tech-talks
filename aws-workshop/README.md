## Launching the Stack

1. Before launching the stack, create a fork of this [sample java application](https://github.com/ebracho/spring-petclinic). It contains specifications on how CodeBuild and CodeDeploy should build and deploy the app.  

2. Launch the CloudFormation stack by clicking this button:
[![Launch Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home#/stacks/new?stackName=cloudformation-codepipeline-example&templateURL=https://s3-us-west-2.amazonaws.com/codepipeline-blog/codepipeline-cfn.yml)  

3. Fill in the parameters and follow the prompts to complete the launch. For the
GitHub user/repo/branch provide your GitHub username and the master branch of the
fork you created in step 1. If you don't have a GitHub OAuth token, you can generate
one [here](https://github.com/settings/tokens).  

4. Watch and wait while CloudFormation provisions all of the resources required
this stack. The process should take roughly 5 minutes.  
