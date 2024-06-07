![image](https://github.com/vanniichan/TryHackMe/assets/112863484/cb5618c7-d1f5-49f6-bbee-d679b93916fa)

Khi vào web thấy được mỗi cánh cửa có 1 đường dẫn đến với mã khác nhau

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/cfa1f747-8b9d-40f4-8cdd-6f4a7e5f3e11)

Click vào thì không có gì

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/a1aba694-9e31-450e-a1a1-539fa13bdc90)

Theo hint của room thì nó là các mã hash đem đi decode thử:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/ea8ba2b2-a4b2-4e30-a8b9-837c3379ccef)

Thử thêm vài mã nữa thì có vẻ nó là số thứ tự 1 -> 13:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/b15161d4-7c4a-4933-bbdc-03a48aeb8596)

Vậy thử encode số 14 mục tiêu để vào room 14 nhưng lỗi nên thử room 0:

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/94cf4fc9-2892-466e-86e4-6ac8e5ce29d2)

![image](https://github.com/vanniichan/TryHackMe/assets/112863484/896fb98c-a77e-4043-9739-b88cac69c8e2)

**FINAL FLAG: flag{2477ef02448ad9156661ac40a6b8862e}**
