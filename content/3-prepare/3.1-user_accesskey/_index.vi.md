---
title : "Tạo User & Access Key"
date :  2025-01-01
weight : 1
chapter : false
pre : " <b> 3.1. </b> "  
---

#### Cài đặt AWS CLI

- Linux:

```bash
sudo apt update
sudo apt install -y unzip curl
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
unzip awscliv2.zip
sudo ./aws/install
```

- MacOS:

```bash
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o AWSCLIV2.pkg
sudo installer -pkg AWSCLIV2.pkg -target /
```

- Windows:

```powershell
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```

#### Tạo User và tạo Access Key

Truy cập [AWS Console](https://console.aws.amazon.com/) và tạo mới một User với quyền AdministratorAccess.

Trên thanh tìm kiếm của AWS Console, nhập "IAM" và chọn "IAM".

![Tạo User và tạo Access Key](/images/3-prepare/3.1-user_accesskey/1.png)

Chọn **Users** trong menu bên trái.

Tiếp theo chọn **Add user** để tạo mới một User.

![Tạo User và tạo Access Key](/images/3-prepare/3.1-user_accesskey/2.png)

Nhập tên cho User: `ecs-workshop`

Chọn **Custom Password** để tạo mật khẩu cho User.

Chọn **Next**.

![Tạo User và tạo Access Key](/images/3-prepare/3.1-user_accesskey/3.png)

Cấu hình quyền cho User:

- **Attach policies directly**: Chọn **AdministratorAccess** để cấu hình quyền cho User.

![Tạo User và tạo Access Key](/images/3-prepare/3.1-user_accesskey/4.png)

Chọn **Next**.

![Tạo User và tạo Access Key](/images/3-prepare/3.1-user_accesskey/5.png)

Chọn **Create user**.

![Tạo User và tạo Access Key](/images/3-prepare/3.1-user_accesskey/6.png)

Như vậy User đã được tạo thành công.

![Tạo User và tạo Access Key](/images/3-prepare/3.1-user_accesskey/7.png)

Tiến hành quay lại IAM Console và chọn **Users** trong menu bên trái.

Chọn User vừa tạo.

![Tạo User và tạo Access Key](/images/3-prepare/3.1-user_accesskey/8.png)

Chọn **Security credentials** tab.

![Tạo User và tạo Access Key](/images/3-prepare/3.1-user_accesskey/9.png)

Chọn **Create access key**.

![Tạo User và tạo Access Key](/images/3-prepare/3.1-user_accesskey/10.png)

Chọn **Command Line Interface (CLI)**. Ấn xác nhận và qua bước tiếp theo

![Tạo User và tạo Access Key](/images/3-prepare/3.1-user_accesskey/11.png)

Description: `Amazon ECS Immersion Day` và tiến hành tạo access key.

![Tạo User và tạo Access Key](/images/3-prepare/3.1-user_accesskey/12.png)

Như vậy access key đã được tạo thành công và có thể ấn **Download .csv** để tải về file `credentials.csv`.

![Tạo User và tạo Access Key](/images/3-prepare/3.1-user_accesskey/13.png)

![Tạo User và tạo Access Key](/images/3-prepare/3.1-user_accesskey/14.png)

#### Cấu hình AWS CLI

Truy cập CLI để tiến hành cấu hình AWS CLI.

Kiểm tra version của AWS CLI.

```bash
aws --version
```

![Tạo User và tạo Access Key](/images/3-prepare/3.1-user_accesskey/15.png)

Cấu hình AWS CLI.

```bash
aws configure
```

![Tạo User và tạo Access Key](/images/3-prepare/3.1-user_accesskey/16.png)

Như vậy AWS CLI đã được cấu hình thành công.

Kiểm tra hoạt động của AWS CLI.

```bash
aws ec2 describe-instances
```

![Tạo User và tạo Access Key](/images/3-prepare/3.1-user_accesskey/17.png)

Như vậy AWS CLI đã được cấu hình thành công và hoạt động tốt.