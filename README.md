# What is the difference between AWS IAM and Azure AD ?

 AWS IAM can be classified as a tool in the "Resource Management" category, while Azure Active Directory is grouped under "Identity Management".
 
 For AWS, users are managed within their AWS account, while for Azure, users are managed outside of the subscription. AWS Organization and AWS IAM Identify Center (used to be called as AWS SSO) can provide with Landing Zone which you can use SSO to each AWS account.

 In Azure, on the other hand, it's the Azure Active Directory that manages users, so you need to associate a subscription with it (you don't need to get a separate subscription to manage users). 
 
Yes, Azure AD remains if subscription expires.


![azure-aws-integration.png](https://github.com/developer-onizuka/What_is_AzureAD/blob/main/azure-aws-integration.png) <br>
> https://learn.microsoft.com/ja-jp/azure/architecture/reference-architectures/aws/aws-azure-ad-security

The followings are steps to attach policys to each user and role in AWS and Azure based on each clould's way of thinking.
---
# 1. AWS IAM
# 1-1. IAM User
- Step1: Getting AWS account (root user), But it is not recommended to use at creating AWS resouces.
- Step2: Create IAM user instead of using the AWS root account, But it is not recommended to share among coworkers.
- Step3: Create IAM policy which can access the services you want to use. 
- Step4: Attach the IAM policy to the IAM user.

For example, if you create a policy called "AmazonS3FullAccess" that grants all operations to S3 (of the AWS storage service) and attach it to the IAM user Bob, Bob will do everything to S3. You might use the User's Access Key for some purposes such as AWS CLI, and before using it you can register Access Key below:
```
$ aws configure

AWS Access Key ID [None]: XXXXXXXXXXXXXXXXXXXXX
AWS Secret Access Key [None]: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Default region name [None]: ap-northeast-1
Default output format [None]: text
```

# 1-2. IAM role
If you want to create the EC2 instance which sends Emails with AWS SES, IAM role is a good solution for it instead of IAM User's Access Key. But you don't forget to write the programs to be aware of ec2-metadata, see also [here](https://aws.amazon.com/jp/blogs/developer/ec2metadata/). In this blog post, you’ve seen how you can query the EC2 instance metadata using curl or the ec2-metadata tool.<br>

- Create the IAM role (role type should be "Amazon EC2 role")
- Attach the access grant of "AmazonSESFullAccess" to it.
- Attach the role to the EC2 instance you created. But note that it is not possible to attach it to existed Instances. Then, you might recreate instance again.
- Run the programs with AWS SES on the EC2 instance. (See also [#1-5](https://github.com/developer-onizuka/What_is_AzureAD/blob/main/README.md#1-4-how-to-retrieve-security-credentials-from-ec2-instance) about how IAM role works inside of EC2 instance.)

# 1-3. Service role
A service role is an IAM role that a service assumes to perform actions on your behalf. An IAM administrator can create, modify, and delete a service role from within IAM. For more information, see [Creating a role to delegate permissions to an AWS service](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html) in the IAM User Guide.<br>
A service role is limited to the services in the IAM role written above, and can be identified as a service role by the ARN path.
When a role serves a specialized purpose for a service, it is categorized as [a service role for EC2 instances](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html#iam-term-service-role-ec2) (for example), or a service-linked role. To see what services support using service-linked roles, or whether a service supports any form of temporary credentials, see [AWS services that work with IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_aws-services-that-work-with-iam.html). 
> https://qiita.com/yuta-katayama-23/items/0606e8d590968e43fd27

# 1-4. Difference between IAM User's Access Key and IAM role
|  | Possibility of leaking | Impact |
| :--- | :--- | :--- |
| IAM User's Access Key | Hard coded usernames, passwords, tokens and other secrets in the source code. | If leaked, it can be used by anyone who obtains it, which can potentially compromise your AWS resources and Account itself. An AWS access key is an authentication key created to authenticate programmatic access to AWS services such as S3 and EC2. So, anyone who obtains it can create resources such as EC2 instances if the access key has another grants such as "ec2:RunInstances". |
| IAM role | No | -<br> (But you can't use IAM role on your onprem servers to access AWS resources, because only AWS instance such as EC2 can have a provider that manages the temporary security credentials transparently (See also [#1-5](https://github.com/developer-onizuka/What_is_AzureAD#1-4-how-to-retrieve-security-credentials-from-ec2-instance)). The way to access from outside is only to use IAM User's Access key. But you might use switch role instead of using Access key bound for each IAM user, directly. See also [#1-6](https://github.com/developer-onizuka/What_is_AzureAD#1-5-switch-role) about Switch role.) |

# 1-5. How to retrieve security credentials from EC2 instance
![IAM-Role.png](https://github.com/developer-onizuka/What_is_AzureAD/blob/main/IAM-Role.png)

> https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html#instance-metadata-security-credentials
>
> https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html
<br>

You can retrieve some secrets directly from the metadata service in your EC2 instance through 169.254.169.254.
```
$ curl http://169.254.169.254/latest/iam/security-credentials/<Your own IAM role's name>
{
  "Code" : "Success",
  "LastUpdated" : "2020-06-11T10:14:39Z",
  "Type" : "...",
  "AccessKeyId" : "...",
  "SecretAccessKey" : "...",
  "Token" : "...",
  "Expiration" : "2020-06-11T16:34:27Z"
}
```

# 1-6. Switch role
In the case of AWS, it is common to create an AWS account that stores only IAM users separately from the AWS account that holds resources, and assign permissions from the AWS account that holds resources. See the URL below, so that you can understand how to switch the role between AWS account that stores only IAM users and IAM role in the AWS account that holds resources.
 > https://www.youtube.com/watch?v=d7R08uPS98M
 > 
 > https://iselegant.hatenablog.com/entry/2020/05/24/215808

The goal of this video above is the table below:
| UserManagement <br>(AWS account: 36989xxxxxxx) | Prod <br>(AWS account: 46017xxxxxxx) | Dev <br>(AWS account: 39355xxxxxxx) |
| :---: | :---: | :---: |
| Vipin <br> (user) | CrossAccount-AppsProds <br> (role) | CrossAccount-AppsDevs <br> (role) |
| Deepak <br> (user) | - | CrossAccount-AppsDevs <br> (role) |

- CrossAccount-AppsProds: Full Access to EC2 and S3 in Product Environment.
- CrossAccount-AppsDevs: Full Access to EC2 and S3 in Develop Environment.
- Vipin and Deepak are managed in the Account of 36989xxxxxxx.
- Vipin and Deepak are in the group AppsTeam whose policy is ReadonlyAccess.
- In addition to the policy above, AssumeRoles which allow them to use AWS STS (Security Token Service) is necessary when Vipin and Deepak switch each role above.
- **AssumeRole API gives them temporary security credential which allows them to act as if they have the role** (CrossAccount-AppsProds or CrossAccount-AppsDevs).
- Trust relationship in AWS account of Prod (46017xxxxxxx) should be used so that the account can accept the only AWS user who already has the trust relationship. 
- "arn:aws:iam::36989xxxxxxx:root" means AWS account itself and all of user in the AWS account 36989xxxxxxx is acceptable. You should use "arn:aws:iam::36989xxxxxxx:/user/Vipin" instead of it. 
- **Edit Trust Relationship** in AWS account: 46017xxxxxxx (Prod), so that **only Vipin** can switch the role to CrossAccount-AppsProds.
```
{
  "Version" : "2012-10-17",
  "Statement" : [
    {
      "Effect" : "Allow",
      "Principal" : {
        "AWS" : "arn:aws:iam::36989xxxxxxx:root"
            --> "arn:aws:iam::36989xxxxxxx:/user/Vipin"
      },
      "Action" : "sts:AssumeRole",
      "Condition" : {}
    }
  ]
}
``` 
- You should use the External ID in addition to above if you use "arn:aws:iam::36989xxxxxxx:root" which is available for everyone to assume a role. **(the confused deputy problem)**
> https://www.youtube.com/watch?v=HP8XSRWrFQc


# 2. Azure Active Directory
- Step1: Create a new tenant which is an instance in Azure Active Directory at Azure portal console.
  > https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-access-create-new-tenant
    - Please note when you create a new Azure AD tenant, you become the first user of that tenant. As the first user, you're automatically assigned the Global Admin role.
  - You may use an existing Azure AD tenant like your organization's Office 365 instead of creating it.

- Step2: Change or add additional domain names / Add users / Add groups and members in the tenant, if you want.
  > https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview

- Step3: Get a subscription which is in accordance with your purposes. (See also [#2-1](https://github.com/developer-onizuka/What_is_AzureAD/blob/main/README.md#2-1-subscription))

- Step4: Attach the subscription to a single Azure AD tenant. 
  > https://www.youtube.com/watch?v=-rudwyS1KNA
  - You can not make any resources such as storage accounts on the Azure AD tenant if you don't associate Azure AD tenant with some specific subscriptions. You might find messages like "You are currently signed into the xxx directory which does not have any subscriptions."
  - You must be an owner of the subscription if you are going to associate the subscription with an Azure AD tenant.
  - One Azure AD tenant can be associated with sevral subscriptions. Example, Prod/Dev subscriptions and IT/HR subscriptions can be associated with an Azure AD tenant. It is convienent to manage security roles and billings.

- Step5: Manage Role-Based Access Control (RBAC)
  - Assign an RBAC role to an Azure AD user to control Virtual Machine or storage account. (See also [#2-2](https://github.com/developer-onizuka/What_is_AzureAD/blob/main/README.md#2-2-role-security-principal-and-scope))
     - Top three RBAC roles are Owner, Contributer and Reader.
  - You can also assign an RBAC role to Managed ID which resides in virtual machine or Service principal. (See also [#2-3](https://github.com/developer-onizuka/What_is_AzureAD#2-3-managed-id) and [#2-4](https://github.com/developer-onizuka/What_is_AzureAD#2-4-service-principal))
     > https://docs.microsoft.com/en-us/learn/modules/implement-managed-identities/
     > 
     > https://docs.microsoft.com/en-us/learn/modules/authenticate-apps-with-managed-identities/

Tips:
---
As you know IAM role in AWS, you can't use Managed ID on your onprem servers to access Azure resources. This is because only Azure instance can have a provider (instance metadata service) that manages the temporary security credentials transparently. 
If you have to access from public, you might use [SAS-token](https://github.com/developer-onizuka/azureBlob#6-download-test-file-to-upload-and-upload-it-to-the-blob) in Azure Storage Account instead of Azure Storage Account's Access key.

# 2-1. Subscription
Subscription is a logical container that Microsoft uses to maintain their billing relationship with the Azure users. The billing relationship starts and stops at the subscription boundary.
> https://www.youtube.com/watch?v=LMAC0IIYSJM

![azure-ad.jpg](https://github.com/developer-onizuka/What_is_AzureAD/blob/main/azure-ad.jpg)

# 2-2. Role, Security principal and Scope
- Security principal is an Azure object (identity) that can be assigned to a role (ex; user, groups, service principal and managed id) 

![security-principal.png](https://github.com/developer-onizuka/What_is_AzureAD/blob/main/security-principal.png)

- When you assign roles to security principal, you must specify a scope. Scope is the set of resources the access applies to. ie, It is a definition of "where it can be done?".

![scope.png](https://github.com/developer-onizuka/What_is_AzureAD/blob/main/scope.png)

> https://www.youtube.com/watch?v=4v7ffXxOnwU
> 
> https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-steps

# 2-3. Managed ID
Manage ID is a locally running internal endpoint (http://169.254.169.254/metadata/identity/oauth2/token) which resides in virtual machine. This endpoint is a micro web service running on that virtual machine. And it is only acceptable from within that virtual machine. So on your locally running code can actually request tokens from it. **Your code just send a token request with no credentials to this endpoint.** The life cycle of Managed ID is tied to that resouce so if you delete that virtual machine the ID will be also deleted. You don't need to put credentials on your code inside because platform manages the credentials. So, it is very secure.

![azure-ManagedID.png](https://github.com/developer-onizuka/What_is_AzureAD/blob/main/azure-ManagedID.png)


Tips:
---
You might create the role which allows to access the blob storage in mystorageaccount20220103 (target azure resource). And attach the role to user-assigned Managed ID you already created in the service of "Managed Identity". The source application on the azure resource such as virtual machine attached the role thru managed ID can access to the blob which is a target of application on the virtual machine.

| user-assigned Managed ID | Role | Scope |
| --- | --- | --- |
| AzDemoUA | Storage Blob Data Contributor | mystorageaccount20220103 |

> https://www.youtube.com/watch?v=sA_mXKy_dKU
> 
> https://www.youtube.com/watch?v=vYUKC0mZFqI
>
> https://www.christofvg.be/2020/05/01/Manage-Azure-Resource-Manager-using-a-Managed-Identity/

Get a token by using C#
---
```
using System;
using System.Collections.Generic;
using System.IO;
using System.Net;
using System.Web.Script.Serialization; 

// Build request to acquire managed identities for Azure resources token
HttpWebRequest request = (HttpWebRequest)WebRequest.Create("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/");
request.Headers["Metadata"] = "true";
request.Method = "GET";

try
{
    // Call /token endpoint
    HttpWebResponse response = (HttpWebResponse)request.GetResponse();

    // Pipe response Stream to a StreamReader, and extract access token
    StreamReader streamResponse = new StreamReader(response.GetResponseStream()); 
    string stringResponse = streamResponse.ReadToEnd();
    JavaScriptSerializer j = new JavaScriptSerializer();
    Dictionary<string, string> list = (Dictionary<string, string>) j.Deserialize(stringResponse, typeof(Dictionary<string, string>));
    string accessToken = list["access_token"];
}
catch (Exception e)
{
    string errorText = String.Format("{0} \n\n{1}", e.Message, e.InnerException != null ? e.InnerException.Message : "Acquire token failed");
}
```
See also https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/how-to-use-vm-token .<br>
The Json structure of the response of metadata service in Azure is here. The above C# code picks up Bearer code through a dictionary of "access_token".
```
{
  "access_token": "eyJ0eXAi...",
  "refresh_token": "",
  "expires_in": "3599",
  "expires_on": "1506484173",
  "not_before": "1506480273",
  "resource": "https://management.azure.com/",
  "token_type": "Bearer"
}
```

# 2-3-1. Difference between System-assigned and User-assigned Managed ID

| | System-assigned | User-assigned |
| --- | --- | --- |
| ID : Resource | 1 : 1 | 1 : N |
| ID's behavior at deleting Resource | gone | remain |
| How to use properly | for Small systems | for Large systems |  

# 2-3-2. Access with Managed ID between Azure subscriptions
A storage account is nothing but an account having a right permission to do something about storage resources. For example, you may add a role on Managed ID in the storage account's Access Control (IAM) menu so that VMs with Managed ID can get reading files in the blob storage.<br>

- [VMs with Managed ID can access the storage account in the different subscription which each belongs to the same Azure AD tenant.](https://stackoverflow.com/questions/59069065/can-managed-identity-of-a-azure-function-have-access-across-multiple-subscriptio) <br> 
- [Managed IDs don't currently support cross-tenant scenarios.](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/managed-identities-faq#can-i-use-a-managed-identity-to-access-a-resource-in-a-different-directorytenant) <br>

![ManagedID_Between_Subscription.drawio.png](https://github.com/developer-onizuka/What_is_AzureAD/blob/main/ManagedID_Between_Subscription.drawio.png)


# 2-4. Service principal (using OAuth 2.0 access token)
Unfortunately, managed ID does not support on-premises applications or services. 
Service principals help us avoid to use each secret when we need to access Azure resources. But it is very similar to Managed Id. Instead of using Managed Id, **you can request an OAuth 2.0 access token** from the Microsoft identity platform, Azure AD. If authentication succeeds, Azure AD returns the access token to the application, and the application can then use the access token to authorize requests to Azure resource such as Azure Blob storage or Queue storage.

The main difference between Service principal and Managed identity is:
- You don’t need to expose any credentials in your code if you use "Managed identity" which manages the creation and automatic renewal of service principal on your behalf.
- But in service principal, you need to expose application id and secret id in your code to let Azure AD generate a token so that your code can access any Azure resource. 
- Exposing the application id (same as client id) and secret id makes some security issues, so you don't push your code directry into github.
- Ideally, you should opt for service principal only if the service you use doesn’t support managed identity.

So, You use these manual processes in only two scenarios:
- Your application or service is running on-premises.
- The resources or applications that you need to access don't support managed identities.

The most secure and convenient way to handle authentication within Azure is to use managed identities.

> https://github.com/developer-onizuka/OAuth2.0_Authorization

![ServicePrincipal.drawio.png](https://github.com/developer-onizuka/Diagrams/blob/main/What_is_AzureAD/ServicePrincipal.drawio.png)


# Refferences:
> https://tech-blog.cloud-config.jp/2020-08-24-azure-authentication-tools/

> https://www.youtube.com/watch?v=7I1eQ_2SjJY

> https://thecloudhub.com/2019/03/22/whats-an-azure-service-principal-and-managed-identity/

> https://azuread.net/archives/9397


