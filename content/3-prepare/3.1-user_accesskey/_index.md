---
title : "Create User & Access Key"
date :  2025-01-01
weight : 1
chapter : false
pre : " <b> 3.1. </b> "  
---

#### Install AWS CLI

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

#### Create User and Create Access Key

Access [AWS Console](https://console.aws.amazon.com/) and create a new User with AdministratorAccess permissions.

In the AWS Console search bar, enter "IAM" and select "IAM".

![Create User and Create Access Key](/images/3-prepare/3.1-user_accesskey/1.png)

Select **Users** in the left menu.

Next select **Add user** to create a new User.

![Create User and Create Access Key](/images/3-prepare/3.1-user_accesskey/2.png)

Enter name for User: `ecs-workshop`

Select **Custom Password** to create password for User.

Select **Next**.

![Create User and Create Access Key](/images/3-prepare/3.1-user_accesskey/3.png)

Configure permissions for User:

- **Attach policies directly**: Select **AdministratorAccess** to configure permissions for User.

![Create User and Create Access Key](/images/3-prepare/3.1-user_accesskey/4.png)

Select **Next**.

![Create User and Create Access Key](/images/3-prepare/3.1-user_accesskey/5.png)

Select **Create user**.

![Create User and Create Access Key](/images/3-prepare/3.1-user_accesskey/6.png)

The User has been created successfully.

![Create User and Create Access Key](/images/3-prepare/3.1-user_accesskey/7.png)

Return to IAM Console and select **Users** in the left menu.

Select the User just created.

![Create User and Create Access Key](/images/3-prepare/3.1-user_accesskey/8.png)

Select **Security credentials** tab.

![Create User and Create Access Key](/images/3-prepare/3.1-user_accesskey/9.png)

Select **Create access key**.

![Create User and Create Access Key](/images/3-prepare/3.1-user_accesskey/10.png)

Select **Command Line Interface (CLI)**. Confirm and proceed to next step

![Create User and Create Access Key](/images/3-prepare/3.1-user_accesskey/11.png)

Description: `Amazon ECS Immersion Day` and proceed to create access key.

![Create User and Create Access Key](/images/3-prepare/3.1-user_accesskey/12.png)

The access key has been created successfully and you can click **Download .csv** to download the `credentials.csv` file.

![Create User and Create Access Key](/images/3-prepare/3.1-user_accesskey/13.png)

![Create User and Create Access Key](/images/3-prepare/3.1-user_accesskey/14.png)

#### Configure AWS CLI

Access CLI to configure AWS CLI.

Check AWS CLI version.

```bash
aws --version
```

![Create User and Create Access Key](/images/3-prepare/3.1-user_accesskey/15.png)

Configure AWS CLI.

```bash
aws configure
```

![Create User and Create Access Key](/images/3-prepare/3.1-user_accesskey/16.png)

AWS CLI has been configured successfully.

Check AWS CLI operation.

```bash
aws ec2 describe-instances
```

![Create User and Create Access Key](/images/3-prepare/3.1-user_accesskey/17.png)

AWS CLI has been configured successfully and is working properly.
