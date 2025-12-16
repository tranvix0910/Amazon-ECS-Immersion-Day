---
title : "Services"
date :  2025-01-01
weight : 3
chapter : false
pre : " <b> 3.3. </b> "  
---

#### Tạo VPC

Truy cập VPC trên AWS Console:

![Tạo VPC](/images/3-prepare/3.3-services/1.png)

Chọn **Create VPC**.

![Tạo VPC](/images/3-prepare/3.3-services/2.png)

Chọn **VPC and more** và cấu hình các thông số sau:

- **VPC Name**: `ecs-workshop`
- **VPC CIDR Block**: `10.0.0.0/16`
- **Number of Availability Zones**: `2`
- **Number of Public Subnets**: `2`
- **Number of Private Subnets**: `2`
- **NAT Gateway**: `None` (NAT sẽ được tạo thủ công sau)
- **VPC Endpoint**: `None`

Sau khi cấu hình xong, chọn **Create VPC**.

![Tạo VPC](/images/3-prepare/3.3-services/3.png)

![Tạo VPC](/images/3-prepare/3.3-services/4.png)

Như vậy VPC đã được tạo thành công.

![Tạo VPC](/images/3-prepare/3.3-services/5.png)

![Tạo VPC](/images/3-prepare/3.3-services/6.png)

#### Tạo NAT Gateway

Trên giao diện VPC Console, chọn **NAT Gateways** và chọn **Create NAT Gateway**.

![Tạo NAT Gateway](/images/3-prepare/3.3-services/7.png)

Cấu hình các thông số sau:

- **NAT Gateway Name**: `ecs-workshop-nat`
- **Availability mode**: `Regional`
- **VPC**: VPC vừa mới tạo.
- **Method of Elastic IP (EIP) allocation**: `Automatic`

Sau khi đã cấu hình xong tiến hành chọn **Create NAT Gateway**

![Tạo NAT Gateway](/images/3-prepare/3.3-services/8.png)

Tiến hành tạo **Elastic IP** trên giao diện VPC Console.

- Chon **Allocation Elastic IP address**

![Tạo NAT Gateway](/images/3-prepare/3.3-services/9.png)

Chọn **Allocate**

![Tạo NAT Gateway](/images/3-prepare/3.3-services/10.png)

Như vậy **Elastic IP** đã được tạo thành công.

![Tạo NAT Gateway](/images/3-prepare/3.3-services/11.png)

NAT Gateway sẽ tự động lấy **Elastic IP** đã tạo.

![Tạo NAT Gateway](/images/3-prepare/3.3-services/12.png)

Sau khi tạo NAT Gateway, tiến hành cấu hình Route Table cho Private Subnet.

Truy cập **Route Table**:

- Chọn **Private Subnet**.
- Chọn **Routes**
- Chọn **Edit Routes**

![Tạo Route Table](/images/3-prepare/3.3-services/37.png)

Cấu hình các thông số sau:

- **Destination**: `0.0.0.0/0`
- **Target**: `NAT Gateway` và chọn ID của NAT Gateway vừa tạo.

![Tạo Route Table](/images/3-prepare/3.3-services/38.png)

Như vậy Route Table đã được cấu hình thành công.

![Tạo Route Table](/images/3-prepare/3.3-services/39.png)

Làm tương tự với Private Subnet thứ hai.

![Tạo Route Table](/images/3-prepare/3.3-services/40.png)

#### Tạo Security Group

Truy cập vào EC2 Console:

![Tạo Security Group](/images/3-prepare/3.3-services/13.png)

Chọn **Security Groups** và chọn **Create Security Group**.

![Tạo Security Group](/images/3-prepare/3.3-services/14.png)

Chúng ta sẽ tiến hành tạo `alb-sg` để cho phép truy cập vào ALB.

- **Security Group Name**: `alb-sg`
- **Description**: `Security Group for ALB`
- **VPC**: VPC vừa mới tạo.

Inbound rules:

- **Type**: `HTTP`
- **Port**: `80`
- **Source**: `0.0.0.0/0`
- **Type**: `HTTPS`
- **Port**: `443`
- **Source**: `0.0.0.0/0`

![Tạo Security Group](/images/3-prepare/3.3-services/15.png)

Outbound rules:

- **Type**: `All traffic`
- **Source**: `0.0.0.0/0`

Sau khi đã cấu hình xong tiến hành chọn **Create Security Group**

![Tạo Security Group](/images/3-prepare/3.3-services/16.png)

Sau khi tạo xong Security Group cho ALB, tiến hành tạo Security Group cho ứng dụng.

![Tạo Security Group](/images/3-prepare/3.3-services/17.png)

- **Security Group Name**: `ui-sg`
- **Description**: `Security Group for UI`
- **VPC**: VPC vừa mới tạo.

Inbound rules:

- **Type**: `Custom TCP Rule`
- **Port**: `8080`
- **Source**: `alb-sg`

![Tạo Security Group](/images/3-prepare/3.3-services/18.png)

