# Cài đặt ELK Stack
## **Mô hình**
## **Các bước cài đặt**
### **Cài đặt ElasticSearch**
- **B1 :** Cài đặt **Java OpenJDK** :
    ```
    # yum -y install java-openjdk-devel java-openjdk
    ```
- **B2 :** Import GPG key :
    ```
    # rpm --import http://packages.elastic.co/GPG-KEY-elasticsearch
    ```
- **B3 :** Thêm repository của **ElasticSearch** :
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
- **B4 :** Update các repo :
    ```
    # yum repolist
    ```   
- **B5 :** Cài đặt **ElasticSearch** :
    ```
    # yum install elasticsearch -y
    ```
- **B6 :**  (tùy chọn) Cấu hình các JVM options như memory limit trong file `jvm.options` :
    ```
    # vi /etc/elasticsearch/jvm.options
    ```
    - Chỉnh sửa các thông số `Xms` và `Xmx` theo thứ tự là initial/maximum size của tổng lượng heap memory :
        <img src=https://i.imgur.com/H0Lu85F.png>

- **B7 :** Chỉnh sửa các tham số `network.host`, `http.port`, `discovery.seed_hosts` trong file `elasticsearch.yml` :
    ```
    # vi /etc/elasticsearch/elasticsearch.yml
    ```
    <img src=https://i.imgur.com/NdXupK9.png>

    - Thêm 2 parameter sau vào cuối file :
        ```ini
        xpack.security.enabled: true

        discovery.type: single-node
        ```
    <img src=https://i.imgur.com/AV1HITv.png>

- **B9 :** Khởi động **ElasticSearch** và cho phép dịch vụ khởi động cùng hệ thống :
    ```
    # systemctl start elasticsearch
    # systemctl enable elasticsearch
    ```
- **B10 :** Khởi tạo password cho các user có sẵn ([tham khảo thêm](https://www.elastic.co/guide/en/elasticsearch/reference/7.16/security-minimal-setup.html)) :
    ```
    # /usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto
    ```
    <img src=https://i.imgur.com/wXkZvOD.png>
    
    > Lưu lại các user và password vừa được gen, vì sẽ không thể chạy lại câu lệnh này lần 2. Có thể tự set password bằng lệnh `elasticsearch-setup-passwords interactive`

### **Cài đặt Logtash**
- **B1 :** Thêm repository của **Logtash** :
    ```
    # cat <<EOF | sudo tee /etc/yum.repos.d/logstash.repo
    [logstash-7.x]
    name=Elastic repository for 7.x packages
    baseurl=https://artifacts.elastic.co/packages/7.x/yum
    gpgcheck=1
    gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
    enabled=1
    autorefresh=1
    type=rpm-md
    EOF
    ```
- **B2 :** Update các repo :
    ```
    # yum repolist
    ```
- **B3 :** Cài đặt **Logtash** :
    ```
    # yum install logstash -y
    ```
- **B4 :** Khởi động **ElasticSearch** và cho phép dịch vụ khởi động cùng hệ thống :
    ```
    # systemctl start logstash
    # systemctl enable logstash
    ```
- **B5 :** Kiểm tra phiên bản `logtash` vừa cài đặt :
    ```
    # /usr/share/logstash/bin/logstash --version
    ```
    <img src=https://i.imgur.com/8Egw4Sb.png>

### **Cài đặt Kibana**
- **B1 :** Thêm repository của **Logtash** :
    ```
    # cat <<EOF | sudo tee /etc/yum.repos.d/kibana.repo
    [kibana-7.x]
    name=Kibana repository for 7.x packages
    baseurl=https://artifacts.elastic.co/packages/7.x/yum
    gpgcheck=1
    gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
    enabled=1
    autorefresh=1
    type=rpm-md
    EOF
    ```
- **B2 :** Update các repo :
    ```
    # yum repolist
    ```
- **B3 :** Cài đặt **Kibana** :
    ```
    # yum install kibana -y
    ```
- **B4 :** Chỉnh sửa file cấu hình `/etc/kibana/kibana.yml` :
    ```
    # vi /etc/kibana/kibana.yml
    ```
    <img src=https://i.imgur.com/fkkQbEj.png>

    <img src=https://i.imgur.com/WDI8LRy.png>

- **B5 :** Cấu hình `firewalld` cho phép port `5601` :
    ```
    # firewall-cmd --zone=public --permanent --add-port=5601/tcp
    # firewall-cmd --reload
    ```
- **B6 :** Khởi động **Kibana** và cho phép dịch vụ khởi động cùng hệ thống :
    ```
    # systemctl start kibana
    # systemctl enable kibana
    ```
- **B7 :** Kiểm tra phiên bản `kibana` vừa cài đặt :
    ```
    # /usr/share/kibana/bin/kibana --version --allow-root
    ```
    <img src=https://i.imgur.com/JYMSAmA.png>

- **B8 :** Tạo Kibana keystore :
    ```
    # /usr/share/kibana/bin/kibana-keystore create
    ```
- **B9 :** Nhập mật khẩu của user `kibana_system` vào Kibana keystore :
    ```
    # /usr/share/kibana/bin/kibana-keystore add elasticsearch.password
    ```
- **B10 :** Khởi động lại dịch vụ `kibana` :
    ```
    # systemctl restart kibana
    ```
- **B11 :** Truy cập **Kibana** từ trình duyệt client, đăng nhập bằng user `elastic` và password được gen tự động trước đó :
    ```
    http://192.168.5.10:5601
    ```
    <img src=https://i.imgur.com/awGR8h0.png>

- **B12 :** Giao diện quản trị của **Kibana** :

    <img src=https://i.imgur.com/4VHmPcF.png>