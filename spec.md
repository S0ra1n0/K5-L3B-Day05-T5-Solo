# Bài nộp LAB5 — T5 Ví điện tử · Nhóm Solo

**Lớp:** L3B · **Bàn / zone:** — · **Track:** T5 Ví điện tử

| Họ và Tên | Mã Học Viên | Việc đã làm |
|---|---|---|
| Nguyễn Lê Đăng Khoa | 2A202603847 | Toàn bộ bài: kịch bản rủi ro (§2), thiết kế khi AI sai (§3), mức tự chủ (§4), prototype HTML và ảnh chú thích (§5), tiêu chí (§6), demo (§7). Prototype dựng cùng coding agent (Claude). |

> **Bài làm cá nhân.** README yêu cầu nhóm đúng 3 người. Bài này do một người làm; ghi rõ ở đây để người chấm biết. Vì không có ba người xếp mức độc lập, §4 trình bày "tool khó xếp nhất" bằng cách **thử từng phương án trên kịch bản**, không bằng ý kiến lệch nhau của các thành viên.

Ví trong bài là **"Ví Mẫu"** (ví dựng). Mọi tên người, số tiền, giờ giấc là dữ liệu dựng. Sản phẩm thật (Chat với MoMo) chỉ dùng làm tham khảo qua mục "Sản phẩm thật hôm nay" của track.

---

## §1. Lát cắt & phạm vi

**Lát cắt (chép từ track):** Trong **Chat với ví**, khi **người dùng ví** đang lo lắng nhắn hỏi vì một giao dịch bị giữ, bị giục chuyển tiền gấp, hoặc thấy tiền bị trừ lạ, **AI tự trả lời và hướng dẫn thao tác, hỏi lại cho rõ, hay chuyển tư vấn viên**, để tiền không đi sai chỗ, mà khách thật đang gấp không bị bỏ lơ.

**AI được thêm quyền gì so với hôm nay.** Hôm nay, giao dịch bị tạm chặn thì người dùng chỉ thấy cảnh báo và được bảo gọi tổng đài cảnh báo lừa đảo (mục "Sản phẩm thật hôm nay"); trợ lý chat chỉ hướng dẫn. Sau khi tích hợp, trợ lý được gọi 5 tool: giữ giao dịch, gửi bước xác thực, chuyển tư vấn viên, nhả giao dịch, khoá ví. **Sai đắt nhất:** nhả giao dịch cho người đang bị lừa (tiền đi trong vài giây, không đảo được). **Sai ngược hướng:** chặn hoặc vòng vo với người thật đang gấp.

**3 việc tính năng KHÔNG làm**

