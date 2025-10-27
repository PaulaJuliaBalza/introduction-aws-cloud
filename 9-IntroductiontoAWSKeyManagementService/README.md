# Introduction to AWS Key Management Service

## Lab overview
This lab introduces you to AWS Key Management Service (AWS KMS). The lab demonstrates the basic steps required to get started with AWS KMS, creating keys, assigning management and usage permissions for the keys, encrypting data and monitoring the access and usage of keys.

## Ojectives
By the end of this lab, you should be able to do the following:
* Create an Encryption Key
* Create an Amazon Simple Storage Service (Amazon S3) bucket with AWS CloudTrail logging functions
* Encrypt data stored in an Amazon S3 bucket using an encryption key
* Monitor encryption key usage using CloudTrail
* Manage encryption keys for users and roles

## Services used in this lab 
### AWS Key Management Service (KMS)
AWS Key Management Service (KMS) is a managed service that makes it easy for you to create and control the encryption keys used to encrypt your data, and uses Hardware Security Modules (HSMs) to protect the security of your keys. AWS Key Management Service is integrated with several other AWS services to help you protect the data you store with these services. AWS Key Management Service is also integrated with AWS CloudTrail to provide you with logs of all key usage to help meet your regulatory and compliance needs.

### AWS CloudTrail
AWS CloudTrail is a service that enables governance, compliance, operational auditing, and risk auditing of your AWS account. With CloudTrail, you can log, continuously monitor, and retain account activity related to actions across your AWS infrastructure. CloudTrail provides event history of your AWS account activity, including actions taken through the AWS Management Console, AWS SDKs, command line tools, and other AWS services. This event history simplifies security analysis, resource change tracking, and troubleshooting.

### Amazon S3
Companies today need the ability to simply and securely collect, store, and analyze their data at a massive scale. Amazon S3 is object storage built to store and retrieve any amount of data from anywhere – web sites and mobile apps, corporate applications, and data from IoT sensors or devices. It is designed to deliver 99.999999999% durability, and stores data for millions of applications used by market leaders in every industry. S3 provides comprehensive security and compliance capabilities that meet even the most stringent regulatory requirements. It gives customers flexibility in the way they manage data for cost optimization, access control, and compliance. S3 is the only cloud storage solution with query-in-place functionality, allowing you to run powerful analytics directly on your data at rest in S3. And Amazon S3 is the most supported storage platform available, with the largest ecosystem of ISV solutions and systems integrator partners.

## Task 1: Create your KMS master key
In this task you create a KMS master key. A KMS master key enables you to easily encrypt your data across AWS services and within your own applications.

1. At the top of the page, in the unified search bar, search for and choose Key Management Service.
2. In the Key Management Service page, choose Create a key.

![key](images/key-1.png)

3. On the Configure key page, select Symmetric.

![key](images/key-2.png)

4. Choose Next
5. On the Add labels page, configure:
* Alias: myFirstKey
* Description - optional: KMS Key for S3 data
It is a good practice to describe what services the encryption key is associated with.
![key](images/key-3.png)
6. Choose Next.
7. On the Define key administrative permissions page, select the user or role you’re signed into the Console with. The user is displayed at the top of the page, to the right of the AWS region. It starts with the characters AWSLabsUser-

You can also enter AWSLabsUser- in the search field to find and select the user.
Key Administrators are users or roles that manage access to the encryption key.

![key](images/key-4.png)

8. Choose Next.
9. On the Define key usage permissions page, select the user or role you’re signed into the Console with. This is the same user you selected in the previous step.
Key Users are the users or roles that use the key to encrypt and decrypt data.


![key](images/key-5.png)
![key](images/key-6.png)

10. Choose Next.
```
{
  "Id": "key-consolepolicy-3",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::139867461305:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow access for Key Administrators",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::139867461305:role/AWSLabsUser-noFQw7afVgSci1AKnewcW2"
      },
      "Action": [
        "kms:Create*",
        "kms:Describe*",
        "kms:Enable*",
        "kms:List*",
        "kms:Put*",
        "kms:Update*",
        "kms:Revoke*",
        "kms:Disable*",
        "kms:Get*",
        "kms:Delete*",
        "kms:TagResource",
        "kms:UntagResource",
        "kms:ScheduleKeyDeletion",
        "kms:CancelKeyDeletion",
        "kms:RotateKeyOnDemand"
      ],
      "Resource": "*"
    },
    {
      "Sid": "Allow use of the key",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::139867461305:role/AWSLabsUser-noFQw7afVgSci1AKnewcW2"
      },
      "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:DescribeKey"
      ],
      "Resource": "*"
    },
    {
      "Sid": "Allow attachment of persistent resources",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::139867461305:role/AWSLabsUser-noFQw7afVgSci1AKnewcW2"
      },
      "Action": [
        "kms:CreateGrant",
        "kms:ListGrants",
        "kms:RevokeGrant"
      ],
      "Resource": "*",
      "Condition": {
        "Bool": {
          "kms:GrantIsForAWSResource": "true"
        }
      }
    }
  ]
}
```
11. In the Review page, take a brief moment to review the parameters of the KMS configuration.
12. Choose Finish.

![key](images/key-7.png)
![key](images/key-8.png)

13. In the Customer managed keys page, the new key - myFirstKey is listed. Copy the Key ID value to a text editor. The value is located under the Key ID column.
3d46abd3-279a-41b2-bcb1-b4b2cdcb76db 

![key](images/key-9.png)

Later in the lab, you use the Key ID value when reviewing the activity log for this KMS key.

 Task complete: You have successfully created a KMS master key.

## Task 2: Configure CloudTrail to store logs in a new S3 bucket
In this task you configure CloudTrail to store log files in a new S3 bucket.

1. At the top of the page, in the unified search bar, search for and choose CloudTrail.
 Note: You can safely ignore any of the following messages if they appear:
* The option to create an organization trail is not available for this AWS account…
* AccessDeniedExceptions…
* You do not have permissions to perform this action….

![trail](images/trail-1.png)

2. In the left navigation pane, choose Trails.
3. Choose Create trail.


4. In the Choose trail attributes page, configure:
* Trail name: myTrail.
* Trail log bucket and folder: mycloudtrailbucketNUMBER.
* Replace NUMBER with a random number.
* De-select Enabled for Log file SSE-KMS encryption.
5. Choose Next.
6. On the Choose log events page, configure:
* Select Management events
* Select Data events
* Select Insights events
7. In the Data events section, choose Switch to basic event selectors
8. Then choose Continue.
9. In the Insights events section, select
* Select API call rate
* Select API error rate
10. Choose Next.
11. In the Review and create page, take a brief moment to review the parameters of the Trail configuration.
12. Choose Create trail.

![trail](images/trail-2.png)
![trail](images/trail-3.png)
![trail](images/trail-4.png)
![trail](images/trail-5.png)
![trail](images/trail-6.png)
![trail](images/trail-7.png)
![trail](images/trail-8.png)
![trail](images/trail-9.png)

The trail is created even though it is not listed due to permission controls - in the lab environment.

 Task complete: You have successfully configured CloudTrail.

## Task 3: Upload an image to your S3 bucket and encrypt it
In this task, you upload an image file to your S3 bucket and encrypt it using the encryption key (myFirstKey) you created earlier.

1. At the top of the page, in the unified search bar, search for and choose S3.
2. Choose mycloudtrailbucket*.
3. From the Objects tab, choose Upload.
4. Choose Add files.
5. Browse to and select an image file on your computer.
6. At the bottom of the screen, expand  Properties.
7. In the Server-side encryption settings section, select Specify an encryption key.
8. For Encryption settings, select Override bucket settings for default encryption.
9. For Encryption type, select Server-side encryption with AWS Key Management Service keys (SSE-KMS).
10. For AWS KMS key, select Choose from your AWS KMS keys.
11. From the Available AWS KMS keys drop down menu, select myFirstKey.
12. Scroll to the bottom of the screen, then choose Upload.
13. In the Upload: status page, a message shows the image was succesfully uploaded.
14. Choose Close.
15. In the mycloudtrailbucket* page, choose the Objects tab.
16. From the row that lists the image you uploaded, record the Last modified timestamp to a text editor. Later in the lab, you use the Last modified timestamp to review CloudTrail logs.

