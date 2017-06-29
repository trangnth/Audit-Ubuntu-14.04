# Audit 

## 1. Overview
### Mục đích 
Audit có thể theo dõi nhiều loại sự kiện để giám sát và kiểm toán hệ thống. Ví dụ như:
* Thống kê các truy cập và sửa đổi tệp tin: 
	<ul>
  	<li>Xem ai truy cập một tập tin cụ thể</li>
  	<li>Phát hiện các thay đổi trái phép</li>
	</ul>	
* Giám sát system call và function
* Phát hiện các bất thường như  crashing processes
* Phát hiện xâm nhập
* Ghi các lệnh được sử dụng bởi người dùng cá nhân

### Các thành phần
#### kernel:
<ul>
<li>audit: kiểm soát tương tác kernel</li>
</ul>

#### binaties:
<ul>
<li>auditd : daemon lưu trữ các event (log file)</li>
<li>auditctl : là một công cụ để kiểm soát hành vi của các daemon,..</li>
<li>audispd : là một công cụ có thể được sử dụng để chuyển tiếp các thông báo sự kiện cho các ứng dụng khác thay vì viết chúng vào disk trong audit log</li>
<li>aureport : là công cụ để tạo ra và xem báo cáo audit (auditd.log)</li>
<li>ausearch : tìm kiếm và xem các sự kiện (auditd.log)</li>
<li>autrace : là một lệnh có thể được sử dụng để theo dõi một quá trình, sử dụng thành phần audit trong kernel để theo dõi những chương trình</li>
<li>aulast : similar to last, but instaed using audit framework</li>
<li>aulastlog : similar to lastlog, also using audit framework instead</li>
<li>ausyscall : map syscall ID and name</li>
<li>auvirt : hiển thị thông tin audit liên quan đến  virtual machines</li>
</ul>
	
### Các tập tin: 

<li>audit.rules: được sử dụng bởi auditctl để đọc những gì rules cần phải được sử dụng</li>
<li>auditd.conf: file cấu hình của auditd</li>


## 2. Installation
Debian/Ubuntu: `apt-get install auditd audispd-plugins`

Red Hat/CentOS/Fedora: thường được cài đặt sẵn (package: audit and audit-libs)

## 3. Configuration

Có hai file cấu hình, một cho các daemon của chính nó ( auditd.conf ) và một cho các quy tắc được sử dụng bởi auditctl tool( audit.rules ).

Ban đầu sẽ không có bất cứ một rule nào được cấu hình. Xem các rule đang chạy bằng cách chạy auditctl với tham số -l: ` auditctl -l `, nếu muốn xóa tất cả thì sử dụng tham số `-D`

Bây giờ có thể theo dõi bất cứ file nào, ví dụ file /etc/passwd. Ta sẽ đặt một 'watch' trên các file bằng cách định nghĩa một rule như sau, gõ lệnh:

`auditctl -a exit,always -F path=/etc/passwd -F perm=wa`

Bốn option:
* r = read
* w = write
* x = execute
* a = attribute change
Gõ:

	[root@localhost ~]# auditctl -l
	-w /etc/passwd -p wa

