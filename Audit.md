#Audit 

##1. Installation
Debian/Ubuntu: `apt-get install auditd audispd-plugins`

Red Hat/CentOS/Fedora: thường được cài đặt sẵn (package: audit and audit-libs)

##2. Configuration

###Cấu hình đẩy log từ audit bằng rsyslog
##a. Trên server
####Cấu hình cho server nhận log theo UDP trên cổng 514

Trên file /etc/rsyslog.conf untag 2 dòng như hình dưới

<img src = "https://github.com/trangnth/Audit-Ubuntu-14.04/blob/master/img/rsyslog-conf-server-udp.png">

####Cấu hình vị trí lưu các log nhận được
Thêm các dòng như sau vào file /etc/rsyslog.conf:

```
$template TmplAuth,"/data/log/%HOSTNAME%/%PROGRAMNAME%.log"
Auth.*  ?TmplAuth
*.*    ?TmplAuth
& ~
```
<img src = "https://github.com/trangnth/Audit-Ubuntu-14.04/blob/master/img/rsyslog-conf-server.png">

##b. Trên client
####Cấu hình audit
* Cấu hình audit tạo log ghi lại lịch sử người dùng
Thêm đoạn sau vào trong /etc/audit/audit.rules
```
-a exit,always -F arch=b64 -S execve
-a exit,always -F arch=b32 -S execve
```

* Chuyển log của audit log sang rsyslog trong file `/var/log/syslog`

Trong file `/etc/audisp/plugins.d/syslog.conf ` sửa *active = no* sang *yes*

<img src = "https://github.com/trangnth/Audit-Ubuntu-14.04/blob/master/img/audit-conf.png">

Sau đó restart audit bằng lệnh: `/etc/init.d/auditd restart`

####Cấu hình rsyslog
* Cấu hình file /etc/rsyslog.d/60-output.conf

Thêm dòng sau:

<img src = "https://github.com/trangnth/Audit-Ubuntu-14.04/blob/master/img/60-output.png">

`@196.168.169.135:514` là địa chỉ server được đẩy đến bằng UDP theo cổng 514 (nếu là TCP thì trước địa chỉ server là @@).

Restart rsyslog: `service rsyslog restart`

####Kết quả:

Cuối cùng các log sẽ được lưu trên server theo đường dẫn đã được cấu hình.

Ví dụ: `/data/log/ubuntu` "ubuntu" là tên hostname của máy client

<img src = "https://github.com/trangnth/Audit-Ubuntu-14.04/blob/master/img/server2.png">

Log audit lưu trong file audispd.log (tên chương trình trong syslog)

<img src = "https://raw.github.com/trangnth/Audit-Ubuntu-14.04/master/img/server3.png">


*Tham khảo:* http://serverfault.com/questions/202044/sending-audit-logs-to-syslog-server
