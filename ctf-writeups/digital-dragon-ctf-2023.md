# Digital Dragon CTF 2023

**The Cybersecurity Challenge 2023. Cuộc thi do Trường Đại học Công nghệ thông tin và Truyền thông Việt- Hàn phối hợp với Công ty EvvoLab, Singapore tổ chức dưới sự bảo trợ chuyên môn của Cục An toàn thông tin-Bộ Thông tin và Truyền thông và FISU Việt Nam.**



## Invisible

### Overview

> Not perceptible!! Our SOC was alerted of a breach and they suspect that some attacker got into one of our employee’s system after she clicked on a phishing mail resulting in a malware download. They also suspect that some exfiltration of data happened but our experts were unable to identify the files extracted. Can you catch the threat and identify what he took??

Challenge được lưu trên Huawrcloud. Ta sử dụng **obsutil** (một công cụ dòng lệnh để truy cập và quản lý OBS trên Huawei Cloud) để download challenge.

Các bước thực hiện download challenge

Step 1 :

Please review the document that provides instructions for installing "obsutil" on various operating systems. https://support.huaweicloud.com/intl/en-us/utiltg-obs/obs\_11\_0003.html

Step 2:

Linux Machine users

command to download obsutil

```
wget https://obs-community-intl.obs.ap-southeast-1.myhuaweicloud.com/obsutil/current/obsutil_linux_amd64.tar.gz
```

command in the directory where the tool package resides

```
tar -xzvf obsutil_linux_amd64.tar.gz
```

```
cd obsutil_linux_amd64_5.4.11/
```

Step 3:

Use the permanent AK and SK for configuring initialization:

```
./obsutil config -i=DEW6H07RVZSJ9SNCMIGN -k=3zD2DgXoT2lt3s8YJyAu61KeloM8BeBYNVevkINe-e=obs.ap-southeast-3.myhuaweicloud.com
```

```
./obsutil cp obs://1nv1s1bl3/evidence-1nv1s1bl3.pcapng /{Local Path}
```

<figure><img src="../.gitbook/assets/image (27) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (28) (1).png" alt=""><figcaption></figcaption></figure>

Ta được cung cấp 1 flle pcapng. Dựa vào description của bài, ta được biết một số hoạt động đánh cắp dữ liệu đã được diễn ra. Ta có thể nghĩ đến DNS exfiltration và tunneling.

Mình sử dụng công cụ Wireshark để tiến hành phân tích.

<figure><img src="../.gitbook/assets/image (29) (1).png" alt=""><figcaption></figcaption></figure>

Sau khi phân tích nhanh file pcap và sử dụng dns query thì dường như suy nghĩ ban đầu đã đúng định hướng, ta có thể thấy hành vi lạ trong thông tin trao đổi DNS.

Sau một thời gian tìm hiểu kĩ và phân tích các packet thì mình biết rằng attacker đang sử dụng DNS exfiltration.&#x20;

### Backgroud Knowledge

#### **DNS**

Domain Name System là một protocol quan trọng trong Internet và mạng máy tính để chuyển đổi các tên miền (như example.com) thành địa chỉ IP (như 192.168.1.1) và ngược lại.

Cách DNS hoạt động:

* Khi bạn nhập một tên miền vào trình duyệt web, trình duyệt sẽ gửi một yêu cầu DNS đến máy chủ DNS (DNS server) để tìm địa chỉ IP tương ứng.
* Máy chủ DNS sẽ trả về địa chỉ IP của tên miền đó, cho phép trình duyệt biết cách kết nối đến máy chủ web tương ứng.

<figure><img src="../.gitbook/assets/image (33) (1).png" alt=""><figcaption></figcaption></figure>

Các loại DNS record quan trọng:

* **A Record (Address Record):** Đây là bản ghi DNS dùng để ánh xạ tên miền đến địa chỉ IPv4.
* **AAAA Record (IPv6 Address Record):** Tương tự như A Record, nhưng sử dụng để ánh xạ tên miền đến địa chỉ IPv6.
* **MX Record (Mail Exchange Record):** Sử dụng để xác định máy chủ email của tên miền.
* **CNAME Record (Canonical Name Record):** Sử dụng để tạo bí danh (alias) cho một tên miền, thường được sử dụng để chuyển hướng từ một tên miền khác.
* **TXT Record (Text Record):** Được sử dụng để chứa thông tin văn bản cho một tên miền, thường được sử dụng cho xác minh và chứng thực.

