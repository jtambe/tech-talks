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
[![Launch Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home#/stacks/new?region=us-east-1&stackName=cloudformation-codepipeline-example&templateURL=https://s3-us-west-2.amazonaws.com/codepipeline-blog/codepipeline-cfn.yml)  
  
3. Fill in the parameters and follow the prompts to complete the launch. For the
GitHub user/repo/branch provide your GitHub username and the master branch of the
fork you created in step 1. If you don't have a GitHub OAuth token, you can generate
one [here](https://github.com/settings/tokens).  
  
4. Watch and wait while CloudFormation provisions all of the resources required
this stack. The process should take roughly 5 minutes.  
  
### Setting up your Alexa Skill

1. Access the [Amazon Developer Panel](https://developer.amazon.com/alexa) and sign in to create a new skill. Select the 'Alexa' tab, and follow the 'Get Started' link for the Alexa Skills Kit

2. Click on 'Add a New Skill' to begin creating your very own skill! The name can be anything you like, but set the invocation name to `code deploy`. Make sure the radio button for `Custom Interaction Model` is selected.

![Entering Skill information](/aws-workshop/img1.png)

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

5. You are ready to move on to the lambda skill for this project. Go The AWS dashboard and navigate to lambda. It is important you create your lambda function in **US-EAST-1**, which is **N. Virginia**. If you create it in a different region, Alexa won't be able to interact with your function. On the lambda dashboard select, 'Create a lambda function'.

6. Select the `Python 2.7` runtime and the `alexa-skills-kit-color-expert-python` blueprint to get started. Name your function whatever you like and fill out description with somethign relevant. For the IAM role, select "Create Custom Role" copy [this](https://s3-us-west-2.amazonaws.com/codepipeline-blog/alexa_iam_role.json) policy document to the new role (make sure to give the IAM role a memorable name). Leaving the rest of the options set to their default values, create the new skill.

7. Add the following Python code to the body of your Lambda function. You may or may not want to edit the code in a local environment before trying it out on lambda/alexa, as the errors generated by those services can be cryptic. **This code is not 100% complete**. We have left some of the finer details up to you, so you can finish writing them yourself and gain a better understanding of the Code\* toolset, but this is a good starting point. 

  * Please change your pin to something other than "1337", so that other people do not deploy your build!
  * There are three *TODO* sections that need to be completed to get the skill working.
  * The [boto3](https://boto3.readthedocs.io/en/latest/reference/services/codepipeline.html) codepipeline documentation has all the information you need to complete this skill.

```

"""
This skill was created to interact with codepipeline deployments. This was designed for a Chico-Liatrio
AWS meetup in Chico, CA April 2017
"""
from __future__ import print_function
import json
import boto3

pin = "1337"
pending = {}

# --------------- functions for querying ec2. ----------------------

codepipeline = boto3.client('codepipeline', region_name='us-east-1')

def get_approval_action_ids():
    """Generates ids for all approval all approval actions in all existing pipelines.
    Yield:
        dict: { 'pipeline': String, 'stage': String, 'action': String }
    """
    for pipeline in codepipeline.list_pipelines()['pipelines']:
        pipeline_details = codepipeline.get_pipeline(name=pipeline['name'])
        for stage in pipeline_details['pipeline']['stages']:
            for action in stage['actions']:
                if action['actionTypeId']['category'] == 'Approval':
                    yield { 
                        'pipeline': pipeline['name'], 
                        'stage': stage['name'], 
                        'action': action['name']
                    }

def get_pending_approval_actions():
    """Generates approval_action_ids and tokens for all pending approval actions accross all pipelines
    Yield: 
        dict: { 'approval_action_id': Dict, 'token': String }
    """
    
    for approval_action_id in get_approval_action_ids():
        #
        # TODO: Use the boto3 to get the pipeline state identified by `approval_action_id` 
        # Hint: use the `codepipline` client created above.
        #
        pipeline_state = "wrongstate" #codepipeline.something
        stage_state = next(s for s in pipeline_state['stageStates'] if s['stageName'] == approval_action_id['stage'])
        action_state = next(a for a in stage_state['actionStates'] if a['actionName'] == approval_action_id['action'])
        if 'latestExecution' in action_state and 'token' in action_state['latestExecution']:
            yield { 
                'approval_action_id': approval_action_id, 
                'token': action_state['latestExecution']['token']
            }


def submit_approval_result(pending_approval_action, status, summary):
    codepipeline.put_approval_result(
        pipelineName=pending_approval_action['approval_action_id']['pipeline'],
        stageName=pending_approval_action['approval_action_id']['stage'],
        actionName=pending_approval_action['approval_action_id']['action'],
        result = {
            'status': status,
            'summary': summary
        },
        token=pending_approval_action['token']
    )


# --------------- Helpers that build all of the responses ----------------------

def build_speechlet_response(title, output, reprompt_text, should_end_session):
    return {
        'outputSpeech': {
            'type': 'PlainText',
            'text': output
        },
        'card': {
            'type': 'Simple',
            'title': title,
            'content': output 
        },
        'reprompt': {
            'outputSpeech': {
                'type': 'PlainText',
                'text': reprompt_text
            }
        },
        'shouldEndSession': should_end_session
    }


def build_response(session_attributes, speechlet_response):
    return {
        'version': '1.0',
        'sessionAttributes': session_attributes,
        'response': speechlet_response
    }



# ------------------- Response Handlers  ---------------------------

def get_welcome_response():

    session_attributes = {}
    card_title = "Welcome"
    speech_output = "You can check your queued code pipeline deployments, and even deploy one simply by speaking commands! Try asking, Do I have any builds queued?"
    reprompt_text = "Try asking, Do I have any builds queued?"
    should_end_session = False
    return build_response({}, build_speechlet_response(
        card_title, speech_output, reprompt_text, should_end_session))

def handle_session_end_request():
    card_title = "Session Ended"
    speech_output = "Goodbye"
    should_end_session = True
    return build_response({}, build_speechlet_response(
        card_title, speech_output, None, should_end_session))

def alexa_get_pending(intent, session):
    card_title = "Pending CodeDeploy Builds:"
    speech_output = get_list_response_string()
    if speech_output == "empty":
        speech_output = "No builds queued, have a nice day!"
        should_end_session = True
        return build_response({}, build_speechlet_response(
            card_title, speech_output, None, should_end_session))
    else:
        should_end_session = False
        reprompt_text = "Select one of the options by saying deploy followed by the number you want, or get the list again by asking me to check the queue"
        return build_response({}, build_speechlet_response(
            card_title, speech_output, reprompt_text, should_end_session))
    
def deploy(intent, session):
    populate_pending_list()
    card_title = "Pin protected.."
    selection = int(intent['slots']['Choice']['value'])
    if not selection in pending:
        speech_output = "Unable to find build number: " + str(selection) + ", try deploying one of the values listed instead, or get the list again by asking me to check the queue"
        reprompt_text = "try deploying one of the values listed instead, or get the list again by asking me to check the queue"
    else:
        speech_output = "to deploy " + pending[selection]['approval_action_id']['pipeline'] + ", please enter your pin number "
        reprompt_text = "try saying Enter Pin followed by your four-digit pin number"
    should_end_session = False
    params = {"Selection":{"name" : "Selection", "value":str(selection) }}
    return build_response(params, build_speechlet_response(
    card_title, speech_output, reprompt_text, should_end_session))
        
def enter_pin(intent, session):
    populate_pending_list()
    entered_pin = intent['slots']['Pin']['value']
    selection = int(session['attributes']['Selection']['value'])
    if selection == 0:
        card_title = "No selection found"
        speech_output = "no selection made, try checking queued builds first"
        should_end_session = False
    elif entered_pin != pin:
        card_title = "Incorrect pin number"
        speech_output = "incorrect pin number, please try again"
        should_end_session = False
    else:
        card_title = "Deploying.."
        speech_output = "Deploying" + pending[int(selection)]['approval_action_id']['pipeline']
        should_end_session = True
        #
        # TODO: Use a function provided in this script to approve the deployment  
        #
    return build_response({}, build_speechlet_response(
        card_title, speech_output, None, should_end_session))

def handle_session_end_request():
    card_title = "Session Ended"
    speech_output = "Goodbye"
    should_end_session = True
    return build_response({}, build_speechlet_response(
        card_title, speech_output, None, should_end_session))

# --------------- Interaction functions for Alexa  -----------------

def populate_pending_list():
    pending.clear()
    counter = 1
    for approval_action in get_pending_approval_actions():
        pipeline_name = approval_action['approval_action_id']['pipeline']
        pending[counter] = approval_action    
        counter +=1

def get_list_response_string():
    if not list(get_pending_approval_actions()):
        response = "empty"
        return response
        
    response = "Your queue: \n"
    counter = 1
    
    for approval_action in get_pending_approval_actions():
        pipeline_name = approval_action['approval_action_id']['pipeline']
    	response += str(counter) + ", " + pipeline_name + ".\n"
    	counter += 1
    response += "\n to deploy, simply say deploy followed by the number you want to approve."
    return response
    
# ------------------------------- DEFAULT Alexa handlers -----------------------

def lambda_handler(event, context):
    """ Route the incoming request based on type (LaunchRequest, IntentRequest,
    etc.) The JSON body of the request is provided in the event parameter.
    """
    print("event.session.application.applicationId=" +
          event['session']['application']['applicationId'])
    
    if event['session']['new']:
        on_session_started({'requestId': event['request']['requestId']},
                           event['session'])

    if event['request']['type'] == "LaunchRequest":
        return on_launch(event['request'], event['session'])
    elif event['request']['type'] == "IntentRequest":
        return on_intent(event['request'], event['session'])
    elif event['request']['type'] == "SessionEndedRequest":
        return on_session_ended(event['request'], event['session'])


def on_session_started(session_started_request, session):
    """ Called when the session starts """

    print("on_session_started requestId=" + session_started_request['requestId']
          + ", sessionId=" + session['sessionId'])


def on_launch(launch_request, session):
    """ Called when the user launches the skill without specifying what they
    want
    """
    print("on_launch requestId=" + launch_request['requestId'] +
          ", sessionId=" + session['sessionId'])
    return get_welcome_response()


def on_intent(intent_request, session):
    """ Called when the user specifies an intent for this skill """

    print("on_intent requestId=" + intent_request['requestId'] +
          ", sessionId=" + session['sessionId'])

    intent = intent_request['intent']
    intent_name = intent_request['intent']['name']


# ---------- custom intents ---------------
    if intent_name == "GetQueueIntent":
        return alexa_get_pending(intent, session)
    elif intent_name == "DeployIntent":    
         # 
         # TODO: Call appropriate handler function (see handlers written above)
         #
         pass
    elif intent_name == "EnterPinIntent":
        return enter_pin(intent, session)
    elif intent_name == "AMAZON.HelpIntent":
        return get_welcome_response()
    elif intent_name == "AMAZON.CancelIntent" or intent_name == "AMAZON.StopIntent":
        return handle_session_end_request()
    else:
        raise ValueError("Invalid intent")
# ------------------------------------------

def on_session_ended(session_ended_request, session):
    """ Called when the user ends the session.
    Is not called when the skill returns should_end_session=true
    """
    print("on_session_ended requestId=" + session_ended_request['requestId'] +
          ", sessionId=" + session['sessionId'])

```

8. With your skill done (or partly done) you can connect the two AWS resources. To do this locate the arn in the top right corner of the lamda dashboard for your specific lambda function. Copy this arn number into the Alexa Developer Console's configuration tab. Make sure North America is checked, and save your skill. You should now be able to access it directly from the echo Dot you set by using the invocation name.

![ASK configuration panel with ARN](/aws-workshop/img2.png)
