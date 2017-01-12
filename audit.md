#Audit - Công cụ kiểm toán
Audit - a set of rules loaded in the kernel audit system
##Mục đích 
Audit có thể theo dõi nhiều loại sự kiện để giám sát và kiểm toán hệ thống. Ví dụ như:
* Audit file access and modification: 
	<ul>
  	<li>Xem ai truy cập một tập tin cụ thể</li>
  	<li>Phát hiện thay đổi trái phép</li></ul>	
* Giám sát cuộc gọi hệ thống và các function
* Phát hiện các bất thường như  crashing processes
* Set tripwires cho mục đích phát hiện xâm nhập
* Ghi các lệnh được sử dụng bởi người dùng cá nhân

##Các thành phần
###kernel:
<li>**audit**: can thiệp vào kernel để bắt các sự kiện và cung cấp cho auditd</li>
###binaties:
<li>**auditd** : daemon để bắt các sự kiện và lưu trữ chúng (log file)</li>
<li>**auditctl** : công cụ client để cấu hình auditd</li>
<li>**audispd** : daemon cho đa sự kiện</li>
<li>**aureport** : công cụ báo cáo những thứ đọc từ tập tin log (auditd.log)</li>
<li>**ausearch** : xem sự kiện (auditd.log)</li>
<li>**autrace** : sử dụng thành phần audit trong kernel để theo dõi những chương trình</li>
<li>**aulast** : similar to last, but instaed using audit framework</li>
<li>**aulastlog** : similar to lastlog, also using audit framework instead</li>
<li>**ausyscall** : map syscall ID and name</li>
<li>**auvirt** : hiển thị thông tin audit liên quan đến  virtual machines</li>

###Các tập tin: 
<li>**audit.rules**: được sử dụng bởi auditctl để đọc những gì rules cần phải được sử dụng</li>
<li>**auditd.conf**: file cấu hình của auditd</li>

##Installation
Debian/Ubuntu: `apt-get install auditd audispd-plugins`

Red Hat/CentOS/Fedora: thường được cài đặt sẵn (package: audit and audit-libs)

##Configuration
Các cấu hình của trình nền kiểm toán được sắp xếp bởi hai tập tin, một cho các daemon chính nó ( auditd.conf ) và một cho các quy tắc sử dụng bởi auditctl tool( audit.rules ).

Như với hầu hết mọi thứ, sử dụng một khởi đầu sạch sẽ và không có bất kỳ quy tắc nạp. Quy tắc hoạt động có thể được xác định bằng cách chạy auditctl với tham số -l: ` auditctl -l `

Thời gian để bắt đầu với giám sát vài thứ, hãy nói những file / etc / passwd. Chúng tôi đặt một 'watch' trên các file bằng cách định nghĩa path và cho phép để tìm kiếm:

`auditctl -a exit,always -F path=/etc/passwd -F perm=wa`

Bốn option:
* r = read
* w = write
* x = execute
* a = attribute change

##Cấu hình đẩy log từ audit bằng rsyslog
### Trên server
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

###Trên client
####Cấu hình audit
* Cấu hình audit tạo log ghi lại lịch sử người dùng
Thêm đoạn sau vào trong /etc/audit/audit.rules
```
-a exit,always -F arch=b64 -S execve
-a exit,always -F arch=b32 -S execve
```
Sau đó restart audit bằng lệnh: `/etc/init.d/auditd restart`

* Chuyển log của audit log sang rsyslog trong file `/var/log/syslog`

Trong file `/etc/audisp/plugins.d/syslog.conf ` sửa *active = no* sang *yes*

<img src = "https://github.com/trangnth/Audit-Ubuntu-14.04/blob/master/img/audit-conf.png">

####Cấu hình rsyslog
* Cấu hình file /etc/rsyslog.d/60-output.conf

Thêm dòng sau:

<img src = "https://github.com/trangnth/Audit-Ubuntu-14.04/blob/master/img/60-output.png">

Dấu `*` thứ nhất thể hiện đẩy log từ tất cả các nguồn sinh log, dấu `*` thứ 2 thể hiện log ở tất cả các mức độ cảnh báo.

Nếu là một mức nào đó thì sẽ là đẩy tất cả các log có mức cảnh báo bằng mức đó và cao hơn.

Nếu muốn bỏ một mức nào đó thì thêm `!` ở trước mức độ đó.

@196.168.169.135:514 là địa chỉ server được đẩy đến bằng UDP theo cổng 514 (nếu là TCP thì trước địa chỉ server là @@).

Restart rsyslog: `service rsyslog restart`

####Kết quả:

Cuối cùng các log sẽ được lưu trên server theo đường dẫn đã được cấu hình.

Ví dụ: `/data/log/ubuntu` "ubuntu" là tên hostname của máy client

<img src = "https://github.com/trangnth/Audit-Ubuntu-14.04/blob/master/img/server2.png">

Log audit lưu trong file audispd.log (tên chương trình trong syslog)

<img src = "https://raw.github.com/trangnth/Audit-Ubuntu-14.04/master/img/server3.png">

**Chú ý:** trong bài này là đẩy tất cả các log, log được lưu trong audispd trên server là log của một chương trình trên syslog được tạo ra từ audispd.

http://serverfault.com/questions/202044/sending-audit-logs-to-syslog-server
