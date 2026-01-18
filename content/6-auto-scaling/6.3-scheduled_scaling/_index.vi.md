---
title : "Scheduled Scaling"
date :  2025-01-01
weight : 3
chapter : false
pre : " <b> 6.3. </b> "  
---

{{% notice warning %}}
Phải hoàn thành phần [Target Tracking Scaling](/vi/6-auto-scaling/6.2-target_tracking_scaling/) trước khi làm phần này.
{{% /notice %}}

Trong phần này, chúng ta sẽ cấu hình ECS Service Auto Scaling bằng Scheduled Scaling. Điều này bao gồm việc thiết lập các quy tắc scale theo thời gian cố định, phù hợp cho các service có mô hình tải dễ dự đoán. 

Chúng ta sẽ tạo một scheduled scaling cho UI service bằng cách sử dụng cron expression (lịch auto scaling) để kiểm tra việc scale-out và scale-in thông qua việc điều chỉnh giá trị minimum.

Tiến hành chạy lệnh sau để tăng minimum capacity từ 2 lên 5 sau 1 phút kể từ thời điểm hiện tại.

```bash
aws application-autoscaling put-scheduled-action \
 --service-namespace ecs \
 --scalable-dimension ecs:service:DesiredCount \
 --resource-id service/retail-store-ecs-cluster/ui \
 --scheduled-action-name "test-scale-up" \
 --schedule "at($(date -u -d "+1 minutes" "+%Y-%m-%dT%H:%M:%S"))" \
 --scalable-target-action MinCapacity=5,MaxCapacity=10
```

![Scheduled Scaling](/images/6-auto-scaling/6.3-scheduled_scaling/1.png)

{{% notice note %}}
Cần phải register scalable target trước khi tạo scheduled scaling.
{{% /notice %}}

Kiểm tra Event:

![Scheduled Scaling](/images/6-auto-scaling/6.3-scheduled_scaling/4.png)

Chạy lệnh sau để quan sát scheduled action. Sẽ thấy danh sách gồm 5 ECS tasks.

```bash
aws ecs describe-tasks \
 --cluster retail-store-ecs-cluster \
 --tasks $(aws ecs list-tasks --cluster retail-store-ecs-cluster --service-name ui --query 'taskArns[]' --output text) \
 --query "tasks[*].[group, launchType, lastStatus, healthStatus, taskArn]" --output table
```

![Scheduled Scaling](/images/6-auto-scaling/6.3-scheduled_scaling/2.png)

Kiểm tra trên ECS Consle:

![Scheduled Scaling](/images/6-auto-scaling/6.3-scheduled_scaling/3.png)

Bây giờ hãy chạy lệnh sau để scale back. Lệnh này sẽ đặt desired capacity về 2, là mức task ban đầu.

```bash
aws ecs update-service \
    --cluster retail-store-ecs-cluster \
    --service ui \
    --task-definition retail-store-ecs-ui \
    --desired-count 2
```

![Scheduled Scaling](/images/6-auto-scaling/6.3-scheduled_scaling/5.png)

Kiểm tra trên ECS Consle:

![Scheduled Scaling](/images/6-auto-scaling/6.3-scheduled_scaling/6.png)

#### Triển khai Scheduled Scaling trong thực tế

Các lệnh có thể được điều chỉnh để lên lịch scale trong các khung thời gian cần thiết dựa trên yêu cầu của ứng dụng. Ví dụ, scheduled scaling có thể được cấu hình để kích hoạt vào 9 giờ sáng UTC và 6 giờ tối UTC mỗi ngày, phù hợp với thời gian cao điểm của một ứng dụng toàn cầu.

Ví dụ sau có thể áp dụng trong thực tế.

Lệnh sau sẽ kích hoạt scale-out, tăng số lượng task lên 5 mỗi ngày lúc 9:00 sáng UTC để đáp ứng lưu lượng truy cập dự kiến tăng cao trong ngày:

```bash
aws application-autoscaling put-scheduled-action \
    --service-namespace ecs \
    --scalable-dimension ecs:service:DesiredCount \
    --resource-id service/retail-store-ecs-cluster/ui \
    --scheduled-action-name "week-day-scale-out" \
    --schedule "cron(0 9 ? * MON-FRI *)" \
    --scalable-target-action MinCapacity=5,MaxCapacity=10
```

Lệnh sau sẽ kích hoạt scale-in, giảm số lượng task trong các khung giờ thấp điểm. Ví dụ, chúng ta sẽ scale down UI service lúc 6 giờ tối mỗi ngày.

```bash
aws application-autoscaling put-scheduled-action \
    --service-namespace ecs \
    --scalable-dimension ecs:service:DesiredCount \
    --resource-id service/retail-store-ecs-cluster/ui \
    --scheduled-action-name "week-day-scale-in" \
    --schedule "cron(0 18 ? * MON-FRI *)" \
    --scalable-target-action MinCapacity=2,MaxCapacity=10
```

