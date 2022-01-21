# Filebeat
## **1) Giới thiệu** <img src=https://i.imgur.com/nosafWg.png align=right width=20%>
- Trang chủ : https://www.elastic.co/beats/
- **Beats** là những data shipper mã nguồn mở mà ta sẽ cài đặt như các agent trên các server mà chúng ta cần thu thập các sự kiện để gửi các kiểu dữ liệu khác nhau tới **Elasticsearch**. Beats có thể gửi dữ liệu trực tiếp tới **Elasticsearch** hay thông qua **Logstash**.
- **Beats** là một platform trong đó có các project nhỏ sinh ra thực hiện trên từng loại dữ liệu nhất định.
- **ELK** cần sử dụng các **beat** để làm shipper giúp gửi các loại dữ liệu từ client tới Server.
- Các beat index pattern cần được cài đặt trên cả ELK server và các client. Trên ELK server, các **beat** sẽ kết hợp với các thành phần để lọc dữ liệu, đánh chỉ mục, hiển thị.
- **Beat** chia ra 7 loại như sau :

    | Beat | Data Capture | Description |
    |------|--------------|-------------|
    | **Auditbeat** | Audit data | 	A supercharged version of Linux auditd. It can interact directly with your Linux system in place of the auditd process. If you already have auditd rules in place, Auditbeat will read from your existing configuration. |
    | **Filebeat** | Log files | Reads and ships system logfiles. It is useful for server logs, such as hardware events or application logs. |
    | **Functionbeat** | Cloud data | Ships data from serverless or cloud infrastructure. If you’re running a cloud-hosted service, use it to collate data from the cloud and export it to Elasticsearch. |
    | **Heartbeat** | Availability | Displays uptime and response time. Use this to keep an eye on critical servers or other systems, to make sure they’re running and available. |
    | **Metricbeat** | Metrics | Reads metric data - CPU usage, memory, disk usage, network bandwidth. Use this as a supercharged system resource monitor. |
    | **Packetbeat** | Network traffic | Analyzes network traffic. Use it to monitor latency and responsiveness, or usage and traffic patterns. |
    | **Winlogbeat** | Windows event logs | Ships data from the Windows Event Log. Track logon events, installation events, even hardware or application errors. |

- Ngoài các loại **Beat** cơ bản, community phát triển thêm một số loại **beat** khác để hỗ trợ monitor các công nghệ khác. Tham khảo tại [link](https://www.elastic.co/guide/en/beats/libbeat/current/community-beats.html).
- Trong các **beats** được kể ở trên thì **filebeat** thường được ưu tiên sử dụng tuy nhiên **filebeat** vẫn còn một số hạn chế cần lưu ý khi sử dụng như :
    - Khó khăn đối với người mới sử dụng cú pháp `YAML`.
    - Nếu cấu hình quá nhiều file log cần đẩy về thì **filebeat registry** sẽ phình to rất nhanh do cần dung lượng để lưu trữ từng trạng thái của từng dòng log (dòng log đã được gửi đi hay chưa).
    - Không nên cấu hình **filebeat** quét các filelog nhỏ hơn `1s` bởi vì điều này sẽ khiến cho **filebeat** chiếm CPU một lượng đáng kể.
## **2) Cách hoạt động của Filebeat**
- Khi khởi động **Filebeat**, nó sẽ bắt đầu tìm kiếm một hoặc nhiều input là các vị trí lưu log đã cấu hình từ trước. Đối với mỗi vị trí log mà **Filebeat** xác định được, nó sẽ khởi động một **haverster**. Mỗi **haverster** sẽ đọc một file log và lấy phần log mới gửi tới **libbeat**, nơi tổng hợp các event và gửi dữ liệu tới đầu output đã được cấu hình :

    <p align=center><img src=https://i.imgur.com/S5GbO0s.png width=60%></p>

## **3) Cài đặt Filebeat**
### **Mô hình**
### **Cài đặt Filebeat trên Client**
- **B1 :** Thêm repository của **ElasticSearch** :
    ```
    # cat <<EOF | sudo tee /etc/yum.repos.d/elasticsearch.repo
    [elasticsearch-7.x]
    name=Elasticsearch repository for 7.x packages
    baseurl=https://artifacts.elastic.co/packages/7.x/yum
    gpgcheck=1
    gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
    enabled=1
    autorefresh=1
    type=rpm-md
    EOF
    ```
- **B2 :** Cài đặt **Filebeat** :
    ```
    # yum install filebeat -y
    ```
- **B3 :** Backup file cấu hình **filebeat** :
    ```
    # mv /etc/filebeat/filebeat.yml /etc/filebeat/filebeat.yml.bak
    ```
- **B4 :** 