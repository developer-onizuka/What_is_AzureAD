# What is AzureAD ?

AWS IAM can be classified as a tool in the "Cloud Access Management" category, while Azure Active Directory is grouped under "Password Management".

# 1. AWS IAM
- Step1: Geting AWS account (root user), But it is not recommended to use at creating AWS resouces.
- Step2: Create IAM user instead of using the AWS root account, But it is not recommended to share among coworkers.
- Step3: Create IAM policy which can access the services you want to use. 
- Step4: Attach the IAM policy to the IAM user.

For example, if you create a policy called "MyS3FullAccess" that grants all operations to S3 (of the AWS storage service) and attach it to the IAM user Bob, Bob will do everything to S3.

# 2. Azure Active Directory



