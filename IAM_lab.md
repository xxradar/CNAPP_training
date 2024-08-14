Assuming a role in AWS CLI allows you to temporarily gain access to AWS resources that are allowed under a different role, which is useful for managing cross-account access or following the principle of least privilege. Here's a step-by-step guide on how to assume a role using the AWS CLI:
### Step 1: Configure Your AWS CLI

Ensure that your AWS CLI is configured with your IAM user credentials.

```bash
aws configure
```
You'll be prompted to enter your AWS Access Key ID, AWS Secret Access Key, Default region name, and Default output format.
### Step 2: Assume the Role Using sts assume-role

To assume a role, use the aws sts assume-role command. You'll need the ARN (Amazon Resource Name) of the role you want to assume, and optionally, a session name.

```bash
aws sts assume-role \
    --role-arn arn:aws:iam::123456789012:role/RoleName \
    --role-session-name SessionName
```
Replace arn:aws:iam::123456789012:role/RoleName with the actual ARN of the role you want to assume, and SessionName with a name for the session (this can be any string).

This command will return a JSON object containing temporary security credentials (AccessKeyId, SecretAccessKey, SessionToken) and an expiration time.
### Step 3: Export the Temporary Credentials

After you get the temporary credentials from the assume-role command, you need to set them as environment variables so that the AWS CLI uses them for subsequent commands.

```bash

export AWS_ACCESS_KEY_ID=<AccessKeyId>
export AWS_SECRET_ACCESS_KEY=<SecretAccessKey>
export AWS_SESSION_TOKEN=<SessionToken>
```
Replace <AccessKeyId>, <SecretAccessKey>, and <SessionToken> with the corresponding values from the JSON output of the assume-role command.
### Step 4: Use the Assumed Role

Now that the temporary credentials are set, you can use the AWS CLI to perform actions with the assumed role's permissions.

For example:

```bash
aws s3 ls
```
This command will list the S3 buckets that the assumed role has access to.
### Step 5: Revert to Your Original Credentials (Optional)

Once you're done using the assumed role, you can unset the temporary credentials and revert to your original IAM user credentials.

```bash
unset AWS_ACCESS_KEY_ID
unset AWS_SECRET_ACCESS_KEY
unset AWS_SESSION_TOKEN
```
Or you can simply close the terminal session to clear the environment variables.
Example Script

### Here’s a simple bash script that automates the process:

```bash

#!/bin/bash

# Assume a role
ROLE_ARN="arn:aws:iam::123456789012:role/RoleName"
SESSION_NAME="MySession"

CREDENTIALS=$(aws sts assume-role --role-arn $ROLE_ARN --role-session-name $SESSION_NAME --query "Credentials" --output json)

export AWS_ACCESS_KEY_ID=$(echo $CREDENTIALS | jq -r .AccessKeyId)
export AWS_SECRET_ACCESS_KEY=$(echo $CREDENTIALS | jq -r .SecretAccessKey)
export AWS_SESSION_TOKEN=$(echo $CREDENTIALS | jq -r .SessionToken)

# Now you can use AWS CLI with the assumed role

# Unset the environment variables when done
unset AWS_ACCESS_KEY_ID
unset AWS_SECRET_ACCESS_KEY
unset AWS_SESSION_TOKEN
```

This script uses jq to parse the JSON output. Make sure you have jq installed (sudo apt-get install jq on Debian-based systems).

This is the basic method to assume a role in AWS CLI. If you need more complex setups (e.g., MFA, external IDs), the commands can be expanded to include those parameters.
