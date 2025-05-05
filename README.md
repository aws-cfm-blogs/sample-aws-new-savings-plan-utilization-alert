# How to Set Up Automated Alerts for Newly Purchased AWS Savings Plans

**Syed Muhammad Tawha**, Principal Technical Account Manager
**Dan Johns**, Sr. Solution Architect

As organizations grow, FinOps teams need a holistic view of their AWS Savings Plans commitments to ensure optimal utilization. The solution involves implementing monitoring systems and automated alerts to identify underutilized Savings Plans within the eligible return period.

When you purchase a Savings Plan, you make a commitment for one or three years. Savings Plans with an hourly commitment of $100 or less can be returned if they were purchased within the last seven days and in the same calendar month, provided you haven't reached your return limit. Once the calendar month ends (UTC time), these purchased Savings Plans cannot be returned.

In this blog post, we provide [AWS CloudFormation](https://aws.amazon.com/cloudformation/) template that creates [AWS Step Functions](https://aws.amazon.com/step-functions/) state machine, [Amazon Simple Notification Service (SNS)](amazon.com/sns/) topic, [Amazon EventBridge](https://aws.amazon.com/eventbridge/) scheduler, and necessary [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/) roles to automate the monitoring of newly purchased Savings Plans and highlight those that are underutilized. The template simplifies the deployment process by creating all required resources with a single stack deployment.

## Overview of Solution:
This solution deploys a state machine that analyzes all Savings Plans in your account, including those purchased across your organization for payer accounts. The workflow filters active Savings Plans based on their purchase date, focusing on those acquired within the last 7 days and the current calendar month. It then evaluates their utilization rates and identifies plans falling below the defined threshold. The state machine executes at your specified frequency and uses Amazon SNS to send email alerts to addresses you provide during CloudFormation stack creation. These alerts contain detailed information about low-utilization Savings Plans and instructions for the return process.

![diagram](https://github.com/user-attachments/assets/4e06c569-353e-4cc9-b885-7c0939f1ac70)


## Solution Walk Through:

### Prerequisites
* An AWS account
* IAM permissions to deploy Step Functions, IAM roles, SNS, and EventBridge scheduler

### Deploy the solution
In this section we will deploy sources for this solution in your account:

1. Login to your [AWS Management Console](https://aws.amazon.com/console/) where you have bought Savings Plans
2. Deploy this [CloudFormation stack](https://github.com/aws-samples/sample-aws-new-savings-plan-utilization-alert/blob/main/sample-aws-new-savings-plan-utilization-alert.yaml)
3. Provide the Stack Name as `new-sp-utilization-alert`
4. In the AlertEmails parameter, enter a comma-separated list of email addresses that will receive notifications about underutilized Savings Plans.
5. In the ScheduleExpression parameter, specify the execution frequency for the Step Functions state machine using cron format (default is daily at 9 AM UTC).
6. In the UtilizationThreshold parameter, specify the minimum utilization percentage for your Savings Plans. You will receive alerts when utilization falls below this threshold.
7. Click Next, select the acknowledge box and create stack.
8. Wait until the stack has finished deploying and is showing as `CREATE-COMPLETE`
9. You will receive an email to confirm your subscription to the SNS topic created by this stack. Please confirm the subscription to begin receiving notifications.

### Test the Solution
Now that the Step Functions state machine and associated resources are deployed in your account, let's test the deployment:

1. Navigate to the Resources tab in your CloudFormation stack and locate the `SavingsPlansAlerts` Step Functions state machine. Click the blue hyperlink.
2. You will be redirected to the Step Functions console. Click the `Start execution` button on the right.
3. View the execution details in the Events section to monitor the state machine's progress. If you have any Savings Plans purchased within the last 7 days and the current calendar month, you will receive email notifications.
4. A successful execution is indicated by a green box in the Graph view. If any Savings Plans fall below your specified utilization threshold, you will receive an email at your provided address.

### Clean Up
All resources deployed for this solution can be removed by deleting the CloudFormation stack. You can delete the stack through either the AWS Management Console or the [AWS CLI](amazon.com/cli/).

To delete the Cost Optimization Conformance Pack solution (CLI):
```bash
aws cloudformation delete-stack --stack-name new-sp-utilization-alert  
```
### Conclusion
In this post, we explored how to use the Savings Plan and Cost Explorer APIs to identify underutilized Savings Plans in your account. We then demonstrated how to use a Step Functions State Machine to filter Savings Plans purchased within the last 7 days and the current calendar month. This timing is crucial because you can return Savings Plans within the return window if they were purchased inadvertently or aren't being utilized effectively. For guidance on returning a purchased Savings Plan, please refer to the [Returning a Purchased Savings Plan](https://docs.aws.amazon.com/savingsplans/latest/userguide/return-sp.html) documentation.


