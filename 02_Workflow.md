# Workflow của ELK Stack
- Một máy tính hay server tạo ra các file log. Các file log đó ở nhiều định dạng khác nhau, thậm chí khá phức tạp để đọc. Một số hệ thống, ví dụ như server cluster, sản sinh ra một lượng lớn file log. **ELK stack** được thiết kế để giúp quản lý lượng dữ liệu lớn.

    <img src=https://i.imgur.com/ZYl1nO4.png>

- Các file log được thu thập bởi một collector gọi là **Beats**. Các loại **Beats** khác nhau tiếp cận các thành phần khác nhau của server, đọc các file, và gửi chúng đi :

    <img src=https://i.imgur.com/abTg492.png>

- Một số người dùng có thể bỏ qua **Beats** mà sử dụng trực tiếp **Logtash**. Một số khác có thể kết nối thẳng **Beats** vào **Elasticsearch**.
- **Logtash** được cấu hình để tiếp cận và thu thập data từ các ứng dụng **Beats** khác nhau (hoặc trực tiếp từ nhiều nguồn khác nhau). **Logtash** có thể filter data từ nhiều hệ thống và tập trung data vào một vị trí.

    <img src=https://i.imgur.com/qAnoD0e.png>

- **Elasticsearch** được sử dụng như một database có khả năng tìm kiếm, mở rộng, dùng để lưu trữ data. **Elasticsearch** chính là kho chứa, nơi mà **Logtash** hoặc **Beats** đẩy toàn bộ dữ liệu vào.

    <img src=https://i.imgur.com/EIs9rRD.png>

- Cuối cùng, **Kibana** cung cấp một giao diện web UI thân thiện với người dùng để review toàn bộ dữ liệu đã thu thập :

    <img src=https://i.imgur.com/G5nWQlJ.png>