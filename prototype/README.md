# Prototype — Ví Mẫu · Chat với ví (T5)

**Mở:** tải repo về rồi mở `index.html` bằng trình duyệt (Chrome / Edge). Một file, không backend, không đăng nhập, không gọi AI thật, không gửi dữ liệu đi đâu.

## Cách bấm qua ca khó (demo 4 phút)

1. **S1 · Bị lừa đòi nhả**:
   - Khung chat bên trái: AI giữ tiền, cảnh báo mạo danh công an, chuyển tư vấn viên. Phía khách **không có nút nhả**.
   - Bấm "Mình không bị lừa — gửi thêm cho tư vấn viên": AI vẫn không nhả, còn bàn tư vấn viên hiện dòng "Khách phản đối".
   - Bàn tư vấn viên bên phải: nút **Nhả** bị khoá cho tới khi tích ô "đã gọi lại qua số đăng ký". Bấm "Không nhả" thì khách nhận thông báo trong chat.
2. **S2 · Gấp ở quầy viện phí**:
   - Bấm "Xác thực khuôn mặt"; thanh 2 bước chuyển sang ② và đồng hồ ước tính chạy.
   - Bên phải, tích ô rồi bấm **Nhả**: khách thấy "Xong rồi".
   - Bấm "Mình đã trả cách khác" để xem đường dự phòng.
3. **Giờ hiện tại → 23:48**: xem đường ngoài giờ trực. Giao dịch giữ nguyên, nút của tư vấn viên bị khoá, AI nói giờ vào ca là 08:00.
4. **Ca lạ**: gõ tin bất kỳ (hoặc bấm tin mẫu). Bộ phân loại bằng luật từ khoá chọn đường đi, hiện ra tool AI đã gọi và mức tự chủ của từng tool.

Mở thẳng một trạng thái bằng hash: `index.html#s1,lock` (S1, đang mở hộp khoá ví) · `index.html#s2,verified,chk` (S2, đã xác thực).

## Ảnh đánh số

| Ảnh | Trạng thái |
|---|---|
| [`anh-1-S1-bi-lua.png`](anh-1-S1-bi-lua.png) | S1, hộp xác nhận khoá ví đang mở — số ❶–❾ trừ ❽ |
| [`anh-2-S2-gap-that.png`](anh-2-S2-gap-that.png) | S2, khách đã xác thực, tư vấn viên đã tích ô kiểm — có ❽ |

Bảng giải thích từng số (nguyên tắc HAX/PAIR và mức tự chủ): `spec.md` §5. Bảng chú thích cũng hiện ngay trên trang.

## Phần nào là mock

- Toàn bộ dữ liệu (Ví Mẫu, Bệnh viện Mẫu, TRAN VAN H, số tổng đài) là dữ liệu dựng.
- Câu trả lời của AI ở S1 và S2 được viết sẵn theo `spec.md` §3.
- Ô "Ca lạ" phân loại bằng luật từ khoá, không phải mô hình.
- Xác thực khuôn mặt chỉ là hiệu ứng giả.