Sau đó, sẽ có báo cáo nếu có bất kỳ thay đổi nào trong file /etc/passwd 
	
	[root@localhost ~]# ausearch -f /etc/passwd
	----
	time->Thu Jun 29 11:05:23 2017
	type=CONFIG_CHANGE msg=audit(1498709123.287:1261036): auid=1000 ses=47672 op="updated_rules" path="/etc/passwd" key=(null) list=4 res=1
	----
	time->Thu Jun 29 11:05:23 2017
	type=PATH msg=audit(1498709123.287:1261037): item=3 name="/etc/passwd~" inode=5027664 dev=fd:00 mode=0100644 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:passwd_file_t:s0 objtype=CREATE
	type=PATH msg=audit(1498709123.287:1261037): item=2 name="/etc/passwd" inode=5027664 dev=fd:00 mode=0100644 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:passwd_file_t:s0 objtype=DELETE
	type=PATH msg=audit(1498709123.287:1261037): item=1 name="/etc/" inode=4194369 dev=fd:00 mode=040755 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:etc_t:s0 objtype=PARENT
	type=PATH msg=audit(1498709123.287:1261037): item=0 name="/etc/" inode=4194369 dev=fd:00 mode=040755 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:etc_t:s0 objtype=PARENT
	type=CWD msg=audit(1498709123.287:1261037):  cwd="/root"
	type=SYSCALL msg=audit(1498709123.287:1261037): arch=c000003e syscall=82 success=yes exit=0 a0=14d2390 a1=168b650 a2=fffffffffffffe90 a3=7ffe4ffffc50 items=4 ppid=28750 pid=17557 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=47672 comm="vim" exe="/usr/bin/vim" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key=(null)
	----
	time->Thu Jun 29 11:05:23 2017
	type=CONFIG_CHANGE msg=audit(1498709123.288:1261038): auid=1000 ses=47672 op="updated_rules" path="/etc/passwd" key=(null) list=4 res=1
	----

	
### Cấu hình đẩy log từ audit bằng rsyslog
## Trên server
#### Cấu hình cho server nhận log theo UDP trên cổng 514

Trên file /etc/rsyslog.conf bỏ dấu `#` 2 dòng như hình dưới

<img src = "https://github.com/trangnth/Audit-Ubuntu-14.04/blob/master/img/rsyslog-conf-server-udp.png">

#### Cấu hình vị trí lưu các log nhận được
Thêm các dòng như sau vào file /etc/rsyslog.conf:

```
$template TmplAuth,"/data/log/%HOSTNAME%/%PROGRAMNAME%.log"
Auth.*  ?TmplAuth
*.*    ?TmplAuth
& ~
```
<img src = "https://github.com/trangnth/Audit-Ubuntu-14.04/blob/master/img/rsyslog-conf-server.png">

## Trên client
#### Cấu hình audit
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

#### Cấu hình rsyslog
* Cấu hình file /etc/rsyslog.d/60-output.conf

Thêm dòng sau:

<img src = "https://github.com/trangnth/Audit-Ubuntu-14.04/blob/master/img/60-output.png">

`@196.168.169.135:514` là địa chỉ server được đẩy đến bằng UDP theo cổng 514 (nếu là TCP thì trước địa chỉ server là @@).

Restart rsyslog: `service rsyslog restart`

### Kết quả:

Cuối cùng các log sẽ được lưu trên server theo đường dẫn đã được cấu hình.

Ví dụ: `/data/log/ubuntu` "ubuntu" là tên hostname của máy client

<img src = "https://github.com/trangnth/Audit-Ubuntu-14.04/blob/master/img/server2.png">

Log audit lưu trong file audispd.log (tên chương trình trong syslog)

<img src = "https://raw.github.com/trangnth/Audit-Ubuntu-14.04/master/img/server3.png">

Chẳng hạn khi ssh đến client và gõ một lệnh `tail -40 /var/log/syslog` log sẽ được tạo ra như hình dưới đây:

<img src = "https://github.com/trangnth/Audit-Ubuntu-14.04/blob/master/img/LOG.png">

Thông thường, khi chạy một lệnh log sẽ được tạo ra với 5 type (SYSCALL, EXECVE, CWD, PATH, EOE)

- Dòng `type=SYSCALL` chứa các thông tin về người dùng (ví dụ uid - User ID, pid - Process ID,  gid - Group ID,...)
- Dòng `type=EXECVE` cho biết lệnh đã được chạy => `argc=3 a0="tail" a1="-40" a2="/var/log/syslog"`
- Dòng `type=CWD` cho biết vị trí gõ lệnh => lệnh được thao tác trên thư mục `/root`

Chi tiết hơn về các tham số trên http://manpages.ubuntu.com/manpages/precise/man8/auditctl.8.html

## Tham khảo:
http://serverfault.com/questions/202044/sending-audit-logs-to-syslog-server

https://linux-audit.com/configuring-and-auditing-linux-systems-with-audit-daemon/

