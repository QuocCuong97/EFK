# Cài đặt ELK Stack
## **Mô hình**
## **Các bước cài đặt**
### **Cài đặt ElasticSearch**
- **B1 :** Cài đặt **Java OpenJDK** :
    ```
    # yum -y install java-openjdk-devel java-openjdk
    ```
- **B2 :** Thêm repository của **ElasticSearch** :
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
- **B3 :** Update các repo :
    ```
    yum update -y
    ```
- **B4 :** Import GPG Key :
    ```
    # rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
    ```    
- **B5 :** Cài đặt **ElasticSearch** :
    ```
    yum install elasticsearch -y
    ```
- **B6 :** Cấu hình các JVM options như memory limit trong file `jvm.options` :
    ```
    # vi /etc/elasticsearch/jvm.options
    ```
    - Chỉnh sửa các thông số `Xms` và `Xmx` theo thứ tự là initial/maximum size của tổng lượng heap memory :
        <img src=https://i.imgur.com/H0Lu85F.png>

- **B7 :** Chỉnh sửa `network.host`, `discovery.seed_hosts`, `cluster.initial_master_nodes` trong file `elasticsearch.yml` :
    ```
    # vi /etc/elasticsearch/elasticsearch.yml
    ```
    <img src=https://i.imgur.com/0fqyLDD.png>

    > Trong đó : `192.168.5.10` là IP để truy cập dịch vụ.
- **B8 :** Cấu hình `firewalld` cho phép port `9200` :
    ```
    # firewall-cmd --zone=public --permanent --add-port=9200/tcp
    # firewall-cmd --reload
    ```
- **B9 :** Khởi động **ElasticSearch** và cho phép dịch vụ khởi động cùng hệ thống :
    ```
    # systemctl start elasticsearch
    # systemctl enable elasticsearch
    ```
- **B10 :** Kiểm tra dịch vụ :
    ```
    # curl -X GET http://localhost:9200
    ```