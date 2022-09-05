# Write-up-BalsnCTF
###1. my first app
![image](https://user-images.githubusercontent.com/82523299/188363439-d3381264-6dc6-467b-aadf-5e1508b4b868.png)

- Bài này được viết bằng nextjs, tôi chưa bao giờ học code js cả nên gặp nextjs là ngáo luôn ^^. Và tôi dành ra 1 tiếng để học cấp tốc tại trang https://nextjs.org/learn
- Đây là source và flag nằm ở ![image](https://user-images.githubusercontent.com/82523299/188363832-5d2212d0-d59a-43e1-9a59-fa0974b3dfa5.png)


- Vì không quen với kiểu code web routes nên mãi một hồi tôi mới biết được cách endpoine cái api hello để xuất ra secret ![image](https://user-images.githubusercontent.com/82523299/188363885-cba16781-280a-4203-a25c-abc45f6c8fa1.png)
- Nhưng thứ tôi cần là flag :((
- Trong source index.js tác giả đã `import globalVars from '../utils/globalVars'` để xuất cái title ở index của trang web và biến `globalVars` cũng chứa flag
- ![image](https://user-images.githubusercontent.com/82523299/188364193-53be8df9-c690-4b56-8680-841609a54592.png)
- Và tôi tìm mọi cách để có thể injected vào đây ^^, và rồi phát hiện không có cách nào cả
- Tôi thử search các version của thư viện nhưng đều là last version, ko có lỗi cve => cùng đường :((
- Sau khi quay lại đọc document của nextjs tôi tìm được từ khóa `one page website` đây là kiểu code mà server gửi các file frontend cho client để hướng dẫn client xử lí giảm gánh nặng cho server. Thế là tôi lục code trong developer tool của trang web
- Vì file index đã import globalVars nên data cũng được gửi về server
- ![image](https://user-images.githubusercontent.com/82523299/188365202-1c148fcd-d038-4981-822f-e57cfda31007.png)
####Flag: BALSN{hybrid_frontend_and_api}
###2.

