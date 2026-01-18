---
title : "Target Tracking Scaling"
date :  2025-01-01
weight : 2
chapter : false
pre : " <b> 6.2. </b> "  
---

#### Register Scalable Target

Trong phần này, chúng ta sẽ cấu hình ECS Service Auto Scaling bằng cách sử dụng Target Tracking Scaling. 

Trước tiên, Register UI service như một scalable target với Application Auto Scaling. Lệnh sau thiết lập scaling range cho UI Service từ minimum 2 đến maximum 10 tasks.

Lệnh `register-scalable-target` dùng để cho phép ECS Service được auto scale và giới hạn phạm vi scale (min/max task); nếu không có bước này thì mọi scaling policy đều vô hiệu.

```bash
aws application-autoscaling register-scalable-target \
    --service-namespace ecs \
    --scalable-dimension ecs:service:DesiredCount \
    --resource-id service/retail-store-ecs-cluster/ui \
    --min-capacity 2 \
    --max-capacity 10
```

![Register Scalable Target](/images/6-auto-scaling/6.2-target_tracking_scaling/1.png)

![Register Scalable Target](/images/6-auto-scaling/6.2-target_tracking_scaling/2.png)

{{% notice note %}}
Scalable Target là một đối tượng có thể bị Auto Scaling. Việc khai báo này với Application Auto Scaling rằng: “ECS Service này ĐƯỢC PHÉP auto scale, và chỉ được scale trong một khoảng xác định.”
{{% /notice %}}

#### Tạo Scaling Policy

Scaling policy là tập các quy tắc xác định khi nào và bằng cách nào Amazon ECS Service sẽ tăng hoặc giảm số lượng task, dựa trên các metric được theo dõi.

Scaling policy không trực tiếp thực hiện việc scale tài nguyên, mà:
- Theo dõi các metric từ CloudWatch
- So sánh metric với các điều kiện hoặc target đã cấu hình
- Gửi yêu cầu tới Application Auto Scaling để điều chỉnh DesiredCount của ECS Service

Trong Amazon ECS:
- Scaling policy được quản lý bởi Application Auto Scaling
- Đối tượng được scale là DesiredCount của ECS Service

Tiếp theo, chúng ta sẽ tạo một scaling policy cho scalable target này.

Đầu tiên, tạo một file cấu hình JSON cho scaling policy. Cấu hình này sử dụng predefined metric type là request count per target liên quan đến Application Load Balancer (ALB) định tuyến request tới ECS service. Trong trường hợp này, mục tiêu của chúng ta là 1.500 requests cho mỗi ECS task (hoặc target).

{{% notice note %}}
Scaling policy này chỉ mang tính ví dụ. Cần hiểu rõ scaling profile của workload cụ thể để xác định các scaling metrics và thresholds phù hợp trước khi bật autoscaling.
{{% /notice %}}

File cấu hình JSON `ui-scaling-policy.json`:

```json
{
    "TargetValue": 1500,
    "PredefinedMetricSpecification": {
        "PredefinedMetricType": "ALBRequestCountPerTarget",
        "ResourceLabel": "$UI_ALB_PREFIX/$UI_TG_PREFIX"
    }
}
```

Lưu ý rằng chúng ta đang sử dụng các environment variables là `$UI_ALB_PREFIX` và `$UI_TG_PREFIX` để chỉ định đúng ALB và Target Group cho UI service.

- `$UI_ALB_PREFIX`: ARN của ALB, tuy nhiên chỉ cần phần prefix là đủ. Ví dụ: `app/ecs-workshop-alb/07eeccd166f4ef38`

![UI ALB ARN](/images/6-auto-scaling/6.2-target_tracking_scaling/6.png)

- `$UI_TG_PREFIX`: Target Group prefix cho UI service. Ví dụ: `targetgroup/ui-tg/2feb79f68add27c7`

![UI TG ARN](/images/6-auto-scaling/6.2-target_tracking_scaling/7.png)

Bây giờ, phần `ResourceLabel` trong file cấu hình JSON như sau: `app/ecs-workshop-alb/07eeccd166f4ef38/targetgroup/ui-tg/2feb79f68add27c7`

```bash
aws application-autoscaling put-scaling-policy \
    --service-namespace ecs \
    --scalable-dimension ecs:service:DesiredCount \
    --resource-id service/retail-store-ecs-cluster/ui \
    --policy-name ui-scaling-policy \
    --policy-type TargetTrackingScaling \
    --target-tracking-scaling-policy-configuration file://ui-scaling-policy.json
```

![Target Tracking Scaling Policy](/images/6-auto-scaling/6.2-target_tracking_scaling/3.png)

Kiểm tra target tracking scaling policy trong Amazon ECS console.