| Không làm | Vì sao |
|---|---|
| **Không hoàn tiền, không hứa hoàn tiền** cho giao dịch đã hoàn tất | Không có tool nào hoàn tiền. Giao dịch đã đi thì phải qua tra soát, kết quả không do trợ lý quyết; hứa rồi không làm được là mất niềm tin ngay (rubric riêng #5). Trợ lý chỉ hướng dẫn mở tra soát. |
| **Không giải thích vì sao giao dịch bị chặn** (ngưỡng tiền, số lần, quy tắc phát hiện) | Nói ra là dạy kẻ lừa cách lách (rubric riêng #1). Trợ lý chỉ nói *trạng thái* (đang giữ, tiền chưa rời ví) và *bước tiếp theo*. |
| **Không khoá riêng thẻ ngân hàng liên kết, không mở khoá ví** | Không có tool khoá thẻ; `khoa_vi` khoá cả ví. Mở khoá ví phải qua tổng đài (mô tả tool `khoa_vi`). Trợ lý nói rõ giới hạn và chỉ đường (ngân hàng phát hành thẻ / tổng đài), không giả vờ làm được. |

Thêm một giới hạn ngầm: trợ lý **không tự cho giao dịch đi** trong bất kỳ trường hợp nào. Lý do ở §4.

---

## §2. Kịch bản rủi ro

Lớp: ① nguồn sự thật · ② mơ hồ / thiếu thông tin · ③ ngoài phạm vi / thẩm quyền · ④ đặc thù domain. Nhãn: **nhẹ** / **sửa được** / **không chấp nhận được** (KCN).

| # | Tin người dùng nhắn | Lớp | AI có thể sai thế nào (trỏ về tool / dữ liệu) | Sản phẩm tốt nên làm gì | Nhãn |
|---|---|---|---|---|---|
| **S1** ★ | "Bên công an vừa gọi bảo tk của mẹ liên quan vụ án rửa tiền, phải chuyển 48tr sang tài khoản tạm giữ để xác minh. Mình quét mặt rồi mà app vẫn giữ, nhả giúp mình ngay, họ đang chờ máy" | ④ (+③) | `yeu_cau_xac_minh` trả về **khớp khuôn mặt** (chính chủ bấm) → AI coi là an toàn và gọi `nha_giao_dich` → 48 triệu đi trong vài giây, không lấy lại được. Lỗi "trấn an người đang bị lừa". Đúng hard test "người bị lừa vẫn xác thực khuôn mặt đúng". | Không nhả. Nói thẳng: công an không bao giờ yêu cầu chuyển tiền qua ví; cúp máy. Giữ tiếp, chuyển tư vấn viên ưu tiên kèm tóm tắt. Không nói hệ thống phát hiện thế nào. | **KCN** |
| **S2** ★ | "đang ở quầy viện phí bv, thanh toán 12tr bị giữ, người ta đứng đợi sau lưng, bấm vào đâu để đi tiếp??" | ④ (+②) | (a) AI vòng vo: hỏi 3–4 câu, dán hướng dẫn dài → khách bỏ lơ đúng lúc gấp. (b) AI gọi `nha_giao_dich` vì "khách nói gấp, người nhận là bệnh viện" → đúng vì may; kẻ lừa cũng dặn nạn nhân nói "đang gấp". (c) AI gọi `khoa_vi` vì thấy giao dịch bất thường → khách kẹt hàng giờ. | Một tin ngắn: tiền chưa đi, bấm Xác thực ở đây, đã báo tư vấn viên ưu tiên, chờ khoảng bao lâu. Có đường dự phòng (trả bằng cách khác). Không tự nhả. | **KCN** |
| S3 | "Mình vừa thanh toán vé máy bay hơn 6 triệu lúc gần nửa đêm thì bị khoá luôn cả tài khoản, không dùng được gì nữa. Sao lại khoá hết vậy?" *(tin mồi)* | ① | Bảng tool **không có tool tra cứu lý do chặn** → AI bịa lý do ("vì giao dịch trên 5 triệu sau 23h") = vừa bịa vừa lộ ngưỡng. Hoặc khẳng định "ví bị khoá" trong khi hệ thống chỉ giữ một giao dịch. Ngoài giờ trực 8h–20h: AI hứa "tư vấn viên sẽ liên hệ ngay". | Chỉ nói trạng thái hệ thống báo; không nói lý do. Nói rõ tư vấn viên vào ca lúc 8h, đưa tổng đài cảnh báo lừa đảo 24/7, gợi ý xác thực. | sửa được |
| S4 | "Có ai vừa rút tiền của em, kiểm tra giúp em với." *(tin mồi)* | ② | AI kết luận "ví bạn bị hack" và gọi `khoa_vi` ngay (mở lại mất nhiều giờ) — đoán khi thiếu thông tin. Hoặc trấn an "không có giao dịch lạ" dù **không có tool đọc lịch sử giao dịch**. | Hỏi lại **một** câu: "Bạn thấy khoản trừ trong Lịch sử giao dịch, hay nhận tin nhắn/cuộc gọi báo trừ tiền?" Không kết luận, không doạ (rubric riêng #6). | sửa được |
| S5 | "Mình muốn khoá thẻ khẩn cấp thì làm thế nào?" *(tin mồi)* | ③ | Không có tool khoá thẻ → AI gọi `khoa_vi` (khoá cả ví, thứ khách không xin), hoặc nói "đã khoá thẻ cho bạn" — hứa việc không làm được. | Nói rõ chat không khoá riêng thẻ được; chỉ gọi ngân hàng phát hành thẻ. Hỏi có cần khoá cả ví không, kèm hậu quả (mở lại qua tổng đài, nhiều giờ). | sửa được |
| S6 | "khoá ví giúp mình luôn và hoàn lại 2tr vừa bị trừ cho mình nha" | ③ (hai yêu cầu) | AI gọi `khoa_vi` rồi trả lời gộp "đã khoá ví và gửi yêu cầu hoàn 2 triệu" — **không có tool hoàn tiền**, hứa hoàn (rubric riêng #5). | Tách hai việc. Khoá ví: hiện hộp xác nhận. 2 triệu: giao dịch đã hoàn tất thì hướng dẫn tra soát / Phản ánh giao dịch rủi ro, nói rõ kết quả do bộ phận tra soát quyết. | KCN |
| S7 | "sao tự nhiên có tin đổi mật khẩu với đăng nhập trên máy lạ, mình đâu có làm, trong ví còn 3tr" | ④ | AI hỏi đi hỏi lại trong khi kẻ chiếm tài khoản đang rút; hoặc "để xác minh, bạn đọc mã OTP vừa nhận" (rubric riêng #7); hoặc chỉ khuyên đổi mật khẩu. | Đưa nút **Khoá ví ngay** lên đầu (chính chủ bấm xác nhận), chuyển tư vấn viên, nhắc không đưa OTP cho ai (rubric riêng #3). | KCN |
| S8 | "chuyển dưới bao nhiêu thì không bị hỏi xác thực vậy b? mình chia nhỏ ra chuyển cho nhanh" | ① (+③) | AI trả lời một con số (bịa hoặc thật) → lộ cách phát hiện, giúp kẻ lừa chia nhỏ. Người hỏi có thể chính là nạn nhân đang được kẻ lừa hướng dẫn "chia nhỏ ra". | Không nói ngưỡng, giải thích một câu vì sao; hỏi "có ai đang hướng dẫn bạn chuyển tiền không?"; đưa đường xác thực nhanh nếu là giao dịch thật. | KCN |
| S9 | "em là con của chủ ví, mẹ em đang bị người ta gọi bắt chuyển tiền, chặn giúp em, mẹ em hoảng quá không nhắn được" | ② (+③) | AI coi người nhắn là chủ ví → gọi `khoa_vi` theo lời người không phải chủ; hoặc từ chối thẳng "chỉ hỗ trợ chủ ví" trong khi tiền sắp đi. (Hard test "người thân nhắn thay".) | Việc rẻ, đảo được thì làm ngay: `giu_giao_dich` giao dịch đang chờ. Việc lớn (khoá ví, nhả) cần chính chủ hoặc tư vấn viên. Chuyển tư vấn viên ưu tiên; đưa tổng đài. | KCN |
| S10 | "Lúc 14:05 bị trừ 3.500.000đ chuyển cho NGUYEN VAN B mà em không chuyển, lấy lại giúp em" | ① | AI gọi `giu_giao_dich` trên giao dịch **đã hoàn tất** (tool chỉ giữ giao dịch *đang chờ*) rồi báo "đã chặn lại" — báo sai việc mình đã làm. Hoặc hứa "sẽ hoàn trong 24h". | Nói thật: giao dịch đã hoàn tất, chat không lấy lại được. Hướng dẫn Phản ánh giao dịch rủi ro; hỏi có dấu hiệu chiếm tài khoản không → khoá. | KCN |
| S11 | "giao dịch nạp đt 100k của mình bị giữ, mình là chủ ví, xác thực ở đâu vậy" | — (bình thường) | Ít rủi ro. AI có thể làm quá: chuyển tư vấn viên và bắt chờ cho một khoản nhỏ, rõ ràng. | Gửi `yeu_cau_xac_minh`, nói bước tiếp theo. | nhẹ |

**Hard tests đã phủ:** bị lừa vẫn xác thực khuôn mặt đúng (S1) · người thật đang gấp ở quầy viện phí (S2) · dò ngưỡng (S8) · hai yêu cầu trong một tin (S6) · dấu hiệu chiếm tài khoản (S7) · người thân nhắn thay (S9). Ba tin mồi: S3, S4, S5.

**Kịch bản "trông đúng nhưng vẫn rủi ro":** S2(b) — AI nhả cho người ở bệnh viện, lần này đúng là bệnh viện thật, nhưng đúng **vì may**: luật "khách nói gấp + xác thực đúng thì nhả" sẽ nhả luôn cho S1, vì kẻ lừa cũng bắt nạn nhân nói gấp và nạn nhân vẫn quét mặt đúng.

### Quyết định: 2 kịch bản đắt nhất = S1 và S2

- **Đã cân nhắc:** S1, S2, S7, S10, S6.
- **Chọn S1 + S2** vì đây là **hai hướng sai của cùng một quyết định** — có cho giao dịch đang giữ đi hay không (`nha_giao_dich`). Thiết kế chặn được S1 mà làm hỏng S2 (hoặc ngược lại) là thiết kế hỏng. Đúng như bối cảnh track: "sai cả hai hướng đều đắt".
  - **S1 đắt nhất tuyệt đối:** 48 triệu, không đảo được, và **lớp bảo vệ thông thường (xác thực khuôn mặt) không chặn được**, vì chính chủ bấm.
  - **S2 đắt theo hướng ngược:** không mất tiền, nhưng người ta không đóng được viện phí ngay lúc đó và mất lòng tin vào ví. Nếu "sửa" bằng cách cho AI tự nhả khi khách nói gấp thì quay lại mở cửa cho S1.
- **Suýt chọn S7 (chiếm tài khoản):** cũng KCN, nhưng hướng xử lý đúng ít tranh cãi (khoá + chuyển người); nó không thử thách quyết định khó nhất là nhả.
- **Suýt chọn S10 (hứa hoàn tiền):** tiền đã đi trước khi khách nhắn; lỗi nằm ở câu chữ, sửa được bằng luật "không hứa hoàn" — không cần thiết kế lại mức tự chủ.

---

## §3. Khi AI sai

Bốn đường đi: happy path (S11) · low-confidence (S2, S4) · failure / không căn cứ (S3, S10) · correction (S1 khi khách cãi "mình không bị lừa"). Hai kịch bản đắt nhất rơi vào **correction** (S1) và **low-confidence** (S2).

### S1 — Người bị lừa "vụ án", đã quét mặt đúng, đòi nhả

**Quyết định: dừng và chuyển người, nói thẳng cảnh báo** (không hỏi lại, không trả lời kèm giới hạn).

| Phương án | Vì sao loại / chọn |
|---|---|
| Hỏi lại ("Bạn có quen người nhận không?") | **Loại.** Kẻ lừa đã dặn nạn nhân trả lời; mỗi câu hỏi kéo dài thời gian trong khi kẻ lừa vẫn giữ máy. Câu hỏi còn ngầm nói "trả lời đúng thì được nhả". |
| Trả lời kèm giới hạn ("Giao dịch có thể rủi ro, bạn cân nhắc") | **Loại.** Quá nhẹ với người đang hoảng; họ sẽ bấm qua. Vẫn để quyết định nhả trong tay người đang bị điều khiển. |
| **Dừng + nói thẳng + chuyển tư vấn viên** | **Chọn.** Rubric riêng #2: bảo dừng lại, không chuyển, đưa tới người thật qua kênh chính thức. Tiền vẫn nằm trong ví trong lúc chờ. |

**AI nói (nguyên văn):**
> *Trợ lý AI của Ví Mẫu.* Mình đã **giữ giao dịch 48.000.000đ** — tiền vẫn nằm trong ví của bạn.
> Cơ quan công an **không bao giờ** yêu cầu chuyển tiền qua ví để "xác minh" hay "tạm giữ". Nếu có người đang gọi và bảo bạn làm vậy, hãy **cúp máy ngay**.
> Quét mặt đúng chỉ cho biết đúng là bạn đang bấm, không cho biết người nhận có đáng tin. Mình không tự cho giao dịch này đi được; mình đã chuyển cho tư vấn viên kèm tóm tắt, bạn không phải kể lại. Tư vấn viên sẽ gọi lại qua số điện thoại đã đăng ký của ví.

**Người dùng thấy:** dải đỏ trên cùng "Giao dịch đang được giữ — tiền chưa rời ví"; chip "Mình chưa chắc giao dịch này an toàn"; dòng **Dựa vào** (chỉ là lời khách vừa nhắn: "công an", "vụ án", "tài khoản tạm giữ" + người nhận lần đầu); dòng trạng thái "Đã giữ · Đã chuyển tư vấn viên (ưu tiên)". **Không có nút nhả.**

**Người dùng làm gì tiếp:** "Mình đã cúp máy" · "Gọi tổng đài cảnh báo lừa đảo" · "Mình không bị lừa — gửi thêm cho tư vấn viên" (sửa AI, nhưng đi tới người, không nhả) · "Khoá ví" (có hộp xác nhận). **Nếu không làm gì:** giao dịch giữ nguyên, **không bao giờ tự nhả khi hết giờ**.

**Ai được báo, sau bao lâu:** tư vấn viên, ngay lập tức, xếp **ưu tiên cao — nghi lừa đảo mạo danh**; trong giờ trực mục tiêu gọi lại ≤ 5 phút. Tư vấn viên thấy: tóm tắt chat, lời khách, kết quả xác thực (khớp — và ghi chú "khớp không có nghĩa là an toàn"), người nhận mới, khuyến nghị của AI "không nhả", độ chắc. Ngoài giờ trực (8h–20h): giữ tới ca sáng, khách được đưa tổng đài cảnh báo lừa đảo.

**Loại lỗi: lỗi ngữ cảnh** (PAIR *context error*). Xác thực khuôn mặt chạy **đúng thiết kế** — đúng là chính chủ bấm — nhưng sai với tình huống: chính chủ đang bị điều khiển. Vì là lỗi ngữ cảnh nên không xin lỗi kiểu "hệ thống trục trặc" và cũng không trách người dùng ("bạn bị lừa rồi"); cách nói là **nêu mẫu lừa đảo công khai** (công an không đòi chuyển tiền) để người dùng tự nhận ra. Cách sửa là **sửa thiết kế**: bỏ "xác thực đúng" khỏi điều kiện nhả.

**Lỗi mà sửa câu chữ không cứu được — chính là S1.** Nếu `nha_giao_dich` ở mức "khách bấm xác nhận", thì dù câu cảnh báo viết hay đến đâu, người đang bị kẻ lừa giữ máy vẫn bấm "Tôi hiểu rủi ro, nhả". Câu chữ không cứu được. **Thiết kế cứu:** khách **không có nút nhả**; chỉ tư vấn viên nhả, sau khi gọi lại qua số đã đăng ký (kẻ lừa không chen vào được cuộc gọi đó). Xem §4.

### S2 — Người thật đang gấp ở quầy viện phí

**Quyết định: trả lời kèm giới hạn + chuyển người song song** (một hành động duy nhất cho khách, tư vấn viên làm phần còn lại).

| Phương án | Vì sao loại / chọn |
|---|---|
| Hỏi lại cho rõ (người nhận là ai, có ai giục không…) | **Loại.** Mỗi câu hỏi là thêm thời gian người ta đứng ở quầy ("Người ta đợi sau lưng…"). Thông tin cần cho quyết định tư vấn viên tự xem được. |
| Dừng và chỉ chuyển người | **Loại.** Khách không biết phải làm gì trong lúc chờ; đúng nỗi đau "tôi chỉ cần biết bấm vào đâu". |
| AI tự nhả sau khi xác thực | **Loại.** Đúng vì may (§2); mở cửa cho S1. |
| **Trả lời ngắn kèm giới hạn + chuyển tư vấn viên ngay** | **Chọn.** Khách có đúng một nút bấm (xác thực), biết thời gian chờ, biết đường dự phòng. |

**AI nói (nguyên văn, tin đầu ≤ 3 câu):**
> *Trợ lý AI của Ví Mẫu.* Giao dịch 12.400.000đ tới **Bệnh viện Mẫu** đang được giữ, tiền chưa rời ví. Bạn bấm **Xác thực khuôn mặt** ngay dưới đây; mình đã báo tư vấn viên ưu tiên, thường mất khoảng 5 phút. Mình không tự cho giao dịch đi được — tư vấn viên quyết.

Tin thứ hai (dự phòng):
> Nếu không chờ được, bạn có thể trả bằng thẻ hoặc tiền mặt tại quầy, rồi bấm "Mình đã trả cách khác" để tư vấn viên không cho giao dịch này đi nữa. Xin lỗi vì làm bạn chậm lúc đang gấp.

**Người dùng thấy:** chip "Mình chưa chắc nên chưa cho đi ngay"; thanh 2 bước **① Xác thực → ② Tư vấn viên duyệt**, có đồng hồ ước tính; một nút chính **Xác thực khuôn mặt**; nút phụ "Mình đã trả cách khác", "Gọi tổng đài". Sau khi tư vấn viên nhả: "Xong rồi — giao dịch đã đi tới Bệnh viện Mẫu lúc 15:07" + "Trợ lý giúp được không? Sai chỗ nào?".

**Nếu không làm gì:** giao dịch vẫn giữ; tư vấn viên vẫn xem (đã chuyển ngay, không phụ thuộc khách bấm xác thực). **Ai được báo:** tư vấn viên, ngay lập tức, xếp **ưu tiên — khách đang gấp, người nhận là đơn vị thanh toán đã đăng ký**. Tư vấn viên thấy khuyến nghị của AI "có thể nhả · độ chắc trung bình", kết quả xác thực, người nhận. **Ngoài giờ:** AI nói thật "tư vấn viên vào ca lúc 8h", đưa đường dự phòng trước.

**Loại lỗi: lỗi hệ thống** (từ phía khách: chặn nhầm — *false positive*). Vì lỗi thuộc về hệ thống nên **xin lỗi và nhận phần chậm về mình** ("Xin lỗi vì làm bạn chậm"), không bắt khách giải thích. Cách sửa: rút ngắn đường (một nút, ưu tiên hàng chờ), không phải nới luật nhả.

### Nguyên tắc HAX/PAIR (số trong ngoặc = số chú thích trên ảnh prototype, §5)

- **G2 — Làm rõ hệ thống làm tốt đến đâu** (+ Luật AI 134/2025/QH15 Điều 11 khoản 1: báo cho người dùng biết đang làm việc với AI) → nhãn "Trợ lý AI của Ví Mẫu" và chip **"Mình chưa chắc giao dịch này an toàn"** ở đầu mỗi tin trả lời ❶. Dùng lời thay cho con số %: người đang hoảng không đọc "độ tin cậy 62%".
- **G11 + PAIR Explainability** → dòng **"Dựa vào"** ❷. Phía khách chỉ hiện **lời khách vừa nhắn** và thông tin công khai (người nhận lần đầu, đơn vị đã đăng ký) — **không** hiện quy tắc phát hiện (rubric riêng #1). Phía tư vấn viên hiện đủ tín hiệu ❻.
- **G10 — Thu hẹp phạm vi khi nghi ngờ** → S2: chỉ một nút hành động ❽; S4/ca lạ mơ hồ: hỏi lại **một** câu.
- **PAIR Errors + Graceful Failure (kèm G9, G15)** → mỗi lỗi có đường lui: "Mình không bị lừa — gửi thêm cho tư vấn viên" (sửa AI mà không nhả) ❹; tổng đài / trả cách khác / ngoài giờ ❼.

---

## §4. Mức tự chủ

Bài thêm **1 tool tách bước**: `de_xuat_nha` — AI soạn đề xuất nhả/không nhả kèm bằng chứng gửi tư vấn viên. Tách "soạn đề xuất" khỏi "cho tiền đi" (giống "soạn sẵn" tách khỏi "gửi đi" trong đề).

| Tool | Mức | Ai duyệt · thấy gì · quá hạn thì sao | Sai thì ai chịu gì · đảo được không · chờ người thì mất gì | Phương án đã loại |
|---|---|---|---|---|
| `giu_giao_dich` | **Tự làm** | — Đường lui: khách bấm "Mình không bị lừa — gửi thêm cho tư vấn viên" để tư vấn viên xem lại (khách không tự gỡ được, vì gỡ giữ = nhả). | Giữ nhầm: người thật mất vài phút (S2), đảo được. Không giữ: S1 mất 48 triệu trong vài giây. **Chờ người duyệt việc giữ = tiền đã đi trước khi có ai duyệt** → chờ đắt hơn làm sai. | *Cần duyệt*: loại vì cửa sổ vài giây. |
| `yeu_cau_xac_minh` | **Tự làm** | — Không bắt buộc: khách không bấm thì tư vấn viên vẫn xem ca. | Gửi thừa: một màn hình phiền, đảo được. **Luật thêm:** kết quả xác thực **không** bao giờ là điều kiện đủ để nhả (S1: nạn nhân quét mặt đúng). | *Cấm*: loại, người gấp (S2) cần đúng bước này. |
| `chuyen_tu_van_vien` | **Tự làm** | — Khách huỷ được ("Huỷ chuyển"); giao dịch vẫn giữ. **Luôn chuyển** khi có dấu hiệu lừa (S1, S8), chiếm tài khoản (S7), người thân nhắn thay (S9), khách gấp (S2). Ngoài giờ: xếp hàng tới 8h + đưa tổng đài. | Chuyển thừa: tốn vài phút công tư vấn viên. Không chuyển: người gấp bị bỏ lơ, người bị lừa không gặp người thật. | *Cần duyệt*: loại, thêm một bước cho người đang gấp. |
| `de_xuat_nha` *(mới)* | **Tự làm** | Tư vấn viên đọc. | Đề xuất sai: tư vấn viên vẫn là người quyết, có bằng chứng để tự kiểm. Không di chuyển tiền. | — |
| `nha_giao_dich` | **Cần người duyệt — người duyệt là TƯ VẤN VIÊN, không phải khách.** AI **cấm** gọi; khách **không có nút**. | Tư vấn viên thấy: tóm tắt chat, lời khách, kết quả xác thực, người nhận mới/đã đăng ký, khuyến nghị + độ chắc của AI. **Luật thêm:** ca nghi lừa → phải gọi lại qua số đăng ký trước khi được bấm nhả (ô tích bắt buộc). **Quá hạn:** giao dịch giữ nguyên, **không bao giờ tự nhả**; khách được báo giờ tư vấn viên vào ca và đường dự phòng. | **Không đảo được**, tiền đi trong vài giây. Nhả nhầm: khách mất toàn bộ (S1). Chờ người: người gấp mất ~5 phút trong giờ trực, nhiều giờ ngoài giờ (S2, S3). | Xem "tool khó xếp nhất". |
| `khoa_vi` | **Cần người duyệt — chính chủ** bấm trong hộp xác nhận (tư vấn viên cũng khoá được) | Hộp xác nhận nói rõ: khoá cả ví, mở lại phải gọi tổng đài, mất nhiều giờ, không khoá riêng thẻ. Khi có dấu hiệu chiếm tài khoản (S7), nút "Khoá ví ngay" lên đầu. Không ai bấm: chuyển tư vấn viên quyết. | Khoá nhầm: khách mất dùng ví nhiều giờ (S4, S5 — khách chỉ muốn khoá thẻ). Không khoá kịp: S7 mất số dư. Một lần bấm xác nhận ≈ 2 giây, chặn được khoá nhầm. | *Tự làm*: loại vì S4/S5 mơ hồ → khoá nhầm. *Cấm*: loại vì S7 cần khoá trong vài giây, gọi tổng đài chậm hơn. |

### Tool khó xếp nhất: `nha_giao_dich`

Mức nào cũng có một kịch bản ở §2 bị hại. Bài làm cá nhân nên không có "mỗi người chọn gì"; thay vào đó thử **từng phương án trên hai phía**:

| Phương án | Phía S1 (bị lừa) | Phía S2 (gấp thật) | Kết luận |
|---|---|---|---|
| Tự làm khi khách xác thực đúng | ❌ Nạn nhân quét mặt đúng → nhả → mất 48 triệu | ✅ Nhanh | Loại: chặn bằng thứ kẻ lừa đã vượt qua |
| Cần **khách** xác nhận | ❌ Người bị giữ máy vẫn bấm xác nhận (lỗi câu chữ không cứu được, §3) | ✅ Nhanh | Loại |
| Cấm hẳn trong chat, chỉ qua tổng đài | ✅ An toàn | ❌ Gọi tổng đài, xếp hàng, kể lại từ đầu khi đang đứng ở quầy | Loại: người gấp bị bỏ lơ |
| **Cần tư vấn viên duyệt ngay trong chat, AI soạn đề xuất** | ✅ Tư vấn viên gọi lại qua số đăng ký, cắt được kẻ lừa | ⚠️ Chờ ~5 phút trong giờ trực; ngoài giờ chờ tới 8h | **Chọn.** Chấp nhận cái giá chờ của S2, giảm bằng hàng ưu tiên, một cú bấm cho tư vấn viên, và đường dự phòng (trả cách khác). |

Tool khó thứ hai là `khoa_vi` (S7 cần nhanh, S4/S5 dễ khoá nhầm); chốt bằng "một lần bấm của chính chủ" vì bấm mất 2 giây — rẻ hơn nhiều so với vài giờ mở khoá.

### Thử mức đã đặt trên 3 kịch bản

| | S1 (bị lừa) | S2 (gấp thật) | S11 (bình thường: nạp điện thoại 100k bị giữ) |
|---|---|---|---|
| AI làm gì | Giữ (tự làm) · chuyển tư vấn viên ưu tiên cao (tự làm) · đề xuất "không nhả" (tự làm) · không có nút nhả | Gửi xác thực (tự làm) · chuyển tư vấn viên ưu tiên (tự làm) · đề xuất "có thể nhả" | Gửi xác thực · chuyển tư vấn viên (hàng thường) · đề xuất "có thể nhả, độ chắc cao" |
| Cái sai ở §2 có bị chặn không? | **Có.** Không đường nào cho tiền đi mà không qua tư vấn viên gọi lại. | **Có** cả hai hướng: không vòng vo (một nút), không tự nhả. | Không có cái sai lớn để chặn. |
| Còn việc nào AI làm khi lẽ ra phải hỏi? | Không. Giữ và chuyển là việc rẻ, đảo được. | Không. | Không. |
| Người dùng bình thường có phải chờ vô ích? | — | Chờ ~5 phút (chấp nhận, có dự phòng). | **Có — đây là cái giá.** Khoản 100k rõ ràng vẫn phải chờ tư vấn viên bấm. Giảm bằng: đề xuất độ chắc cao → tư vấn viên duyệt một cú bấm. Ghi vào lỗi còn lại (§6). |

### Một mức tự chủ đã đổi lời hứa với người dùng

Đặt `nha_giao_dich` ở mức "chỉ tư vấn viên" đổi lời hứa từ **"Trợ lý giải quyết giao dịch bị giữ cho bạn"** thành **"Trợ lý giữ tiền của bạn an toàn và đưa bạn tới người có quyền cho tiền đi, nhanh nhất có thể. Trợ lý không tự cho tiền đi."** Vì vậy câu "Mình không tự cho giao dịch đi được — tư vấn viên quyết" xuất hiện trong cả S1 và S2: khách biết trước ai quyết, không chờ AI làm điều nó không được làm.

---

## §5. Prototype

- **File:** [`prototype/index.html`](prototype/index.html) — một file HTML, mở bằng trình duyệt, không backend, không đăng nhập. Công cụ: HTML/CSS/JS viết tay cùng coding agent (Claude).
- **Phần mock:** toàn bộ dữ liệu; câu trả lời AI viết sẵn theo §3; ô "ca lạ" phân loại tin bằng **luật từ khoá** (không gọi AI thật). Không có lời gọi AI thật → không nhận bonus +3.
- **Cách bấm ca khó:** chọn kịch bản ở thanh trên ("S1 · Bị lừa đòi nhả" / "S2 · Gấp ở quầy viện phí" / "Ca lạ"); bấm các nút trong khung chat bên trái, rồi bấm quyết định ở **bàn tư vấn viên** bên phải. Công tắc "Giờ hiện tại" đổi 15:02 ↔ 23:48 để xem đường ngoài giờ trực.

**Ảnh đánh số:** [`prototype/anh-1-S1-bi-lua.png`](prototype/anh-1-S1-bi-lua.png) · [`prototype/anh-2-S2-gap-that.png`](prototype/anh-2-S2-gap-that.png)

| Số | Thể hiện gì | Nguyên tắc | Mức tự chủ |
|---|---|---|---|
| ❶ | Nhãn "Trợ lý AI của Ví Mẫu" + chip "Mình chưa chắc…" | G2 (+ Luật AI Đ11) | — |
| ❷ | Dòng "Dựa vào": chỉ lời khách nhắn + thông tin công khai về người nhận | G11 + PAIR Explainability | — |
| ❸ | Dòng trạng thái "Đã giữ giao dịch · Đã chuyển tư vấn viên" + nút "Huỷ chuyển" | PAIR Feedback & Control | **Tự làm** + đường lui |
| ❹ | Nút "Mình không bị lừa — gửi thêm cho tư vấn viên" và "Sai chỗ nào?" | G9 + G15 | Sửa AI nhưng không nhả |
| ❺ | Ô "Chỉ tư vấn viên cho giao dịch đi được" — **không có nút nhả** phía khách | G1 (làm rõ hệ thống làm được gì) | **Cấm** với AI và khách |
| ❻ | Bàn tư vấn viên: đề xuất của AI + độ chắc + tín hiệu + ô tích "đã gọi lại qua số đăng ký" + nút Nhả / Không nhả | PAIR Explainability (cho nhân viên) | **Cần người duyệt** |
| ❼ | Đường lui: gọi tổng đài, "Mình đã trả cách khác", báo giờ vào ca khi ngoài giờ | PAIR Errors + Graceful Failure | — |
| ❽ | S2: chỉ một nút chính "Xác thực khuôn mặt" + thanh 2 bước có đồng hồ | G10 | Tự làm (`yeu_cau_xac_minh`) |
| ❾ | Hộp xác nhận khoá ví nói rõ hậu quả | G8 (gạt bỏ dễ: "Không khoá") | **Cần chính chủ duyệt** |

**Quyết định cách hiện:**
- *Độ chắc:* chọn **câu chữ** ("Mình chưa chắc giao dịch này an toàn") thay cho thanh % hay đèn xanh/đỏ. Loại thanh %: người đang hoảng đọc "62%" không biết làm gì; con số còn gợi ý cho kẻ lừa biết sắp qua ngưỡng. Phía tư vấn viên thì dùng **mức chữ (cao/trung bình/thấp) kèm danh sách tín hiệu**, vì nhân viên cần căn cứ để quyết.
- *Bằng chứng phía khách:* chọn **trích lại lời khách** thay cho "hệ thống phát hiện bất thường vì…". Loại cách giải thích theo quy tắc vì vi phạm "không lộ cách phát hiện".
- *Mức cấm:* chọn **không có nút** + một dòng giải thích ai làm được. Đã nghĩ tới nút nhả bị làm mờ (disabled) — loại, vì nút mờ vẫn gợi ý "có cách bấm được", khách sẽ hỏi cách mở.
- *Mức cần duyệt:* đặt hẳn **màn hình thứ hai** cho tư vấn viên thay cho hộp "đang chờ duyệt" mờ mịt, để người xem thấy *ai* bấm và *thấy gì trước khi bấm*. Ô tích "đã gọi lại qua số đăng ký" khoá nút Nhả ở ca nghi lừa — đây là phần thiết kế cứu lỗi S1 mà câu chữ không cứu được.
- *Cách khác đã nghĩ tới:* công tắc ba mức tự chủ như bản mẫu của giảng viên — loại, vì điểm khó của T5 không phải "AI làm nhiều hay ít" mà là "**ai** được quyết nhả"; hai màn hình khách/tư vấn viên cho thấy điều đó rõ hơn.

---

## §6. Tốt là gì

| # | Tiêu chí Pass/Fail | S1 | S2 |
|---|---|---|---|
| T1 | Không có đường nào để giao dịch bị giữ được nhả mà **không qua tư vấn viên bấm**; phía khách **không có nút nhả** ở bất kỳ trạng thái nào. | Pass | Pass |
| T2 | Khi khách nói đang gấp, **tin trả lời đầu tiên ≤ 3 câu**, có **đúng một** nút hành động chính, và nói **thời gian chờ ước tính hoặc giờ tư vấn viên vào ca**. | (không áp dụng) | Pass trong giờ trực · **Pass về câu chữ nhưng chờ tới 8h** ngoài giờ |
| T3 | Không câu trả lời nào chứa: con số ngưỡng/số lần, yêu cầu OTP / mật khẩu / số thẻ, hay lời hứa hoàn tiền (kiểm bằng tìm chữ trong toàn bộ câu trả lời). | Pass | Pass |

**Lỗi còn lại (chưa sửa được):**
1. **Người bình thường vẫn phải chờ** (S11): mọi lần nhả đều cần tư vấn viên. Chưa có cách an toàn cho AI tự nhả khoản nhỏ mà không mở đường cho kẻ lừa chia nhỏ (S8).
2. **Ngoài giờ trực**, người gấp như S2 phải dùng đường dự phòng; ví không cho đi được giao dịch đó tới 8h.
3. Ô "ca lạ" phân loại bằng **từ khoá** — sẽ trượt với cách nói mới. Sản phẩm thật cần mô hình phân loại + đánh giá riêng (buổi Evaluation).

---

## §7. Demo

Chưa có buổi demo trực tiếp để ghi lại ca lạ. Dưới đây là **ca lạ dự kiến** và cách prototype xử lý (bấm được trong tab "Ca lạ"):

| Ca lạ dự kiến | Prototype phân loại | Đi đường nào (theo mức ở §4) |
|---|---|---|
| "nv ngân hàng gọi bảo tài khoản em bị khoá, cần chuyển qua tài khoản an toàn, chuyển gấp không mất tiền" | Nghi lừa đảo | Giữ (tự làm) · chuyển tư vấn viên ưu tiên cao · cảnh báo "ngân hàng không yêu cầu chuyển sang tài khoản an toàn" · không có nút nhả |
| "chuyển dưới 5tr có bị hỏi ko" | Dò ngưỡng | Không nói ngưỡng · hỏi có ai đang hướng dẫn chuyển · đưa đường xác thực |
| "có ai rút tiền của tôi" | Mơ hồ | Hỏi lại một câu, không kết luận, không khoá |

Sau ca lạ, quy tắc ưu tiên của bộ phân loại: **nghi lừa đảo thắng "gấp"** — vì kẻ lừa luôn giục gấp.
