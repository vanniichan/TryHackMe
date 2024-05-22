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

<h2>Task 5 - Decoding base64</h2>

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/91b3cc65-8a6c-44fa-bbc4-d067b74f3faa)

<h2>Task 6 - Encryption/Decryption using gpg</h2>

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/75977548-77c5-4b9e-8b2e-5d776d6b16d6)

**Q2**: `gpg --cipher-algo AES-128 --symmetric history_logs.txt`

**Q3**: `gpg history_logs.txt.gpg`

**Q4**:

Tìm và vào thư mục có `layer4.txt`:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/e1a8e10c-5ba3-4742-9ae4-40843ac6294e)

Sử dụng `gpg layer4.txt` để lấy decrypt file:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/042bbabc-cdb1-4f86-ab8a-07e6efd9a860)

Ta sẽ lấy được nội dung, tuy nhiên nó cứ dẫn sang các layer khác và ta cũng làm tương tự:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/81059c68-77c3-4d2a-a129-3b6cda246f18)

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/0244d53e-d54d-4e7e-b4d8-dfda5d0e175b)

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/afe3ccf5-ebdc-4a91-9739-fef56b1a6a48)

Về đến `layer1.txt` thì lấy được flag : ) :

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/f2f4796e-b015-4e67-8a5b-b174a41f2b3d)

<h2>Task 7 - Cracking encrypted gpg files</h2>

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/7b6dc6af-17f5-4ece-bb8f-cd562c876419)

**Q2**:

Tìm 2 file theo yêu cầu đề bài:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/c6feb2af-3b34-42af-b031-05f9c63037a5)

Tuy nhiên server nầy không hỗ trợ nên lại phải làm như `Task 5` là lấy file về máy để dùng:

`gpg2john personal.txt.gpg > hash.txt`

Đợi lâu vcut để lấy được password: `valamanezivonia`:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/56128078-b92c-423d-857e-6d3e9387a62d)

**Q3**:

`gpg personal.txt.gpg` với password là `valamanezivonia` sau đó cat file ra và lấy được nội dung file:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/2d5893d2-482b-4c34-b9d8-2f7101d3a036)

<h2>Task 8 - Reading SQL databases</h2>


<h2>Task 9 - Reading SQL databases</h2>

**Q2**:

Sau khi theo `Q1` thì khá là bị bịp nếu không đọc hint. Sử dụng lệnh dưới để tìm đến đoạn hội thoại chính xác để lấy key:

```
grep -iRl "Sameer" /home 2>/dev/null

///-i được sử dụng để tìm kiếm không phân biệt chữ hoa chữ thường, -R là tìm kiếm đệ quy trong tất cả các tệp và -l chỉ in các tệp khớp với từ tôi đang tìm kiếm
```
![image](https://github.com/vanniichan/TryHackMe/assets/112863484/e9cc7272-d26f-43ec-bc66-d3835ed2f1e5)

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/5c7d2a69-b25e-4c04-b92c-fcbc4b7987b5)

**Q3**:

Quay lại đoạn hội thoại tìm được từ `Q2` ta thấy có 1 file có size 50M chứa key:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/18c98d2d-b23c-4687-a102-50d4b1de88c8)

Sử dụng `ls -lah`với h là in ra dung lượng tệp:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/1d0abbf9-bfcc-419f-adae-0dca94e19ef2)

Lấy được nội dung đem đi dcode và tấy láy được dir của file wordlist:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/24c1dcdc-c9c6-4c90-b09f-c5ae80605a6a)

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/aba757b4-1328-4873-9a9a-5190d2525b82)