Sau khi đã cấu hình xong tiến hành chọn **Create Security Group**

![Tạo Security Group](/images/3-prepare/3.3-services/19.png)

Như vậy Security Group đã được tạo thành công.

![Tạo Security Group](/images/3-prepare/3.3-services/20.png)

#### Tạo Target Group

Truy cập EC2 Console:

![Tạo Target Group](/images/3-prepare/3.3-services/21.png)

Chọn phần **Target Groups** và chọn **Create Target Group**.

![Tạo Target Group](/images/3-prepare/3.3-services/22.png)

Cấu hình các thông số sau:

- **Target Group Name**: `ui-tg`
- **Target Type**: `IP Addresses`
- **Protocol**: `HTTP`
- **Port**: `8080`
- **VPC**: VPC vừa mới tạo.
- **Health Check Protocol**: `HTTP`
- **Health Check Path**: `/actuator/health`

![Tạo Target Group](/images/3-prepare/3.3-services/23.png)

![Tạo Target Group](/images/3-prepare/3.3-services/24.png)

Sau khi cấu hình xong chọn **Next**.

![Tạo Target Group](/images/3-prepare/3.3-services/25.png)

![Tạo Target Group](/images/3-prepare/3.3-services/26.png)

Chọn **Next**.

![Tạo Target Group](/images/3-prepare/3.3-services/27.png)

Kiểm tra lại các thông số và chọn **Create Target Group**.

![Tạo Target Group](/images/3-prepare/3.3-services/28.png)

Như vậy Target Group đã được tạo thành công.

![Tạo Target Group](/images/3-prepare/3.3-services/29.png)

#### Tạo Application Load Balancer

Tại giao diện EC2 Console, chọn **Load Balancers** và chọn **Create Load Balancer**.

![Tạo Application Load Balancer](/images/3-prepare/3.3-services/30.png)

Chọn Type là **Application Load Balancer**.

![Tạo Application Load Balancer](/images/3-prepare/3.3-services/31.png)

Cấu hình các thông số sau:

- **Load Balancer Name**: `ecs-workshop-alb`
- **Scheme**: `Internet-facing`

![Tạo Application Load Balancer](/images/3-prepare/3.3-services/32.png)

- **VPC**: VPC vừa mới tạo.
- Cấu hình 2 Publib Subnet cho ALB.

![Tạo Application Load Balancer](/images/3-prepare/3.3-services/33.png)

- Chọn `alb-sg` vừa tạo.

![Tạo Application Load Balancer](/images/3-prepare/3.3-services/34.png)

Review và chọn **Create Load Balancer**.

![Tạo Application Load Balancer](/images/3-prepare/3.3-services/35.png)

![Tạo Application Load Balancer](/images/3-prepare/3.3-services/36.png)

#### Tạo Service ECS

Sau khi tạo xong các thành phần ở trên, tiến hành tạo Service ECS.

Sử dụng CLI sau để tiến hành tạo Service ECS.

Chú ý các tham số sau:

- **UI_TG_ARN**: ARN của Target Group của ứng dụng.
- **PRIVATE_SUBNET1**: ID của Private Subnet thứ nhất.
- **PRIVATE_SUBNET2**: ID của Private Subnet thứ hai.
- **UI_SG_ID**: ID của Security Group của ứng dụng.


```bash
aws ecs create-service \
    --cluster retail-store-ecs-cluster \
    --service-name ui \
    --task-definition retail-store-ecs-ui \
    --desired-count 2 \
    --launch-type FARGATE \
    --load-balancers targetGroupArn=${UI_TG_ARN},containerName=application,containerPort=8080 \
    --network-configuration "awsvpcConfiguration={subnets=[${PRIVATE_SUBNET1},${PRIVATE_SUBNET2}],securityGroups=[${UI_SG_ID}],assignPublicIp=DISABLED}"
```

Truy cập vào Cluster kiểm tra thì có thể thấy Service đã được tạo thành công.

![Tạo Service ECS](/images/3-prepare/3.3-services/41.png)

Kiểm tra thông tin Service:

![Tạo Service ECS](/images/3-prepare/3.3-services/42.png)

Kiểm tra thông tin Task:

![Tạo Service ECS](/images/3-prepare/3.3-services/43.png)

Ta có thể thấy 2 Task đã được tạo thành công.

![Tạo Service ECS](/images/3-prepare/3.3-services/44.png)

Tiến hành truy cập Web với ALB URL để kiểm tra trang web.

`http://ecs-workshop-alb-653740503.ap-northeast-2.elb.amazonaws.com`

![Tạo Service ECS](/images/3-prepare/3.3-services/45.png)

Như vậy ứng dụng đã được triển khai thành công.

Sau khi tạo ra các thành phần cơ bản, kiến ​​trúc hiện tại của các Service đã cấu hình được hiển thị bên dưới.

![Tạo Service ECS](/images/3-prepare/3.3-services/46.png)


