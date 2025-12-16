---
title : "Chuẩn bị"
date :  2025-01-01
weight : 3
chapter : false
pre : " <b> 3. </b> "  
---

#### Cài đặt AWS CLI

- Linux/MacOS:

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

- Windows:

```powershell
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```

#### Tạo User và tạo Access Key

Truy cập [AWS Console](https://console.aws.amazon.com/) và tạo mới một User với quyền AdministratorAccess.

Chọn service IAM.

