# Troubleshooting AWS Secrets Manager Rotation of Secrets<a name="org_troubleshoot_rotation"></a>

Use the information here to help you diagnose and fix common errors found when rotating Secrets Manager secrets\.

Rotating secrets in AWS Secrets Manager requires the use of a Lambda function that defines how to interact with the database or service that owns the secret\. Other than errors in the code

**Topics**
+ [How to find the diagnostic logs for your Lambda rotation function](#tshoot-rotation-find-logs)
+ [I get "access denied" when trying to configure rotation for my secret](#tshoot-lambda-initialconfig-perms)
+ [My first rotation fails after I enable rotation](#tshoot-lambda-initialconfig-mastersecret)
+ [Secrets Manager says I successfully configured rotation, but the password is not rotating](#tshoot-lambda-connection-with-internet)
+ [CloudTrail shows access denied errors during rotation](#tshoot-lambda-accessdeniedduringrotation)

## How to find the diagnostic logs for your Lambda rotation function<a name="tshoot-rotation-find-logs"></a>

When the rotation function is not operating the way you expect, the first thing you should check are the logs\. The Lambda rotation function template code that Secrets Manager provides for you writes error messages to the CloudWatch event log\.

**To view the CloudWatch logs for your Lambda function**

1. Open the AWS Lambda console at [https://console\.aws\.amazon\.com/lambda/](https://console.aws.amazon.com/lambda/)\.

1. On the list of functions, choose the name of the Lambda function associated with your secret\.

1. Choose the **Monitoring** tab\.

1. In the **Invocation errors** section, choose **Jump to Logs**\.

   The CloudWatch console opens and displays the logs for your function\.

## I get "access denied" when trying to configure rotation for my secret<a name="tshoot-lambda-initialconfig-perms"></a>

When you add a Lambda rotation function's ARN to your secret, Secrets Manager checks the permissions of the function\. The function's role policy must grant the Secrets Manager service principal `secretsmanager.amazonaws.com` permission to invoke the function \(`lambda:InvokeFunction`\)\. 

You can add this by running the following AWS CLI command:

```
aws lambda add-permission --function-name ARN_of_lambda_function --principal secretsmanager.amazonaws.com --action lambda:InvokeFunction --statement-id SecretsManagerAccess
```

## My first rotation fails after I enable rotation<a name="tshoot-lambda-initialconfig-mastersecret"></a>

When you enable rotation for a secret that uses a "master" secret to change the credentials on the secured service, Secrets Manager configures most elements required for rotation automatically\. However, Secrets Manager cannot automatically grant permission to read the master secret to your Lambda function\. You must explicitly grant this permission yourself\. Specifically, you grant the permission by adding it to the policy attached to the IAM role that is attached to your Lambda rotation function\. That policy must include the following statement \(this is only a statement, not a complete policy\)\. For the complete policy, see the second sample policy in the section [CloudTrail shows access denied errors during rotation](#tshoot-lambda-accessdeniedduringrotation)\.

```
{
    "Sid": "AllowAccessToMasterSecret",
    "Effect": "Allow",
    "Action": "secretsmanager:GetSecretValue",
    "Resource": "ARN_of_master_secret"
}
```

This enables the rotation function to retrieve the credentials from the master secret, which it can then use to change the credentials for the secret being rotated\. 

## Secrets Manager says I successfully configured rotation, but the password is not rotating<a name="tshoot-lambda-connection-with-internet"></a>

This can occur if there are network configuration issues that prevent the Lambda function from communicating with either your secured database/service or the Secrets Manager service endpoint, which is on the public Internet\. If your database/service is running in a VPC, then you can configure things one of two ways:
+ Make the database in the VPC publicly accessible with an EC2 Elastic IP address\.
+ Configure the Lambda rotation function to operate in the same VPC as the database/service\. In this case your VPC must be equipped with a NAT gateway so that the Lambda rotation function can reach the public Secrets Manager service endpoint\.

To determine if this type of configuration issue is the cause of the rotation failure, perform the following steps:

**To diagnose connectivity issues between your rotation function and the database or Secrets Manager**

1. Follow the procedure [How to find the diagnostic logs for your Lambda rotation function](#tshoot-rotation-find-logs) to open your logs\.

1. Examine the log files to look for indications that there are timeouts occurring between either the Lambda function and the AWS Secrets Manager service, or between the Lambda function and the secured database or service\.

1. Refer to the Amazon Virtual Private Cloud documentation at [https://aws\.amazon\.com/documentation/vpc/](https://aws.amazon.com/documentation/vpc/) and the [AWS Lambda Developer Guide](http://docs.aws.amazon.com/lambda/latest/dg/) for information about how to configure services and Lambda functions to interoperate within the VPC environment\.

## CloudTrail shows access denied errors during rotation<a name="tshoot-lambda-accessdeniedduringrotation"></a>

When you configure rotation, if you let Secrets Manager create the rotation function for you, it is automatically provided with a policy attached to the function's IAM role that grants the appropriate permissions\. If you create the function on your own, you need to grant the following permissions to the role attached to the function\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "secretsmanager:DescribeSecret",
                "secretsmanager:GetRandomPassword",
                "secretsmanager:GetSecretValue",
                "secretsmanager:PutSecretValue",
                "secretsmanager:UpdateSecretVersionStage",
            ],
            "Resource": "*"
        }
    ]
}
```

Also, if your rotation uses a separate master secret's credentials to rotate this secret then you must also grant permission to retrieve the secret value from the master secret\. See [My first rotation fails after I enable rotation](#tshoot-lambda-initialconfig-mastersecret) for more information\. The combined policy might look like this:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowAccessToSecretsManagerAPIs",
            "Effect": "Allow",
            "Action": [
                "secretsmanager:DescribeSecret",
                "secretsmanager:GetRandomPassword",
                "secretsmanager:GetSecretValue",
                "secretsmanager:PutSecretValue",
                "secretsmanager:UpdateSecretVersionStage",
            ],
            "Resource": "*"
        },
        {
            "Sid": "AllowAccessToMasterSecret",
            "Effect": "Allow",
            "Action": "secretsmanager:GetSecretValue",
            "Resource": "ARN_of_master_secret"
        }
    ]
}
```