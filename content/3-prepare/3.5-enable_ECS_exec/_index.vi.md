---
title : "Enable ECS Exec"
date :  2025-01-01
weight : 5
chapter : false
pre : " <b> 3.5. </b> "  
---

Trong phần này chúng ta sẽ kích hoạt tính năng ECS Exec để có thể chạy lệnh hoặc mở shell trực tiếp vào container đang chạy trên EC2 instance hoặc Fargate.

Việc bật ECS Exec mang lại nhiều lợi ích cho vận hành hệ thống (operational management) và đặc biệt tốt cho bảo mật. Tính năng này cho phép truy cập có kiểm soát vào container đang chạy trong ECS task, giúp troubleshoot một cách an toàn, có audit, mà không cần SSH vào host.

Bằng cách tận dụng IAM policies và IAM roles, có thể kiểm soát chặt chẽ ai được phép thực thi lệnh bên trong container, từ đó nâng cao tư thế bảo mật tổng thể (security posture) của hệ thống.

Ngoài ra, mọi lệnh được thực thi thông qua ECS Exec đều được ghi log vào CloudWatch, giúp tạo audit trail phục vụ mục đích giám sát và tuân thủ (compliance).

#### Set IAM Role cho User

{{% notice note %}}
Đây là bước thiết lập quyền cho User để có thể sử dụng ECS Exec. Nếu thực hiện theo bài lab này thì chúng ta đã cấu hình User với quyền AdministratorAccess, nên có thể bỏ qua bước này.
{{% /notice %}}

Tiến hành Policy với các quyền sau:

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

Thêm json trên vào file `ecs-exec-command-policy.json` và chạy lệnh sau:

```bash
aws iam put-role-policy --role-name $(aws sts get-caller-identity --query 'Arn' | cut -d'/' -f2) --policy-name AmazonECSExecCommand --policy-document file://ecs-exec-command-policy.json
```

#### Set IAM Role cho ECS Task Role

ECS Exec yêu cầu task IAM role để giao tiếp với AWS Systems Manager (SSM).

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

Thêm json trên vào file `ecs-exec-task-role-policy.json` và chạy lệnh sau:

```bash
aws iam put-role-policy --role-name retailStoreEcsTaskRole --policy-name AmazonECSExecTaskRolePolicy --policy-document file://ecs-exec-task-role-policy.json
```

Kiểm tra lại Policy của role vừa tạo

```bash
aws iam list-role-policies --role-name retailStoreEcsTaskRole --query 'PolicyNames[0]' --output text
```

![Kiểm tra Policy của role vừa tạo](/images/3-prepare/3.5-enable_ECS_exec/1.png)

Kiểm tra quyền của role bằng lệnh sau:

```bash
aws iam get-role-policy --role-name retailStoreEcsTaskRole --policy-name AmazonECSExecTaskRolePolicy
```

![Kiểm tra quyền của role vừa tạo](/images/3-prepare/3.5-enable_ECS_exec/2.png)

#### Chuẩn bị môi trường

Cài đặt các công cụ cần thiết:

- AWS CLI.
- Session Manager plugin cho AWS CLI.

Chúng ta đã thực hiện cài đặt AWS CLI ở phần [Tạo User và tạo Access Key](/vi/3-prepare/3.1-user_accesskey/). Tiếp theo chúng ta sẽ cài đặt Session Manager plugin cho AWS CLI.

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

Chạy lệnh sau để kiểm tra Session Manager plugin đã được cài đặt thành công:

```bash
session-manager-plugin
```

![Kiểm tra Session Manager plugin đã được cài đặt thành công](/images/3-prepare/3.5-enable_ECS_exec/3.png)


#### Enable ECS Exec trên Service

Cập nhật UI Service để bật ECS Exec bằng cách sử dụng flag `--enable-execute-command`:

```bash
aws ecs update-service \
    --cluster retail-store-ecs-cluster \
    --service ui \
    --task-definition retail-store-ecs-ui \
    --enable-execute-command --force-new-deployment
```

![Cập nhật UI Service để bật ECS Exec](/images/3-prepare/3.5-enable_ECS_exec/4.png)

#### Kiểm tra ARN của ECS task có bật ECS Exec

Chạy lệnh sau để chọn một UI task đang chạy và có `enableExecuteCommand = true`:

```bash
ECS_EXEC_TASK_ARN=$(aws ecs list-tasks --cluster retail-store-ecs-cluster \
    --service-name ui --query 'taskArns[]' --output text | \
    xargs -n1 aws ecs describe-tasks --cluster retail-store-ecs-cluster --tasks | \
    jq -r '.tasks[] | select(.enableExecuteCommand == true) | .taskArn' | \
    head -n 1)
```

Kiểm tra kết quả bằng cách chạy lệnh `echo`:

```bash
echo $ECS_EXEC_TASK_ARN
```

![Kiểm tra ARN của ECS task có bật ECS Exec](/images/3-prepare/3.5-enable_ECS_exec/5.png)

#### Kết nối tới ECS Task

Tiến hành kết nối tới ECS Task bằng cách chạy lệnh sau:

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

Chúng ta sẽ thấy Output như bên dưới:

```bash
The Session Manager plugin was installed successfully. Use the AWS CLI to start a session.

Starting session with SessionId: ecs-execute-command-vvdysulqbcz2txr2d262sw2s64
bash-5.2#

```

![Kết nối tới ECS Task](/images/3-prepare/3.5-enable_ECS_exec/6.png)