October 26, 2025, 21:01:14 (UTC-03:00)

![s3](images/s3-1.png)
![s3](images/s3-2.png)
![s3](images/s3-3.png)
![s3](images/s3-4.png)

 Task complete: You have successfully uploaded an image to S3, and encrypted it as well.

## Task 4: Access the encrypted image
In this task, you try to access the encrypted image through both the AWS Management Console, and the S3 link.

1. In the Objects tab, select the image name and then choose Open .
The image opens in a new tab/window.

Amazon S3 and AWS KMS perform the following actions when you request that your data be decrypted.
* Amazon S3 sends the encrypted data key to AWS KMS
* AWS KMS decrypts the key by using the appropriate master key and sends the plaintext key back to Amazon S3
* Amazon S3 decrypts the ciphertext and removes the plaintext data key from memory as soon as possible
* Close the window/tab that displays your image.

2. With the image selected, choose Copy URL, and paste the URL to a text editor.

The URL should look similar to https://mycloudtrailbucket10619.s3-us-west-2.amazonaws.com/Eiffel.jpg

3. Paste the S3 Object URL that you copied into a new browser/window.

4. Press Enter.

5. What does the page show?
It shows Access Denied. This is because, by default public access is not allowed.
Access denied ->
https://mycloudtrailbucket583495869.s3.us-west-2.amazonaws.com/0-cronograma.png 

![s3](images/s3-5.png)

6. Return to the AWS Management Console.
7. In the mycloudtrailbucket* page, choose the Permissions tab.
8. In the Block public access (bucket settings) section, choose Edit.
9. De-select Block all public access.
10. Choose Save changes then:
* Enter confirm
* Choose Confirm.
11. Scroll to the Object Ownership section, choose Edit.
12. Select ACLs enabled.
13. For Enabling ACLs turns off the bucket owner enforced setting for Object Ownership, select - I acknowledge that ACLs will be restored.
14. Choose Save changes.

![s3](images/s3-6.png)
![s3](images/s3-7.png)
![s3](images/s3-8.png)
![s3](images/s3-9.png)
![s3](images/s3-10.png)

15. In the Object tab, select your image.
16. Choose Actions 
17. From the drop-down menu, select Make public using ACL.
18. Choose Make public.
19. In the Make public: status page, a message shows the access change was succesfull.
20. Choose Close.

![s3](images/s3-11.png)
![s3](images/s3-12.png)
![s3](images/s3-13.png)

21. Navigate to the browser tab/window with the S3 Object URL.
22. Refresh  the tab/window.
23. What do you see?

Because the image is encrypted, you are not able to view it using the public link. You should see a message saying Requests specifying Server Side Encryption with AWS KMS managed keys require AWS Signature Version 4.

If you are uploading or accessing objects encrypted by SSE-KMS, you need to use AWS Signature Version 4 for added security. Signature Version 4 is the process to add authentication information to AWS requests. When you use the AWS Command Line Interface (AWS CLI) or one of the AWS SDKs to make requests to AWS, these tools automatically sign the requests for you with the access key that you specify when you configure the tools. When you use these tools, you don’t need to learn how to sign requests yourself. For more information on this process read this blog post: blog post --> https://aws.amazon.com/es/blogs/security/how-to-use-the-rest-api-to-encrypt-s3-objects-by-using-aws-kms/ 

![s3](images/s3-14.png)

24. Close the S3 Object URL tab/window.
 Task complete: You have attempted to access the S3 object through both the AWS Management Console, and the S3 link.

## Task 5: Monitor KMS activity using CloudTrail logs
In this task, you access CloudTrail log files and review logs related to your encryption operations.

1. In the mycloudtrailbucket* page, choose the Objects tab.
2. Drill-down through the  AWSLogs/ folders till you get to a folder that contains log file(s).
The path should look similar to: Amazon S3 > AWSLogs/ > 197167081626/ > CloudTrail/ > Region > 2024 > 02 > 01
In the above example, Region is the AWS region where your lab is currently running from. The region should match with the LabRegion value to the left of these instructions.

If you don’t see any log files, choose the refresh  button every few seconds till you see a log file.

The log files have a .json.gz extension.

3. Do you see a log file that has a Last modified timestamp that is later than the timestamp for the image file you downloaded? (In an earlier step, you copied the image upload timestamp to a text editor, use that timestamp when identifying the log files to review).
* If yes, continue to the next step.
* If no, continue to choose the refresh  button every few seconds till there is.

Note: It can take up to 5 minutes to see a log file that has a Last modified time stamp that is greater than the time stamp of the image file that you uploaded.

4. Select the log file that has the closest time after the Last modified of the image you uploaded.

5. Choose Open.

![task](images/task5-1.png)

This opens a log file in a new browser tab.

6. If you see a pop-up security warning, confirm that you want to open the file. If not, continue to the next step.
 Note: Your browser security settings may simply ignore the pop-up. If you do not see any file being opened and do not see a pop-up alert, you should enable pop-ups within in your browser’s settings section.

If you are not using Google Chrome or Firefox, you may need to download and decompress the gz compressed file using a local utility on your own computer. Once the .gz file is decompressed - open it in a text editor.

The log file is in a JSON format and contains each API call that has been logged by CloudTrail. Depending upon the browser you are using the log file might look slightly different.

7. Search for the following in your log file:
* The encryption Key ID that you copied to your text editor
* The name of the file that you upload. (You should see name of the file in the same log file that contains your encryption Key ID)

If you cannot locate the items above, try either one of the following:
* Perhaps the log file you have open - was generated before the Last modified timestamp of the image you uploaded. In this case, navigate to the S3 console, and open to search a log file that has a timestamp with the closest time after the Last modified of the image you uploaded.
* If you still don’t see the image name or the encryption Key ID, open the next file after the one you just checked, and try searching again. Please note, you might have to repeat this action for consecutive log files until you locate your image (or the encryption Key ID). Typically, you should be able to find your image or the encryption Key ID in one of the files that was generated around 5 minutes after the timestamp of the Last modified time of the image you uploaded.

 Task complete: You have monitored KMS activity using CloudTrail logs.

