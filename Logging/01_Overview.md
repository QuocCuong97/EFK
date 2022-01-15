# Tổng quan về Log
## **1) Giới thiệu**
- File **log** của hệ thống giống như những quyển sổ nhật ký, ghi lại toàn bộ quá trình hoạt động của hệ thống.
- Những vai trò dễ nhận thấy của **log** là :
    - Troubleshoot  trong quá trình cài đặt các service.
    - Tra cứu nhanh các thông tin của hệ thống.
    - Truy vết các event đã và đang xảy ra.
## **2) Phân loại log trong Linux**
### **2.1) Message log : `/var/log/messages`**
- Chứa dữ liệu log của hầu hết các thông báo hệ thống nói chung, bao gồm cả các thông báo trong quá trình khởi động hệ thống. Ví dụ như, khi bạn restart service network, sẽ có các log về networking xuất hiện. 
- Sử dụng câu lệnh sau để xem các file log được realtime :
    ```
    tail -f /var/log/messages
    ```
    ```bash
    Nov  9 21:32:51 elk systemd: Stopping LSB: Bring up/down networking...
    Nov  9 21:32:52 elk network: Shutting down interface ens160:  [  OK  ]
    Nov  9 21:32:52 elk network: Shutting down interface ens192:  [  OK  ]
    Nov  9 21:32:52 elk network: Shutting down loopback interface:  [  OK  ]
    Nov  9 21:32:52 elk systemd: Starting LSB: Bring up/down networking...
    Nov  9 21:32:53 elk network: Bringing up loopback interface:  [  OK  ]
    Nov  9 21:32:53 elk kernel: vmxnet3 0000:03:00.0 ens160: intr type 3, mode 0, 3 vectors allocated
    Nov  9 21:32:53 elk kernel: vmxnet3 0000:03:00.0 ens160: NIC Link is Up 10000 Mbps
    Nov  9 21:32:57 elk network: Bringing up interface ens160:  [  OK  ]
    Nov  9 21:32:57 elk kernel: vmxnet3 0000:0b:00.0 ens192: intr type 3, mode 0, 3 vectors allocated
    Nov  9 21:32:57 elk kernel: vmxnet3 0000:0b:00.0 ens192: NIC Link is Up 10000 Mbps
    Nov  9 21:33:01 elk network: Bringing up interface ens192:  [  OK  ]
    Nov  9 21:33:01 elk systemd: Started LSB: Bring up/down networking.
    ```
    - Trong file log message thể hiện rõ luồng xử lý của việc restart networking như sau :
        - Shutdown dịch vụ networking :
            ```
            Nov  9 21:32:51 elk systemd: Stopping LSB: Bring up/down networking...
            ```
        - Các interface bị down xuống :
            ```
            Nov  9 21:32:52 elk network: Shutting down interface ens160:  [  OK  ]
            Nov  9 21:32:52 elk network: Shutting down interface ens192:  [  OK  ]
            Nov  9 21:32:52 elk network: Shutting down loopback interface:  [  OK  ]
            Nov  9 21:32:52 elk systemd: Stopped LSB: Bring up/down networking.
            ```
        - Dịch vụ network được bật lên :
            ```
            Nov  9 21:32:53 elk network: Bringing up loopback interface:  [  OK  ]
            ```
        - Các interface được bật lên :
            ```
            Nov  9 21:32:53 elk network: Bringing up loopback interface:  [  OK  ]
            Nov  9 21:32:53 elk kernel: vmxnet3 0000:03:00.0 ens160: intr type 3, mode 0, 3 vectors allocated
            Nov  9 21:32:53 elk kernel: vmxnet3 0000:03:00.0 ens160: NIC Link is Up 10000 Mbps
            Nov  9 21:32:57 elk network: Bringing up interface ens160:  [  OK  ]
            Nov  9 21:32:57 elk kernel: vmxnet3 0000:0b:00.0 ens192: intr type 3, mode 0, 3 vectors allocated
            Nov  9 21:32:57 elk kernel: vmxnet3 0000:0b:00.0 ens192: NIC Link is Up 10000 Mbps
            Nov  9 21:33:01 elk network: Bringing up interface ens192:  [  OK  ]
            ```
        - Dịch vụ network xác thực đã được bật :
            ```
            Nov  9 21:33:01 elk systemd: Started LSB: Bring up/down networking.
            ```
    - Những dòng log trên cho chúng ta 2 cách nhìn :
        - Nếu như có lỗi liên quan tới network, việc đầu tiên cần làm là xem log trong file message.
        - Nếu cứ có các dòng log trên xuất hiện, thì nghĩa là đã có event Networking đã restart.
