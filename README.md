# Systems Manager Change Calendar replace the XXX Enterprise Calendar Solution.

The demonstration of AWS System Manager with Change Calendar to manage the "holiday" event. if your batch application depends on the enterprise calendar, please keep reading the instratuion. 

First of all, we will have the Capital One Enterprise Calendar which inlcudes all the holidays. there is no limiation for the downstream service to get the Systems Manager Change Calendar state. 
>note: if you are using different AWS Account, across AWS Account or create a new enterprise holiday calendar are needed. 

The XXX Team is able to privode you the holiday josn foramat to easiler for you to input, also, you are able to create your own calendars.




from this demo, I will proivde two options to you to choose:
1. **Get Change Calendar State in Lambda Code**. you are your own EventBridge rule, like runs daily, your batch task need to check the enterprise calendar (code changes is required), if today is holiday, the batch job is suppsoed to stop running. 

![Lambda Check Change Calendar States](./img/lambda_check_change_calendar_state.jpg)

2. **Change Calender Triggers EventBridge to invoke downstream Lambda**, Leveraging the EventBridge rule to disable/enable your rules, there is no code changes.

![Lambda Check Change Calendar States](./img/lambda_check_change_calendar_state.jpg)

no matter which way you'd like to choose, let's steps on the Change Calendar Creation.

# Get Change Calendar State in Lambda Code 
There is the code snippet how to get the Change Calendar State
```python
import json
import boto3
import datetime

def lambda_handler(event, context):
    # The Change Calendar Name created on the Systems Manager
    calendar_name = "Holiday_Calendar"
    # Log the event received from EventBridge
    print("Received event: " + json.dumps(event, indent=2))

    ssm = boto3.client('ssm')
    now = datetime.datetime.utcnow().strftime("%Y-%m-%dT%H:%M:%SZ")
    response = ssm.get_calendar_state(
        CalendarNames = [calendar_name], # its able to get muliple calendar states.
        AtTime=now)
        
    print("Checking the ssm calender status:" + json.dumps(response, indent=2))

    if response['State'] == 'OPEN':
        print ("Executing daily task by getting calendar states")
    else:
        print ("Holiday, task not executed  by getting calendar states")
    return {
        "statusCode": 200,
        "body": "Task status logged"
    }
```
### Let's Follow Below Steps to Archieve Solution.
1. Create the Change Calendar with DEFAULT_OPEN.
    1. Open the AWS console, go to AWS Systems Managar.
    2. Click Change Calendar under the Change Management on the left navigator.
    3. Click "Create calendar", and fill out the fields following the screenshot as example
    ![Creating a Calendar](./img/creating_calendar.jpg)
    > (Note: the change calender default should be OPEN, until the events be created, which events have CLOSED state in detail)
    4. Once the calendar has been created, open the calendar and select Details, check the **Calendar use** as below screenshot
    ![Systems Manager Change Calendar Creation with Default Open](./img/holiday_calendar.jpg)

    Explanation: 
    1. Service is SSM (Systems Managr)
    2. Api: GetCalendarState - check the calendar state. _later you will see the creating a role to allow lambda to get the GetCalendarState_
    ```text
    Automation use
    - name: checkChangeCalendarOpen
    action: aws:assertAwsResourceProperty # Asserts an event state for Change Calendar
    onFailure: step:closedCalendar # If Change Calendar state is CLOSED branch to "closedCalendar" step
    timeoutSeconds: 600
    inputs:
    Service: ssm
    Api: GetCalendarState
    CalendarNames: ## List of calendars to check the status.
    - arn:aws:ssm:us-east-1:674415394971:document/Holiday_Calendar
    PropertySelector: "$.State" # Returns OPEN / CLOSED as state
    DesiredValues:
    - OPEN
    nextStep: openCalendar # if Change Calendar state is OPEN “openCalendar” step is executed.
    AWS CLI
    aws ssm get-calendar-state --calendar-names arn:aws:ssm:us-east-1:674415394971:document/Holiday_Calendar --region us-east-1
    CloudWatch Events use. Learn more 
    {
        "source": [
            "aws.ssm"
        ],
        "detail-type": [
            "Calendar State Change"
        ]
    }
    ```
    5. Create events - go to Events tab, and click "Create event".
    
    ![Creating an event](./img/creating_an_event.jpg)
    ![Creating an event](./img/created_an_event.jpg)

2. Create the EventBridge Rule which runs the lambda daily
![EventBridge rule runs daily](./img/eventbridge_rule_runs_daily.jpg)




# Change Calender Triggers EventBridge to invoke downstream Lambda

## Overview
1. Create a Systems Manager Change Calender: define your calendar events
2. Create an EventBridge Rule: Triggered by the change Calendar Evetns,
3. Create a Lambda function: Triggered by EventBridge only when the calendar is OPEN
4. Configure IAM Roles: Ensure proper permissions for all services. 



