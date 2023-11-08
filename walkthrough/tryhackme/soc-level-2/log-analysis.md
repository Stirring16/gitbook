# Log Analysis

Làm thế nào để xác định các hoạt động độc hại và các evidence nào được tạo ra khi attacker xâm nhập vào network của chúng ta. Ở phần này chúng ta sẽ tìm hiểu những điều đó.

Ta sẽ vào luôn ngữ cảnh tấn công để hiểu

**Scenario**: Máy chủ web của Swiff Spend Financial liên tục bị scans từ attacker. Là một sysadmin  bạn phải xác định attacker đang làm gì bằng phân tích logs được ghi trên hệ thống.

Trong thế giới kĩ thuật số, mọi sự tương tác với hệ thống máy tính như xác thực, cấp quyền, access file, connecting to a network  cho đến các lỗi hệ thống, mọi event đó đều được lưu lại dưới dạng logs.

Logs thường bao gồm các thông tin như:

* Timestamp khi 1 event được ghi lại
* Tên của hệ thống hoặc ứng dụng được ghi
* Loại event xảy ra
* IP of device,...

<figure><img src="../../../.gitbook/assets/image (51).png" alt="" width="375"><figcaption></figcaption></figure>

Khi bắt đầu phân tích logs từ 1 event nào đó, ta sẽ bắt đầu với quy tắc 6Ws (Who?, What?, When?, Where?, and Why?)&#x20;

\
5W khi phân tích logs là một cách tiếp cận để nắm bắt thông tin quan trọng từ các bản ghi logs (nhật ký) trong hệ thống máy tính hoặc ứng dụng. 5W là viết tắt của "Who, What, When, Where, Why," và nó giúp bạn trả lời năm câu hỏi cơ bản sau đây khi bạn phân tích logs:

1. Who (Ai): Đây là người hoặc thực thể đã thực hiện hành động ghi lại trong logs. Thông tin về người dùng hoặc máy chủ liên quan đến hoạt động này có thể được xác định từ logs.
2. What (Gì): Điều này liên quan đến hoạt động cụ thể đã xảy ra. Bạn cần xác định loại hoặc tên sự kiện, lỗi, hoặc hành động mà logs đang ghi lại.
3. When (Khi nào): Thời điểm mà sự kiện hoặc hoạt động đã xảy ra. Điều này thường bao gồm ngày, giờ, và thời gian cụ thể.
4. Where (Ở đâu): Đây có thể là vị trí vật lý hoặc máy chủ mà sự kiện xảy ra. Nếu hệ thống của bạn phân tán, bạn có thể cần xác định địa chỉ IP hoặc tên máy chủ liên quan.
5. Why (Tại sao): Điều này thường là câu hỏi khó khăn nhất để trả lời từ logs. Tuy nhiên, bạn có thể tìm hiểu về nguyên nhân hoặc ngữ cảnh xung quanh sự kiện bằng cách xem các thông tin liên quan trong logs và thực hiện phân tích bổ sung.