#### **DNS Exfiltration** <a href="#daef" id="daef"></a>

DNS exfiltration là quá trình trái phép truyền dữ liệu từ một hệ thống hoặc mạng nội bộ ra ngoài mạng Internet hoặc hệ thống không được ủy quyền. Mục tiêu của data exfiltration thường là lấy cắp hoặc tiết lộ thông tin nhạy cảm, bí mật hoặc quan trọng của tổ chức hoặc cá nhân.&#x20;

<figure><img src="../.gitbook/assets/image (34) (1).png" alt=""><figcaption></figcaption></figure>

Để hiểu rõ hơn trực quan hơn về DNS exfiltration, ta có thể xem video dưới đây:

{% embed url="https://www.youtube.com/watch?v=fQ4Y8napHzw" %}

Ngữ cảnh tấn công DNS exfiltration như sau:

Một attacker đã xâm nhập vào mạng nội bộ của một tổ chức và muốn lấy cắp dữ liệu quan trọng từ máy chủ nội bộ của họ. Attacker không thể truy cập trực tiếp vào máy chủ này từ bên ngoài mạng vì tường lửa đã chặn mọi kết nối đến máy chủ.

**Cách thực hiện:**

1. Attacker cài đặt một máy chủ DNS kiểm soát bên ngoài mạng, mà họ có quyền quản lý.
2. Attacker sử dụng phần mềm độc hại đã cài đặt trên máy tính nội bộ để chia nhỏ dữ liệu cần lấy cắp thành các phần nhỏ hơn.
3. Attacker cấu hình máy tính nội bộ để thực hiện các truy vấn DNS đến máy chủ DNS kiểm soát ngoài mạng. Các truy vấn này chứa dữ liệu bị mã hóa hoặc ẩn đi.
4. Máy chủ DNS kiểm soát của attacker được cấu hình để nhận các truy vấn DNS này và lưu trữ chúng.
5. Attacker sau đó có thể truy cập máy chủ DNS kiểm soát để giải mã hoặc lấy dữ liệu đã lưu trữ.
6. Dữ liệu có thể được gửi đến máy chủ DNS kiểm soát thông qua các truy vấn DNS mới hoặc qua một cơ chế gửi dữ liệu chờ phản hồi (DNS tunneling)

#### DNS exfiltration Detection

Vì cuộc tấn công Data Exfiltration thông qua DNS rất lén lút và vì dữ liệu đang được truyền qua mạng nên việc phát hiện ra cuộc tấn công này là một thách thức

Dưới đây là một số cách để phát hiện DNS exfiltration:

1. **Kiểm tra lưu lượng DNS không thường xuyên:**
   * Kiểm tra các truy vấn DNS đến các tên miền không thường xuyên hoặc không liên quan đến hoạt động bình thường của tổ chức.
   * Quan sát sự xuất hiện của các truy vấn DNS lớn hơn kích thước bình thường, đặc biệt là các gói tin DNS UDP lớn hơn 512 byte (gói tin này có thể được sử dụng trong các cuộc tấn công amplification).
   * Theo dõi các gói tin DNS có thời gian sống (TTL) không thường xuyên.
2. **Xem xét các  DNS transaction không thường xuyên:**
   * Theo dõi sự xuất hiện của các truy vấn DNS và phản hồi DNS không cân xứng (không có truy vấn tương ứng).
3. **Theo dõi sự tương tác giữa máy tính và DNS server:**
   * Xem xét các máy tính nội bộ và quyết định xem liệu có sự tương tác không thường giữa các máy tính nội bộ và máy chủ DNS ngoại vi không được kiểm soát.
   * Đảm bảo rằng các máy tính nội bộ được cấu hình để sử dụng máy chủ DNS nội bộ, và không thể truy cập máy chủ DNS ngoại vi trực tiếp.
4. **Sử dụng các công cụ và giải pháp phát hiện DNS exfiltration:**
   * Có thể sử dụng các công cụ phát hiện an ninh mạng để giám sát và phát hiện các hoạt động DNS exfiltration không thường xuyên.
   * Một số giải pháp phát hiện DNS exfiltration có thể theo dõi giao dịch DNS và tạo cảnh báo khi phát hiện sự không bình thường.
