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


