# How to Set Up Automated Alerts for Newly Purchased AWS Savings Plans

**Syed Muhammad Tawha**, Principal Technical Account Manager
**Dan Johns**, Sr. Solution Architect

As organizations expand, FinOps teams require a comprehensive overview of AWS [Savings Plans](https://aws.amazon.com/savingsplans/) commitments to maximize utilization efficiency. The solution involves implementing monitoring systems and automated alerts to identify underutilized Savings Plans within the eligible return period.

When you purchase a Savings Plan, you make a commitment for one or three years. Savings Plans with an hourly commitment of $100 or less can be returned if they were purchased within the last seven days and in the same calendar month, provided you haven't reached your return limit. Once the calendar month ends (UTC time), these purchased Savings Plans cannot be returned.

In this blog post, we provide [AWS CloudFormation](https://aws.amazon.com/cloudformation/) templates that create [AWS Step Functions](https://aws.amazon.com/step-functions/) state machine, [Amazon Simple Notification Service (SNS)](amazon.com/sns/) topic, [Amazon EventBridge](https://aws.amazon.com/eventbridge/) scheduler, and necessary [AWS Identity and Access Management (IAM)](https://aws.amazon.com/iam/) roles to automate the monitoring of newly purchased Savings Plans and highlight those that are underutilized. 

## Overview of Solution:
This solution follows AWS security best practices by separating the deployment across two accounts. One CloudFormation stack will be created in the [Management account](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_getting-started_concepts.html#management-account) to establish necessary IAM roles for fetching Savings Plans utilization. Another CloudFormation stack will be deployed in your chosen [Member Account](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_getting-started_concepts.html#member-account) within your [AWS Organization](https://docs.aws.amazon.com/organizations/). 

The CloudFormation stack in a member account creates a state machine that assumes a role in your management account and analyzes all Savings Plans in your management account, including those purchased across your organization. The workflow filters active Savings Plans based on their purchase date, focusing specifically on plans acquired within the last 7 days and the current calendar month. It then evaluates their utilization rates and identifies plans falling below the defined threshold.

The state machine executes at your specified frequency and uses Amazon SNS to send email alerts to addresses you provide during CloudFormation stack creation. These alerts contain detailed information about low-utilization Savings Plans and instructions for the return process.

![samples-aws-savings-plan-alert-utilization](https://github.com/user-attachments/assets/86c78e4c-b501-4ea0-8048-603c0a9a15c8)

## Solution Walk Through:

### Prerequisites

* An AWS account
* IAM permissions to create a CloudFormation Stack and deploy an IAM role in the management Account
* IAM permissions to create a CloudFormation Stack and deploy Step Functions, IAM roles, SNS, and EventBridge scheduler in your chosen member Account

### Deploy the solution

In this section we will deploy resources for this solution in your accounts:

#### Part 1 - Member Account Deployment

In this section, we will deploy resources for this solution in your chosen member account. 

1. Log in to your AWS Management Console of the member account where you want this solution to run
2. Deploy this [CloudFormation stack](https://github.com/aws-samples/sample-aws-new-savings-plan-utilization-alert/blob/main/sample-aws-new-savings-plan-utilization-alert_member.yaml)
3. Provide the `Stack Name` as **new-sp-utilization-alert-member**
4. In the `AlertEmails` parameter, enter a comma-separated list of email addresses that will receive notifications about underutilized Savings Plans.
5. In the `ManagementAccountId` parameter, enter the 12 digit AWS Account Id of your AWS management account.
6. In the `ScheduleExpression` parameter, specify the execution frequency for the Step Functions state machine using cron format (default is daily at 9 AM UTC).
7. In the `UtilizationThreshold` parameter, specify the minimum utilization percentage for your Savings Plans. You receive alerts when utilization falls below this threshold.
8. Click **Next**, select the acknowledgment box, and create the stack
9. Wait until the stack has finished deploying and is showing as **CREATE-COMPLETE**
10. You will receive an email to confirm your subscription to the SNS topic created by this stack. Please confirm the subscription to begin receiving notifications.
11. Visit the Outputs tab of the stack you just created and make a note of the values of the `ExecutionRoleArn` and `StateMachineArn` Keys, you will need these in the next part.

#### Part 2 - Management Account Deployment

1.	Log in to your AWS Management Console. **Note**: This must be the same account as the one entered in the ManagementAccountId parameter in the previous part.
2.	Deploy this [CloudFormation stack](https://github.com/aws-samples/sample-aws-new-savings-plan-utilization-alert/blob/main/sample-aws-new-savings-plan-utilization-alert_management.yaml)
3.	Provide the `Stack Name` as **new-sp-utilization-alert-management**
4.	In the `ExecutionRoleArn` parameter, provide the value copied from the stack outputs of the stack deployed in the member account.
5.	In the `StateMachineArn` parameter, provide the value copied from the stack outputs of the stack deployed in the member account.
6.	Click **Next**, select the acknowledgment box, and create the stack
7.	Wait until the stack has finished deploying and is showing as **CREATE-COMPLETE**

### Test the Solution

Now that the Step Functions state machine and associated resources are deployed in your member account, let's test the deployment:

* Login back in to your AWS Management Console of the member account where you deployed part 1 of this solution.
* Navigate to the Resources tab in your CloudFormation stack and locate the **SavingsPlansAlerts** Step Functions state machine. Click the blue hyperlink.
* You will be redirected to the Step Functions console. Click the **Start execution** button on the right.
* View the execution details in the Events section to monitor the state machine's progress. If you have any Savings Plans purchased within the last 7 days and the current calendar month, you will receive email notifications.
* A successful execution is indicated by a green box in the Graph view. If any Savings Plans fall below your specified utilization threshold, you will receive an email at your provided address.  

### Clean Up

All resources deployed for this solution can be removed by deleting the CloudFormation stacks. You can delete the stack through either the AWS Management Console or the [AWS CLI](amazon.com/cli/).

**To delete the management account stack (CLI):**

```bash
aws cloudformation delete-stack --stack-name new-sp-utilization-alert-management
```

**To delete the member account stack (CLI):**

```bash
aws cloudformation delete-stack --stack-name new-sp-utilization-alert-member
```

### Conclusion

In this post, we explored how to use the Savings Plan and Cost Explorer APIs to identify underutilized Savings Plans in your organization. We then demonstrated how to use a Step Functions State Machine to filter Savings Plans purchased within the last 7 days and the current calendar month. This timing is crucial because you can return Savings Plans within the return window if they were purchased inadvertently or aren't being utilized effectively. For guidance on returning a purchased Savings Plan, please refer to the [Returning a Purchased Savings Plan](https://docs.aws.amazon.com/savingsplans/latest/userguide/return-sp.html) documentation.
