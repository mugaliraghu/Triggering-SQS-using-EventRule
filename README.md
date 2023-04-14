
## This CloudFormation template creates an EventBridge rule that filters for EC2 instance state-change notifications and sends the filtered events to an SNS topic and an SQS queue. The SNS topic has a subscription for email notifications.

```t
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: Serverless patterns - EventBridge to SNS

Resources:

  # Define the SNS topic
  MySnsTopic:
    Type: AWS::SNS::Topic
    Properties:
      DisplayName: "My SNS Topic"
      TopicName: "my-sns-topic"
      Subscription:
        - Protocol: "email"
          Endpoint: "raghavendra.m@zapcg.com"

  MySqsQueue:
    Type: AWS::SQS::Queue
    Properties:
      QueueName : "MYSQSQUEUE"


  # Define the event rule to filter for events
  EventRule:
    Type: AWS::Events::Rule
    Properties:
      Description: "EventRule"
      EventPattern:
        source:
          - "aws.ec2"
        detail-type:
          - "EC2 Instance State-change Notification"
      Targets:
        - Arn: !Ref MySnsTopic
          Id: "SNStopic"
          InputTransformer:
            InputPathsMap:
              "instance" : "$.detail.instance-id"
              "state" : "$.detail.state"
            InputTemplate: |
              "instance <instance> is in <state>"
        - Arn: !GetAtt MySqsQueue.Arn
          Id: "SQSqueue"
          InputTransformer:
            InputPathsMap:
              "instance" : "$.detail.instance-id"
              "state" : "$.detail.state"
            InputTemplate: |
              "instance <instance> is in <state>"
            
          
          
  # Allow EventBridge to invoke SNS
  EventBridgeToToSnsPolicy:
    Type: AWS::SNS::TopicPolicy
    Properties:
      PolicyDocument:
        Statement:
        - Effect: Allow
          Principal:
            Service: events.amazonaws.com
          Action: sns:Publish
          Resource: !Ref MySnsTopic
      Topics:
        - !Ref MySnsTopic
        
  EventBridgeToToSqsPolicy:
    Type: AWS::SQS::QueuePolicy
    Properties:
      PolicyDocument:
        Statement:
        - Effect: Allow
          Principal:
            Service: events.amazonaws.com
          Action: SQS:SendMessage
          Resource:  !GetAtt MySqsQueue.Arn
      Queues:
        - Ref: MySqsQueue
   ```
