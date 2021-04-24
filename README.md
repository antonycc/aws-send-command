# aws-send-command
Send commands synchronously using this shell wrapper around the AWS CLI's ssm send-command. https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ssm/send-command.html

Usage:
```sh
$ ./aws-send-command <instance-id> <commands>
```

Example:
```sh
instance_id=$(aws ec2 describe-instances \
   --filters 'Name=tag:Name,Values=my-instance-tag' 'Name=instance-state-name,Values=running' \
   --query 'Reservations[0].Instances[0].InstanceId' --output text)
aws ssm get-connection-status \
   --target "${instance_id?}" \
   --query "Status" --output text
connected
./aws-send-command "${instance_id?}" '"#!/usr/bin/env sh","uname"'
Linux
```
