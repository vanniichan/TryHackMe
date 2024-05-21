<h2>Task 3 - Working with files</h2>

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/14c60062-f9f7-4c2f-bc82-a6e0f1adbb74)

**Q1**: `mv * /home/francis/logs`

**Q2**: `scp /home/james/Desktop/script.py john@192.168.10.5:/home/john/scripts`

**Q3**: Để đổi tên một file có dấu gạch ngang (-) lúc đầu chúng ta sử dụng hai dấu gạch ngang (- -) trước tên file

`mv -- -logs -newlogs`

**Q4**: `cp "encryption keys" /home/john/logs`

**Q5**: Sử dụng `find / -type f -name readME_hint.txt 2>/dev/null` để tìm đến file

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/34e04b0a-d4df-47d3-bbe2-39a259f320b2)

Sau đó `cat` để để đọc file và ta có nội dung sau:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/355959da-285d-4898-98b7-0e590afc650e)

`ls` ra ta thấy được `-MoveMe.txt` ở đây luôn:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/6a74808e-6aa4-4e62-8562-20bd06af8e57)

Sử dụng `mv -- -MoveMe.txt "'-march folder'"` để chuyển `-MoveMe.txt` vào `'-march folder'` theo yêu cầu:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/31b11344-337d-44bf-a289-8ff30dc9607d)

Sau khi xong thì `./-runME.sh` để lấy flag:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/b82ac9d4-0f0e-4fd8-b7c5-07fac4b0af03)

<h2>Task 4 - Hashing - introduction</h2>

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/ce9c8101-7882-4286-9563-5a6a167137b2)

**Q1**: Tìm và chuyển file vào tong cùng thư mục có `wordlist` để dùng `john-the-ripper`

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/c49dff76-91a8-4dd1-8b41-bd3e4a242fcf)

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/07204182-b3d4-4af6-89e8-d6966d61550b)

Sau đó dùng lệnh như bên dưới và lấy được answer:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/f8c8851a-9acd-40c2-8058-fc739eaa43f2)

**Q2**:

Tương tự **Q1** tìm file và sau đó lấy nội dung, ở đây là mã hash

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/10c49223-a29c-4054-ae6e-bd90bef5eb7b)

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/9b518b5d-06d3-4741-8db9-e3fb9355b9aa)

Đem lên https://crackstation.net/ để crack hash:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/5be05930-1ada-4e17-bdc1-83df2e45055b)

**Q3+Q5**:

Tương tự **Q2** chúng ta sẽ tìm file và lấy nội dung đem lên https://crackstation.net/ để crack hash:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/0e89c4d9-bb28-4fed-b34c-b88463d0fb09)

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/bbdeb646-5240-4600-bb85-8642c93ab129)

**Q4**:

Tìm và đọc nội dung file:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/2e0a88ef-d131-4e84-8089-a79940808ebc)

 **Q4** này có 2 cách để để đưa file từ server ra ngoài. 1 là dùng lệnh theo `Task 3` hoặc 2 là làm theo [WriteUp](https://07kingslayer.medium.com/linux-strength-training-walkthrough-tryhackme-247faa4fa65d) của trang này

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/c8f17ad3-b2d9-4df2-bacb-52e27396b04a)

Sau khi biết được nó là loại hash gì thì tiến hành crack ra:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/8d78609a-1fcc-4c82-a473-079f569ffe2f)
