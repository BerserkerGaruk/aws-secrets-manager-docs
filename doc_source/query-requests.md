# Calling the API by Making HTTP Query Requests<a name="query-requests"></a>

This section contains general information about using the Query API for AWS Secrets Manager\. For details about the API operations and errors, see the [AWS Secrets Manager API Reference](http://docs.aws.amazon.com/secretsmanager/latest/apireference/)\.

**Note**  
Instead of making direct calls to the AWS Secrets Manager Query API, you can use one of the AWS SDKs\. The AWS SDKs consist of libraries and sample code for various programming languages and platforms \(Java, Ruby, \.NET, iOS, Android, and more\)\. The SDKs provide a convenient way to create programmatic access to Secrets Manager and AWS\. For example, the SDKs take care of tasks such as cryptographically signing requests, managing errors, and retrying requests automatically\. For information about the AWS SDKs, including how to download and install them, see [Tools for Amazon Web Services](http://aws.amazon.com/tools/)\.

The Query API for AWS Secrets Manager lets you call service actions\. Query API requests are HTTPS requests that must contain an `Action` parameter to indicate the operation to be performed\. AWS Secrets Manager supports GET and POST requests for all operations\. That is, the API does not require you to use GET for some actions and POST for others\. However, GET requests are subject to the limitation size of a URL\. Although this limit is browser dependent, a typical limit is 2048 bytes\. Therefore, for Query API requests that require larger sizes, you must use a POST request\.

The response is an XML document\. For details about the response, see the individual action pages in the [AWS Organizations API Reference](http://docs.aws.amazon.com/organizations/latest/APIReference/)\.

**Topics**
+ [Endpoints](#ASMEndpoints)
+ [HTTPS Required](#IAMHTTPSRequired)
+ [Signing Secrets Manager API Requests](#SigVersion)

## Endpoints<a name="ASMEndpoints"></a>

AWS Secrets Manager has endpoints in most AWS regions\. For the complete list, see [the endpoint list for AWS Secrets Manager](http://docs.aws.amazon.com/general/latest/gr/rande.html#asm_region) in the *AWS General Reference*\.

For more information about AWS endpoints and regions for all services, see [Regions and Endpoints](http://docs.aws.amazon.com/general/latest/gr/index.html?rande.html), also in the *AWS General Reference*\. 

## HTTPS Required<a name="IAMHTTPSRequired"></a>

Because the Query API returns sensitive information such as security credentials, you must use HTTPS to encrypt all API requests\. 

## Signing Secrets Manager API Requests<a name="SigVersion"></a>

Requests must be signed using an access key ID and a secret access key\. We strongly recommend that you do not use your AWS root account credentials for everyday work with Secrets Manager\. You can use the credentials for an IAM user or temporary credentials such as you use with an IAM role\.

To sign your API requests, you must use AWS Signature Version 4\. For information about using Signature Version 4, see [Signature Version 4 Signing Process](http://docs.aws.amazon.com/general/latest/gr/signature-version-4.html) in the *AWS General Reference*\. 

For more information, see the following:
+  [AWS Security Credentials](http://docs.aws.amazon.com/general/latest/gr/aws-security-credentials.html)\. Provides general information about the types of credentials that you can use to access AWS\. 
+ [IAM Best Practices](http://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)\. Offers suggestions for using the IAM service to help secure your AWS resources, including those in Secrets Manager\. 
+ [Temporary Credentials](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html)\. Describes how to create and use temporary security credentials\. 