5. **Cấu hình máy chủ DNS để ghi log chi tiết:**
   * Cấu hình máy chủ DNS để ghi log chi tiết về các truy vấn DNS và phản hồi DNS.
   * Theo dõi và phân tích log để xác định sự không bình thường trong lưu lượng DNS.

### Back to the challenge

Mình kiểm tra các DNS queries dài bất thường trong wireshark bằng query

```
dns && dns.qry.name.len > 35
```

<figure><img src="../.gitbook/assets/image (35) (1).png" alt=""><figcaption></figcaption></figure>

Ta có thể thấy một lượng lớn traffic truy cập đến suspicious domain và IP&#x20;

`ie13y39.net | 172.25.20.5`

Mình sử dụng tshark để trích xuất tất cả truy vấn đến domain và IP này

```sh
tshark -r evidence-1nv1s1bl3.pcapng -Y "dns.qry.name contains ie13y39.net && ip.dst==172.25.20.5" -T fields -e frame.time_relative -e dns.qry.name
```

Trong đó:&#x20;

* `-r evidence-1nv1s1bl3.pcapng`: Chọn tệp pcapng  để phân tích.
* `-Y "dns.qry.name contains ie13y39.net && ip.dst==172.25.20.5"`:   filter các gói tin trong tệp pcapng:
  * `dns.qry.name contains ie13y39.net`: Lọc các gói tin DNS query mà chứa chuỗi "ie13y39.net" trong domain query.
  * `ip.dst==172.25.20.5`: Lọc các gói tin có địa chỉ IP đích (destination IP) là 172.25.20.5.&#x20;
* `-T fields`: Sử dụng tùy chọn `-T` để định dạng đầu ra.&#x20;
* `-e frame.time_relative`: Với `-e`, chúng ta chỉ định các trường mà chúng ta muốn trích xuất. Trong trường hợp này, chúng ta đang trích xuất thời gian tương đối của các gói tin.
* `-e dns.qry.name`:  ta đang trích xuất tên miền DNS từ các gói tin.

<figure><img src="../.gitbook/assets/image (38) (1).png" alt=""><figcaption></figcaption></figure>

Từ output ta có thể thấy ở subdomains chứa Hex number

Một chút magic để tách toàn bộ hex number với domain

<figure><img src="../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

Thêm `awk '{print $2}' | cut -d "." -f "1"` vào query&#x20;

1. `awk '{print $2}'`: Lệnh này chọn phần thứ hai từ mỗi dòng đầu vào. Biến `$2` đại diện cho phần tử thứ hai trên mỗi dòng, và sau đó, nó sẽ in ra phần tử này.&#x20;
2. `cut -d "." -f "1"`: Lệnh này được sử dụng để cắt các chuỗi dựa trên dấu chấm (.) và sau đó chọn phần tử thứ nhất sau khi cắt. Trong đó:
   * `-d "."` xác định rằng dấu chấm (.) sẽ được sử dụng làm ký tự ngăn cách (delimiter) để cắt chuỗi.
   * `-f "1"` chỉ định rằng chúng ta muốn chọn phần tử thứ nhất sau khi cắt.

Sau đó lưu lại file này dưới dạng binary data bằng cách

&#x20;**`tr -d "\n\t" | xxd -r -p > flag/flag`**

* `tr -d "\n\t"` là một lệnh dùng để xóa các ký tự xuống dòng ('\n') và ('\t') từ văn bản đầu vào
* `xxd -r -p` là một lệnh trong Unix và các hệ thống tương tự được sử dụng để chuyển đổi dữ liệu từ định dạng hex (hexadecimal) thành dạng dữ liệu nhị phân (binary data)

<figure><img src="../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

### Get Flag

Ta có file OpenDocument Text (một định dạng tập tin thuộc họ XML bị nén Zip dùng để lưu những tài liệu điện tử như bức thư, văn bản, bảng tính, biểu đồ, và bản thuyết trình.)

Ta có thể dùng 7z để extract file

<figure><img src="../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

Check content.xml file ta có flag:

<figure><img src="../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

Easy hơn ta có thể thêm .odt vào file flag và mở bằng word

<figure><img src="../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

## To be continue
