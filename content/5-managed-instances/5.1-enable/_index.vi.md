---
title : "Enable ECS Managed Instances"
date :  2025-01-01
weight : 1
chapter : false
pre : " <b> 5.1. </b> "  
---

Trong phần này, bạn sẽ thực hiện thiết lập để triển khai workload trên Amazon ECS Managed Instances.

Cụ thể, chúng ta sẽ thiết lập **Amazon ECS capacity providers** mặc định cho **ECS Managed Instances**.

Amazon ECS capacity providers chịu trách nhiệm quản lý việc scale hạ tầng cho các task trong ECS cluster.

Amazon ECS hiện cung cấp ba loại capacity provider cho cluster:

- **Fargate capacity providers**: Đây là các capacity provider được định nghĩa sẵn cho AWS Fargate. Đây cũng là capacity provider mà chúng ta đã sử dụng từ đầu workshop.

- **Managed Instances capacity providers**: Cho phép sử dụng Amazon ECS Managed Instances để chạy workload.
    - 👉 Đây là capacity provider sẽ được cấu hình trong phần này.

- **Auto Scaling group capacity providers**: Cho phép sử dụng Amazon EC2 instances để chạy workload thông qua Auto Scaling Group.

![ECS Capacity Providers](/images/5-managed-instances/5.1-enable/1.png)

**Instance Profile** là một đối tượng IAM được gắn trực tiếp vào EC2 instance, đóng vai trò cầu nối giữa EC2 và IAM Role.

Thông qua Instance Profile, EC2 (và các process/daemon chạy trên EC2 như ECS Agent) có thể assume IAM Role tương ứng và sử dụng các quyền AWS đã được cấp mà không cần cấu hình access key/secret key.

#### Tạo ECS Managed Instances Capacity Provider

