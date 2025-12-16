---
title : "Amazon ECS Service"    
date :  2025-01-01
weight : 2.2
chapter : false
pre : " <b> 2.2. </b> "  
---

### Amazon ECS Service

ECS Service là một công cụ quản lý cho phép bạn chạy và duy trì một số lượng task nhất định liên tục trong một ECS cluster. Nó là một phần thiết yếu của ECS dành cho các ứng dụng dài hạn, không giống như standalone tasks dùng cho các batch jobs ngắn hạn.

Nếu một trong các task bị lỗi hoặc dừng, **ECS Service Scheduler** sẽ khởi chạy một instance khác của Task Definition để thay thế.

Điều này giúp duy trì đúng số lượng task mong muốn (desired number of tasks) cho service.

Ngoài ra, chỉ Service mới có thể được cấu hình với Load Balancer. Nếu cần phân phối traffic đến các container, bắt buộc phải sử dụng Service, không phải standalone Task. LB giúp:
- High availability
- Scale
- Health check

#### Khi nào nên dùng Service Scheduler?

AWS khuyến nghị sử dụng Service Scheduler cho:
- Các service
- Ứng dụng stateless
- Workload chạy dài hạn (long-running)

Service Scheduler đảm bảo:
- Chiến lược scheduling bạn cấu hình được tuân thủ
- Task được reschedule khi bị lỗi

Ví dụ:
Nếu hạ tầng bên dưới gặp sự cố, Service Scheduler sẽ chạy lại task.

#### Các Thành Phần Cấu Hình của ECS Service

- **Service Name**: Tên của service để nhận diện nó trong cluster

- **Task Definition**: Task Definition nào sẽ được sử dụng để khởi chạy tasks. Service sẽ tham chiếu Task Definition này.

- **Desired Count**: Số lượng task mà bạn muốn Service giữ chạy liên tục. Nếu bạn chỉ định desired count là 3, Service sẽ luôn cố gắng duy trì 3 tasks đang chạy. Nếu một task bị dừng, Service sẽ khởi chạy task mới để thay thế.​

- **Scheduling Strategy**: Có hai lựa chọn:

    - **REPLICA**: Service chạy một số lượng mong muốn của tasks

    - **DAEMON**: Service chạy một task trên mỗi container instance trong cluster (không có desired count)​

- **Deployment Type**: Cách thức triển khai các phiên bản mới:​

    - **ECS**: Sử dụng ECS Scheduler (rolling update mặc định)

    - **CODE_DEPLOY**: Sử dụng AWS CodeDeploy (hỗ trợ blue-green deployments)

    - **EXTERNAL**: Cho phép triển khai thủ công qua TaskSets (dùng với CloudFormation, Jenkins, v.v.)

- **Load Balancer Configuration**: Nếu bạn muốn phân phối traffic:

    - **Loại Load Balancer**: Application Load Balancer (ALB) hoặc Network Load Balancer (NLB)

    - **Target Group**: Xác định những container nào sẽ nhận traffic

    - **Port Mapping**: Ánh xạ từ port load balancer đến port container

    - **Health Check**: Cấu hình kiểm tra sức khỏe để xác định task có hoạt động không​

#### Vòng Đời của ECS Service

Service hoạt động theo chu trình sau:

- **Khởi tạo**: Bạn tạo Service và chỉ định desired count (ví dụ: 3 tasks)

- **Khởi chạy Tasks**: Service scheduler khởi chạy 3 instances của Task Definition

- **Đăng ký với Load Balancer**: Nếu có load balancer, các tasks được tự động đăng ký với target group

- **Giám sát liên tục**: Service liên tục kiểm tra trạng thái của các tasks

- **Thay thế Tasks lỗi**: Nếu một task gặp sự cố, Service khởi chạy task mới để duy trì desired count

- **Mở rộng/Thu nhỏ**: Nếu auto scaling được kích hoạt, desired count sẽ tự động điều chỉnh dựa trên metrics

#### Deployment Configuration cho Rolling Updates

Khi bạn triển khai phiên bản mới của ứng dụng (cập nhật Task Definition), ECS sử dụng deployment configuration để kiểm soát quá trình:

- **MinimumHealthyPercent**: Phần trăm tối thiểu của tasks phải duy trì ở trạng thái RUNNING trong suốt quá trình deployment. Ví dụ, nếu desired count là 4 và MinimumHealthyPercent là 50%, ECS đảm bảo ít nhất 2 tasks (50% của 4) vẫn đang chạy khi triển khai các phiên bản mới.​

- **MaximumPercent**: Tổng số maximum tasks có thể chạy đồng thời trong suốt quá trình deployment, tính dưới dạng phần trăm của desired count. Ví dụ, nếu desired count là 4 và MaximumPercent là 200%, ECS cho phép tối đa 8 tasks chạy cùng lúc - 4 tasks cũ + 4 tasks mới. Điều này cho phép triển khai nhanh hơn nhưng yêu cầu nhiều tài nguyên hơn.​​

Ví dụ quá trình deployment:

- Bạn có 4 tasks đang chạy version v1.      

- Bạn cập nhật Task Definition thành version v2.

    - **MinimumHealthyPercent**: 100%

    - **MaximumPercent**: 200%

- ECS khởi chạy 4 tasks mới (v2) trước khi dừng 4 tasks cũ (v1).

- Lúc này bạn có 8 tasks chạy đồng thời.

- Sau đó, ECS dừng 4 tasks v1 cũ.

- Kết quả: 4 tasks v2 đang chạy.

#### Service Auto Scaling​

Service Auto Scaling cho phép ECS tự động điều chỉnh số lượng tasks dựa trên các metrics như CPU, memory, hoặc custom metrics. Có bốn loại scaling:

- **Target Tracking Scaling**: Duy trì một metric cụ thể ở mức target. Ví dụ, bạn có thể nói "duy trì CPU ở 70%". ECS sẽ tự động thêm tasks khi CPU vượt 70% và xóa tasks khi CPU dưới 70%.​

- **Step Scaling**: Sử dụng CloudWatch alarms với các thang bậc (steps). Ví dụ, khi CPU > 80%, thêm 2 tasks; khi CPU > 90%, thêm 4 tasks.​

- **Scheduled Scaling**: Điều chỉnh desired count dựa trên lịch thời gian. Ví dụ, mở rộng vào lúc 9 sáng khi công ty mở cửa, thu nhỏ vào 6 tối khi đóng cửa.​

- **Predictive Scaling**: Sử dụng machine learning để phân tích các pattern lịch sử và dự báo traffic trong tương lai.​

#### Cơ Chế Duy Trì Desired Count

Điều quan trọng cần hiểu là desired count và số tasks thực tế đang chạy có thể khác nhau tạm thời:
- **Desired Count**: Số lượng tasks mà bạn muốn Service duy trì. Nó là một giá trị mục tiêu mà ECS cố gắng đạt được.​
- **Running Tasks**: Số tasks thực tế đang chạy tại thời điểm hiện tại. Khi bạn thay đổi desired count, ECS cần thời gian để khởi chạy hoặc dừng tasks để phù hợp với desired count mới.​

Ví dụ:
- Bạn đặt desired count = 3
- Hiện tại có 1 task đang chạy
- ECS sẽ khởi chạy 2 tasks mới
- Trong quá trình khởi chạy, running tasks = 1 nhưng desired count = 3
- Sau vài giây, running tasks = 3 (khớp với desired count)

#### Triển Khai So Với Auto Scaling​

Service Auto Scaling Min/Max là dải cho phép mà Service Auto Scaling có thể điều chỉnh desired count trong nó. 

Nếu bạn đặt Min = 2 và Max = 10:

- **Service Auto Scaling sẽ không bao giờ để desired count xuống dưới 2**

- **Service Auto Scaling sẽ không bao giờ để desired count vượt quá 10**

Deployment Configuration Min/Max Percent chỉ áp dụng trong suốt quá trình deployment (khi cập nhật Task Definition). Nó kiểm soát số tasks có thể chạy trong lúc triển khai, không liên quan đến auto scaling bình thường.