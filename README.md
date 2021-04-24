# aws-send-command
Send commands synchronously using this shell wrapper around the AWS CLI's [ssm send-command](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ssm/send-command.html).


AWS Systems Manager lets you execute shell commands on your Linux or Windows managed instances.
A managed instance is any Amazon Elastic Compute Cloud instance (EC2 instance), or any on-premises
server or virtual machine (VM) in your hybrid environment that has been configured for Systems Manager.


`aws-send-command` wraps serveral AWS Systems Manager commands to provide sychronous execution of commands and
retrieval of `stdout`, `stderr` and (for Linux) the exit status.


## Usage
The execution of `aws-send-command`  requires:
- The [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide) to be installed.
- AWS credentials to be configured in the shell environment.
The first example invokes AWS commands to locate and instance by tag, thens checks the connection status using a Systems Manager command.


Usage:
```sh
$ ./aws-send-command <instance id> <commands> [document name (default: AWS-RunShellScript)]
```

## Examples

Example - Shell set-up for AWS:
```sh
$ my_instance_tag='repository'
$ instance_id=$(aws ec2 describe-instances \
   --filters "Name=tag:Name,Values=${my_instance_tag?}" 'Name=instance-state-name,Values=running' \
   --query 'Reservations[0].Instances[0].InstanceId' --output text)
$ aws ssm get-connection-status \
   --target "${instance_id?}" \
   --query "Status" --output text
connected
$ 
```

Example - Remote uname to Linux:
```sh
$ ./aws-send-command "${instance_id?}" '"#!/usr/bin/env sh","uname"'
Linux
$ 
```

Example - Remote unknown command with non-zero exit status:
```sh
$ ./aws-send-command "${instance_id?}" '"#!/usr/bin/env sh","Xuname"'
/var/lib/amazon/ssm/i-0bdf0b8c4b56adbe3/document/orchestration/2ec34c23-b446-4b8c-8550-559777aba979/awsrunShellScript/0.awsrunShellScript/_script.sh: 2: Xuname: not found
run commands: exit status 127
$ echo $?
127
$ 
```

Example - Remote OS information to Windows:
```sh
$ my_instance_tag='windows'
$ instance_id=$(aws ec2 describe-instances \
   --filters "Name=tag:Name,Values=${my_instance_tag?}" 'Name=instance-state-name,Values=running' \
   --query 'Reservations[0].Instances[0].InstanceId' --output text)
$ aws ssm get-connection-status \
   --target "${instance_id?}" \
   --query "Status" --output text
connected
$ ./aws-send-command "${instance_id?}" '"Get-CimInstance Win32_OperatingSystem | Select-Object Caption"' \
	'AWS-RunPowerShellScript'

Server 2019 Datacenter

$ 
```
(PowerShell script errors do not bubble up to Linux shell exit status.)

## License
```text
MIT License:

Copyright (c) 2021 Antony Cartwright

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
