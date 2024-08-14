# 1. Obtain all the roles that a user or identity can assume in AWS
You generally need to perform a combination of the following steps:

### 1. List All IAM Roles

You can list all IAM roles in your account, but this doesn't tell you if the current user can assume them. Use the following command:

```bash

aws iam list-roles --query 'Roles[*].{RoleName:RoleName, Arn:Arn}'
```
This command will list all IAM roles and their ARNs in your account.
### 2. Check if a User Can Assume a Role

To determine if the current IAM user or role can assume another role, you need to check the role's trust policy, which specifies the entities that can assume the role. You can do this using the following steps:
a. Retrieve the Trust Policy of a Role

For each role you're interested in, retrieve its trust policy using the following command:

```bash

aws iam get-role --role-name <RoleName> --query 'Role.AssumeRolePolicyDocument'
```
This command returns the trust policy of the specified role. The trust policy contains the Principal element, which indicates which AWS accounts, users, roles, or services are allowed to assume the role.
b. Check the Principal Element

Look at the Principal element in the trust policy. If it matches the ARN of the user, role, or account you're working with, then your user can assume this role.
### 3. Automating the Process

If you want to automate the process of finding all roles a user can assume, you would need to script the following:

    List all roles.
    For each role, check its trust policy to see if it includes the current user, role, or account.
    Output the roles that the user can assume.

Here’s a simple example of a Python script using the AWS SDK (boto3) that automates this process:

```python

import boto3

# Initialize a session using your credentials
session = boto3.Session()

# Initialize the IAM client
iam_client = session.client('iam')

# Get the current user identity
sts_client = session.client('sts')
identity = sts_client.get_caller_identity()

# List all roles
roles = iam_client.list_roles()

# Check each role's trust policy
assumable_roles = []
for role in roles['Roles']:
    role_name = role['RoleName']
    role_arn = role['Arn']
    
    try:
        # Get the role's trust policy
        role_policy = iam_client.get_role(RoleName=role_name)
        assume_role_policy = role_policy['Role']['AssumeRolePolicyDocument']
        
        # Check if the current user/account is in the trust policy
        for statement in assume_role_policy['Statement']:
            if 'AWS' in statement['Principal']:
                allowed_entities = statement['Principal']['AWS']
                if isinstance(allowed_entities, str):
                    allowed_entities = [allowed_entities]
                
                if identity['Arn'] in allowed_entities or identity['Account'] in allowed_entities:
                    assumable_roles.append(role_name)
                    
    except Exception as e:
        print(f"Error processing role {role_name}: {e}")

print(f"Roles that can be assumed by {identity['Arn']}:")
for role in assumable_roles:
    print(role)
```
#### Explanation of the Script:
Session Initialization: The script starts by initializing a Boto3 session.
STS Client: The script uses the STS client to get the identity of the current user or role.
List Roles: It lists all roles using the list_roles API.
Check Trust Policy: For each role, it checks the trust policy to see if the current user, role, or account is allowed to assume the role.
Output: It prints out the list of roles that the user can assume.

#### Limitations:
This script checks only roles within the same AWS account. Cross-account role assumption would require additional considerations.
The process relies on the permissions of the user running the script. The user needs permission to list roles and retrieve their trust policies.

This approach should give you a list of all roles that the current user or role can assume, either directly or by checking trust policies.
