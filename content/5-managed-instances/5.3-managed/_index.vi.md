---
title : "Managed Instances Infrastructure"
date :  2025-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "  
---

#### Managed Instances Infrastructure

Trong phần này, sẽ đi sâu tìm hiểu các EC2 instances mà Amazon ECS Managed Instances đã tạo ra để hỗ trợ UI service mà chúng ta đã triển khai ở phần trước.

Chạy lệnh sau để xem các EC2 instances:
```bash
aws ec2 describe-instances \
    --filters \
        "Name=tag:aws:ec2:managed-launch,Values=ecs-managed-instances" \
        "Name=instance-state-name,Values=running" \
    --query 'Reservations[*].Instances[*].{InstanceId:InstanceId,InstanceType:InstanceType,State:State.Name,PrivateIpAddress:PrivateIpAddress}' \
    --output table
```

Vì UI service được triển khai theo mô hình high-availability (HA) với 2 running tasks, và Amazon ECS Managed Instance capacity provider được gắn với 2 subnets (ở các AZ khác nhau), sẽ thấy 2 EC2 instances khác nhau đang chạy.

Có thể thấy các EC2 instance types khác nhau đang chạy trong cluster, vì chúng ta không chỉ định bất kỳ điều kiện nào trong capacity provider cho Amazon ECS Managed Instances.

![EC2 instances](/images/5-managed-instances/5.3-managed/1.png)

```
------------------------------------------------------------------------
|                           DescribeInstances                          |
+----------------------+---------------+--------------------+----------+
|      InstanceId      | InstanceType  | PrivateIpAddress   |  State   |
+----------------------+---------------+--------------------+----------+
|  i-056556a991639db3c |  c5a.large    |  10.0.145.243      |  running |
|  i-08b1aa2b2ec697a12 |  c5a.large    |  10.0.135.156      |  running |
+----------------------+---------------+--------------------+----------+
```

Có thể xem các Instances trên ECS Console:

![ECS Instances](/images/5-managed-instances/5.3-managed/2.png)

Bây giờ, hãy kiểm tra cách Amazon ECS Managed Instance xử lý workload bổ sung bằng cách tăng số running tasks của UI service từ 2 lên 6. Trong trường hợp này, sẽ nhận thấy rằng vì các EC2 instances hiện tại không đủ tài nguyên để triển khai các running tasks, nên các EC2 instances bổ sung sẽ được triển khai để hỗ trợ workload mở rộng. Xem [AWS Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/managed-instances-instance-selection-best-practices.html) để biết thêm thông tin về instance selection.

```bash
aws ecs update-service \
    --cluster retail-store-ecs-cluster \
    --service ui-managed-instance \
    --desired-count 6 \
    --placement-constraints \
    --task-definition retail-store-ecs-ui-managed
```

![ECS Tasks](/images/5-managed-instances/5.3-managed/3.png)

Sau khi quá trình triển khai hoàn tất, hãy xem cách các task được phân bố trên các EC2 instances.

![ECS Tasks](/images/5-managed-instances/5.3-managed/4.png)

![ECS Tasks](/images/5-managed-instances/5.3-managed/5.png)

Có thể thấy có 2 task đang chạy trên `m7i-flex.large` và 2 task đang chạy trên `m5.large` còn lại 2 task chia đều cho `c5a.large`.

Kiểm tra instance types bằng cách mở EC2 console để xem các EC2 instances mới. Amazon ECS Managed Instance sẽ khởi tạo thêm các instance để đáp ứng yêu cầu capacity mới.

![EC2 instances](/images/5-managed-instances/5.3-managed/6.png)

Cuối cùng, hãy scale down số lượng task của UI service về 2.

```bash
aws ecs update-service \
    --cluster retail-store-ecs-cluster \
    --service ui-managed-instance \
    --desired-count 2 \
    --task-definition retail-store-ecs-ui-managed \
    --placement-constraints type="distinctInstance"
```

![ECS Tasks](/images/5-managed-instances/5.3-managed/7.png)

4 tasks đã được gỡ bỏ và hiện có 2 instances đang ở trạng thái idle.

![EC2 instances](/images/5-managed-instances/5.3-managed/8.png)

Sau vài phút nữa, các idle instances sẽ được capacity provider deregister và terminate. Thực thi lệnh sau, sau vài phút sẽ chỉ thấy còn 2 EC2 instances đang chạy:

```bash
aws ec2 describe-instances \
    --filters \
        "Name=tag:aws:ec2:managed-launch,Values=ecs-managed-instances" \
        "Name=instance-state-name,Values=running" \
    --query 'Reservations[*].Instances[*].{InstanceId:InstanceId,InstanceType:InstanceType,State:State.Name,PrivateIpAddress:PrivateIpAddress}' \
    --output table
```

![EC2 instances](/images/5-managed-instances/5.3-managed/9.png)

![EC2 instances](/images/5-managed-instances/5.3-managed/10.png)

