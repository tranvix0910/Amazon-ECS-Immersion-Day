---
title : "Enable ECS Exec"
date :  2025-01-01
weight : 5
chapter : false
pre : " <b> 3.5. </b> "  
---

In this section we will enable ECS Exec feature to be able to run commands or open shell directly into containers running on EC2 instances or Fargate.

Enabling ECS Exec brings many benefits for operational management and is especially good for security. This feature allows controlled access to containers running in ECS tasks, helping to troubleshoot safely, with audit, without needing SSH into the host.

By leveraging IAM policies and IAM roles, you can tightly control who is allowed to execute commands inside containers, thereby enhancing the overall security posture of the system.

Additionally, all commands executed through ECS Exec are logged to CloudWatch, helping to create an audit trail for monitoring and compliance purposes.

#### Set IAM Role for User

{{% notice note %}}
This is the step to set permissions for User to be able to use ECS Exec. If following this lab, we have configured User with AdministratorAccess permissions, so this step can be skipped.
{{% /notice %}}

Proceed to create Policy with the following permissions:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecs:ExecuteCommand",
                "ecs:DescribeTasks"
            ],
            "Resource": [
                "arn:aws:ecs:${AWS_REGION}:${ACCOUNT_ID}:task/retail-store-ecs-cluster/*",
                "arn:aws:ecs:${AWS_REGION}:${ACCOUNT_ID}:cluster/*"
            ]
        }
    ]
}
```

Add the above json to file `ecs-exec-command-policy.json` and run the following command:

```bash
aws iam put-role-policy --role-name $(aws sts get-caller-identity --query 'Arn' | cut -d'/' -f2) --policy-name AmazonECSExecCommand --policy-document file://ecs-exec-command-policy.json
```

#### Set IAM Role for ECS Task Role

ECS Exec requires task IAM role to communicate with AWS Systems Manager (SSM).

```json
{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
            "Action": [
                "ssmmessages:CreateControlChannel",
                "ssmmessages:CreateDataChannel",
                "ssmmessages:OpenControlChannel",
                "ssmmessages:OpenDataChannel"
            ],
            "Resource": "*"
        }
    ]
}
```

Add the above json to file `ecs-exec-task-role-policy.json` and run the following command:

```bash
aws iam put-role-policy --role-name retailStoreEcsTaskRole --policy-name AmazonECSExecTaskRolePolicy --policy-document file://ecs-exec-task-role-policy.json
```

Check the Policy of the role just created

```bash
aws iam list-role-policies --role-name retailStoreEcsTaskRole --query 'PolicyNames[0]' --output text
```

![Check Policy of the role just created](/images/3-prepare/3.5-enable_ECS_exec/1.png)

Check role permissions with the following command:

```bash
aws iam get-role-policy --role-name retailStoreEcsTaskRole --policy-name AmazonECSExecTaskRolePolicy
```

![Check permissions of the role just created](/images/3-prepare/3.5-enable_ECS_exec/2.png)

#### Prepare environment

Install necessary tools:

- AWS CLI.
- Session Manager plugin for AWS CLI.

We have already installed AWS CLI in the [Create User and Create Access Key](/3-prepare/3.1-user_accesskey/) section. Next we will install Session Manager plugin for AWS CLI.

- Linux:

```bash
curl "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/ubuntu_64bit/session-manager-plugin.deb" -o "session-manager-plugin.deb"

sudo dpkg -i session-manager-plugin.deb
```

- MacOS:

```bash
curl "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/mac/session-manager-plugin.pkg" -o "session-manager-plugin.pkg"

sudo installer -pkg session-manager-plugin.pkg -target /
sudo ln -s /usr/local/sessionmanagerplugin/bin/session-manager-plugin /usr/local/bin/session-manager-plugin
```

- Windows:

```
https://s3.amazonaws.com/session-manager-downloads/plugin/latest/windows/SessionManagerPluginSetup.exe
```

Run the following command to check Session Manager plugin has been installed successfully:

```bash
session-manager-plugin
```

![Check Session Manager plugin has been installed successfully](/images/3-prepare/3.5-enable_ECS_exec/3.png)


#### Enable ECS Exec on Service

Update UI Service to enable ECS Exec by using the `--enable-execute-command` flag:

```bash
aws ecs update-service \
    --cluster retail-store-ecs-cluster \
    --service ui \
    --task-definition retail-store-ecs-ui \
    --enable-execute-command --force-new-deployment
```

![Update UI Service to enable ECS Exec](/images/3-prepare/3.5-enable_ECS_exec/4.png)

#### Check ARN of ECS task with ECS Exec enabled

Run the following command to select a running UI task with `enableExecuteCommand = true`:

```bash
ECS_EXEC_TASK_ARN=$(aws ecs list-tasks --cluster retail-store-ecs-cluster \
    --service-name ui --query 'taskArns[]' --output text | \
    xargs -n1 aws ecs describe-tasks --cluster retail-store-ecs-cluster --tasks | \
    jq -r '.tasks[] | select(.enableExecuteCommand == true) | .taskArn' | \
    head -n 1)
```

Check the result by running the `echo` command:

```bash
echo $ECS_EXEC_TASK_ARN
```

![Check ARN of ECS task with ECS Exec enabled](/images/3-prepare/3.5-enable_ECS_exec/5.png)

#### Connect to ECS Task

Proceed to connect to ECS Task by running the following command:

```bash
if [ -z "${ECS_EXEC_TASK_ARN}" ]; then
    echo "ECS_EXEC_TASK_ARN is not correctly configured!"
else

aws ecs execute-command \
    --cluster retail-store-ecs-cluster \
    --task $ECS_EXEC_TASK_ARN \
    --container application \
    --interactive \
    --command "/bin/bash"
fi
```

We will see Output as below:

```bash
The Session Manager plugin was installed successfully. Use the AWS CLI to start a session.

Starting session with SessionId: ecs-execute-command-vvdysulqbcz2txr2d262sw2s64
bash-5.2#

```

![Connect to ECS Task](/images/3-prepare/3.5-enable_ECS_exec/6.png)




