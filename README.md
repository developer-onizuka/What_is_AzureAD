# What is the difference between AWS IAM and Azure AD ?

 AWS IAM can be classified as a tool in the "Cloud Access Management" category, while Azure Active Directory is grouped under "Password Management".
 
 For AWS, users are managed within their AWS account, while for Azure, users are managed outside of the subscription. In the case of AWS, it is common to create an AWS account that stores only IAM users separately from the AWS account that holds resources, and assign permissions from the AWS account that holds resources. See the URL below, so that you can understand how to switch the role between AWS account that stores only IAM users and IAM role in the AWS account that holds resources.
- https://www.youtube.com/watch?v=d7R08uPS98M

In Azure, on the other hand, it's the Azure Active Directory that manages users, so you need to associate a subscription with it (you don't need to get a separate subscription to manage users). 
 
Yes, Azure AD remains if subscription expires.

# 1. AWS IAM
- Step1: Geting AWS account (root user), But it is not recommended to use at creating AWS resouces.
- Step2: Create IAM user instead of using the AWS root account, But it is not recommended to share among coworkers.
- Step3: Create IAM policy which can access the services you want to use. 
- Step4: Attach the IAM policy to the IAM user.

For example, if you create a policy called "MyS3FullAccess" that grants all operations to S3 (of the AWS storage service) and attach it to the IAM user Bob, Bob will do everything to S3. But this might not be secure because IAM user's access key and secret will be used in the EC2 instances. The secret of IAM user account might be used as minings.

IAM role is a good solution for it. When you want to create the EC2 instance which sends Emails with AWS SES, then
- Create the IAM role (role type should be "Amazon EC2 role")
- Attach the access grant of "AmazonSESFullAccess" to it.
- Attach the role to the EC2 instance you created. But note that it is not possible to attach it to existed Instances. Then, you might recreate instance again.
- Run the programs with AWS SES on the EC2 instance.

# 2. Azure Active Directory
- Step1: Create a new tenant which is an instance in Azure Active Directory at Azure portal console.
  - https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-access-create-new-tenant
    - Please note when you create a new Azure AD tenant, you become the first user of that tenant. As the first user, you're automatically assigned the Global Admin role.
  - You may use an existing Azure AD tenant like your organization's Office 365 instead of creating it.

- Step2: Change or add additional domain names / Add users / Add groups and members in the tenant, if you want.
  - https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview
```
Tips:
It's Free up to this point if you use Azure Free Edition. 
But it costs money if you start to use a subscription.
```

- Step3: Get a subscription which is in accordance with your purposes. (See also #2-1)

- Step4: Attach the subscription to a single Azure AD tenant. 
  - https://www.youtube.com/watch?v=-rudwyS1KNA
  - You can not make any resources such as storage accounts on the Azure AD tenant if you don't associate Azure AD tenant with some specific subscriptions. You might find messages like "You are currently signed into the xxx directory which does not have any subscriptions."
  - You must be a owner of the subscription if you are going to associate the subscription with a Azure AD tenant.
  - One Azure AD tenant can be associated with sevral subscriptions. Example, Prod/Dev subscriptions and IT/HR subscriptions can be associated with a Azure AD tenant. It is convienent to manage security roles and billings.

- Step5: Manage Role-Based Access Control (RBAC)
  - Assign an RBAC role to a Azure AD user to control Virtual Machine or storage account.
     - Top three RBAC roles are Owner, Contributer and Reader.
  - You can also assign an RBAC role to a Managed ID which resides in virtual machine. (See also #2-2)
     - https://docs.microsoft.com/en-us/learn/modules/implement-managed-identities/
     - https://docs.microsoft.com/en-us/learn/modules/authenticate-apps-with-managed-identities/

# 2-1. Subscription
Subscription is a logical container that Microsoft uses to maintain their billing relationship with the Azure users. The billing relationship starts and stops at the subscription boundary.
- https://www.youtube.com/watch?v=LMAC0IIYSJM

# 2-2. Managed ID
Manage ID is a locally running internal endpoint (http://169.254.169.254/metadata/identity/oauth2/token) which resides in virtual machine. This endpoint is a micro web service running on that virtual machine. And it is only acceptable from within that virtual machine. So on your locally running code can actually request tokens from it. Your code just send a token request with no credentials to this endpoint. The life cycle of Managed ID is tied to that resouce so if you delete that virtual machine the ID will be also deleted. You don't need to put credentials on your code inside because platform manages the credentials. So, it is very secure.
- https://www.youtube.com/watch?v=sA_mXKy_dKU
- https://www.youtube.com/watch?v=vYUKC0mZFqI

```
Tips:
You might create the role which allows to access the blob storage in mystorageaccount20220103 (target azure resource). 
And attach the role to user-assigned Managed ID you already created in the service of "Managed Identity".
The application on the source azure resource such as virtual machine attached the role thru managed ID can access to the blob.
```
| user-assigned Managed ID | Role | Scope |
| --- | --- | --- |
| AzDemoUA | Storage Blob Data Contributor | mystorageaccount20220103 |


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
See also https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/how-to-use-vm-token

# 2-3. Scope
When you assign roles, you must specify a scope. Scope is the set of resources the access applies to. ie, It is a definition of "where it can be done?".
- https://www.youtube.com/watch?v=4v7ffXxOnwU
