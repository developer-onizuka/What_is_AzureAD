# What is the difference between AWS IAM and Azure AD ?

 AWS IAM can be classified as a tool in the "Cloud Access Management" category, while Azure Active Directory is grouped under "Password Management".
 
 For AWS, users are managed within their AWS account, while for Azure, users are managed outside of the subscription. In the case of AWS, it is common to create an AWS account that stores only IAM users separately from the AWS account that holds resources, and assign permissions from the AWS account that holds resources. In Azure, on the other hand, it's the Azure Active Directory that manages users, so you need to associate a subscription with it (you don't need to get a separate subscription to manage users).

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
- Step1: Create a new tenant (Azure AD's instance) in Azure Active Directory in Azure portal 
  - (or you may use an existing Azure AD tenant like your organization's Office 365)
  - https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-access-create-new-tenant
    - Please note when you create a new Azure AD tenant, you become the first user of that tenant. As the first user, you're automatically assigned the Global Admin role.
- Step2: Change or add additional domain names / Add users / Add groups and members in the tenant
- Step3: Get any subscriptions and attach a single Azure AD tenant to it when signing up for the subscription. (https://www.youtube.com/watch?v=-rudwyS1KNA)

# 2-1. What is Subscription:
- https://www.youtube.com/watch?v=LMAC0IIYSJM

