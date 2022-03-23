# Get notified when your AWS Lambda Python code raises an exception using SAM CLI

*Tags:* AWS | Serverless | Lamda | SNS | CloudWatch

A couple of months ago, I started working on a project which involves some Python code on AWS Lambda. The code handles some critical movements of data between 
API's and a database. With critical I mean if the data does not get stored correctly it can cost you (ok me) big time.
For that reason I have paid extra attention to the exception handling in the code. When the code raises an exception it means that there is a serious, critical issue
that needs to be addresses asap. The code will stop running for safety however I really wanted a notification when such an event occurs. 

## Setup CloudWatch alarm manually
Luckily, when using AWS that's really simple to do. You log into your account and find your way to CloudWatch > Alarms and from there create a new alarm. 
In your metrics just select "Lambda" and find your function by function name. Check the one containing "errors" and proceed. In the next steps you can set a couple of settings. If have just kept it simple, you can pick them out of the example code below. 
I wanted an alarm for every exception raised. Your requirements might be different.

After setting your threshold settings the fun part starts, the Notification settings. Here you just pick an existing SNS topic or create a new one. If an alarm is triggered, you will receive a notification in your email address at the address you provided. That's basically it.

## Setup with SAM CLI (CloudFormation)
I am not a fan of doing those things manually because it leaves a lot of room for error and inconsistency for example if someone needs to recreate your setup or architecture. Therefore I like to add this kind of actions to my CloudFormation file. Or in my case the SAM CLI template.yaml. The only thing you have to do manually now is to add your email address to the subscriptions of your SNS topic. I did not yet find a way to automate that. If you know, please enlighten me. I would not be a decent nerd if I just left it with an email notification. Next step is definitely a notification on my smartwatch. This is work in progress though.

### template.yaml

```python
LambdaFunctionAlarm:
  Type: 'AWS::CloudWatch::Alarm'
  Properties:
    ActionsEnabled: true
    AlarmName: !Sub '${FunctionName}-alarm'
    AlarmActions:
      - !Sub 'arn:aws:sns:${AWS::Region}:${AWS::AccountId}:${SnsTopicName}'
    Namespace: 'AWS/Lambda'
    MetricName: Errors
    Dimensions:
        - Name: FunctionName
          Value: !Sub '${FunctionName}'
    Statistic: Average
    ComparisonOperator: GreaterThanOrEqualToThreshold
    Threshold: 1
    DatapointsToAlarm: 1
    Period: 5
    EvaluationPeriods: 1
```
