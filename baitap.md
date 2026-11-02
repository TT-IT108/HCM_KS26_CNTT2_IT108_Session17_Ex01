# BÁO CÁO PHÂN TÍCH VÀ XỬ LÝ SỰ CỐ WEBSITE (HỆ THỐNG GIÁM SÁT)

## Phần 1: Phân tích và chỉ ra lỗi trong lập luận của Nam

* **Về vấn đề DNS Lookup:** 
  * Lập luận của Nam rằng *"DNS Lookup không tìm thấy địa chỉ IP nên máy mới báo lỗi 500"* là **hoàn toàn sai**. 
  * **Bản chất thực tế:** Nếu DNS Lookup không tìm thấy địa chỉ IP, trình duyệt sẽ trả về lỗi không thể phân giải tên miền (ví dụ: *DNS_PROBE_FINISHED_NXDOMAIN*), chứ không bao giờ trả về mã trạng thái HTTP. Việc Nam nhận được mã lỗi HTTP chứng tỏ quá trình phân giải DNS đã thành công và máy tính đã kết nối thành công tới Server.
* **Về mã trạng thái HTTP 500 (Internal Server Error):**
  * Mã `500` thuộc nhóm **Server Error** (lỗi từ phía máy chủ), thông báo rằng Server đã nhận được yêu cầu (Request) từ client nhưng gặp sự cố bất ngờ trong quá trình xử lý mã nguồn, cơ sở dữ liệu hoặc cấu hình hệ thống nội bộ. Lỗi này **không liên quan** đến mạng internet cá nhân hay DNS của người dùng, do đó việc đổi DNS sang `8.8.8.8` hoàn toàn không khắc phục được vấn đề.

---

## Phần 2: Mô tả sơ đồ luồng dữ liệu (Từ DNS Lookup đến Server Response)

1. **DNS Lookup (Thành công):** 
   * Trình duyệt gửi yêu cầu tới DNS Server và nhận lại chính xác địa chỉ IP của trang web `lms.ptit.edu.vn`. Do bước này thành công, trình duyệt mới tiếp tục gửi HTTP Request tới Server.
2. **TCP Connection & TLS Handshake:** 
   * Thiết lập thành công kết nối an toàn giữa máy client của Nam và máy chủ (Server) của trường.
3. **HTTP Request Sent:** 
   * Trình duyệt gửi gói tin yêu cầu tài nguyên trang web tới Server.
4. **Server Processing & Lỗi phát sinh:** 
   * Server tiếp nhận yêu cầu, nhưng trong quá trình xử lý logic bên trong (gặp ngoại lệ code, lỗi kết nối database, hoặc lỗi cấu hình), Server thất bại trong việc tạo ra phản hồi hợp lệ.
5. **Server Response (Mã lỗi 500):** 
   * Thay vì trả về mã `200 OK` kèm nội dung trang web, Server phản hồi về trình duyệt mã trạng thái `500 Internal Server Error`, khiến trình duyệt xoay vòng lâu và hiển thị thông báo lỗi như hiện trạng của Nam.
