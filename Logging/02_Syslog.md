# Syslog
## **1) Giới thiệu**
- **Syslog** là một giao thức dùng để xử lý các file log Linux. Các file log có thể được lưu tại chính máy Linux đó, hoặc có thể di chuyển và lưu tại 1 máy khác. (Lý do phải chuyển các log lưu tại nơi khác bạn hãy xem ở phần 4 nhé !).
- Một số đặc điểm của **Syslog** cần lưu ý :
    - **Syslog** có thể gửi qua UDP hoặc TCP.
    - Các dữ liệu log được gửi dạng cleartext.
    - **Syslog** mặc định dùng cổng `514`.
- **Syslog** được phát triển năm `1980` bởi **Eric Allman**, nó là một phần của dự án **Sendmail**, và ban đầu chỉ được sử dụng duy nhất cho **Sendmail**. Nó đã thể hiện giá trị của mình và các ứng dụng khác cũng bắt đầu sử dụng nó. **Syslog** hiện nay trở thành giải pháp khai thác log tiêu chuẩn trên Unix-Linux cũng như trên hàng loạt các hệ điều hành khác và thường được tìm thấy trong các thiết bị mạng như router Trong năm `2009`, **Internet Engineering Task Forec (IETF)** đưa ra chuẩn syslog trong `RFC 5424`.
- **Syslog** ban đầu sử dụng UDP, điều này là không đảm bảo cho việc truyền tin. Tuy nhiên sau đó **IETF** đã ban hành `RFC 3195` (đảm bảo tin cậy cho **syslog**) và `RFC 6587` (truyền tải thông báo **syslog** qua TCP). Điều này có nghĩa là ngoài UDP thì giờ đây **syslog** cũng đã sử dụng TCP để đảm bảo an toàn cho quá trình truyền tin.
- **Syslog** là một giao thức, và được sử dụng bới dịch vụ `Rsyslog`. Dịch vụ `Rsyslog` mới là người đưa ra các quyết định như sử dụng port nào để vận chuyển log, sau bao nhiêu lâu thì log sẽ được rotate,...
## **2) Phân tích cấu hình của Syslog**
- Trong CentOS, file cấu hình là `/etc/rsyslog.conf` . File này chứa cả các rule về log.
- Trong Ubuntu, file cấu hình là `/etc/rsyslog.conf` nhưng các rule được định nghĩa riêng trong `/etc/rsyslog.d/50-default.conf` .
- **VD :** File `/etc/rsyslog.conf` :
    ```conf
    # rsyslog configuration file

    # For more information see /usr/share/doc/rsyslog-*/rsyslog_conf.html
    # If you experience problems, see http://www.rsyslog.com/doc/troubleshoot.html

    #### MODULES ####

    # The imjournal module bellow is now used as a message source instead of imuxsock.
    $ModLoad imuxsock # provides support for local system logging (e.g. via logger command)
    $ModLoad imjournal # provides access to the systemd journal
    #$ModLoad imklog # reads kernel messages (the same are read from journald)
    #$ModLoad immark  # provides --MARK-- message capability

    # Provides UDP syslog reception
    #$ModLoad imudp
    #$UDPServerRun 514

    # Provides TCP syslog reception
    #$ModLoad imtcp
    #$InputTCPServerRun 514


    #### GLOBAL DIRECTIVES ####

    # Where to place auxiliary files
    $WorkDirectory /var/lib/rsyslog

    # Use default timestamp format
    $ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

    # File syncing capability is disabled by default. This feature is usually not required,
    # not useful and an extreme performance hit
    #$ActionFileEnableSync on

    # Include all config files in /etc/rsyslog.d/
    $IncludeConfig /etc/rsyslog.d/*.conf

    # Turn off message reception via local log socket;
    # local messages are retrieved through imjournal now.
    $OmitLocalLogging on

    # File to store the position in the journal
    $IMJournalStateFile imjournal.state


    #### RULES ####

    # Log all kernel messages to the console.
    # Logging much else clutters up the screen.
    #kern.*                                                 /dev/console

    # Log anything (except mail) of level info or higher.
    # Don't log private authentication messages!
    *.info;mail.none;authpriv.none;cron.none                /var/log/messages

    # The authpriv file has restricted access.
    authpriv.*                                              /var/log/secure

    # Log all the mail messages in one place.
    mail.*                                                  -/var/log/maillog


    # Log cron stuff
    cron.*                                                  /var/log/cron

    # Everybody gets emergency messages
    *.emerg                                                 :omusrmsg:*

    # Save news errors of level crit and higher in a special file.
    uucp,news.crit                                          /var/log/spooler

    # Save boot messages also to boot.log
    local7.*                                                /var/log/boot.log


    # ### begin forwarding rule ###
    # The statement between the begin ... end define a SINGLE forwarding
    # rule. They belong together, do NOT split them. If you create multiple
    # forwarding rules, duplicate the whole block!
    # Remote Logging (we use TCP for reliable delivery)
    #
    # An on-disk queue is created for this action. If the remote host is
    # down, messages are spooled to disk and sent when it is up again.
    #$ActionQueueFileName fwdRule1 # unique name prefix for spool files
    #$ActionQueueMaxDiskSpace 1g   # 1gb space limit (use as much as possible)
    #$ActionQueueSaveOnShutdown on # save messages to disk on shutdown
    #$ActionQueueType LinkedList   # run asynchronously
    #$ActionResumeRetryCount -1    # infinite retries if host is down
    # remote host is: name/ip:port, e.g. 192.168.0.1:514, port optional
    #*.* @@remote-host:514
    # ### end of the forwarding rule ###
    ```
    - File cấu hình của syslog cho ta thấy được nơi nơi lưu log của các service cơ bản trong hệ thống. Ví dụ như :
        ```
        cron.*                                                  /var/log/cron
        ```
