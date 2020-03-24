# AWS Config Managed Instance Policy Remediation

## Overview
This is a sample solution using [AWS Config](https://aws.amazon.com/config/) rules to audit IAM entities (users, groups, and roles) for the attachment of the IAM Managed Policy *AmazonEC2RoleforSSM* which is being deprecated. 

### Features
* IAM entities which are found to have the policy attached are marked non-compliant. 
* A custom [Systems Manager](https://aws.amazon.com/systems-manager) Automation document allows automated remediation of IAM roles by replacing the deprecated policy with a specified list of policies.

## Deployment
### Prerequisites
To deploy the sample solution, you must first enable [AWS Config](https://aws.amazon.com/config/) and collect IAM resource types IAM:Role, IAM:User, and IAM:Group. If you choose to collect all resources, ensure *Include global resources* is enabled.

Once AWS Config is enabled, use the [AWS CloudFormation](https://aws.amazon.com/cloudformation/) template provided in this project, found here: [SSMManagedPolicyBestPractice.yaml](SSMManagedPolicyBestPractice.yaml). The resulting CloudFormation stack will create the following resources:

* AWS Config rule to scan/remediate IAM roles
* AWS Config rule to scan IAM users and groups
* Systems Manager Automation remediation document
* IAM Automation service role used for remediation
* AWS Config remediation configuration

When launching the stack, specify your replacement IAM policies in the **policiesToAdd** parameter, using Amazon Resource Names (ARNs) in comma-separated format. Upon choosing to remediate any non-compliant role(s), the specified policies will be attached to the noncompliant IAM role, replacing the *AmazonEC2RoleforSSM* policy specified in **policyToRemove**. Note that the default limit is 10 managed policies attached to a role. For more details, see [IAM Object Limits](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_iam-limits.html#reference_iam-limits-entities).

Note: The following AWS Managed Policies are specified by default:
arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
arn:aws:iam::aws:policy/AmazonSSMDirectoryServiceAccess
arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy

For more details about launching a stack, refer to [Creating a Stack on the AWS CloudFormation Console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html).

### Template Parameters
The stack template includes the following parameters:

| Parameter | Required | Description |
| --- | --- | --- |
| policyToRemove | Yes | ARN of the IAM policy to scan in AWS Config rule and remediate by replacing with "policiesToAdd". |
| policiesToAdd | Yes | ARN(s) of an IAM policy or policies to attach to the role, in comma separated list format. Note the default limit is 10 policies attached to a role. |
| RoleConfigRuleName | Yes | The name that you assign to the AWS Config rule to scan and remediate IAM roles. |
| UserGroupConfigRuleName | Yes | The name that you assign to the AWS Config rule to scan IAM users and groups. |
| DocumentName | Yes | Name of the Automation remediation document. Must not begin with reserved prefixes: aws, amazon, or amzn. |
| exceptionList | No | Comma separated list of resourcetypes and list of resource name pairs to exclude from Config rule scan. (for example, users:[user1;user2], groups:[group1;group2], roles:[role1;role2;role3]). |

## Usage
Once deployed, navigate to the [AWS Config console](https://console.aws.amazon.com/config) and choose Rules to view the results of the created Config rules. Note that it can take some time for your resources to be recorded by AWS Config.

To remediate any roles detected as **Noncompliant** (roles which have *AmazonEC2RoleforSSM* policy attached), choose the rule targeting the IAM Role resource, select the role(s) and choose **Remediate**. For more details, see [Remediating Noncompliant AWS Resources by AWS Config Rules](https://docs.aws.amazon.com/config/latest/developerguide/remediation.html).

## License

This library is licensed under the MIT-0 License. See the LICENSE file.
