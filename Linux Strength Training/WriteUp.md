<h2>Task 3 - Working with files</h2>

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