Trước khi tạo capacity provider, chúng ta cần hai IAM role (tham khảo [AWS Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ManagedInstances.html#managed-instances-iam-roles) để biết thêm chi tiết):

- **Infrastructure Role**: Cho phép Amazon ECS quản lý ECS Managed Instances thay mặt bạn.

- **Instance profile**: Cung cấp quyền cho Amazon ECS container agent chạy trên các managed instances.

Chúng ta sẽ tạo hai IAM role này bằng cách sử dụng AWS CLI. Với **Infrastructure Role** chúng ta sẽ tiến hành tạo role với tên là `ecsManagedInstanceInfrastructureRole`. Đầu tiên chúng ta cần tạo trust policy cho role này với nội dung file json bên dưới:

{{% notice note %}}
ECS Agent là một daemon (process chạy nền) trên EC2 instance.
Agent này sử dụng Instance Profile gắn với EC2 để assume IAM Role tương ứng, từ đó có thể thực thi các quyền AWS đã được cấp (ví dụ: gọi ECS API, pull image từ ECR, gửi log lên CloudWatch).
{{% /notice %}}


```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ecs.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

Tiến hành chạy lệnh sau để tạo Role:

```bash
aws iam create-role \
  --role-name ecsManagedInstanceInfrastructureRole \
  --assume-role-policy-document file://ecs-managed-instance-infrastructure-role-trust.json
```

![Tạo Infrastructure Role](/images/5-managed-instances/5.1-enable/2.png)

Tiếp theo chúng ta cần Attach policy AWS-managed vào role vừa tạo:

- Policy 1 - Quản lý Managed Instances:

```bash
aws iam attach-role-policy \
  --role-name ecsManagedInstanceInfrastructureRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonECSInfrastructureRolePolicyForManagedInstances
```

- Policy 2 - Quản lý Volume (EBS):

```bash
aws iam attach-role-policy \
  --role-name ecsManagedInstanceInfrastructureRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSInfrastructureRolePolicyForVolumes
```

![Attach Policy](/images/5-managed-instances/5.1-enable/3.png)

Kiểm tra lại Role đã được gắn policy chưa bằng lệnh:

```bash
aws iam list-attached-role-policies \
  --role-name ecsManagedInstanceInfrastructureRole
```

![List Attached Role Policies](/images/5-managed-instances/5.1-enable/4.png)

Như vậy là đã tạo role Infrastructure Role thành công.

Tiếp theo chúng ta sẽ tạo Instance Role bằng cách tạo tương tự như trên với file json là `ecs-managed-instance-role-trust.json` với nội dung file json bên dưới:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Chạy lệnh sau để tạo Role:

```bash
aws iam create-role \
  --role-name ecsInstanceRoleManagedInstance \
  --assume-role-policy-document file://ecs-managed-instance-role-trust.json
```

![Tạo Instance Role](/images/5-managed-instances/5.1-enable/5.png)

Tiến hành gán Policy ECS Agent:

```bash
aws iam attach-role-policy \
  --role-name ecsInstanceRoleManagedInstance \
  --policy-arn arn:aws:iam::aws:policy/AmazonECSInstanceRolePolicyForManagedInstances
```

![Attach Policy ECS Agent](/images/5-managed-instances/5.1-enable/6.png)

Kiểm tra lại Role:

```bash
aws iam list-attached-role-policies \
  --role-name ecsInstanceRoleManagedInstance
```

![List Attached Role Policies](/images/5-managed-instances/5.1-enable/7.png)

#### Tạo Instance Profile

Tiến hành tạo Instance Profile với Instance Role vừa tạo:

```bash
aws iam create-instance-profile --instance-profile-name ecsInstanceRoleManagedInstanceProfile
```

![Tạo Instance Profile](/images/5-managed-instances/5.1-enable/8.png)

Tiến hành gán Role vào Instance Profile:

```bash
aws iam add-role-to-instance-profile \
  --instance-profile-name ecsInstanceRoleManagedInstanceProfile \
  --role-name ecsInstanceRoleManagedInstance
```

![Gán Role vào Instance Profile](/images/5-managed-instances/5.1-enable/9.png)

Kiểm tra lại Instance Profile:

```bash
aws iam list-instance-profiles
```

![List Instance Profiles](/images/5-managed-instances/5.1-enable/10.png)

Như vậy là đã tạo Instance Profile thành công.

Cuối cùng, hãy tạo **ECS Managed Instances capacity provider**. Xem [AWS Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/managed-instances-capacity-providers-concept.html) để biết thêm thông tin.

Capacity provider bên dưới sẽ cung cấp các instance chạy trong 2 private subnets, sử dụng **UI_SG_ID security group**, và có disk size 100GB. Xem [AWS Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-capacity-provider-managed-instances.html) để biết thêm ví dụ về cách cấu hình capacity provider cho Amazon ECS Managed Instances.

Chúng ta sẽ tạo capacity provider với nội dung file json bên dưới với tên là `retail-store-managed-instances-cp.json`:

```json
{
    "name": "retail-store-managed-instances-cp",
    "cluster": "retail-store-ecs-cluster",
    "managedInstancesProvider": {
        "infrastructureRoleArn": "arn:aws:iam::${ACCOUNT_ID}:role/ecsManagedInstanceInfrastructureRole",
        "instanceLaunchTemplate": {
            "ec2InstanceProfileArn": "arn:aws:iam::${ACCOUNT_ID}:instance-profile/ecsInstanceRoleManagedInstanceProfile",
            "networkConfiguration": {
                "subnets": [
                    "$PRIVATE_SUBNET1",
                    "$PRIVATE_SUBNET2"
                ],
                "securityGroups": [
                    "${UI_SG_ID}"
                ]
            },
            "storageConfiguration": {
                "storageSizeGiB": 100
            }
        }
    }
}
```

Sử dụng CLI sau để tạo capacity provider:

```bash
aws ecs create-capacity-provider --cli-input-json file://retail-store-managed-instances-cp.json
```

![Tạo Capacity Provider](/images/5-managed-instances/5.1-enable/11.png)

Kiểm tra lại capacity provider trên AWS Console:

![Kiểm tra Capacity Provider](/images/5-managed-instances/5.1-enable/12.png)

**ECS Capacity Provider Details** được hiển thị như sau:

![Kiểm tra Capacity Provider](/images/5-managed-instances/5.1-enable/13.png)

ECS Managed Instances capacity provider mới của mình với các đặc điểm chính sau:

- **Type**: Managed Instances.

- **Infrastructure Role**: Được cấu hình cho việc quản lý instance tự động.

- **Instance Profile**: Cho phép các instance tham gia ECS cluster.

- **Network Configuration**: Sử dụng các private subnets hiện có của bạn.

- **Storage**: 100 GiB EBS storage cho mỗi instance.

- **Monitoring**: Bật CloudWatch monitoring cơ bản.

Sau khi đã tạo ECS Managed Instances capacity provider, hãy tiếp tục bước tiếp theo để triển khai ứng dụng bằng tùy chọn compute mới này.

