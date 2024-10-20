# Box has an assigned ROLE
## Use STS to get access tokens
```
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
```
```
INSTANCE_ID=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id)
```
## Find the network interface
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
### This will fail -- no access to EIP
```
ALLOCATE_ID=$(aws ec2 allocate-address --domain vpc)
```
### This will fail -- no access to EIP
```
aws ec2 associate-address \
    --allocation-id $ALLOCATE_ID \
    --network-interface-id $ENI_ID \
    --private-ip-address <secondary-ip-address>


```