```
{"Records":[{"eventVersion":"1.10","userIdentity":{"type":"AssumedRole","principalId":"AROAI2YMI2XZQPW3IJ5T6:AVS-AsyncResourceMonitor","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsController/AVS-AsyncResourceMonitor","accountId":"830834166987","accessKeyId":"ASIA4C4MTJTFRHKZZFJW","sessionContext":{"sessionIssuer":{"type":"Role","principalId":"AROAI2YMI2XZQPW3IJ5T6","arn":"arn:aws:iam::830834166987:role/AWSLabsController","accountId":"830834166987","userName":"AWSLabsController"},"attributes":{"creationDate":"2025-10-27T00:00:52Z","mfaAuthenticated":"false"}}},"eventTime":"2025-10-27T00:00:52Z","eventSource":"ec2.amazonaws.com","eventName":"DescribeInstances","awsRegion":"us-east-1","sourceIPAddress":"3.89.144.38","userAgent":"aws-sdk-js/3.895.0 ua/2.1 os/linux#5.10.244-267.968.amzn2.x86_64 lang/js md/nodejs#20.19.4 api/ec2#3.895.0 exec-env/AWS_Lambda_nodejs20.x m/F,e","requestParameters":{"instancesSet":{},"filterSet":{"items":[{"name":"instance-state-name","valueSet":{"items":[{"value":"running"}]}}]}},"responseElements":null,"requestID":"c35cefbc-ad85-40e0-a5ed-786a87683209","eventID":"97a99623-b1d0-4e2f-9388-45c8ea458c5d","readOnly":true,"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","eventCategory":"Management","tlsDetails":{"tlsVersion":"TLSv1.3","cipherSuite":"TLS_AES_128_GCM_SHA256","clientProvidedHostHeader":"ec2.us-east-1.amazonaws.com"}},{"eventVersion":"1.11","userIdentity":{"type":"AssumedRole","principalId":"AROAI2YMI2XZQPW3IJ5T6:AVS-AsyncResourceMonitor","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsController/AVS-AsyncResourceMonitor","accountId":"830834166987","accessKeyId":"ASIA4C4MTJTFRHKZZFJW","sessionContext":{"sessionIssuer":{"type":"Role","principalId":"AROAI2YMI2XZQPW3IJ5T6","arn":"arn:aws:iam::830834166987:role/AWSLabsController","accountId":"830834166987","userName":"AWSLabsController"},"attributes":{"creationDate":"2025-10-27T00:00:52Z","mfaAuthenticated":"false"}}},"eventTime":"2025-10-27T00:00:52Z","eventSource":"cloudtrail.amazonaws.com","eventName":"LookupEvents","awsRegion":"us-east-1","sourceIPAddress":"3.89.144.38","userAgent":"aws-sdk-nodejs/2.1692.0 linux/v20.19.4 exec-env/AWS_Lambda_nodejs20.x promise","requestParameters":{"lookupAttributes":[{"attributeKey":"ReadOnly","attributeValue":"false"}],"startTime":"Oct 26, 2025, 11:50:52 PM","endTime":"Oct 27, 2025, 12:00:52 AM"},"responseElements":null,"requestID":"231d57c5-e066-41ca-9250-33fbe3748cf7","eventID":"7625ad7e-5b11-4d89-843b-8e83f536d2c2","readOnly":true,"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","eventCategory":"Management","tlsDetails":{"tlsVersion":"TLSv1.3","cipherSuite":"TLS_AES_128_GCM_SHA256","clientProvidedHostHeader":"cloudtrail.us-east-1.amazonaws.com"}},{"eventVersion":"1.11","userIdentity":{"type":"AssumedRole","principalId":"AROAI2YMI2XZQPW3IJ5T6:AVS-AsyncResourceMonitor","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsController/AVS-AsyncResourceMonitor","accountId":"830834166987","accessKeyId":"ASIA4C4MTJTFRHKZZFJW","sessionContext":{"sessionIssuer":{"type":"Role","principalId":"AROAI2YMI2XZQPW3IJ5T6","arn":"arn:aws:iam::830834166987:role/AWSLabsController","accountId":"830834166987","userName":"AWSLabsController"},"attributes":{"creationDate":"2025-10-27T00:00:52Z","mfaAuthenticated":"false"}}},"eventTime":"2025-10-27T00:00:52Z","eventSource":"ecs.amazonaws.com","eventName":"ListClusters","awsRegion":"us-east-1","sourceIPAddress":"3.89.144.38","userAgent":"aws-sdk-nodejs/2.1692.0 linux/v20.19.4 exec-env/AWS_Lambda_nodejs20.x promise","requestParameters":null,"responseElements":null,"requestID":"7d98dee8-cb3a-4315-b015-cab14de5c46c","eventID":"4ec718cc-259b-4509-b6d2-4d80104818c6","readOnly":true,"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","eventCategory":"Management","tlsDetails":{"tlsVersion":"TLSv1.3","cipherSuite":"TLS_AES_128_GCM_SHA256","clientProvidedHostHeader":"ecs.us-east-1.amazonaws.com"}},{"eventVersion":"1.11","userIdentity":{"type":"AssumedRole","principalId":"AROAI2YMI2XZQPW3IJ5T6:AVS-AsyncResourceMonitor","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsController/AVS-AsyncResourceMonitor","accountId":"830834166987","accessKeyId":"ASIA4C4MTJTFZD5OBYRX","sessionContext":{"sessionIssuer":{"type":"Role","principalId":"AROAI2YMI2XZQPW3IJ5T6","arn":"arn:aws:iam::830834166987:role/AWSLabsController","accountId":"830834166987","userName":"AWSLabsController"},"attributes":{"creationDate":"2025-10-27T00:00:52Z","mfaAuthenticated":"false"}},"invokedBy":"lambda.amazonaws.com"},"eventTime":"2025-10-27T00:00:52Z","eventSource":"kms.amazonaws.com","eventName":"Decrypt","awsRegion":"us-east-1","sourceIPAddress":"lambda.amazonaws.com","userAgent":"lambda.amazonaws.com","requestParameters":{"encryptionAlgorithm":"SYMMETRIC_DEFAULT","encryptionContext":{"aws:lambda:FunctionArn":"arn:aws:lambda:us-east-1:830834166987:function:LabStack-5afab8dd-572c-45-S3PublicAccessBlockFunct-LkLzhrffL5DL"}},"responseElements":null,"additionalEventData":{"keyMaterialId":"249ff1560efc7b313c30919fb4f95b43831f4dddb51134f69816742654cf2919"},"requestID":"226b3e95-e212-4224-b49b-3006b2395dbc","eventID":"3be8ee01-58ae-4409-9982-da6d5cba400c","readOnly":true,"resources":[{"accountId":"830834166987","type":"AWS::KMS::Key","ARN":"arn:aws:kms:us-east-1:830834166987:key/69ba0c3d-57aa-4fe5-8318-b2a40e1366b6"}],"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","eventCategory":"Management"},{"eventVersion":"1.11","userIdentity":{"type":"AssumedRole","principalId":"AROAI2YMI2XZQPW3IJ5T6:AVS-AsyncResourceMonitor","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsController/AVS-AsyncResourceMonitor","accountId":"830834166987","accessKeyId":"ASIA4C4MTJTFZD5OBYRX","sessionContext":{"sessionIssuer":{"type":"Role","principalId":"AROAI2YMI2XZQPW3IJ5T6","arn":"arn:aws:iam::830834166987:role/AWSLabsController","accountId":"830834166987","userName":"AWSLabsController"},"attributes":{"creationDate":"2025-10-27T00:00:52Z","mfaAuthenticated":"false"}},"invokedBy":"lambda.amazonaws.com"},"eventTime":"2025-10-27T00:00:52Z","eventSource":"kms.amazonaws.com","eventName":"Decrypt","awsRegion":"us-east-1","sourceIPAddress":"lambda.amazonaws.com","userAgent":"lambda.amazonaws.com","requestParameters":{"encryptionAlgorithm":"SYMMETRIC_DEFAULT","encryptionContext":{"aws:lambda:FunctionArn":"arn:aws:lambda:us-east-1:830834166987:function:LabEnforcer_Function"}},"responseElements":null,"additionalEventData":{"keyMaterialId":"249ff1560efc7b313c30919fb4f95b43831f4dddb51134f69816742654cf2919"},"requestID":"b000428f-97a1-44da-8207-8a9df18e68a6","eventID":"c61e04a5-a5bb-4a15-a38b-5d8415aacb62","readOnly":true,"resources":[{"accountId":"830834166987","type":"AWS::KMS::Key","ARN":"arn:aws:kms:us-east-1:830834166987:key/69ba0c3d-57aa-4fe5-8318-b2a40e1366b6"}],"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","eventCategory":"Management"},{"eventVersion":"1.11","userIdentity":{"type":"AssumedRole","principalId":"AROA4C4MTJTFWGPZCZOEB:5afab8dd-572c-45b1-8b68-6b9e00f836c7","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsUser-eo48QcrNoHcJZfkNoBbNHr/5afab8dd-572c-45b1-8b68-6b9e00f836c7","accountId":"830834166987","accessKeyId":"ASIA4C4MTJTFQ6JCKKNF","sessionContext":{"sessionIssuer":{"type":"Role","principalId":"AROA4C4MTJTFWGPZCZOEB","arn":"arn:aws:iam::830834166987:role/AWSLabsUser-eo48QcrNoHcJZfkNoBbNHr","accountId":"830834166987","userName":"AWSLabsUser-eo48QcrNoHcJZfkNoBbNHr"},"attributes":{"creationDate":"2025-10-26T23:53:26Z","mfaAuthenticated":"false"}}},"eventTime":"2025-10-27T00:00:54Z","eventSource":"kms.amazonaws.com","eventName":"ListAliases","awsRegion":"us-east-1","sourceIPAddress":"190.139.250.213","userAgent":"Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36","requestParameters":{"limit":1000},"responseElements":null,"requestID":"0d924abc-6bae-4c7e-b968-3e1697820462","eventID":"441141ed-3324-4589-9edb-3b423a5b067f","readOnly":true,"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","eventCategory":"Management","tlsDetails":{"tlsVersion":"TLSv1.3","cipherSuite":"TLS_AES_256_GCM_SHA384","clientProvidedHostHeader":"kms.us-east-1.amazonaws.com"},"sessionCredentialFromConsole":"true"},{"eventVersion":"1.11","userIdentity":{"type":"AssumedRole","principalId":"AROAI2YMI2XZQPW3IJ5T6:AVS-AsyncResourceMonitor","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsController/AVS-AsyncResourceMonitor","accountId":"830834166987","accessKeyId":"ASIA4C4MTJTFRHKZZFJW","sessionContext":{"sessionIssuer":{"type":"Role","principalId":"AROAI2YMI2XZQPW3IJ5T6","arn":"arn:aws:iam::830834166987:role/AWSLabsController","accountId":"830834166987","userName":"AWSLabsController"},"attributes":{"creationDate":"2025-10-27T00:00:52Z","mfaAuthenticated":"false"}}},"eventTime":"2025-10-27T00:00:52Z","eventSource":"codebuild.amazonaws.com","eventName":"ListBuilds","awsRegion":"us-east-1","sourceIPAddress":"3.89.144.38","userAgent":"aws-sdk-nodejs/2.1692.0 linux/v20.19.4 exec-env/AWS_Lambda_nodejs20.x promise","requestParameters":null,"responseElements":null,"requestID":"ba17617d-c326-4dd0-a149-2dad3af0997a","eventID":"3182977b-c31c-4d17-a7d8-a11c3c906f4d","readOnly":true,"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","eventCategory":"Management","tlsDetails":{"tlsVersion":"TLSv1.3","cipherSuite":"TLS_AES_128_GCM_SHA256","clientProvidedHostHeader":"codebuild.us-east-1.amazonaws.com"}},{"eventVersion":"1.11","userIdentity":{"type":"AssumedRole","principalId":"AROAI2YMI2XZQPW3IJ5T6:AVS-AsyncResourceMonitor","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsController/AVS-AsyncResourceMonitor","accountId":"830834166987","accessKeyId":"ASIA4C4MTJTFRHKZZFJW","sessionContext":{"sessionIssuer":{"type":"Role","principalId":"AROAI2YMI2XZQPW3IJ5T6","arn":"arn:aws:iam::830834166987:role/AWSLabsController","accountId":"830834166987","userName":"AWSLabsController"},"attributes":{"creationDate":"2025-10-27T00:00:52Z","mfaAuthenticated":"false"}}},"eventTime":"2025-10-27T00:00:52Z","eventSource":"sagemaker.amazonaws.com","eventName":"ListNotebookInstances","awsRegion":"us-east-1","sourceIPAddress":"3.89.144.38","userAgent":"aws-sdk-nodejs/2.1692.0 linux/v20.19.4 exec-env/AWS_Lambda_nodejs20.x promise","requestParameters":{"statusEquals":"InService"},"responseElements":null,"requestID":"d4c4e803-332c-41a0-9f61-984234790422","eventID":"382018e7-b5af-4926-a66f-a23b6c548a47","readOnly":true,"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","eventCategory":"Management","tlsDetails":{"tlsVersion":"TLSv1.3","cipherSuite":"TLS_AES_128_GCM_SHA256","clientProvidedHostHeader":"api.sagemaker.us-east-1.amazonaws.com"}},{"eventVersion":"1.09","userIdentity":{"type":"AssumedRole","principalId":"AROAI2YMI2XZQPW3IJ5T6:AVS-AsyncResourceMonitor","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsController/AVS-AsyncResourceMonitor","accountId":"830834166987","accessKeyId":"ASIA4C4MTJTFRHKZZFJW","sessionContext":{"sessionIssuer":{"type":"Role","principalId":"AROAI2YMI2XZQPW3IJ5T6","arn":"arn:aws:iam::830834166987:role/AWSLabsController","accountId":"830834166987","userName":"AWSLabsController"},"attributes":{"creationDate":"2025-10-27T00:00:52Z","mfaAuthenticated":"false"}}},"eventTime":"2025-10-27T00:00:52Z","eventSource":"eks.amazonaws.com","eventName":"ListClusters","awsRegion":"us-east-1","sourceIPAddress":"3.89.144.38","userAgent":"aws-sdk-nodejs/2.1692.0 linux/v20.19.4 exec-env/AWS_Lambda_nodejs20.x promise","requestParameters":null,"responseElements":null,"requestID":"a1dd8ab8-8920-415a-91dc-47e24b4acdea","eventID":"87dcc6da-c932-4e4f-ab6f-dc13818be4e0","readOnly":true,"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","eventCategory":"Management"},{"eventVersion":"1.11","userIdentity":{"type":"AssumedRole","principalId":"AROA4C4MTJTFWGPZCZOEB:5afab8dd-572c-45b1-8b68-6b9e00f836c7","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsUser-eo48QcrNoHcJZfkNoBbNHr/5afab8dd-572c-45b1-8b68-6b9e00f836c7","accountId":"830834166987","accessKeyId":"ASIA4C4MTJTFUJVXY444","sessionContext":{"sessionIssuer":{"type":"Role","principalId":"AROA4C4MTJTFWGPZCZOEB","arn":"arn:aws:iam::830834166987:role/AWSLabsUser-eo48QcrNoHcJZfkNoBbNHr","accountId":"830834166987","userName":"AWSLabsUser-eo48QcrNoHcJZfkNoBbNHr"},"attributes":{"creationDate":"2025-10-26T23:53:26Z","mfaAuthenticated":"false"}},"invokedBy":"AWS Internal"},"eventTime":"2025-10-27T00:01:13Z","eventSource":"kms.amazonaws.com","eventName":"GenerateDataKey","awsRegion":"us-east-1","sourceIPAddress":"AWS Internal","userAgent":"AWS Internal","requestParameters":{"keyId":"arn:aws:kms:us-east-1:830834166987:key/313e9dc0-fec9-4cac-b343-fc7233316514","encryptionContext":{"aws:s3:arn":"arn:aws:s3:::mycloudtrailbucket0968"},"keySpec":"AES_256"},"responseElements":null,"additionalEventData":{"keyMaterialId":"1a02a9acc80f3d4c49d6f7d70807ece174ae904b7ccae2619a75dc81b7d69513"},"requestID":"920375cb-8a54-4aee-a9a1-c0f185c0b9d2","eventID":"220b7c53-3946-49e7-b282-598c3084ef9a","readOnly":true,"resources":[{"accountId":"830834166987","type":"AWS::KMS::Key","ARN":"arn:aws:kms:us-east-1:830834166987:key/313e9dc0-fec9-4cac-b343-fc7233316514"}],"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","eventCategory":"Management","sessionCredentialFromConsole":"true"},{"eventVersion":"1.11","userIdentity":{"type":"AssumedRole","principalId":"AROAI2YMI2XZQPW3IJ5T6:AVS-AsyncResourceMonitor","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsController/AVS-AsyncResourceMonitor","accountId":"830834166987","accessKeyId":"ASIA4C4MTJTFRHKZZFJW","sessionContext":{"sessionIssuer":{"type":"Role","principalId":"AROAI2YMI2XZQPW3IJ5T6","arn":"arn:aws:iam::830834166987:role/AWSLabsController","accountId":"830834166987","userName":"AWSLabsController"},"attributes":{"creationDate":"2025-10-27T00:00:52Z","mfaAuthenticated":"false"}}},"eventTime":"2025-10-27T00:00:52Z","eventSource":"lambda.amazonaws.com","eventName":"ListProvisionedConcurrencyConfigs","awsRegion":"us-east-1","sourceIPAddress":"3.89.144.38","userAgent":"aws-sdk-js/3.895.0 ua/2.1 os/linux#5.10.244-267.968.amzn2.x86_64 lang/js md/nodejs#20.19.4 api/lambda#3.895.0 exec-env/AWS_Lambda_nodejs20.x m/F,e","requestParameters":{"functionName":"LabEnforcer_Function"},"responseElements":null,"requestID":"80254970-d6d5-490a-ab52-03ee8aab14c5","eventID":"c9b62bfc-e30a-4a74-a3fe-6c00739c3b52","readOnly":true,"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","eventCategory":"Management","tlsDetails":{"tlsVersion":"TLSv1.3","cipherSuite":"TLS_AES_128_GCM_SHA256","clientProvidedHostHeader":"lambda.us-east-1.amazonaws.com"}},{"eventVersion":"1.11","userIdentity":{"type":"AssumedRole","principalId":"AROAI2YMI2XZQPW3IJ5T6:AVS-AsyncResourceMonitor","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsController/AVS-AsyncResourceMonitor","accountId":"830834166987","accessKeyId":"ASIA4C4MTJTFRHKZZFJW","sessionContext":{"sessionIssuer":{"type":"Role","principalId":"AROAI2YMI2XZQPW3IJ5T6","arn":"arn:aws:iam::830834166987:role/AWSLabsController","accountId":"830834166987","userName":"AWSLabsController"},"attributes":{"creationDate":"2025-10-27T00:00:52Z","mfaAuthenticated":"false"}}},"eventTime":"2025-10-27T00:00:52Z","eventSource":"lambda.amazonaws.com","eventName":"ListProvisionedConcurrencyConfigs","awsRegion":"us-east-1","sourceIPAddress":"3.89.144.38","userAgent":"aws-sdk-js/3.895.0 ua/2.1 os/linux#5.10.244-267.968.amzn2.x86_64 lang/js md/nodejs#20.19.4 api/lambda#3.895.0 exec-env/AWS_Lambda_nodejs20.x m/F,e","requestParameters":{"functionName":"LabStack-5afab8dd-572c-45-S3PublicAccessBlockFunct-LkLzhrffL5DL"},"responseElements":null,"requestID":"76a5b320-c675-4e0b-a4cc-4f4915a8690d","eventID":"ee754ed7-086f-4d78-8028-c263e8065946","readOnly":true,"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","eventCategory":"Management","tlsDetails":{"tlsVersion":"TLSv1.3","cipherSuite":"TLS_AES_128_GCM_SHA256","clientProvidedHostHeader":"lambda.us-east-1.amazonaws.com"}},{"eventVersion":"1.11","userIdentity":{"type":"AssumedRole","principalId":"AROAI2YMI2XZQPW3IJ5T6:AVS-AsyncResourceMonitor","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsController/AVS-AsyncResourceMonitor","accountId":"830834166987","accessKeyId":"ASIA4C4MTJTFRHKZZFJW","sessionContext":{"sessionIssuer":{"type":"Role","principalId":"AROAI2YMI2XZQPW3IJ5T6","arn":"arn:aws:iam::830834166987:role/AWSLabsController","accountId":"830834166987","userName":"AWSLabsController"},"attributes":{"creationDate":"2025-10-27T00:00:52Z","mfaAuthenticated":"false"}}},"eventTime":"2025-10-27T00:00:52Z","eventSource":"sagemaker.amazonaws.com","eventName":"ListMlflowTrackingServers","awsRegion":"us-east-1","sourceIPAddress":"3.89.144.38","userAgent":"aws-sdk-js/3.895.0 ua/2.1 os/linux#5.10.244-267.968.amzn2.x86_64 lang/js md/nodejs#20.19.4 api/sagemaker#3.895.0 exec-env/AWS_Lambda_nodejs20.x m/F,e","requestParameters":null,"responseElements":null,"requestID":"3c932bfa-5d61-4429-9835-7063bd5bc6e6","eventID":"78c0dc1a-e268-4e65-bc37-ac1b3d8b0115","readOnly":true,"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","eventCategory":"Management","tlsDetails":{"tlsVersion":"TLSv1.3","cipherSuite":"TLS_AES_128_GCM_SHA256","clientProvidedHostHeader":"api.sagemaker.us-east-1.amazonaws.com"}},{"eventVersion":"1.11","userIdentity":{"type":"AssumedRole","principalId":"AROAI2YMI2XZQPW3IJ5T6:AVS-AsyncResourceMonitor","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsController/AVS-AsyncResourceMonitor","accountId":"830834166987","accessKeyId":"ASIA4C4MTJTFRHKZZFJW","sessionContext":{"sessionIssuer":{"type":"Role","principalId":"AROAI2YMI2XZQPW3IJ5T6","arn":"arn:aws:iam::830834166987:role/AWSLabsController","accountId":"830834166987","userName":"AWSLabsController"},"attributes":{"creationDate":"2025-10-27T00:00:52Z","mfaAuthenticated":"false"}}},"eventTime":"2025-10-27T00:00:52Z","eventSource":"lambda.amazonaws.com","eventName":"ListProvisionedConcurrencyConfigs","awsRegion":"us-east-1","sourceIPAddress":"3.89.144.38","userAgent":"aws-sdk-js/3.895.0 ua/2.1 os/linux#5.10.244-267.968.amzn2.x86_64 lang/js md/nodejs#20.19.4 api/lambda#3.895.0 exec-env/AWS_Lambda_nodejs20.x m/F,e","requestParameters":{"functionName":"LabEnforcer_Helper_Function"},"responseElements":null,"requestID":"0ed8d3b9-b9da-471b-b52d-a4909024b320","eventID":"49d0cd29-3ab6-4356-a267-82dabfd0ee7d","readOnly":true,"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","eventCategory":"Management","tlsDetails":{"tlsVersion":"TLSv1.3","cipherSuite":"TLS_AES_128_GCM_SHA256","clientProvidedHostHeader":"lambda.us-east-1.amazonaws.com"}},{"eventVersion":"1.11","userIdentity":{"type":"AWSService","invokedBy":"cloudtrail.amazonaws.com"},"eventTime":"2025-10-27T00:01:27Z","eventSource":"s3.amazonaws.com","eventName":"GetBucketAcl","awsRegion":"us-east-1","sourceIPAddress":"cloudtrail.amazonaws.com","userAgent":"cloudtrail.amazonaws.com","requestParameters":{"bucketName":"mycloudtrailbucket0968","Host":"mycloudtrailbucket0968.s3.us-east-1.amazonaws.com","acl":""},"responseElements":null,"additionalEventData":{"SignatureVersion":"SigV4","CipherSuite":"TLS_AES_128_GCM_SHA256","bytesTransferredIn":0,"AuthenticationMethod":"AuthHeader","x-amz-id-2":"9/RjpZg69zz2Hbuycp96GYMWBWjMrSjQoMIR8fEIazPXgQsusm/TBLaFqzaA3ZnfqYzpttJkZ7g=","bytesTransferredOut":480},"requestID":"1AVH6THV9ZBN2GPP","eventID":"61e2c3a4-9ad3-379d-b06e-196778db5fc2","readOnly":true,"resources":[{"accountId":"830834166987","type":"AWS::S3::Bucket","ARN":"arn:aws:s3:::mycloudtrailbucket0968"}],"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","sharedEventID":"3b847176-d1c8-4332-8886-d8663e177524","eventCategory":"Management"},{"eventVersion":"1.11","userIdentity":{"type":"AssumedRole","principalId":"AROA4C4MTJTFWGPZCZOEB:5afab8dd-572c-45b1-8b68-6b9e00f836c7","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsUser-eo48QcrNoHcJZfkNoBbNHr/5afab8dd-572c-45b1-8b68-6b9e00f836c7","accountId":"830834166987","accessKeyId":"ASIA4C4MTJTFQ6JCKKNF","sessionContext":{"sessionIssuer":{"type":"Role","principalId":"AROA4C4MTJTFWGPZCZOEB","arn":"arn:aws:iam::830834166987:role/AWSLabsUser-eo48QcrNoHcJZfkNoBbNHr","accountId":"830834166987","userName":"AWSLabsUser-eo48QcrNoHcJZfkNoBbNHr"},"attributes":{"creationDate":"2025-10-26T23:53:26Z","mfaAuthenticated":"false"}}},"eventTime":"2025-10-27T00:01:29Z","eventSource":"s3.amazonaws.com","eventName":"GetBucketVersioning","awsRegion":"us-east-1","sourceIPAddress":"190.139.250.213","userAgent":"[Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36]","requestParameters":{"bucketName":"mycloudtrailbucket0968","Host":"s3.us-east-1.amazonaws.com","versioning":""},"responseElements":null,"additionalEventData":{"SignatureVersion":"SigV4","CipherSuite":"TLS_CHACHA20_POLY1305_SHA256","bytesTransferredIn":0,"AuthenticationMethod":"AuthHeader","x-amz-id-2":"fFwTp+HAaYXx21k2/x8k3QKNT9ioU3b+QTUWYn270Co4/cwLujelxYlpGvT8WgZ8RWvkksdLWBI=","bytesTransferredOut":113},"requestID":"FK4NTV2GC6KK1RWN","eventID":"45763ffa-b2fb-43b2-a30e-aabe5cc48ded","readOnly":true,"resources":[{"accountId":"830834166987","type":"AWS::S3::Bucket","ARN":"arn:aws:s3:::mycloudtrailbucket0968"}],"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","eventCategory":"Management","tlsDetails":{"tlsVersion":"TLSv1.3","cipherSuite":"TLS_CHACHA20_POLY1305_SHA256","clientProvidedHostHeader":"s3.us-east-1.amazonaws.com"}},{"eventVersion":"1.08","userIdentity":{"type":"AWSService","invokedBy":"AWS Internal"},"eventTime":"2025-10-27T00:01:57Z","eventSource":"sts.amazonaws.com","eventName":"AssumeRole","awsRegion":"us-east-1","sourceIPAddress":"AWS Internal","userAgent":"AWS Internal","requestParameters":{"roleArn":"arn:aws:iam::830834166987:role/AWSLabsController","roleSessionName":"AVS-AsyncResourceMonitor","policy":"{\"Version\":\"2012-10-17\",\"Statement\":[{\"Effect\":\"Allow\",\"Action\":[\"cloudtrail:LookupEvents\",\"cloudwatch:GetMetricData\",\"codebuild:BatchGetBuilds\",\"codebuild:ListBuilds\",\"ec2:List*\",\"ec2:Describe*\",\"ec2:TerminateInstances\",\"ec2:ModifyInstanceAttribute\",\"ecs:List*\",\"ecs:Describe*\",\"ecs:StopTask\",\"eks:ListClusters\",\"eks:DescribeCluster\",\"glue:ListJobs\",\"glue:GetJob\",\"cloudwatch:Get*\",\"lambda:Get*\",\"lambda:List*\",\"lambda:DeleteFunction\",\"sagemaker:DeleteMlflowTrackingServer\",\"sagemaker:DescribeMlflowTrackingServer\",\"sagemaker:ListMlflowTrackingServers\",\"sagemaker:ListNotebookInstances\"],\"Resource\":\"*\"}]}","durationSeconds":900},"responseElements":{"credentials":{"accessKeyId":"ASIA4C4MTJTF66AVRECM","sessionToken":"IQoJb3JpZ2luX2VjEOD//////////wEaCXVzLWVhc3QtMSJHMEUCICmdcyAsgW6TJN/9UJKq5zb7NuBpdJGytmnNZ6aG+DQkAiEAyN43EMWHYMU0ZtO8OvAqgkowqLb89O3CicVIDuDkgU8qxAQImf//////////ARAEGgw4MzA4MzQxNjY5ODciDMtofJyPxHvb77oRlCqYBKx1gLaV4RoyTMX37D4oWKawcKg5gMnS4b5QiOrNWNDIn2ZU0DvC3KDVikxG9MI4sZrNtdphs9eBtsXok27adG82Otj2y9w0603DemgJE26RDGYlW6JjuksojHiTTVQUobmjsTQtNYZ7IxMHfF6ud0wt8OU8M6aLzE9Dy5F3g+drZUQ0iX2VUTYntiSoANLFbX9Ja2o2+2gC9gL2QHj+a8BKSSF9XzbAhL/H2AVZ2Z8+6ebTHCFVrDNnmLZp+zl657gFbJN3THglYa751OV/RhXsGrrhvUWmlmFGBkY3gaHGfkGHXBMGyAlbpd8q+AOe0rWWVFxEEZcCG4umZ4c4ycrDDEQfVmzHg8z2UWcW9OHM/0X/0Xfn0y2tVwdJW3IOSBm1Oggr+WeJNUpgnjoC75MlCg4GywUq5jlcWrL1v/P2YeycZCWj+fVibqWdQEl+Vc6ke6JV6jBMfyQc1omPkFFqLVfo0dU9RYwotYZ/XnCNL+ayMSxEgM9ny8YLvq80/rzTHt6HH9N978cYs3wlvpxiZuNOgaFrrIEJtwYxkxFuWigXdU3jC4+SHPf2Fs7LXy8BQBb7F+F0BAc1Cnase+hR8hX3FjMWrD0aUa4WozPZ7jr12DXM61i+FP4SPv4hRlWZJlyQgHyJm3GDDV8VeogDU8KrAmPp2dCCFBxrd5Mt7YpDvWWySTAa0HOpvrDR3WfsPa0NrblCMPXs+scGOo8Bz22FeSpI8JPWyBF8hQ4Y1IxLbJ86oK7Zvk/JrCdEZ9IPtJBcpBtuNJuUmIoBWc7E6B6HmSvsr2rJIhUC/ilC/TxkKhRk7/vEUNcgSDNI35/VmWatHshJORRMYREal9a0S+YvZZqUndKLvaaOBBpv0s0sYjPntneCvpKcIKsA1IdmAmiGbReWK4QfuTRr864=","expiration":"Oct 27, 2025, 12:16:57 AM"},"assumedRoleUser":{"assumedRoleId":"AROAI2YMI2XZQPW3IJ5T6:AVS-AsyncResourceMonitor","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsController/AVS-AsyncResourceMonitor"},"packedPolicySize":64},"additionalEventData":{"ExtendedRequestId":"MTp1cy1lYXN0LTE6MTc2MTUyMzMxNzYyMzpSOllRUmE0Snht"},"requestID":"7f250d7e-d459-4c33-b232-6c5bbbd8cc08","eventID":"34809761-029f-323a-a86e-e905ca6092f6","readOnly":true,"resources":[{"accountId":"830834166987","type":"AWS::IAM::Role","ARN":"arn:aws:iam::830834166987:role/AWSLabsController"}],"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","sharedEventID":"ca1e4ed8-cefc-4856-9e0c-db88eae2dcd1","eventCategory":"Management"},{"eventVersion":"1.11","userIdentity":{"type":"AssumedRole","principalId":"AROAI2YMI2XZQPW3IJ5T6:AVS-AsyncResourceMonitor","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsController/AVS-AsyncResourceMonitor","accountId":"830834166987","accessKeyId":"ASIA4C4MTJTFYYEJVMJY","sessionContext":{"sessionIssuer":{"type":"Role","principalId":"AROAI2YMI2XZQPW3IJ5T6","arn":"arn:aws:iam::830834166987:role/AWSLabsController","accountId":"830834166987","userName":"AWSLabsController"},"attributes":{"creationDate":"2025-10-27T00:01:57Z","mfaAuthenticated":"false"}},"invokedBy":"lambda.amazonaws.com"},"eventTime":"2025-10-27T00:01:57Z","eventSource":"kms.amazonaws.com","eventName":"Decrypt","awsRegion":"us-east-1","sourceIPAddress":"lambda.amazonaws.com","userAgent":"lambda.amazonaws.com","requestParameters":{"encryptionAlgorithm":"SYMMETRIC_DEFAULT","encryptionContext":{"aws:lambda:FunctionArn":"arn:aws:lambda:us-east-1:830834166987:function:LabStack-5afab8dd-572c-45-S3PublicAccessBlockFunct-LkLzhrffL5DL"}},"responseElements":null,"additionalEventData":{"keyMaterialId":"249ff1560efc7b313c30919fb4f95b43831f4dddb51134f69816742654cf2919"},"requestID":"b9c7cbdb-e26c-4ea5-bf02-c179139b9fd5","eventID":"70a9f056-8931-4f9e-877d-c2f85ab95297","readOnly":true,"resources":[{"accountId":"830834166987","type":"AWS::KMS::Key","ARN":"arn:aws:kms:us-east-1:830834166987:key/69ba0c3d-57aa-4fe5-8318-b2a40e1366b6"}],"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","eventCategory":"Management"},{"eventVersion":"1.11","userIdentity":{"type":"AssumedRole","principalId":"AROAI2YMI2XZQPW3IJ5T6:AVS-AsyncResourceMonitor","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsController/AVS-AsyncResourceMonitor","accountId":"830834166987","accessKeyId":"ASIA4C4MTJTFYYEJVMJY","sessionContext":{"sessionIssuer":{"type":"Role","principalId":"AROAI2YMI2XZQPW3IJ5T6","arn":"arn:aws:iam::830834166987:role/AWSLabsController","accountId":"830834166987","userName":"AWSLabsController"},"attributes":{"creationDate":"2025-10-27T00:01:57Z","mfaAuthenticated":"false"}},"invokedBy":"lambda.amazonaws.com"},"eventTime":"2025-10-27T00:01:57Z","eventSource":"kms.amazonaws.com","eventName":"Decrypt","awsRegion":"us-east-1","sourceIPAddress":"lambda.amazonaws.com","userAgent":"lambda.amazonaws.com","requestParameters":{"encryptionAlgorithm":"SYMMETRIC_DEFAULT","encryptionContext":{"aws:lambda:FunctionArn":"arn:aws:lambda:us-east-1:830834166987:function:LabEnforcer_Function"}},"responseElements":null,"additionalEventData":{"keyMaterialId":"249ff1560efc7b313c30919fb4f95b43831f4dddb51134f69816742654cf2919"},"requestID":"50186093-3582-4888-8bfd-2e7bc775582a","eventID":"15b0445f-e2d2-4cb2-ad51-b703b92c9cd9","readOnly":true,"resources":[{"accountId":"830834166987","type":"AWS::KMS::Key","ARN":"arn:aws:kms:us-east-1:830834166987:key/69ba0c3d-57aa-4fe5-8318-b2a40e1366b6"}],"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","eventCategory":"Management"},{"eventVersion":"1.11","userIdentity":{"type":"AssumedRole","principalId":"AROAI2YMI2XZQPW3IJ5T6:AVS-AsyncResourceMonitor","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsController/AVS-AsyncResourceMonitor","accountId":"830834166987","accessKeyId":"ASIA4C4MTJTF66AVRECM","sessionContext":{"sessionIssuer":{"type":"Role","principalId":"AROAI2YMI2XZQPW3IJ5T6","arn":"arn:aws:iam::830834166987:role/AWSLabsController","accountId":"830834166987","userName":"AWSLabsController"},"attributes":{"creationDate":"2025-10-27T00:01:57Z","mfaAuthenticated":"false"}}},"eventTime":"2025-10-27T00:01:57Z","eventSource":"cloudtrail.amazonaws.com","eventName":"LookupEvents","awsRegion":"us-east-1","sourceIPAddress":"44.222.105.239","userAgent":"aws-sdk-nodejs/2.1692.0 linux/v20.19.4 exec-env/AWS_Lambda_nodejs20.x promise","requestParameters":{"lookupAttributes":[{"attributeKey":"ReadOnly","attributeValue":"false"}],"startTime":"Oct 26, 2025, 11:51:57 PM","endTime":"Oct 27, 2025, 12:01:57 AM"},"responseElements":null,"requestID":"c277965a-098a-4407-84c2-ce1decefe9d8","eventID":"fa784979-a061-49e3-913c-775311310d4c","readOnly":true,"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","eventCategory":"Management","tlsDetails":{"tlsVersion":"TLSv1.3","cipherSuite":"TLS_AES_128_GCM_SHA256","clientProvidedHostHeader":"cloudtrail.us-east-1.amazonaws.com"}},{"eventVersion":"1.11","userIdentity":{"type":"AssumedRole","principalId":"AROAI2YMI2XZQPW3IJ5T6:AVS-AsyncResourceMonitor","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsController/AVS-AsyncResourceMonitor","accountId":"830834166987","accessKeyId":"ASIA4C4MTJTF66AVRECM","sessionContext":{"sessionIssuer":{"type":"Role","principalId":"AROAI2YMI2XZQPW3IJ5T6","arn":"arn:aws:iam::830834166987:role/AWSLabsController","accountId":"830834166987","userName":"AWSLabsController"},"attributes":{"creationDate":"2025-10-27T00:01:57Z","mfaAuthenticated":"false"}}},"eventTime":"2025-10-27T00:01:57Z","eventSource":"ecs.amazonaws.com","eventName":"ListClusters","awsRegion":"us-east-1","sourceIPAddress":"44.222.105.239","userAgent":"aws-sdk-nodejs/2.1692.0 linux/v20.19.4 exec-env/AWS_Lambda_nodejs20.x promise","requestParameters":null,"responseElements":null,"requestID":"6aa10166-c66a-4370-a696-7ca66c30c733","eventID":"c4738aee-44bb-4375-b70e-20d4ddd8ea68","readOnly":true,"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","eventCategory":"Management","tlsDetails":{"tlsVersion":"TLSv1.3","cipherSuite":"TLS_AES_128_GCM_SHA256","clientProvidedHostHeader":"ecs.us-east-1.amazonaws.com"}},{"eventVersion":"1.10","userIdentity":{"type":"AssumedRole","principalId":"AROAI2YMI2XZQPW3IJ5T6:AVS-AsyncResourceMonitor","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsController/AVS-AsyncResourceMonitor","accountId":"830834166987","accessKeyId":"ASIA4C4MTJTF66AVRECM","sessionContext":{"sessionIssuer":{"type":"Role","principalId":"AROAI2YMI2XZQPW3IJ5T6","arn":"arn:aws:iam::830834166987:role/AWSLabsController","accountId":"830834166987","userName":"AWSLabsController"},"attributes":{"creationDate":"2025-10-27T00:01:57Z","mfaAuthenticated":"false"}}},"eventTime":"2025-10-27T00:01:57Z","eventSource":"ec2.amazonaws.com","eventName":"DescribeInstances","awsRegion":"us-east-1","sourceIPAddress":"44.222.105.239","userAgent":"aws-sdk-js/3.895.0 ua/2.1 os/linux#5.10.244-267.968.amzn2.x86_64 lang/js md/nodejs#20.19.4 api/ec2#3.895.0 exec-env/AWS_Lambda_nodejs20.x m/F,e","requestParameters":{"instancesSet":{},"filterSet":{"items":[{"name":"instance-state-name","valueSet":{"items":[{"value":"running"}]}}]}},"responseElements":null,"requestID":"a3d543c3-ab4b-450a-b066-c5f9be477476","eventID":"850b97b9-4515-4042-bfe5-2dfe3a98ae9d","readOnly":true,"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","eventCategory":"Management","tlsDetails":{"tlsVersion":"TLSv1.3","cipherSuite":"TLS_AES_128_GCM_SHA256","clientProvidedHostHeader":"ec2.us-east-1.amazonaws.com"}},{"eventVersion":"1.09","userIdentity":{"type":"AssumedRole","principalId":"AROAI2YMI2XZQPW3IJ5T6:AVS-AsyncResourceMonitor","arn":"arn:aws:sts::830834166987:assumed-role/AWSLabsController/AVS-AsyncResourceMonitor","accountId":"830834166987","accessKeyId":"ASIA4C4MTJTF66AVRECM","sessionContext":{"sessionIssuer":{"type":"Role","principalId":"AROAI2YMI2XZQPW3IJ5T6","arn":"arn:aws:iam::830834166987:role/AWSLabsController","accountId":"830834166987","userName":"AWSLabsController"},"attributes":{"creationDate":"2025-10-27T00:01:57Z","mfaAuthenticated":"false"}}},"eventTime":"2025-10-27T00:01:57Z","eventSource":"eks.amazonaws.com","eventName":"ListClusters","awsRegion":"us-east-1","sourceIPAddress":"44.222.105.239","userAgent":"aws-sdk-nodejs/2.1692.0 linux/v20.19.4 exec-env/AWS_Lambda_nodejs20.x promise","requestParameters":null,"responseElements":null,"requestID":"a41fca99-cfa7-44ea-93f2-261e27fbe4ac","eventID":"d9ba3be8-fa90-459f-ab70-36fc81d7d132","readOnly":true,"eventType":"AwsApiCall","managementEvent":true,"recipientAccountId":"830834166987","eventCategory":"Management"}]}
```

