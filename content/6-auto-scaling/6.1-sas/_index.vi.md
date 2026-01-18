---
title : "Service Auto Scaling"
date :  2025-01-01
weight : 1
chapter : false
pre : " <b> 6.1. </b> "  
---

#### Service Auto Scaling

Amazon ECS tích hợp với Amazon CloudWatch để cho phép scale ECS services một cách hiệu quả dựa trên các real-time metrics. Các metric này được truyền từ Amazon ECS sang CloudWatch theo chu kỳ mỗi một phút, cho phép giám sát chính xác và đưa ra các quyết định scale kịp thời. 

Khi các metric vượt quá thresholds được định nghĩa trong scaling policy, CloudWatch sẽ kích hoạt một alarm để điều chỉnh desired number of tasks trong service của bạn. Quá trình điều chỉnh động này sẽ tăng desired capacity trong các sự kiện scale-out và giảm trong các sự kiện scale-in, đảm bảo việc sử dụng tài nguyên tối ưu.

![Service Auto Scaling](/images/6-auto-scaling/6.1-sas/1.png?width=80%)

Amazon ECS cung cấp ba chiến lược service scaling nâng cao:

**Target Tracking Scaling**:

- Phương pháp này nhằm duy trì một scaling metric ở một target value xác định bằng cách tự động điều chỉnh số lượng task. Target tracking scaling được ưu tiên nhờ tính đơn giản và yêu cầu bảo trì thấp, khiến nó trở thành lựa chọn lý tưởng cho các doanh nghiệp muốn đạt hiệu quả vận hành mà không cần can thiệp thủ công thường xuyên.

**Step Scaling**:

- Chiến lược này cung cấp khả năng kiểm soát chi tiết hơn đối với các hành động scaling. Người dùng có thể chọn metric, thiết lập threshold values, và định nghĩa step adjustments để chỉ định số lượng tài nguyên cần thêm hoặc bớt. Nó cũng cho phép tùy chỉnh breach evaluation periods cho các metric alarm, mang lại cách tiếp cận linh hoạt để xử lý các workload biến động hiệu quả.

**Scheduled Scaling**:

- Phương pháp này phù hợp nhất khi các hành động scaling có thể được dự đoán trước dựa trên các known demand patterns. Nó lý tưởng cho các ứng dụng có sự biến động traffic có thể dự đoán, cho phép quản lý tài nguyên một cách chủ động nhằm đảm bảo tính ổn định và hiệu năng của service trong các thời điểm peak.

#### Target Tracking Scaling

Tự động giữ metric quanh một giá trị mục tiêu. ECS không quan tâm cần thêm bao nhiêu task, mà chỉ quan tâm:

- “Metric hiện tại có đang lệch khỏi target value không?”

- Nếu lệch → ECS tự tính toán số task cần scale.

```
desiredTasks = currentTasks × (currentMetric / targetMetric)
```

Ví dụ: CPU-based scaling
- Metric: ECSServiceAverageCPUUtilization
- Target value: 50%
- Hiện tại:
    - 2 tasks
    - CPU = 90%
```
desiredTasks = 2 × (90 / 50) = 3.6 ≈ 4 tasks
```
-> ECS scale 2 → 4 tasks.
- **Ưu điểm**:
    - Dễ cấu hình
    - Không cần tuning phức tạp
    - Ít bảo trì
    - Tránh scale quá đà
- **Nhược điểm**:
    - Ít kiểm soát chi tiết
    - Không phù hợp workload có spike cực nhanh

**Khi nào nên dùng?**

- 80–90% ECS service.
- Web service, API, microservices.
- Load tăng dần hoặc tương đối ổn định.

#### Step Scaling

Scale theo “bậc” (step) dựa trên threshold.

Chủ động định nghĩa:
- Metric nào
- Khi vượt ngưỡng bao nhiêu
- Thì scale bao nhiêu task

ECS không tự tính, mà làm đúng theo rule được thiết lập.

Ví dụ: CPU-based Step Scaling

- Metric: CPUUtilization

| CPU   | Hành động |
| ----- | --------- |
| ≥ 60% | +1 task   |
| ≥ 75% | +2 tasks  |
| ≥ 90% | +4 tasks  |
| ≤ 30% | -1 task   |

- Service có 2 task và CPU tăng lên 85% -> ECS tăng lên 2 tasks.

Step scaling dùng CloudWatch Alarm:

Ví dụ:
- CPU ≥ 75%
- Duy trì 2 phút liên tục
→ Trigger scale action

Tránh scale do spike ngắn hạn ( tăng trong tời gian ngắn rồi giảm ).

**Ưu điểm**
- Kiểm soát rất chi tiết
- Phù hợp workload biến động mạnh
- Dùng được metric không proportional

**Nhược điểm**
- Cấu hình phức tạp
- Dễ sai nếu tuning kém
- Cần theo dõi & điều chỉnh thường xuyên

**Khi nào nên dùng?**
- Workload spike mạnh, không tuyến tính
- Batch processing
- Legacy system
- Khi cần kiểm soát chính xác số task tăng/giảm

#### Scheduled Scaling

Scale theo lịch trình (schedule).

Người cấu hình biết trước: Khi nào load cao, khi nào load thấp.

ECS scale theo thời gian định sẵn, không cần metric.

Ví dụ: Website giờ hành chính có giờ hoạt động từ 8:00 đến 18:00.

| Thời gian     | Tasks |
| ------------- | ----- |
| 08:00 – 18:00 | 6     |
| 18:00 – 08:00 | 2     |

**Ưu điểm**
- Rất ổn định
- Không phụ thuộc metric
- Tránh cold start

**Nhược điểm**
- Không phản ứng được traffic bất ngờ
- Phải dự đoán đúng

**Khi nào nên dùng?**
- Traffic có pattern rõ ràng
- Business hours
- Event cố định (sale, livestream, exam system)
