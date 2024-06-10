![image](https://github.com/vanniichan/TryHackMe/assets/112863484/470d1223-654e-4bc1-b958-7fdf4d675542)

<h2>Task 1,2 - Begin</h2>

**Q1:** admin@juice-sh.op

**Q2:** q

**Q3:** Star Trek

<h2>Task 3 - Inject the juice</h2>

Sử dụng SQLi cơ bản để lấy được flag: ` ' or 1=1 --`

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/189255d7-301a-4db0-8c7b-e6bc96b13fde)

**Q1:** 32a5e0f21372bcc1000a6088b93b458e41f0e02a

Làm tương tự để login k cần password ta được flag:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/9355a3b4-650d-435e-8e72-16ea605db1ab)

**Q2:** fb364762a3c102b2db932069c0e6b78e738d4066

<h2>Task 4 - Who broke my lock?!</h2>

**Q1:** c2110d06dc6f81c67cd8099ff0ba601241f1ac0e

Bài này weak cridental nên chỉ cần brute-force pass là lấy được flag:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/f7eb1346-9327-4f68-b65e-3840dcbe7cbf)

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/351385dc-8555-408e-86d1-32627c9d4792)

**Q2:** 094fbc9b48e525150ba97d05b942bbf114987257

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/45b22df6-5d37-4881-b288-d35adec35d54)

Câu hỏi để reset password có thể đoán ra được luôn

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/5179975b-4bd9-46af-8cfe-3559ce43d696)

<h2>Task 5 - AH! Don't look!</h2>

**Q1:** edf9281222395a1c5fee9b89e32175f1ccf50c5b

Nhiều khi đường dẫn của các trang web không được ẩn dẫn đến lộ đường dẫn chứa các file nhạy cảm 

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/447ea8e6-3a93-4e90-811b-d7d6959fd691)

Đường dẫn không được bật ẩn sẽ chứa các file:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/cc5a19de-79b2-4366-bd7d-ee0dac5f0b6c)

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/9eef9e4a-7a9d-4d59-98a6-41a1175af2be)

**Q2:** 66bdcffad9e698fd534003fbb3cc7e2b7b55d7f0

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/3ad0ba61-a59b-49a6-b22f-7b33b17552d4)

**Q3:** bfc1e6b4a16579e85e06fee4c36ff8c02fb13795

Bài này file backup bị chặn tuy nhiên lại có thể bypass bằng **null byte** `%00` sau đó encode thành `%2500`. Do đó ta có thể download file về và lấy được flag

<h2>Task 6 - Who's flying this thing?</h2>

**Q1:** 946a799363226a24822008503f5d1324536629a0

**Q2:** 169940f83378cc420ae4fdeb9c1f73631a2baee6

**Q3:** 50c97bcce0b895e446d61c83a21df371ac2266ef

<h2>Task 7 - Where did that come from?</h2>

**Q1:** 9aaf4bbea5c30d00a1f5bbcfce4db6d4b0efe0bf

`<iframe src="javascript:alert(`xss`)"> `

**Q2:** 149aa8ce13d7a4a8a931472308e269c94dc5f156

**Q3:** 23cefee1527bde039295b2616eeb29e1edc660a0

**DONE**

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/be344d03-073e-4efd-ac37-c7958ffad422)