- Cấu hình **Syslog** như hình trên được chia thành `2` trường:
    - Trường 1: Trường Seletor (số 1) :
        - Trường Seletor : Chỉ ra nguồn tạo ra log và mức cảnh bảo của log đó.
        - Trong trường seletor có 2 thành phần và được tách nhau bằng dấu “.”
    - Trường 2: Trường action (số 2) :
        - Chỉ ra nơi lưu log của tiến trình đó.Có 2 loại là lưu tại file trong localhost hoặc gửi đến IP của Máy chủ Log tập trung
## **3) Các nguồn tạo log (Facility Level)**

| Nguồn tạo log	| Ý nghĩa |
|---------------|---------|
| `kernel` | Những log mà do kernel sinh ra |
| `user` | Log ghi lại cấp độ người dùng |
| `mail` | Log của hệ thống mail |
| `daemon` | Log của các tiến trình trên hệ thống |
| `auth` | Log từ quá trình đăng nhập hệ hoặc xác thực hệ thống |
| `syslog` | Log từ chương trình `syslogd` |
| `lpr` | Log từ quá trình in ấn |
| `news` | Thông tin từ hệ thống |
| `uucp` | Log UUCP subsystem |
| `cron` | Clock deamon |
| `authpriv` | Quá trình đăng nhập hoặc xác thực hệ thống |
| `ftp` | Log của FTP deamon |
| `ntp` | Log từ dịch vụ NTP của các subserver |
| `security` | Kiểm tra đăng nhập |
| `console` | Log cảnh báo hệ thống |
| `local 0 -local 7` | Log dự trữ cho sử dụng nội bộ |

