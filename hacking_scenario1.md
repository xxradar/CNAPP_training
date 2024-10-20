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
