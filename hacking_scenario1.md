# Box has an assigned ROLE
### Use STS to get access tokens
```
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
```
```
INSTANCE_ID=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id)
```
### Find the network interface and assign an additional IP address
```
ENI_ID=$(aws ec2 describe-instances \
    --instance-ids $INSTANCE_ID \
    --query "Reservations[].Instances[].NetworkInterfaces[].NetworkInterfaceId" \
    --output text)
```
```
aws ec2 assign-private-ip-addresses \
    --network-interface-id $ENI_ID \
    --secondary-private-ip-address-count 1 \
    --allow-reassignment
```
or 
```
aws ec2 assign-private-ip-addresses \
    --network-interface-id $ENI_ID \
    --private-ip-addresses "10.1.1.221" \
    --allow-reassignment
```
### This will fail -- no priviledge to allocate EIP
```
ALLOCATE_ID=$(aws ec2 allocate-address --domain vpc)
```
```
aws ec2 associate-address \
    --allocation-id $ALLOCATE_ID \
    --network-interface-id $ENI_ID\
    --private-ip-address "10.1.1.221"
```
### This will not fail
```
aws ec2 associate-address \
    --allocation-id eipalloc-0130a4659a3849f09 \
    --network-interface-id $ENI_ID \
    --private-ip-address "10.1.1.221"
```

### Trying to figure out your role
```
ROLE_NAME=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/iam/info | jq -r '.InstanceProfileArn' | cut -d '/' -f 2)
echo $ROLE_NAME
```
Things to try:<br>
Attached Roles:
```
aws iam list-attached-role-policies --role-name $ROLE_NAME
```
Inline policies:
```
aws iam list-role-policies --role-name $ROLE_NAME
```
Try to simulate
```
aws iam get-policy --policy-arn <policy-arn>
aws iam get-role-policy --role-name $ROLE_NAME --policy-name <policy-name>
```
Bruteforcing ...
```
aws iam simulate-principal-policy --policy-source-arn $ROLE_ARN --action-names "s3:ListBucket" "ec2:DescribeInstances"
```
```
# Check if you can list S3 buckets
aws s3 ls

# Check if you can describe EC2 instances
aws ec2 describe-instances
```
## Policies for reference
```
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Action": [
				"ec2:Describe*",
				"ec2:AssociateAddress",
				"ec2:AssignPrivateIpAddresses",
				"ec2:UnassignPrivateIpAddresses",
				"ec2:ReplaceRoute"
			],
			"Effect": "Allow",
			"Resource": "*"
		},
		{
			"Sid": "Statement1",
			"Effect": "Allow",
			"Action": [],
			"Resource": []
		}
	]
}
```
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "",
            "Effect": "Allow",
            "Principal": {
                "Service": "ec2.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```