### **2.2) Crontab log : `/var/log/cron`**
- File log cron chứa dữ liệu log của cron deamon. Bắt đầu và dừng cron cũng như cronjob thất bại. Crob deamon là tiến trình giúp chúng ta thực hiện một hành động trên hệ thống một cách tự động theo mốc thời gian cố định.
- **VD :** Khi chúng ta thực hiện chạy một crontab thực hiện việc backup database với thời gian là 2h / lần. Sử dụng `crontab -e` để xem nội dung setup crontab :
    ```sh
    0 */2 * * * /backup/backup_database.sh
    ```
    - Khi đó, trong file log `/var/log/cron` sẽ ghi lại quá trình crobtab được thực hiện. Sử dụng câu lệnh sau để lọc thông tin về việc chạy cron cho file `/backup/backup_database.sh` :
        ```
        cat /var/log/cron  | grep /backup/backup_database.sh
        ```
        ```
        Jan 24 00:00:01 controller1 CROND[34033]: (root) CMD (/backup/backup_database.sh)
        Jan 24 02:00:01 controller1 CROND[24735]: (root) CMD (/backup/backup_database.sh)
        Jan 24 04:00:01 controller1 CROND[14661]: (root) CMD (/backup/backup_database.sh)
        Jan 24 06:00:01 controller1 CROND[3334]: (root) CMD (/backup/backup_database.sh)
        Jan 24 08:00:01 controller1 CROND[57850]: (root) CMD (/backup/backup_database.sh)
        Jan 24 10:00:01 controller1 CROND[46964]: (root) CMD (/backup/backup_database.sh)
        Jan 24 12:00:01 controller1 CROND[37449]: (root) CMD (/backup/backup_database.sh)
        Jan 24 14:00:01 controller1 CROND[28280]: (root) CMD (/backup/backup_database.sh)
        Jan 24 16:00:01 controller1 CROND[19629]: (root) CMD (/backup/backup_database.sh)
        Jan 24 18:00:01 controller1 CROND[10219]: (root) CMD (/backup/backup_database.sh)
        Jan 24 20:00:01 controller1 CROND[723]: (root) CMD (/backup/backup_database.sh)
        Jan 24 22:00:01 controller1 CROND[57462]: (root) CMD (/backup/backup_database.sh)
        Jan 25 00:00:01 controller1 CROND[48920]: (root) CMD (/backup/backup_database.sh)
        ```
    - Ta có thể thấy là việc chạy file `/backup/backup_database.sh` được diễn ra đều đặn 2 tiếng 1 lần.
### **2.3) Log về Login - Logout : `/var/log/wtmp`**
- File log `wtmp` chứa tất cả các đăng nhập và đăng xuất lịch sử.
- Tuy nhiên, file log `wtmp` có định dạng khá đặc biệt. Để đọc được file log này, chúng ta sử dụng câu lệnh sau :
    ```
    utmpdump /var/log/wtmp | less
    ```
    ```
    Utmp dump of /var/log/wtmp
    [8] [20302] [    ] [     ] [pts/0 ] [             ] [0.0.0.0      ] [Sat Nov 10 20:21:59 2018 +07]
    [7] [20500] [ts/0] [root ] [pts/0 ] [27.72.59.xxx ] [27.72.59.xxx ] [Sat Nov 10 20:22:14 2018 +07]
    [8] [20441] [    ] [     ] [pts/1 ] [             ] [0.0.0.0      ] [Sat Nov 10 20:22:20 2018 +07]
    ```
    - Mỗi khi có người dùng logout khỏi hệ thống, định dạng log sẽ như sau :
        ```
        [8] [20302] [    ] [     ] [pts/0 ] [        ] [0.0.0.0] [Sat Nov 10 20:21:59 2018 +07]
        ```
    - Còn đây là định dạng log mỗi khi có người login vào hệ thống :
        ```
        [7] [20500] [ts/0] [root ] [pts/0 ] [27.72.59.xxx ] [27.72.59.xxx ] [Sat Nov 10 20:22:14 2018 +07]
        ```
        > Trong đó, IP `27.72.59.xxx` là IP của người dùng đang login vào hệ thống.