## Task 6: Manage encryption keys
In this task you manage encryption keys for users and roles.
1. Navigate to the browser tab/window S3 service.
2. At the top of the page, in the unified search bar, search for and choose Key Management Service.
3. In the left navigation panel, choose Customer managed keys.
4. Choose myFirstKey.

This opens a page where you can make changes to the KMS settings. One of the configurable setting is Add or Remove - key administrators, and key users.

5. In the Key users section, select the user or role that you are signed in with.
6. Choose Remove.
Immediately, the user’s permission to use this key is removed.

7. In the Key users section, choose Add then:
Search and select the user or role that you are signed in with.
The user starts with the characters AWSLabsUser-

8. Choose Add.
Immediately, the user’s permission to use this key is restored.

This shows how you can control which IAM users or roles can use KMS Keys that you create. The same process can be used to control which IAM users can manage KMS keys.

 Task complete: You have managed encryption keys user permissions.

![task](images/task6-1.png)
![task](images/task6-2.png)

# Conclusion
You have successfully done the following:
* Created an Encryption Key
* Created an S3 bucket with CloudTrail logging functions
* Encrypted an image and stored it in your S3 bucket
* Viewed the encrypted image using the AWS Management Console
* Monitored encryption key usage using CloudTrail
* Managed encryption keys for users and roles

