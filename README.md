# systems-management

The demonstration of AWS System Manager with Change Calendar to manage the "holiday" event. if your batch application depends on the enterprise calendar, please keep reading the instratuion. 

First of all, we will have the Capital One Enterprise Calendar which inlcudes all the holidays. there is no limiation for the downstream service to get the Systems Manager Change Calendar state. 
>note: if you are using different AWS Account, across AWS Account or create a new enterprise holiday calendar are needed. 

The Falcon9 Team is able to privode you the holiday josn foramat to easiler for you to input, also, you are able to create your own calendars.




from this demo, I will proivde two options to you to choose:
1. you are your own EventBridge rule, like runs daily, your batch task need to check the enterprise calendar (code changes is required), if today is holiday, the batch job is suppsoed to stop running. 
![Lambda Check Change Calendar States](./img/lambda_check_change_calendar_state.jpg)
2. Leveraging the EventBridge rule to disable/enable your rules, there is no code changes.
![Lambda Check Change Calendar States](./img/lambda_check_change_calendar_state.jpg)

no matter which way you'd like to choose, let's steps on the Change Calendar Creation.

## Overview
1. Create a Systems Manager Change Calender: define your calendar events
2. Create an EventBridge Rule: Triggered by the change Calendar Evetns,
3. Create a Lambda function: Triggered by EventBridge only when the calendar is OPEN
4. Configure IAM Roles: Ensure proper permissions for all services. 

### Steps
1. Create the Change Calendar with DEFAULT_OPEN. (Note: the change calender default should be OPEN, until the events be created, which events have CLOSED state in detail)
![Systems Manager Change Calendar Creation with Default Open](./img/holiday_calendar.jpg)

the calendar use will be looked like below. 
Expliantaion: 
1. Service is SSM (Systems Managr)
2. API: check the calendar state.
```json
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

2. Create the EventBridge Rule which runs the lambda daily
![EventBridge rule runs daily](./img/eventbridge_rule_runs_daily.jpg)
## Daily Lambda Execution Before checking the Change Calendar State

## Daily Lambda execution will be triggerred by two eventbridge rule. 