### **2.4) Log phần cứng : `/var/log/dmesg`**
- Chứa thông tin bộ đệm kernel ring. Khi hệ thống khởi động, file log sẽ chứa thông tin về các thiết bị phần cứng mà kernel phát hiện được. Các message này có sẵn trong kernel ring buffer và bất cứ khi nào có message mới xuất hiện, message sẽ bị ghi đè. Bạn cũng có thể xem nội dung của tệp này bằng lệnh `dmesg`.
- Chúng ta hãy thử xem `10` dòng log cuối cùng trong file log `dmesg` mỗi khi một máy ảo trên hệ thống Cloud Nhân Hòa được tạo :
    ```
    tail -n 10 /var/log/dmesg
    ```
    ```sh
    [    6.542930] systemd[1]: Inserted module 'ip_tables'
    [    3.358422]  vda: vda1
    [    7.014517] EXT4-fs (vda1): re-mounted. Opts: (null)
    [    7.107034] systemd-journald[1355]: Received request to flush runtime journal from PID 1
    [    2.449704] systemd[1]: Starting Journal Service...
    [    2.455997] systemd[1]: Starting Create list of required static device nodes for the current kernel...
    [    2.465960] systemd[1]: Starting Setup Virtual Console...
    [    2.472855] systemd[1]: Starting Apply Kernel Variables...
    [    7.583101] piix4_smbus 0000:00:01.3: SMBus Host Controller at 0x700, revision 0
    [    7.773026] input: PC Speaker as /devices/platform/pcspkr/input/input5
    ```
- Ta hãy tập trung vào một số file log liên quan tới phần cứng như :
    ```sh
    [    3.358422]  vda: vda1
    [    7.014517] EXT4-fs (vda1): re-mounted. Opts: (null)
    ...
    [    7.583101] piix4_smbus 0000:00:01.3: SMBus Host Controller at 0x700, revision 0
    ```
    > 2 log trên cho chúng ta thấy thông tin về ổ cứng và SMBus của máy ảo được in ra khi máy ảo được khởi tạo.
### **2.5) Log về SSH : `/var/log/secure`**
- Tiếp theo, chúng ta hãy cùng phân tích một file log rất quan trọng và mang lại nhiều thông tin hữu ích. Đó chính là file log secure chứa các nội dung về việc SSH tới hệ thống. Việc SSH tới hệ thống có thể chia thành 3 trường hợp chính : login thành công, login thất bại và logout.
#### **2.5.1) Login SSH thành công**
- Mỗi khi user SSH thành công vào hệ thống, các dòng log sau sẽ xuất hiện tại `/var/log/secure`.
    ```
    Jan 18 15:39:30 web sshd[14838]: Accepted password for duydm from 27.72.59.xxx port 49572 ssh2
    Jan 18 15:39:30 web sshd: Started Session 3057 of user duydm
    ```
    - Từ log SSH thành công, ta có thể phân tích thành các dữ liệu sau :
        - User login SSH (`duydm`)
        - IP login SSH (`27.72.59.xxx`)
        - Source Port của tiến trình SSH (`49572`)
        - Kết quả việc SSH (`Accepted password`)
#### **2.5.2) Login SSH thất bại**
- Nội dung log :
    ```sh
    Nov 10 20:54:32 elk sshd[26205]: Failed password for root from 218.92.1.148 port 21450 ssh2
    Nov 10 20:54:32 elk sshd[26205]: Received disconnect from 218.92.1.148 port 21450:11:  [preauth]
    Nov 10 20:54:32 elk sshd[26205]: Disconnected from 218.92.1.148 port 21450 [preauth]
    ```
- Về cơ bản, các thông tin được sàng lọc ra vẫn giống với khi có login SSH thành công. Tuy nhiên, kết quả việc SSH sẽ khác :
    - Kết quả việc SSH : `Failed password`
#### **2.5.3) Logout SSH**
- Và cuối cùng, mỗi khi user login phiên SSH thì các file log sẽ có các thông tin sau :
    ```sh
    Nov 10 21:06:46 elk sshd[28506]: Received disconnect from 27.72.59.xxx port 56671:11: disconnected by user
    Nov 10 21:06:46 elk sshd[28506]: Disconnected from 27.72.59.xxx port 56671
    Nov 10 21:06:46 elk sshd[28506]: pam_unix(sshd:session): session closed for user root
    Nov 10 21:06:47 elk sshd[28511]: Connection closed by 27.72.59.xxx port 56673 [preauth]
    ```
## **3) Các câu lệnh đọc log hay dùng**
- Xem file log realtime với `tail -f` :
    ```
    tail -f /var/log/messages
    ```
- Lọc thông tin với `grep` :
    ```
    cat /var/log/cron  | grep /backup/backup_database.sh
    ```
- Đọc định dạng file `wtmp` với `utmpdump`, và sử dụng `less` để hiện thị một phần của file :
    ```
    utmpdump /var/log/wtmp | less
    ```
- In ra màn hình `10` dòng log cuối cùng với `tail -n 10` . Thay `10` với số dòng bạn muốn in ra :
    ```
    tail -n 10 /var/log/dmesg
    ```
- In ra tất cả các file log có chứa cụm từ `duydm` trong thư mục `/var/log/` với câu lệnh :
    ```
    grep -Rin "duydm" /var/log/
    ```