# Additional resources
https://aws.amazon.com/es/kms/ 
https://aws.amazon.com/es/s3/ 

# Assesment
Question 1
Answer status:
Correct
Question
What is the primary purpose of configuring CloudTrail to store logs in an S3 bucket?
Question Type
Multiple Choice
Answer Options
Option
Your Selection
Correct Answer
Rationale
To improve S3 bucket performance
Not selected
Incorrect
Storing CloudTrail logs does not directly improve S3 bucket performance.
To maintain an audit trail of AWS account activity
Selected
Correct
This is the correct answer. CloudTrail logs stored in S3 provide a detailed record of actions taken on your AWS account, serving as an audit trail.
To reduce storage costs
Not selected
Incorrect
Storing logs in S3 is not primarily about reducing costs, but about maintaining records.
To enable real-time log analysis
Not selected
Incorrect
While logs can be analyzed, real-time analysis is not the primary purpose of storing logs in S3.
Question 2
Answer status:
Correct
Question
What is the benefit of using AWS KMS for S3 bucket encryption instead of S3-managed keys?
Question Type
Multiple Choice
Answer Options
Option
Your Selection
Correct Answer
Rationale
It's the only way to encrypt data in S3
Not selected
Incorrect
This is incorrect as S3 offers multiple encryption options, including S3-managed keys.
It allows for unlimited storage in S3 buckets
Not selected
Incorrect
The encryption method does not affect storage limits in S3 buckets.
It's faster to encrypt and decrypt data
Not selected
Incorrect
KMS encryption is not necessarily faster than S3-managed encryption.
It provides additional key management features and auditing
Selected
Correct
AWS KMS offers more control over encryption keys, including the ability to create, rotate, disable, and define usage policies. It also provides detailed auditing through AWS CloudTrail.
Question 3
Answer status:
Correct
Question
What is the basic principle of access control for KMS keys?
Question Type
Multiple Choice
Answer Options
Option
Your Selection
Correct Answer
Rationale
Only the root user can access KMS keys
Not selected
Incorrect
While the root user has full access, it's not the only entity that can access KMS keys. Access can be granted to other IAM users and roles.
KMS keys are automatically shared across all IAM users
Not selected
Incorrect
This is false. KMS keys are not automatically shared. Access must be explicitly granted to specific IAM users or roles.
All users have full access to all keys by default
Not selected
Incorrect
This is incorrect and would be a security risk. Access to KMS keys is restricted by default.
Access is denied by default unless explicitly granted
Selected
Correct
This is the correct principle. KMS follows the principle of least privilege, where access to keys is denied by default and must be explicitly granted through key policies or IAM policies.
Question 4
Answer status:
Incorrect
Question
Which CloudTrail event type would you see for AWS KMS key usage monitoring?
Question Type
Multiple Choice
Answer Options
Option
Your Selection
Correct Answer
Rationale
Management events
Not selected
Correct
AWS KMS API calls are considered management events in CloudTrail, as they involve operations on AWS resources.
Data events
Selected
Incorrect
Data events typically relate to object-level activity in services like S3 or Lambda, not KMS operations.
Insight events
Not selected
Incorrect
Insight events are used for detecting unusual API activity, not for standard KMS operations.
Log events
Not selected
Incorrect
This is not a standard CloudTrail event type. CloudTrail itself generates log entries, not 'log events'.
Question 5
Answer status:
Correct
Question
What is the purpose of defining key administrative permissions when creating a KMS master key?
Question Type
Multiple Choice
Answer Options
Option
Your Selection
Correct Answer
Rationale
To control who can manage the key's properties and policies
Selected
Correct
Key administrative permissions define who can manage the key, including updating its properties and key policies.
To set the key's rotation schedule
Not selected
Incorrect
Key rotation is a separate feature and not directly related to administrative permissions.
To determine who can use the key for encryption and decryption
Not selected
Incorrect
This is controlled by key usage permissions, not administrative permissions.
To specify which AWS services can use the key
Not selected
Incorrect
This is typically controlled through key policies and resource policies, not administrative permissions.
