### 1. IAM: Users & Groups

Identity and Access Management

- **Global** service
- **Root account** created by default, shouldn’t be used or shared
- **Users** are people within your organization, and can be grouped
- **Groups** only contain users, not other groups
- Users don’t have to belong to a group, and user can belong to multiple groups

### 2. IAM: Permissions

- **Users or Groups** can be assigned JSON documents called policies
- These policies define the **permissions** of the users
- In AWS you apply the **least privilege principle**: don't give more permissions than a user needs

### 3. IAM Policies Structure
<center><img src="iam.png" style="zoom:75%"/></center>

Version: policy language version, always include `"2012-10-17"`
Id: an identifier for the policy (optional)
Statement: one or more individual statements (required)

- Sid: an identifier for the statement (optional) 
- Effect: whether the statement allows or denies access (Allow, Deny)
- Principal: account/user/role to which this policy applied to
- Action: list of actions this policy allows or denies
- Resource: list of resources to which the actions applied to
- Condition: conditions for when this policy is in effect (optional)

[使用标签控制对 IAM 用户和角色的访问以及他们进行的访问](https://docs.aws.amazon.com/zh_cn/IAM/latest/UserGuide/access_iam-tags.html)

### 4. How can users access AWS ?

- **AWS Management Console** (protected by password + MFA)
- **[AWS Command Line Interface (CLI)](https://github.com/aws/aws-cli)**: protected by access keys (is built on AWS SDK for Python)
- **AWS Software Developer Kit (SDK)** - for code: protected by access keys

### 5. IAM Security Tools
IAM Credentials Report (account-level):

- a report that lists all your account's users and the status of their various credentials

IAM Access Advisor (user-level):
- Access advisor shows the service permissions granted to a user and when those services were last accessed
- You can use this information to revise your policies

### 6. IAM Guidelines & Best Practices

- Don’t use the root account except for AWS account setup
- One physical user = One AWS **user**
- **Assign users to groups** and assign permissions to groups
- Create a **strong password policy**
- Use and enforce the use of **Multi Factor Authentication (MFA)**
- Create and use **Roles** for giving permissions to AWS services
- Use Access Keys for Programmatic Access (CLI / SDK)
- Audit permissions of your account with the IAM Credentials Report
- **Never share IAM users & Access Keys**

### 7. Shared Responsibility Model for IAM

| <center>aws</center>                                         | <center>you</center>                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| - Infrastructure (global network security) <br>- Configuration and vulnerability analysis <br>- Compliance validation | - Users, Groups, Roles, Policies management and monitoring <br>- Enable MFA on all accounts <br>- Rotate all your keys often <br>- Use IAM tools to apply appropriate permissions <br>- Analyze access patterns & review permissions |