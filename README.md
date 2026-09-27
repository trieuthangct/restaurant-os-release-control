# Điều phối phát hành ROS

Repo công khai này chỉ chứa GitHub Actions và hướng dẫn phát hành. Mã ứng dụng nằm trong repo riêng tư `trieuthangct/restaurant-os`. Không đưa dữ liệu Brand, khóa hoặc bản build vào repo hay artifact công khai.

## Điều kiện trước khi bật workflow

1. Nhánh `main` được bảo vệ, chỉ nhận thay đổi qua pull request đã review; không cho quản trị viên bỏ qua quy tắc.
2. Environment `production-candidate` và `production` chỉ cho phép nhánh `main` chạy. `production` phải có người duyệt được xác minh, bật **Prevent self-review** và tắt **Allow administrators to bypass**.
3. Có một tài khoản khởi chạy khác tài khoản duyệt. Nếu chưa xác định được hai danh tính độc lập, không chạy phát hành.
4. Repo riêng tư đã hợp nhất mã cần phát hành vào `codex/production-release`; `ROS release guard` xanh trên đúng SHA. Nghiệm thu Marketing và migration production là cổng riêng trước khi phát hành giao diện.
5. Không còn workflow private có quyền phát hành song song. Người điều phối kiểm tra task ROS và GitHub Actions đang chạy trước khi kích hoạt.

## Environment Secrets

| Secret | `production-candidate` | `production` |
|---|---|---|
| `ROS_SOURCE_TOKEN` | Có: fine-grained token chỉ đọc Contents và Actions của repo ROS riêng tư | Có: cùng quyền đọc |
| `CLOUDFLARE_API_TOKEN` | Có: upload candidate và đọc version | Có: promote version đã duyệt |
| `CLOUDFLARE_ACCOUNT_ID` | Có | Có |
| `VITE_SUPABASE_URL` | Có | Không |
| `VITE_SUPABASE_ANON_KEY` | Có | Không |
| `VITE_UPLOAD_URL` | Có | Không |
| `VITE_UPLOAD_TOKEN` | Có | Không |
| `VITE_VAPID_PUBLIC_KEY` | Có | Không |

Không ghi giá trị secret vào issue, pull request, file hoặc log. `VITE_*` được đóng vào bundle frontend; chỉ dùng giá trị được phép công khai cho trình duyệt.

## Luồng phát hành

Người khởi chạy độc lập nhập SHA đầy đủ của nhánh private `codex/production-release`. Job candidate kiểm workflow guard, trạng thái phát hành khác, SHA và cấu hình; build từ source private; upload version không nhận traffic; kiểm preview. Job promote chỉ bắt đầu sau khi người duyệt duyệt environment `production`, kiểm lại SHA và production baseline, rồi promote đúng Version ID đã kiểm. Nếu bất kỳ điều kiện nào thiếu, workflow dừng.

Đây là thiết kế đang chờ review. Việc tạo repo hoặc pull request không đồng nghĩa đã deploy production.