## **4) Mức cảnh báo của Log (Severity Level)**
- Các bản tin log trong hệ thống là vô cùng nhiều. Vì vậy để thuận tiện cho việc phân tích mức độ quan trọng của log, mỗi dòng log đều được gắn một mã cảnh báo, tương ứng mức độ quan trọng của dòng log đó.
- Thống kê số lượng về mức độ cảnh báo của các file log cũng cho chúng ta thấy một phần tình trạng hệ thống. Nếu số lượng log `WARN` và `ERROR` xuất hiện nhiều thì chứng tỏ hệ thống của bạn không ổn một chút nào rồi.

    | Code | Mức cảnh báo | Ý nghĩa |
    |------|--------------|---------|
    | 0	| `emerg` | Thông báo tình trạng khẩn cấp
    | 1	| `alert`	| Hệ thống cần can thiệp ngay
    | 2	| `crit` | Tình trạng nguy kịch
    | 3	| `error`	| Thông báo lỗi đối với hệ thống
    | 4	| `warn` | Mức cảnh báo đối với hệ thống
    | 5	| `notice` | Chú ý đối với hệ thống
    | 6	| `info` | Thông tin của hệ thống
    | 7	| `debug`	| Quá trình kiểm tra hệ thống

- Với các file log do **Syslog** quản lý, ta có thể tùy chỉnh việc lưu các log với mức độ như thế nào. Ví dụ với dịch vụ `mail` :
    - Nếu chỉ muốn lưu các Log với mức độ cảnh báo là `INFO` trở lên (từ mức `6` tới mức `0`) :
        ```conf
        mail.info         /var/log/mail
        ```
    - Nếu chỉ muốn mail ghi các log với mức là `INFO` :
        ```conf
        mail.=info /var/log/mail
        ```
    - Nếu muốn lưu lại tất cả các mức của dịch vụ mail vào log :
        ```conf
        mail.*         /var/log/mail
        ```
    - Nếu muốn lưu lại tất cả, ngoài trừ các log `INFO` :
        ```conf
        mail.!info         /var/log/mail
        ```
## **5) Log Rotation**
- Phần lớn các distro sẽ cài đặt một cấu hình **syslog** mặc định, bao gồm logging to messages và các log files khác trong `/var/log`.
- Để ngăn cản những files này ngày càng trở nên cồng kềnh và khó kiểm soát, một hệ thống quay vòng log file (a log file rotation scheme) nên được cài đặt.
- Hệ thống cron đưa ra các lệnh để thiết lập những log files mới, những file cũ được đổi tên bằng cách thay một con số ở hậu tố.
- Với loại quay vòng này, `/var/log/messages` của ngày hôm qua sẽ trở thành `messages.1` của ngày hôm nay và một messages mới được tạo. Sự luân phiên này được cấu hình cho một số lượng lớn các file, và các log files cũ nhất sẽ được xoá khi sự luân phiên bắt đầu chạy. Ví dụ trong `/var/log` có các messages sau: `messages`, `messages.1`, `messages-20181111`, `messages-20181118`, …
- Tiện ích thi hành rotation là `logrotate`. Lệnh này được cấu hình sử dụng cho một hoặc nhiều files - được xác định bởi các tham số đi cùng.
- File cấu hình mặc định là `/etc/logrotate.conf` :
    ```conf
    # see "man logrotate" for details
    # rotate log files weekly
    weekly

    # keep 4 weeks worth of backlogs
    rotate 4

    # create new (empty) log files after rotating old ones
    create

    # use date as a suffix of the rotated file
    dateext

    # uncomment this if you want your log files compressed
    #compress

    # RPM packages drop log rotation information into this directory
    include /etc/logrotate.d

    # no packages own wtmp and btmp -- we'll rotate them here
    /var/log/wtmp {
        monthly
        create 0664 root utmp
        rotate 1
    }

    /var/log/btmp {
        missingok
        monthly
        create 0600 root utmp
        rotate 1
    }

    # system-specific logs may be also be configured here.
    ```
    - Trong file cấu hình trên :
        - Hệ thống sẽ quay vòng log files hàng tuần
        - Lưu lại những thông tin logs đáng giá trong 4 tuần
        - Sử dụng định dạng Ngày tháng thêm vào để làm hậu tố của log files (`20181111`, `20181118`, ...)
        - Thông tin về sự quay vòng log của các gói RPM nằm trong `/etc/logrotate.d`
        - Rotation được thiết lập cho 2 files: `/var/log/wtmp` và `/var/log/btmp` .