![Target Tracking Scaling Policy](/images/6-auto-scaling/6.2-target_tracking_scaling/5.png)

#### CloudWatch Alarm

Trạng thái alarm có thể thay đổi tùy theo số lượng request. Ví dụ, `UI-AlarmLow` sẽ trigger khi số request giảm xuống dưới 1350.

![CloudWatch Alarm](/images/6-auto-scaling/6.2-target_tracking_scaling/4.png)

**Cấu hình High Alarm (Scale-out):**

- Metric: ALBRequestCountPerTarget > 1500

- Evaluation: 3 data points trong 3 phút

- Action: Khi ở trạng thái ALARM, sẽ thêm task vào ECS service

- Behavior: Tiếp tục thêm task trong các evaluation period tiếp theo nếu alarm vẫn tồn tại, tối đa đến giới hạn đã chỉ định trong scaling policy

**Cấu hình Low Alarm (Scale-in):**

- Metric: ALBRequestCountPerTarget < 1350

- Evaluation: 15 data points trong 15 phút

- Action: Khi ở trạng thái ALARM, sẽ giảm số lượng task

- Behavior: Scale-in chậm hơn để ưu tiên high availability

#### Trigger Auto Scaling

Tiến hành tạo synthetic load để trigger auto scaling.

Đầu tiên chúng ta cần lấy DNS Name của Application Load Balancer được gắn cho UI service.

```bash
export RETAIL_ALB=$(aws elbv2 describe-load-balancers --name ecs-workshop-alb \
  --query 'LoadBalancers[0].DNSName' --output text)
echo "Retail ALB: ${RETAIL_ALB}"
```

![Trigger Auto Scaling](/images/6-auto-scaling/6.2-target_tracking_scaling/8.png)

Tiếp theo, chúng ta sẽ sử dụng [hey tool](https://github.com/rakyll/hey) để gửi request đến path /home của UI service:

Có thể cài đặt hey tool bằng lệnh sau:

```bash
sudo apt update
sudo apt install hey -y
```

```bash
hey -n 1000000 -c 5 -q 40 http://$RETAIL_ALB/home &
```

![Trigger Auto Scaling](/images/6-auto-scaling/6.2-target_tracking_scaling/9.png)

Hoạt động scaling sẽ được kích hoạt khi high alarm cho scaling metric bị breach trong 3 chu kỳ liên tiếp, mỗi chu kỳ 1 phút. Nếu bạn muốn tự động chờ cho đến khi alarm được kích hoạt, bạn có thể chạy lệnh sau (khoảng ~ 4 phút):

```bash
sleep 90 && aws cloudwatch wait alarm-exists --alarm-name-prefix \
 TargetTracking-service/retail-store-ecs-cluster/ui-AlarmHigh --state-value ALARM
```
Khi alarm được kích hoạt, bạn sẽ thấy task count của service scale out từ 2 lên con số cao hơn:

```bash
aws ecs describe-tasks \
    --cluster retail-store-ecs-cluster \
    --tasks $(aws ecs list-tasks --cluster retail-store-ecs-cluster --query 'taskArns[]' --output text) \
    --query "tasks[*].[group, launchType, lastStatus, healthStatus, taskArn]" --output table
```

![Trigger Auto Scaling](/images/6-auto-scaling/6.2-target_tracking_scaling/11.png)

![Trigger Auto Scaling](/images/6-auto-scaling/6.2-target_tracking_scaling/12.png)

Có thể quan sát high alarm liên quan đến scaling policy chuyển sang trạng thái ALARM trong CloudWatch console, như hình bên dưới.

![Trigger Auto Scaling](/images/6-auto-scaling/6.2-target_tracking_scaling/10.png)

Bạn cũng có thể kiểm tra tab Events trong trang UI Service để xem hoạt động scaling, nơi desired count tăng lên vượt quá số task ban đầu.

![Trigger Auto Scaling](/images/6-auto-scaling/6.2-target_tracking_scaling/13.png)

Tiếp theo chúng ta sẽ tiến hành giảm số lượng các task trong ECS service.

Dừng tiến trình Hey:

```bash
pkill -9 hey
```

Thông thường, sau vài phút, số lượng task sẽ scale back về mức tối thiểu là 2. Tuy nhiên, để tiết kiệm thời gian, chúng ta có thể force scale down service bằng cách chạy các lệnh sau:

```bash
aws ecs wait services-stable --cluster retail-store-ecs-cluster --services ui
```

```bash
aws ecs update-service \
    --cluster retail-store-ecs-cluster \
    --service ui \
    --task-definition retail-store-ecs-ui \
    --desired-count 2
```

![Force Scale Down](/images/6-auto-scaling/6.2-target_tracking_scaling/14.png)

![Force Scale Down](/images/6-auto-scaling/6.2-target_tracking_scaling/15.png)