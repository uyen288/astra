# ASTRA — Tài liệu nghiệp vụ (Business Requirements)

> Web app Tử Vi: **"Lập lá số để xem — Sổ tay để hiểu — Cộng đồng để học."** Tài liệu liên quan: [`ui.md`](https://claude.ai/chat/ui.md) (giao diện User + Admin) · [`technical-user.md`](https://claude.ai/chat/technical-user.md) · [`technical-admin.md`](https://claude.ai/chat/technical-admin.md)

|Thông tin|Nội dung|
|---|---|
|Sản phẩm|Astra — Web app Tử Vi|
|Phiên bản tài liệu|1.0|
|Công nghệ dự kiến|React, Supabase, Vercel|
|Ngôn ngữ sản phẩm|Tiếng Việt|
|Trạng thái|Bản nháp để thống nhất phạm vi MVP|

---

## 1. Tổng quan

### 1.1 Bối cảnh

Người quan tâm Tử Vi hiện phải dùng nhiều nơi riêng lẻ: một nơi lập lá số, một nơi đọc kiến thức, một nơi hỏi đáp. Các công cụ lập lá số thường thiếu giải thích đi kèm, còn diễn đàn thì rời rạc, không gắn với lá số cụ thể.

### 1.2 Tầm nhìn sản phẩm

Một nơi duy nhất để **xem lá số → hiểu từng sao/cung → học và trao đổi**, trong đó 3 phần bổ trợ nhau:

|Trụ cột|Vai trò|
|---|---|
|🔮 Lập lá số|Xem — tạo giá trị ngay lập tức, thu hút người dùng|
|📚 Sổ tay|Hiểu — giải thích sao, cung, cách cục; giữ chân người dùng|
|💬 Cộng đồng|Học — hỏi đáp gắn với lá số và kiến thức|

### 1.3 Mục tiêu kinh doanh

1. Cung cấp công cụ lập lá số **chính xác, dễ dùng**, không bắt đăng nhập.
2. Xây thư viện kiến thức Tử Vi có cấu trúc, tìm kiếm được.
3. Chuyển người dùng ẩn danh thành người dùng đăng ký (lưu lá số, bookmark).
4. Hình thành cộng đồng Q&A chuyên đề **sau khi** đã có lượng người dùng đủ.
5. Vận hành được bởi đội nhỏ nhờ công cụ Admin đầy đủ.

### 1.4 Chỉ số thành công (KPI)

|Nhóm|Chỉ số|Mục tiêu tham khảo (giai đoạn đầu)|
|---|---|---|
|Thu hút|Số lá số được lập / tuần|Tăng đều theo tuần|
|Chuyển đổi|% khách lập lá số → đăng ký|≥ 15%|
|Giữ chân|% user quay lại trong 30 ngày|≥ 25%|
|Nội dung|Số bài Sổ tay đã xuất bản|≥ 60 bài cốt lõi khi launch|
|Tìm kiếm|% lượt tìm có kết quả|≥ 90%|
|Cộng đồng|Tỉ lệ câu hỏi có trả lời trong 48h|≥ 60% (khi mở Q&A)|
|Chất lượng|Tỉ lệ báo lỗi lá số sai|< 1%|

> Các con số là mục tiêu gợi ý, cần điều chỉnh theo thực tế sau khi đo dữ liệu đầu tiên.

---

## 2. Đối tượng & Vai trò (Actors)

### 2.1 Nhóm người dùng mục tiêu (Persona)

|Persona|Mô tả|Nhu cầu chính|
|---|---|---|
|**Người xem tò mò**|Biết sơ về Tử Vi, muốn xem lá số của mình/người thân|Lập lá số nhanh, hiểu cơ bản, không phải đăng ký|
|**Người mới học**|Muốn học Tử Vi bài bản|Lộ trình, giải thích dễ hiểu, quiz, áp dụng vào lá số thật|
|**Người nghiên cứu**|Đã có kiến thức, luận giải nhiều lá số|Lưu nhiều lá số, ghi chú, thảo luận chuyên sâu|
|**Người đóng góp**|Giàu kinh nghiệm, muốn chia sẻ|Trả lời câu hỏi, viện dẫn kiến thức|

### 2.2 Vai trò trong hệ thống

|Vai trò|Mô tả|Quyền chính|
|---|---|---|
|**Khách**|Chưa đăng nhập|Lập lá số, xem Sổ tay, xem hỏi đáp công khai, xem lá số chia sẻ bằng link/công khai|
|**User**|Đã đăng nhập|Lưu và quản lý lá số, bookmark, ghi chú, học, hỏi/đáp, vote, báo cáo|
|**Editor**|Biên tập nội dung|Tạo/sửa/xuất bản bài Sổ tay, quản lý bài học & quiz|
|**Moderator**|Kiểm duyệt cộng đồng|Xử lý báo cáo, ẩn/xoá nội dung, cảnh cáo/khoá tạm user|
|**Admin**|Quản trị toàn hệ thống|Mọi quyền trên + quản lý user, vai trò, cài đặt, thông báo, audit log|

> Không vai trò nào (kể cả Admin) được đọc lá số **riêng tư** của người dùng khác.

---

## 3. Phạm vi

### 3.1 Trong phạm vi

|Module|Mô tả|
|---|---|
|M1. Lập lá số|Nhập thông tin sinh → lá số 12 cung, tương tác, đại hạn/tiểu hạn|
|M2. Chia sẻ lá số|Riêng tư / Link / Công khai, URL riêng|
|M3. Tài khoản|Đăng nhập, lưu & quản lý nhiều lá số|
|M4. Sổ tay|Thư viện kiến thức, danh mục, bài viết có cấu trúc|
|M5. Tìm kiếm|Tìm sao, cung, cách cục, bài viết (hỗ trợ không dấu)|
|M6. Bookmark & Ghi chú|Lưu bài, ghi chú cá nhân|
|M7. Hỏi đáp (Q&A)|Hỏi–đáp chuyên Tử Vi, vote, tag, báo cáo|
|M8. Học Tử Vi|Lộ trình, bài học, quiz, tiến độ|
|M9. Quản trị (Admin)|Dashboard, biên tập, kiểm duyệt, người dùng, cài đặt, audit log|

### 3.2 Ngoài phạm vi (giai đoạn đầu)

- Thanh toán, gói trả phí, bán dịch vụ xem số.
- Luận giải tự động bằng AI cho toàn bộ lá số.
- Ứng dụng mobile native (chỉ web responsive).
- Chat/nhắn tin riêng giữa người dùng.
- Flashcard / spaced repetition (đề xuất cho giai đoạn sau Phase 6).
- Các môn khác (Bát Tự, Chiêm tinh phương Tây...).
- Đa ngôn ngữ (chỉ tiếng Việt).

### 3.3 Giả định & ràng buộc

- Nội dung Sổ tay do đội ngũ/biên tập viên có chuyên môn soạn; cần xác định nguồn tham khảo và trường phái áp dụng.
- Quy tắc an sao dựa trên **một trường phái được chọn và công bố rõ** trong sản phẩm.
- Sản phẩm mang tính **tham khảo/giải trí – học thuật**, không thay thế tư vấn chuyên môn (y tế, pháp lý, tài chính).
- Dữ liệu ngày giờ sinh là dữ liệu cá nhân, phải bảo vệ theo mặc định.

---

## 4. Quy trình nghiệp vụ (Business Flows)

### 4.1 Khách lập lá số và đăng ký

```
Trang chủ → Lập lá số → Nhập ngày/giờ/giới tính/nơi sinh
→ Hệ thống tính toán → Hiển thị lá số 12 cung
→ (Khách muốn lưu) "Đăng nhập để lưu lá số"
→ Đăng nhập → Lá số được gắn vào tài khoản → Xem trong "Lá số của tôi"
```

**Điểm nghiệp vụ:** khách xem trọn vẹn lá số trước; chỉ khi cần **lưu/chia sẻ lâu dài** mới yêu cầu đăng nhập.

### 4.2 Chia sẻ lá số

```
Chủ lá số → Chọn mức riêng tư (Riêng tư / Link / Công khai)
→ [Link] hệ thống sinh token → Chủ sao chép link gửi người khác
→ Người nhận mở link → xem lá số (chỉ đọc)
→ Chủ có thể đổi mức riêng tư / thu hồi bất cứ lúc nào
```

### 4.3 Học một bài trong lộ trình

```
Chọn lộ trình → Chọn bước → Đọc bài → Làm quiz
→ Đạt yêu cầu → "Áp dụng vào lá số" (mở lá số của user với phần liên quan được làm nổi bật)
→ Đánh dấu hoàn thành → Cập nhật tiến độ → Gợi ý bài tiếp theo
```

### 4.4 Đặt câu hỏi và trả lời

```
User đặt câu hỏi (tiêu đề, nội dung, danh mục, tag, [đính kèm lá số])
→ Câu hỏi hiển thị ở danh sách
→ User khác trả lời (có thể dẫn Sổ tay, trích sao/cung, đính kèm phần lá số)
→ Cộng đồng vote → Người hỏi chấp nhận câu trả lời tốt nhất
```

### 4.5 Kiểm duyệt nội dung

```
User báo cáo nội dung (chọn lý do)
→ Báo cáo vào hàng đợi (gộp theo đối tượng)
→ Moderator xem ngữ cảnh → quyết định:
   ├─ Bỏ qua
   ├─ Ẩn / Xoá nội dung
   └─ Cảnh cáo / Khoá tạm tác giả
→ Hệ thống ghi audit log → (tuỳ chọn) thông báo cho tác giả
```

### 4.6 Xuất bản bài viết Sổ tay

```
Editor tạo bài (Nháp) → Soạn nội dung, chọn loại/danh mục/entity_key
→ Gửi duyệt (Review) → Admin xem → Xuất bản (hoặc hẹn lịch) 
→ Bài hiển thị trên Sổ tay và đưa vào chỉ mục tìm kiếm
→ Có thể gỡ về Nháp / Lưu trữ khi cần (giữ lịch sử phiên bản)
```

### 4.7 Xử lý vi phạm người dùng

```
Cảnh cáo → (tái phạm) Khoá tạm có thời hạn → (nghiêm trọng) Khoá vĩnh viễn
Mọi bước: ghi lý do, thời hạn, người thực hiện; user bị khoá thấy thông báo rõ lý do.
```

---

## 5. Yêu cầu chức năng

> Ký hiệu độ ưu tiên: **P0** = bắt buộc cho MVP · **P1** = quan trọng, làm sau MVP · **P2** = nên có.

### 5.1 M1 — Lập lá số

|Mã|Yêu cầu|Ưu tiên|
|---|---|---|
|FR-1.1|Cho phép nhập: ngày sinh (dương hoặc âm lịch), giờ sinh (12 canh giờ hoặc giờ-phút), giới tính, nơi sinh, năm xem hạn|P0|
|FR-1.2|Khi nhập âm lịch, hỗ trợ chọn **tháng nhuận**|P0|
|FR-1.3|Tính và hiển thị lá số 12 cung kèm Can Chi (năm/tháng/ngày/giờ), Mệnh/Thân, Cục, Âm Dương|P0|
|FR-1.4|Hiển thị Chính tinh, Phụ tinh (phân biệt cát/sát), Tứ Hóa trong từng cung|P0|
|FR-1.5|Hiển thị Đại hạn (12 hạn) và Tiểu hạn/lưu niên theo năm xem|P0|
|FR-1.6|Click cung: làm nổi bật tam hợp, xung chiếu và hiện chi tiết|P0|
|FR-1.7|Click sao: hiện giải thích ngắn và liên kết tới bài trong Sổ tay|P0|
|FR-1.8|Chuyển giữa các đại hạn để xem lá số theo hạn đó|P0|
|FR-1.9|Phóng to / thu nhỏ / đặt lại khung xem lá số|P0|
|FR-1.10|Khách dùng được toàn bộ chức năng trên mà không cần đăng nhập|P0|
|FR-1.11|Kiểm tra dữ liệu đầu vào (ngày hợp lệ, giờ hợp lệ) với thông báo tiếng Việt rõ ràng|P0|
|FR-1.12|Hiển thị rõ **trường phái / quy ước** đang áp dụng và disclaimer|P1|
|FR-1.13|Xuất lá số ra ảnh/PDF để lưu hoặc in|P2|

### 5.2 M2 — Chia sẻ lá số

|Mã|Yêu cầu|Ưu tiên|
|---|---|---|
|FR-2.1|Mỗi lá số lưu có URL riêng dạng `/la-so/:id`|P0|
|FR-2.2|Ba mức: 🔒 Riêng tư · 🔗 Chia sẻ bằng link · 🌎 Công khai; **mặc định Riêng tư**|P0|
|FR-2.3|Chủ lá số đổi mức riêng tư bất cứ lúc nào; đổi về Riêng tư thì link cũ **vô hiệu**|P0|
|FR-2.4|Cảnh báo khi chọn Công khai ("ngày giờ sinh sẽ hiển thị cho mọi người")|P0|
|FR-2.5|Tuỳ chọn ẩn tên / ẩn ngày sinh đầy đủ khi chia sẻ|P1|
|FR-2.6|Người không có quyền mở lá số riêng tư thấy trang 403 thân thiện|P0|
|FR-2.7|Trang lá số công khai có Open Graph để chia sẻ mạng xã hội (không lộ dữ liệu nhạy cảm)|P2|

### 5.3 M3 — Tài khoản

|Mã|Yêu cầu|Ưu tiên|
|---|---|---|
|FR-3.1|Đăng nhập bằng Google hoặc Email (magic link)|P0|
|FR-3.2|Lá số khách vừa lập được gắn vào tài khoản sau khi đăng nhập|P0|
|FR-3.3|Quản lý nhiều lá số: thêm, đổi tên/nhãn (VD "Bố", "Mẹ"), xoá, chia sẻ|P0|
|FR-3.4|Trang "Bài viết đã lưu", "Ghi chú", "Câu hỏi của tôi", "Câu trả lời của tôi"|P0/P1|
|FR-3.5|Cập nhật hồ sơ (tên hiển thị, ảnh đại diện)|P1|
|FR-3.6|Xoá tài khoản kèm toàn bộ dữ liệu cá nhân (có xác nhận)|P0|
|FR-3.7|Trang chủ cá nhân hoá khi đã đăng nhập (lá số gần đây, đang học, câu hỏi của bạn)|P1|

### 5.4 M4 — Sổ tay

|Mã|Yêu cầu|Ưu tiên|
|---|---|---|
|FR-4.1|Cây danh mục: Nhập môn · Âm Dương–Ngũ Hành · Thiên Can–Địa Chi · Các sao (Chính tinh, Phụ tinh, Cát tinh, Sát tinh, Tứ Hóa) · 12 Cung · Tam hợp–Xung chiếu · Đại hạn–Tiểu hạn · Cách cục · Bài viết|P0|
|FR-4.2|Mỗi bài theo mẫu: Tổng quan · Đặc điểm · Ở 12 cung · Khi gặp sao khác · Ví dụ · Đọc tiếp|P0|
|FR-4.3|Bài có "Xem lá số mẫu" mở lá số minh hoạ|P1|
|FR-4.4|Liên kết hai chiều: sao/cung trên lá số ↔ bài tương ứng (qua mã định danh `entity_key`)|P0|
|FR-4.5|Mục lục bài và gợi ý "Đọc tiếp"|P0|
|FR-4.6|Chỉ nội dung **đã xuất bản** hiển thị cho công chúng|P0|

### 5.5 M5 — Tìm kiếm

|Mã|Yêu cầu|Ưu tiên|
|---|---|---|
|FR-5.1|Một ô tìm kiếm toàn cục: sao, cung, cách cục, bài viết|P0|
|FR-5.2|Kết quả nhóm theo loại (Sao / Cung / Cách cục / Bài viết)|P0|
|FR-5.3|Gõ **không dấu** vẫn ra kết quả ("tu vi" → "Tử Vi")|P0|
|FR-5.4|Gợi ý khi đang gõ, điều hướng bằng bàn phím, phím tắt mở nhanh|P1|
|FR-5.5|Ghi nhận từ khoá tìm không có kết quả (ẩn danh) để biên tập bổ sung nội dung|P1|

### 5.6 M6 — Bookmark & Ghi chú

|Mã|Yêu cầu|Ưu tiên|
|---|---|---|
|FR-6.1|User lưu / bỏ lưu bài viết|P0|
|FR-6.2|Danh sách bài đã lưu|P0|
|FR-6.3|Ghi chú cá nhân gắn với bài viết hoặc lá số; chỉ chủ ghi chú xem được|P1|

### 5.7 M7 — Hỏi đáp (Q&A)

|Mã|Yêu cầu|Ưu tiên|
|---|---|---|
|FR-7.1|Đặt câu hỏi: tiêu đề, nội dung, danh mục, tag, đính kèm lá số (tuỳ chọn)|P1|
|FR-7.2|Danh mục: Tử Vi căn bản · Lá số · Cung Mệnh · Công danh · Tài lộc · Tình duyên · Đại vận · Tiểu vận · Khác|P1|
|FR-7.3|Danh sách có 3 chế độ: Mới nhất · Nhiều tương tác · Chưa có câu trả lời; lọc theo danh mục/tag|P1|
|FR-7.4|Trả lời: viết nội dung, chèn link Sổ tay, trích sao/cung, đính kèm phần lá số|P1|
|FR-7.5|Khối **"Câu trả lời này dựa trên kiến thức nào?"** liệt kê bài Sổ tay được viện dẫn (nối forum ↔ knowledge base)|P1|
|FR-7.6|Vote câu hỏi/câu trả lời (mỗi user 1 vote/đối tượng)|P1|
|FR-7.7|Người hỏi chấp nhận một câu trả lời tốt nhất|P1|
|FR-7.8|Báo cáo nội dung vi phạm kèm lý do|P1|
|FR-7.9|Thông báo khi có trả lời mới, khi câu trả lời được chấp nhận|P2|
|FR-7.10|Lá số đính kèm tuân thủ mức riêng tư: người xem chỉ thấy phần người đăng đã đồng ý chia sẻ|P1|
|FR-7.11|Chỉ mở module Hỏi đáp khi đủ lượng người dùng (điều khiển bằng feature flag)|P1|

### 5.8 M8 — Học Tử Vi

|Mã|Yêu cầu|Ưu tiên|
|---|---|---|
|FR-8.1|Lộ trình 6 bước: Âm Dương–Ngũ Hành → Thiên Can–Địa Chi → 12 Cung → 14 Chính tinh → Tam hợp–Xung chiếu → Đại hạn|P1|
|FR-8.2|Mỗi bài: Đọc → Quiz → Áp dụng vào lá số|P1|
|FR-8.3|Quiz trắc nghiệm, phản hồi đúng/sai và giải thích ngay|P1|
|FR-8.4|Lưu và hiển thị tiến độ (✓ hoàn thành / → đang học); % tiến độ tổng|P1|
|FR-8.5|Hiển thị "Đang học" ở trang chủ cá nhân|P1|
|FR-8.6|Flashcard / spaced repetition|P2 (sau)|

### 5.9 M9 — Quản trị (Admin)

|Mã|Yêu cầu|Ưu tiên|
|---|---|---|
|FR-9.1|Dashboard: số liệu lá số, user mới, câu hỏi, báo cáo chờ; khối "Cần xử lý"|P0|
|FR-9.2|Quản lý bài Sổ tay: danh sách, soạn (Markdown + xem trước + tự lưu nháp), danh mục, bài liên quan|P0|
|FR-9.3|Quy trình bài: Nháp → Chờ duyệt → Xuất bản → Lưu trữ; hẹn lịch xuất bản; lịch sử phiên bản và khôi phục|P0/P1|
|FR-9.4|Nhập hàng loạt nội dung từ Markdown/CSV|P1|
|FR-9.5|Quản lý người dùng: xem, tìm, lọc; đổi vai trò; khoá/mở khoá; xoá tài khoản|P0|
|FR-9.6|Hàng đợi báo cáo: gộp theo đối tượng, xem ngữ cảnh, ẩn/xoá/bỏ qua/cảnh cáo, ghi chú nội bộ|P1|
|FR-9.7|Quản lý Q&A: ẩn/khôi phục/xoá, ghim, đổi danh mục, gộp câu hỏi trùng|P1|
|FR-9.8|Quản lý lộ trình, bài học, quiz|P1|
|FR-9.9|Thống kê lá số (tổng hợp, ẩn danh) + kiểm duyệt lá số công khai|P1|
|FR-9.10|Thông báo hệ thống: banner, thông báo trong app, email|P2|
|FR-9.11|Nhật ký hành động (audit log): lọc, xuất CSV, không sửa/xoá|P0|
|FR-9.12|Cài đặt hệ thống & bật/tắt tính năng (Hỏi đáp, Học, bảo trì...)|P1|

---

## 6. Quy tắc nghiệp vụ (Business Rules)

### 6.1 Lá số & riêng tư

|Mã|Quy tắc|
|---|---|
|BR-1|Lá số mới tạo luôn ở mức **Riêng tư** trừ khi chủ đổi.|
|BR-2|Chỉ chủ sở hữu được xem/sửa/xoá lá số riêng tư của mình. **Không ai khác**, kể cả Admin, đọc được lá số riêng tư.|
|BR-3|Lá số "Link" chỉ truy cập được khi có token hợp lệ; thu hồi (đổi về Riêng tư) thì token cũ vô hiệu ngay.|
|BR-4|Lá số "Công khai" hiển thị cho mọi người; hệ thống cảnh báo trước khi chuyển sang mức này.|
|BR-5|Admin chỉ xem **số liệu tổng hợp ẩn danh** về lá số; chỉ xem chi tiết lá số khi nó **Công khai** hoặc được người dùng đính kèm vào nội dung bị báo cáo.|
|BR-6|Khách có thể lập lá số mà không cần tài khoản; lá số khách không lưu vĩnh viễn trừ khi đăng nhập.|
|BR-7|Khi xoá lá số/tài khoản, dữ liệu liên quan bị xoá; nội dung Q&A đính kèm lá số đó chuyển về trạng thái "lá số không còn khả dụng".|
|BR-8|Lá số lưu kèm **phiên bản engine** tính ra nó, để khi quy tắc được cải tiến vẫn truy vết/tính lại được.|

### 6.2 Tính lá số

|Mã|Quy tắc|
|---|---|
|BR-10|Giờ sinh quy đổi về **12 canh giờ** (Tý, Sửu, Dần, Mão, Thìn, Tị, Ngọ, Mùi, Thân, Dậu, Tuất, Hợi).|
|BR-11|Ngày sinh dương lịch được đổi sang âm lịch (có xử lý tháng nhuận) để an sao; tất cả phép an sao dựa trên âm lịch và Can Chi.|
|BR-12|**Âm Dương** xác định theo Can năm sinh; kết hợp giới tính để quyết định **chiều đi của Đại hạn** (Dương Nam, Âm Nữ: thuận · Âm Nam, Dương Nữ: nghịch).|
|BR-13|Cục (Thủy nhị / Mộc tam / Kim tứ / Thổ ngũ / Hỏa lục) xác định từ Can năm sinh và cung Mệnh; số Cục là cơ sở khởi Đại hạn và an Tử Vi.|
|BR-14|Mỗi Đại hạn kéo dài **10 năm**; có 12 Đại hạn tương ứng 12 cung.|
|BR-15|Tứ Hóa (Lộc, Quyền, Khoa, Kỵ) xác định theo **Thiên Can năm sinh** (và theo Can cung khi xem hạn) theo bảng của trường phái đã chọn.|
|BR-16|Mọi quy ước có khác biệt giữa các trường phái (cách an một số sao, Tiểu hạn...) phải **chọn một cách, ghi tài liệu, và hiển thị cho người dùng**.|
|BR-17|Kết quả phải đúng với **bộ lá số chuẩn** (golden cases) do chuyên gia xác nhận trước khi phát hành.|

### 6.3 Nội dung Sổ tay

|Mã|Quy tắc|
|---|---|
|BR-20|Chỉ bài có trạng thái **Xuất bản** hiển thị công khai và vào chỉ mục tìm kiếm.|
|BR-21|Mỗi bài có `slug` duy nhất; bài về sao/cung có `entity_key` duy nhất để nối với lá số.|
|BR-22|Xuất bản bài phải có tiêu đề, tóm tắt, danh mục, slug hợp lệ.|
|BR-23|Mọi lần xuất bản/chỉnh sửa lưu lịch sử phiên bản; xoá bài là **xoá mềm**, khôi phục được.|
|BR-24|Nội dung phải ghi nguồn tham khảo khi trích dẫn; không sao chép nguyên văn nội dung có bản quyền.|

### 6.4 Cộng đồng Q&A

|Mã|Quy tắc|
|---|---|
|BR-30|Phải đăng nhập mới được hỏi, trả lời, vote, báo cáo.|
|BR-31|Mỗi user chỉ 1 vote cho mỗi câu hỏi/câu trả lời; không vote nội dung của chính mình.|
|BR-32|Chỉ người đặt câu hỏi được chấp nhận câu trả lời tốt nhất.|
|BR-33|Nội dung bị ẩn không hiển thị với người dùng thường nhưng vẫn lưu để Moderator xem xét.|
|BR-34|Nội dung bị ≥ **N** báo cáo độc lập được tự ẩn tạm chờ Moderator duyệt (N cấu hình được).|
|BR-35|Cấm nội dung: spam, xúc phạm/thù ghét, quảng cáo, lộ thông tin cá nhân người khác, **phán đoán tuyệt đối gây hoảng sợ** (VD khẳng định chết chóc, bệnh nặng).|
|BR-36|Có giới hạn tốc độ đăng (rate limit) để chống spam; user mới có giới hạn chặt hơn.|
|BR-37|Khi đính kèm lá số vào câu hỏi, người đăng chọn mức hiển thị; không thể đính kèm lá số của người khác.|

### 6.5 Người dùng & vi phạm

|Mã|Quy tắc|
|---|---|
|BR-40|Cấp độ xử lý: Cảnh cáo → Khoá tạm (có hạn) → Khoá vĩnh viễn. Mọi bước ghi lý do, người thực hiện.|
|BR-41|Moderator được cảnh cáo và khoá tạm; chỉ Admin khoá vĩnh viễn, xoá tài khoản, đổi vai trò.|
|BR-42|User bị khoá vẫn xem được nội dung công khai và dữ liệu của mình, nhưng **không** được tạo nội dung mới.|
|BR-43|Không thể tự đổi vai trò của chính mình; không thể hạ/xoá Admin cuối cùng.|
|BR-44|User được quyền xoá tài khoản và dữ liệu cá nhân bất cứ lúc nào.|

### 6.6 Quản trị & kiểm toán

|Mã|Quy tắc|
|---|---|
|BR-50|Mọi hành động nhạy cảm (xuất bản/gỡ bài, ẩn/xoá nội dung, đổi vai trò, khoá/xoá user, đổi cài đặt, xuất log) đều ghi **audit log**.|
|BR-51|Audit log chỉ đọc, giữ tối thiểu 12 tháng.|
|BR-52|Editor không có quyền kiểm duyệt Q&A; Moderator không có quyền xuất bản bài.|
|BR-53|Hành động rủi ro cao (xoá user, đổi vai trò) yêu cầu xác nhận lại.|
|BR-54|Admin bắt buộc bật xác thực 2 lớp (2FA).|

### 6.7 Học

|Mã|Quy tắc|
|---|---|
|BR-60|Bài học thuộc đúng một lộ trình và có thứ tự; bước sau mở khi hoàn thành bước trước (có thể cấu hình cho phép học tự do).|
|BR-61|Hoàn thành bài khi đạt ngưỡng điểm quiz (mặc định ≥ 60%); được làm lại không giới hạn.|
|BR-62|Tiến độ tổng = số bài hoàn thành / tổng số bài đang hiển thị.|

---

## 7. Câu chuyện người dùng (User Stories) tiêu biểu

|Mã|Với vai trò|Tôi muốn|Để|Tiêu chí chấp nhận|
|---|---|---|---|---|
|US-01|Khách|lập lá số chỉ bằng vài thông tin|xem ngay mà không cần đăng ký|Nhập đủ thông tin → thấy lá số 12 cung; không bị chặn đăng nhập|
|US-02|Khách|bấm vào một sao|hiểu sao đó nghĩa là gì|Hiện giải thích ngắn + link tới bài Sổ tay|
|US-03|User|lưu lá số của cha mẹ|xem lại sau này|Có nhãn, thấy trong "Lá số của tôi"|
|US-04|User|chia sẻ lá số cho bạn qua link nhưng người lạ không xem được|giữ riêng tư|Chọn "Link"; người không có link → 403; đổi về Riêng tư thì link cũ hỏng|
|US-05|Người mới học|một lộ trình học từng bước|không bị lạc giữa nhiều kiến thức|Thấy 6 bước, trạng thái từng bước, tiến độ %|
|US-06|User|tìm "tham lang" không dấu|nhanh chóng tìm bài|Kết quả hiển thị Sao Tham Lang và các bài liên quan|
|US-07|User|lưu bài và ghi chú|đọc lại khi học đến phần liên quan|Bài xuất hiện trong "Bài đã lưu"; ghi chú chỉ mình thấy|
|US-08|User|hỏi về cung Phu Thê kèm lá số của mình|nhận góp ý có dẫn chứng|Câu hỏi hiển thị kèm lá số theo mức riêng tư đã chọn|
|US-09|Người đóng góp|dẫn các bài Sổ tay vào câu trả lời|câu trả lời có căn cứ|Khối "Dựa trên kiến thức nào?" hiển thị link các bài|
|US-10|User|báo cáo câu trả lời spam|cộng đồng sạch|Báo cáo được ghi nhận, vào hàng đợi Moderator|
|US-11|Editor|viết và xuất bản bài mà không đụng database|vận hành nội dung độc lập|Soạn → gửi duyệt → xuất bản; bài hiện ở Sổ tay và tìm được|
|US-12|Moderator|xem ngữ cảnh đầy đủ của nội dung bị báo cáo|quyết định đúng|Panel hiển thị câu hỏi + chuỗi trả lời; có nút Ẩn/Bỏ qua/Cảnh cáo|
|US-13|Admin|đổi vai trò một user thành Editor|giao việc biên tập|Vai trò đổi; audit log ghi lại; người đó thấy menu Editor|
|US-14|Admin|tắt tạm module Hỏi đáp|launch MVP khi chưa có cộng đồng|Menu/route Hỏi đáp ẩn đi mà không cần deploy|
|US-15|Admin|xem thống kê lá số|hiểu người dùng|Thấy số liệu tổng hợp; **không** thấy dữ liệu sinh cá nhân|

---

## 8. Dữ liệu & thực thể nghiệp vụ

|Thực thể|Mô tả|Quan hệ chính|
|---|---|---|
|Người dùng (User/Profile)|Tài khoản, vai trò, trạng thái|Có nhiều Hồ sơ sinh, Câu hỏi, Bookmark, Tiến độ|
|Hồ sơ sinh (Birth Profile)|Thông tin ngày giờ sinh của một người (có nhãn)|Thuộc User; có nhiều Lá số|
|Lá số (Chart)|Kết quả tính toán + mức riêng tư|Thuộc Hồ sơ sinh/User; đính kèm vào Câu hỏi|
|Bài viết (Article)|Nội dung Sổ tay, thuộc danh mục, có `entity_key`|Có Phiên bản, Bài liên quan, Bookmark|
|Danh mục (Category)|Cây phân loại nội dung|Chứa Bài viết|
|Câu hỏi (Question)|Câu hỏi + tag + danh mục|Có nhiều Câu trả lời, có thể đính kèm Lá số|
|Câu trả lời (Answer)|Nội dung trả lời|Viện dẫn nhiều Bài viết|
|Vote / Báo cáo|Tương tác và kiểm duyệt|Gắn Câu hỏi/Câu trả lời|
|Lộ trình / Bài học / Tiến độ|Chương trình học|Bài học gắn Bài viết|
|Chế tài (Sanction)|Cảnh cáo/khoá tạm/khoá vĩnh viễn|Gắn User|
|Audit log|Nhật ký hành động quản trị|Gắn người thực hiện|
|Cài đặt / Feature flag|Cấu hình vận hành|Toàn hệ thống|

Quan hệ cốt lõi:

```
User ─┬─ Birth Profile ── Chart
      ├─ Question ── Answer ── (viện dẫn) Article
      ├─ Article Bookmark / Note
      └─ Learning Progress
```

### 8.1 Dữ liệu cá nhân & bảo vệ

|Loại dữ liệu|Mức nhạy cảm|Quy định|
|---|---|---|
|Ngày giờ sinh, giới tính, nơi sinh|**Cao**|Mặc định riêng tư; chỉ chủ xem; xoá khi yêu cầu|
|Email, tên|Trung bình|Chỉ dùng cho tài khoản/thông báo; không công khai email|
|Nội dung hỏi đáp|Công khai theo thiết kế|Người dùng được thông báo trước khi đăng|
|Log tìm kiếm|Thấp|Ẩn danh, không gắn user|

---

## 9. Yêu cầu phi chức năng

|Nhóm|Yêu cầu|
|---|---|
|**Độ chính xác**|Lá số phải khớp 100% bộ golden cases; có tài liệu quy ước trường phái|
|**Hiệu năng**|Tính lá số < 1 giây cảm nhận được; trang chính tải nhanh trên 4G (mục tiêu Lighthouse Performance ≥ 90)|
|**Tương thích**|Chrome, Safari, Firefox, Edge bản mới; ưu tiên mobile (iOS/Android)|
|**Responsive**|Dùng tốt từ 360px; lá số hỗ trợ zoom/pan trên mobile|
|**Bảo mật**|Phân quyền theo vai trò ở tầng dữ liệu; mã hoá khi truyền; không lộ khoá dịch vụ ở client; 2FA cho Admin|
|**Riêng tư**|Tuân thủ nguyên tắc tối thiểu hoá dữ liệu; quyền xoá dữ liệu; chính sách riêng tư rõ ràng|
|**Khả dụng (a11y)**|Đạt WCAG AA: tương phản, điều hướng bàn phím, hỗ trợ screen reader cho lá số|
|**SEO**|Trang Sổ tay có meta, Open Graph, sitemap, có thể được lập chỉ mục|
|**Độ tin cậy**|Sao lưu dữ liệu định kỳ; theo dõi lỗi (monitoring); có trang bảo trì|
|**Khả năng mở rộng**|Thêm trường phái/quy tắc mới mà không làm hỏng lá số cũ (nhờ phiên bản engine)|
|**Bản địa hoá**|Toàn bộ giao diện và thông báo bằng tiếng Việt, đúng thuật ngữ Tử Vi|

---

## 10. Lộ trình triển khai theo giá trị kinh doanh

|Giai đoạn|Nội dung|Giá trị mang lại|
|---|---|---|
|**Phase 1**|Nền tảng, hạ tầng, khung Admin cơ bản|Sẵn sàng phát triển|
|**Phase 2**|**Lập lá số** (phần quan trọng nhất)|Giá trị cốt lõi, thu hút người dùng|
|**Phase 3**|Tài khoản, lưu & chia sẻ lá số|Chuyển đổi khách → thành viên|
|**Phase 4**|Sổ tay + tìm kiếm + bookmark + công cụ biên tập|Giữ chân, SEO|
|🚀 **Launch MVP**|Phase 1–4|Lập lá số + Sổ tay + Tài khoản|
|**Phase 5**|Hỏi đáp + kiểm duyệt|Hình thành cộng đồng (khi đã có lượng người dùng)|
|**Phase 6**|Học (lộ trình, quiz, tiến độ)|Tăng gắn bó, giáo dục|

**Lý do hoãn Q&A:** diễn đàn trống nhìn "chết" và làm giảm niềm tin; nên launch trước với Lập lá số + Sổ tay, rồi mở Hỏi đáp khi đã có lưu lượng và nhu cầu thật.

**Điều kiện mở Hỏi đáp (gợi ý):** đủ số người dùng đăng ký hoạt động, có đội kiểm duyệt, có bộ quy tắc cộng đồng, có vài câu hỏi/mồi nội dung ban đầu.

---

## 11. Rủi ro nghiệp vụ & hướng xử lý

|Rủi ro|Mức|Hướng xử lý|
|---|---|---|
|Lá số tính sai / khác trường phái, mất uy tín|Cao|Golden cases do chuyên gia xác nhận; công bố trường phái; kênh báo lỗi lá số|
|Lộ dữ liệu ngày giờ sinh|Cao|Mặc định riêng tư; không ai (kể cả Admin) đọc lá số riêng tư; cảnh báo khi công khai|
|Nội dung Sổ tay thiếu / sai|Trung bình|Bắt đầu với nhóm cốt lõi; quy trình duyệt; ghi nguồn; cho phép user báo lỗi bài|
|Forum rỗng hoặc bị spam|Trung bình|Hoãn đến khi có người dùng; kiểm duyệt, rate limit, từ khoá cấm|
|Nội dung gây hoảng sợ / phán đoán cực đoan|Trung bình|Disclaimer; quy tắc cộng đồng (BR-35); Moderator xử lý nhanh|
|Phạm vi phình to|Trung bình|Bám theo Phase, có tiêu chí hoàn thành từng Phase|
|Vấn đề bản quyền nội dung|Trung bình|Biên soạn nội dung gốc; ghi nguồn; không sao chép nguyên văn|
|Thiếu nhân lực vận hành|Thấp–TB|Công cụ Admin tự phục vụ; feature flag; tự ẩn tạm theo ngưỡng báo cáo|

---

## 12. Điều khoản & thông báo bắt buộc

Cần soạn trước khi launch:

- **Điều khoản sử dụng** (quy tắc cộng đồng, quyền Admin xử lý nội dung).
- **Chính sách riêng tư** (dữ liệu thu thập, mục đích, thời gian lưu, quyền xoá).
- **Disclaimer:** _"Tử vi mang tính tham khảo, không thay thế lời khuyên chuyên môn."_
- Thông báo rõ khi chuyển lá số sang **Công khai**.

---

## 13. Thuật ngữ (Glossary)

|Thuật ngữ|Giải thích ngắn|
|---|---|
|Lá số|Bản đồ 12 cung với các sao được an theo ngày giờ sinh|
|12 cung|Mệnh, Phụ Mẫu, Phúc Đức, Điền Trạch, Quan Lộc, Nô Bộc (Giao Hữu), Thiên Di, Tật Ách, Tài Bạch, Tử Tức, Phu Thê, Huynh Đệ|
|Cung Mệnh / Thân|Hai cung trọng tâm của lá số, xác định theo tháng và giờ sinh|
|Chính tinh|14 sao chính: Tử Vi, Thiên Cơ, Thái Dương, Vũ Khúc, Thiên Đồng, Liêm Trinh, Thiên Phủ, Thái Âm, Tham Lang, Cự Môn, Thiên Tướng, Thiên Lương, Thất Sát, Phá Quân|
|Phụ tinh|Các sao phụ; chia cát tinh và sát tinh|
|Tứ Hóa|Bốn biến hóa của sao: Hóa Lộc, Hóa Quyền, Hóa Khoa, Hóa Kỵ|
|Thiên Can – Địa Chi|10 Can và 12 Chi, dùng để tính năm/tháng/ngày/giờ|
|Âm Dương|Tính chất Âm/Dương của Can năm sinh|
|Cục|Ngũ hành cục (Thủy nhị, Mộc tam, Kim tứ, Thổ ngũ, Hỏa lục); số Cục khởi Đại hạn|
|Tam hợp / Xung chiếu|Các cung có quan hệ hỗ trợ / đối xung với một cung|
|Đại hạn|Chu kỳ 10 năm theo từng cung|
|Tiểu hạn / Lưu niên|Vận hạn theo từng năm|
|Cách cục|Tổ hợp sao có ý nghĩa đặc biệt (VD Tử Phủ Vũ Tướng, Cơ Nguyệt Đồng Lương)|
|Canh giờ|12 khung giờ theo Địa Chi (mỗi khung 2 tiếng)|
|Trường phái|Cách áp dụng quy tắc an sao/luận giải khác nhau giữa các phái|
|Golden cases|Bộ lá số chuẩn đã xác nhận, dùng để kiểm tra tính chính xác|
|`entity_key`|Mã định danh một sao/cung/cách cục để nối lá số với bài viết|

---

## 14. Các điểm cần quyết định / xác nhận

|#|Câu hỏi|Người quyết định|Ảnh hưởng|
|---|---|---|---|
|1|Chọn **trường phái** an sao và Tứ Hóa nào làm chuẩn?|Chủ sản phẩm + chuyên gia Tử Vi|Toàn bộ Chart Engine|
|2|Ai là nguồn nội dung Sổ tay và ai duyệt?|Chủ sản phẩm|Tiến độ launch|
|3|Số bài Sổ tay tối thiểu khi launch?|Chủ sản phẩm|Điều kiện launch|
|4|Editor có được tự xuất bản, hay luôn cần Admin duyệt?|Chủ sản phẩm|Quy trình biên tập|
|5|Có cần nơi sinh để hiệu chỉnh giờ (múi giờ/giờ mặt trời) không?|Chuyên gia + chủ sản phẩm|Form nhập và độ chính xác|
|6|Ngưỡng tự ẩn khi bị báo cáo (N) và rate limit?|Moderator + chủ sản phẩm|Kiểm duyệt Q&A|
|7|Thời điểm / điều kiện mở Hỏi đáp?|Chủ sản phẩm|Roadmap|
|8|Có hỗ trợ xuất PDF/ảnh lá số ở MVP?|Chủ sản phẩm|Phạm vi Phase 2|
|9|Mô hình doanh thu (nếu có) trong tương lai?|Chủ sản phẩm|Ngoài phạm vi hiện tại|

---

## 15. Ma trận truy vết (Module ↔ Tài liệu)

|Module|Giao diện (`ui.md`)|Kỹ thuật User|Kỹ thuật Admin|
|---|---|---|---|
|M1 Lập lá số|Mục 5, 6|Mục 6 (Chart Engine), Phase 2|A2 (thống kê)|
|M2 Chia sẻ|Mục 6 (modal chia sẻ)|Mục 7.3–7.4, Phase 3|A2 (kiểm duyệt lá số công khai)|
|M3 Tài khoản|Mục 7|Mục 9, Phase 3|A2 (quản lý user)|
|M4 Sổ tay|Mục 8|Mục 7, Phase 4|A3|
|M5 Tìm kiếm|Mục 9|Mục 8, Phase 4|A3 (search logs, reindex)|
|M6 Bookmark/Ghi chú|Mục 7, 8|Phase 4|—|
|M7 Hỏi đáp|Mục 11|Phase 5|A4|
|M8 Học|Mục 10|Phase 6|A5|
|M9 Admin|Phần B (mục 16–27)|—|A1–A6|