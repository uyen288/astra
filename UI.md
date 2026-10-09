# ASTRA — UI / UX Spec (User + Admin)

## Mục lục

- **Phần A — Giao diện User** (mục 1–15)
- **Phần B — Giao diện Admin** (mục 16–27)

## Vai trò & quyền xem giao diện

|Vai trò|Giao diện|Quyền chính|
|---|---|---|
|Khách|Phần A (không đăng nhập)|Lập lá số, đọc Sổ tay, xem hỏi đáp|
|User|Phần A (đã đăng nhập)|Lưu lá số, bookmark, ghi chú, hỏi/đáp, học|
|Moderator|Phần B (giới hạn)|Duyệt báo cáo, ẩn/xoá nội dung hỏi đáp|
|Editor|Phần B (giới hạn)|Viết/sửa bài Sổ tay, quản lý bài học|
|Admin|Phần B (đầy đủ)|Tất cả, kèm quản lý user, vai trò, cài đặt|

---

# PHẦN A — GIAO DIỆN USER

> Concept: **"Lập lá số để xem — Sổ tay để hiểu — Cộng đồng để học."**

---

## 1. Nguyên tắc thiết kế

1. **Hành động trước, giới thiệu sau.** Homepage không có đoạn "Chào mừng...", đưa user vào thẳng 3 hành động chính.
2. **Lá số là trung tâm.** Mọi trang khác (sổ tay, hỏi đáp, học) đều có đường quay về lá số.
3. **Không bắt đăng nhập sớm.** Lập lá số → xem → mới gợi ý "Đăng nhập để lưu".
4. **Riêng tư mặc định.** Dữ liệu sinh là dữ liệu cá nhân; mọi nơi hiển thị trạng thái chia sẻ rõ ràng.
5. **Mobile-first.** Lá số 12 cung phải dùng được trên điện thoại (zoom, pan, bottom sheet).
6. **Nối kiến thức.** Click sao/cung ở đâu cũng ra giải thích, link tới Sổ tay.

---

## 2. Sitemap

```
/
├── /lap-la-so
├── /la-so/:id
│
├── /dang-nhap
├── /tai-khoan
│   ├── /la-so
│   ├── /bai-viet-da-luu
│   └── /ghi-chu
│
├── /so-tay
│   ├── /nhap-mon
│   ├── /sao
│   ├── /cung
│   ├── /cach-cuc
│   └── /bai-viet
│
├── /hoc
│   ├── /lo-trinh
│   └── /bai/:slug
│
└── /hoi-dap
    ├── /cau-hoi/:id
    └── /dat-cau-hoi
```

---

## 3. Layout chung

### Header (desktop)

```
[Logo ASTRA]  Lập lá số | Sổ tay | Học | Hỏi đáp      [🔍 Tìm...]  [Đăng nhập / Avatar]
```

### Mobile

- Top bar: Logo + 🔍 + Avatar
- **Bottom tab bar:** 🔮 Lá số · 📚 Sổ tay · 🎓 Học · 💬 Hỏi đáp · 👤 Tôi

### Footer

Giới thiệu, Điều khoản, Chính sách riêng tư, Liên hệ, Disclaimer ("Tử vi mang tính tham khảo").

---

## 4. Trang chủ `/`

### Chưa đăng nhập

```
              TỬ VI
       Khám phá lá số của bạn

      [ 🔮 LẬP LÁ SỐ ]

────────────────────────────
📚 HỌC TỬ VI
Từ cơ bản đến nâng cao
[ Bắt đầu học ]

────────────────────────────
💬 CỘNG ĐỒNG
Hỏi — đáp — thảo luận
[ Xem diễn đàn ]
```

### Đã đăng nhập

```
Chào bạn 👋

Lá số gần đây
┌────────────────────┐
│ Nguyễn Văn A       │
│ 12/03/2000 14:30   │
│ [Xem lá số]        │
└────────────────────┘

📖 Đang học: Cung Mệnh — 60%   [Học tiếp]

💬 Câu hỏi của bạn: 3 câu hỏi
```

- CTA chính **Lập lá số** luôn là nút nổi bật nhất (primary).
- Giai đoạn đầu (chưa có forum) ẩn khối "Cộng đồng" hoặc thay bằng "Sắp ra mắt".

---

## 5. Lập lá số `/lap-la-so`

### Flow

```
Trang chủ → Lập lá số → Nhập thông tin → Tính toán → Lá số 12 cung
```

### Form

|Trường|Component|Ghi chú|
|---|---|---|
|Họ tên / nhãn|Text input|Tuỳ chọn, VD "Bố", "Mẹ"|
|Ngày sinh|Date picker|Có toggle **Dương lịch / Âm lịch**|
|Giờ sinh|Select 12 canh giờ + nhập giờ chính xác|Hiển thị "Tý (23h–1h)", ...|
|Giới tính|Segmented control Nam / Nữ||
|Nơi sinh|Autocomplete tỉnh/thành|Phục vụ múi giờ / giờ mặt trời nếu cần|
|Năm xem hạn|Select|Mặc định năm hiện tại|

- Nút **[ Lập lá số ]** sticky ở đáy trên mobile.
- Validation inline, thông báo tiếng Việt thân thiện.
- Loading ngắn (skeleton lá số) khi tính toán.

---

## 6. Xem lá số `/la-so/:id`

### Bố cục desktop

```
┌──────────────────────────────┬────────────────────┐
│                              │ Thông tin chung    │
│      LƯỚI 12 CUNG            │ - Họ tên, giới tính│
│      (zoom / pan)            │ - Can Chi năm/tháng│
│                              │ - Mệnh / Thân      │
│                              │ - Cục, Âm dương    │
│                              ├────────────────────┤
│                              │ Panel chi tiết     │
│                              │ (cung / sao đã     │
│                              │  chọn + giải thích)│
└──────────────────────────────┴────────────────────┘
 [Đại hạn: 4–13 | 14–23 | 24–33 ...]   [Tiểu hạn: năm]
```

### Mobile

- Lưới 12 cung ở trên, hỗ trợ **pinch-to-zoom**.
- Click cung/sao → **bottom sheet** trượt lên.
- Đại hạn là thanh cuộn ngang bên dưới lá số.

### Mỗi ô cung hiển thị

- Tên cung (Mệnh, Phụ Mẫu, Phúc Đức, ...)
- Địa chi + Can cung
- Chính tinh (nổi bật, đậm)
- Phụ tinh (nhỏ hơn, phân màu cát/sát)
- Tứ Hóa (badge: Lộc / Quyền / Khoa / Kỵ)
- Đại hạn của cung
- Dấu hiệu **Mệnh / Thân**

### Tương tác

|Hành động|Kết quả|
|---|---|
|Click cung|Highlight cung + tam hợp + xung chiếu; mở panel chi tiết|
|Click sao|Popover giải thích ngắn + link "Đọc trong Sổ tay"|
|Chọn đại hạn|Lá số đổi sang chế độ xem đại hạn đó (cung hạn, tứ hóa đại hạn)|
|Chọn tiểu hạn / lưu niên|Highlight cung lưu niên|
|Zoom + / − / reset|Điều khiển góc lưới|
|Chia sẻ|Mở modal chia sẻ|

### Modal chia sẻ

```
Quyền riêng tư
( ) 🔒 Riêng tư — chỉ mình bạn
( ) 🔗 Chia sẻ bằng link
( ) 🌎 Công khai

[ Sao chép link ]   https://astra.app/la-so/8f3a2...
```

- Mặc định **Riêng tư**.
- Có cảnh báo khi chọn Công khai: "Ngày giờ sinh sẽ hiển thị cho mọi người."
- Tuỳ chọn **ẩn tên / ẩn ngày sinh đầy đủ** khi chia sẻ.

### Banner đăng nhập (khách)

> "Đăng nhập để lưu lá số này" — dismissible, không chặn nội dung.

---

## 7. Tài khoản `/tai-khoan`

```
Tài khoản
├── Lá số của tôi   (grid thẻ: tên, ngày sinh, trạng thái chia sẻ, ⋯ menu)
├── Bài viết đã lưu
├── Câu hỏi của tôi
├── Câu trả lời của tôi
└── Ghi chú
```

- **Lá số của tôi:** thẻ lá số, nút `+ Thêm lá số`, tìm/lọc theo nhãn, menu ⋯ (đổi tên, chia sẻ, xoá).
- **Bài viết đã lưu:** danh sách ⭐ kèm ghi chú nhanh.
- **Ghi chú:** soạn thảo đơn giản, gắn được với bài viết hoặc lá số.
- Đăng nhập `/dang-nhap`: Google + Email magic link; giữ lại lá số vừa lập ở phiên khách để gắn vào tài khoản sau khi đăng nhập.

---

## 8. Sổ tay `/so-tay`

### Điều hướng (sidebar desktop / drawer mobile)

```
SỔ TAY
├── 🌱 Nhập môn
├── ☯ Âm Dương - Ngũ Hành
├── 🏮 Thiên Can - Địa Chi
├── ⭐ Các sao
│    ├── 14 Chính Tinh
│    ├── Phụ tinh
│    ├── Cát tinh
│    ├── Sát tinh
│    └── Tứ Hóa
├── 🏠 12 Cung
├── 🔗 Tam hợp - Xung chiếu
├── ⏳ Đại hạn - Tiểu hạn
├── 🧩 Cách cục
└── 📖 Bài viết
```

### Template trang bài (VD: Thiên Cơ)

```
# Thiên Cơ
⭐ Tổng quan
## Đặc điểm
## Thiên Cơ ở 12 cung      (tab hoặc accordion theo cung)
## Khi gặp các sao khác
## Ví dụ                   [ Xem lá số mẫu ]
## Đọc tiếp                → Thái Âm  → Cự Môn  → Cơ Nguyệt Đồng Lương
```

- Mục lục nổi bên phải (desktop).
- Nút ⭐ **Lưu bài** + 📝 **Ghi chú** cố định.
- Nút **[ Hỏi về bài này ]** (mở form đặt câu hỏi gắn sẵn bài viết, khi Phase Q&A mở).
- Link nội bộ sao/cung hiển thị hover-card xem nhanh.

---

## 9. Tìm kiếm

- Một ô duy nhất trên header: `🔍 Tìm sao, cung, cách cục, bài viết...`
- Phím tắt `/` hoặc `Ctrl/⌘ + K` mở command palette.
- Kết quả nhóm theo loại:

```
SAO        ⭐ Tử Vi
CUNG       🏠 Cung Mệnh · 🏠 Cung Quan Lộc
CÁCH CỤC   🧩 Tử Phủ Vũ Tướng
BÀI VIẾT   📖 Tìm hiểu về Tử Vi · 📖 Tử Vi tại cung Mệnh
```

- Hỗ trợ **gõ không dấu** ("tu vi" ra "Tử Vi").
- Điều hướng bằng bàn phím ↑ ↓ Enter.
- Trạng thái rỗng: gợi ý tìm kiếm phổ biến.

---

## 10. Học Tử Vi `/hoc`

### Lộ trình

```
Bước 1  ☯ Âm Dương - Ngũ Hành     ✓
Bước 2  🏮 Thiên Can - Địa Chi     ✓
Bước 3  🏠 12 Cung                 →
Bước 4  ⭐ 14 Chính Tinh           →
Bước 5  🔗 Tam hợp - Xung chiếu    →
Bước 6  ⏳ Đại hạn                 →
```

- Trạng thái bước: hoàn thành ✓ / đang học → / chưa mở.
- Progress bar tổng ở đầu trang.

### Một bài học

```
📖 Đọc  →  ❓ Quiz  →  🔮 Áp dụng vào lá số
```

- Quiz: câu trắc nghiệm, phản hồi đúng/sai ngay, giải thích ngắn.
- Áp dụng: mở lá số của user (hoặc lá số mẫu) với highlight đúng phần vừa học.
- Nút **Bài tiếp theo** sau khi hoàn thành.

---

## 11. Hỏi đáp `/hoi-dap`

### Danh sách

```
HỎI ĐÁP TỬ VI
[ 🔥 Mới nhất ] [ 🔥 Nhiều tương tác ] [ Chưa có câu trả lời ]
[ + Đặt câu hỏi ]

Danh mục: Tử Vi căn bản · Lá số · Cung Mệnh · Công danh · Tài lộc
          · Tình duyên · Đại vận · Tiểu vận · Khác
```

### Thẻ câu hỏi

```
Cung Phu Thê có Tham Lang thì luận thế nào?
Mình mới học Tử Vi nên chưa hiểu trường hợp...
📎 Lá số đính kèm    #phuthe #thamlang
👍 12   💬 5
```

### Trang câu hỏi `/hoi-dap/cau-hoi/:id`

- Nội dung câu hỏi + lá số đính kèm (xem thu nhỏ, bấm để mở).
- Danh sách câu trả lời, sắp theo vote / mới nhất.
- Mỗi câu trả lời có khối **"Câu trả lời này dựa trên kiến thức nào?"**

```
📚 Tham Lang   📚 Cung Phu Thê   📚 Tam hợp
```

- Editor trả lời cho phép: chèn link bài Sổ tay, trích sao/cung, đính kèm một phần lá số.
- Vote 👍, báo cáo (report), chấp nhận câu trả lời tốt nhất.

### Đặt câu hỏi `/hoi-dap/dat-cau-hoi`

Tiêu đề · Nội dung · Danh mục · Tag · Đính kèm lá số (chọn từ lá số của tôi) · Gửi.

---

## 12. Design system

### Phong cách

Huyền bí nhưng **sạch và dễ đọc**: nền tối sâu hoặc giấy ấm, điểm nhấn vàng đồng / tím chàm. Tránh trang trí quá đà làm khó đọc lá số.

### Token gợi ý

|Token|Light|Dark|
|---|---|---|
|`--bg`|`#FAF6EE`|`#0F1020`|
|`--surface`|`#FFFFFF`|`#181A2E`|
|`--text`|`#1C1A17`|`#ECE8DC`|
|`--primary`|`#5B3FA8`|`#9B84E8`|
|`--accent`|`#B8892B`|`#D9AE4E`|
|`--cat` (cát tinh)|`#2F7D4F`|`#5BBE86`|
|`--sat` (sát tinh)|`#B3402F`|`#E5705E`|

### Hóa / màu sao

- Hóa Lộc: xanh lá · Hóa Quyền: xanh dương · Hóa Khoa: vàng · Hóa Kỵ: đỏ
- Ngũ hành sao có thể dùng viền/chấm màu nhỏ (Kim, Mộc, Thủy, Hỏa, Thổ).
- **Không dựa vào màu duy nhất** — luôn kèm chữ/biểu tượng (accessibility).

### Typography

- Tiêu đề: serif có hỗ trợ tiếng Việt đầy đủ (VD _Playfair Display_, _Noto Serif_).
- Nội dung & UI: sans (VD _Inter_, _Be Vietnam Pro_).
- Lá số: font số/chữ rõ ở cỡ nhỏ; chính tinh ≥ 14px, phụ tinh ≥ 11px.

### Components chính

`Button`, `Input`, `DatePicker`, `Select`, `SegmentedControl`, `Card`, `Badge` (Hóa), `Tabs`, `Accordion`, `Modal`, `BottomSheet`, `Popover/HoverCard`, `CommandPalette`, `Toast`, `ProgressBar`, `Skeleton`, `EmptyState`, `ChartGrid`, `PalaceCell`, `StarChip`.

---

## 13. Trạng thái & edge cases

|Màn hình|Loading|Empty|Error|
|---|---|---|---|
|Lá số|Skeleton lưới 12 ô|—|"Không tìm thấy lá số hoặc bạn không có quyền xem"|
|Lá số của tôi|Skeleton thẻ|"Chưa có lá số. Lập lá số đầu tiên →"|Retry|
|Bài đã lưu|Skeleton|"Bạn chưa lưu bài nào"|Retry|
|Hỏi đáp|Skeleton|"Chưa có câu hỏi. Hãy là người đầu tiên!"|Retry|
|Tìm kiếm|Spinner nhỏ|"Không có kết quả cho '…'" + gợi ý|Retry|

- Link lá số không có quyền → trang 403 thân thiện, có CTA "Lập lá số của bạn".
- Offline/mạng chậm: giữ lá số vừa xem trong cache.

---

## 14. Responsive & Accessibility

- Breakpoint: `sm 640 · md 768 · lg 1024 · xl 1280`.
- Lá số: desktop lưới 4×4 (12 cung + ô trung tâm); mobile cho phép cuộn/zoom, hoặc chế độ xem danh sách 12 cung.
- Điều hướng bàn phím toàn bộ, focus ring rõ.
- Độ tương phản ≥ WCAG AA.
- `aria-label` cho từng cung/sao; hỗ trợ screen reader đọc "Cung Mệnh, chính tinh Tử Vi, Thiên Phủ".
- Tôn trọng `prefers-reduced-motion` và `prefers-color-scheme`.

---

## 15. Microcopy mẫu

- CTA chính: **Lập lá số**
- Gợi ý đăng nhập: _"Đăng nhập để lưu lá số và xem lại bất cứ lúc nào."_
- Cảnh báo công khai: _"Lá số công khai sẽ hiển thị ngày giờ sinh cho mọi người."_
- Disclaimer: _"Tử vi mang tính tham khảo, không thay thế lời khuyên chuyên môn."_
- Empty hỏi đáp: _"Chưa có câu hỏi nào. Bạn muốn hỏi gì về lá số của mình?"_

---

# PHẦN B — GIAO DIỆN ADMIN

## 16. Nguyên tắc thiết kế Admin

1. **Desktop-first** (quản trị chủ yếu trên máy tính), vẫn xem được trên tablet; mobile chỉ cần duyệt báo cáo nhanh.
2. **Tách hẳn khỏi giao diện User**: layout, màu nhấn và đường dẫn `/admin/*` riêng, tránh nhầm lẫn.
3. **Quyền theo vai trò**: menu chỉ hiện mục mà vai trò được phép; truy cập trái quyền → trang 403.
4. **Ưu tiên tốc độ thao tác**: bảng dữ liệu có lọc, tìm, sắp xếp, chọn nhiều dòng, phím tắt.
5. **Riêng tư**: admin **không xem ngày giờ sinh / lá số riêng tư** của user; chỉ xem số liệu tổng hợp. Chỉ xem được lá số công khai hoặc lá số đính kèm trong nội dung bị báo cáo.
6. **Mọi hành động nhạy cảm đều xác nhận và ghi audit log.**

---

## 17. Sitemap Admin

```
/admin
├── /admin                    Dashboard
├── /admin/bai-viet           Quản lý bài Sổ tay
│   ├── /moi                  Tạo bài
│   ├── /:id                  Sửa bài
│   └── /danh-muc             Danh mục
├── /admin/hoc                Quản lý lộ trình & bài học
│   ├── /lo-trinh
│   └── /bai/:id              Sửa bài học + quiz
├── /admin/hoi-dap            Quản lý câu hỏi / câu trả lời
│   └── /bao-cao              Hàng đợi báo cáo
├── /admin/nguoi-dung         Quản lý user
│   └── /:id
├── /admin/la-so              Thống kê lá số (ẩn dữ liệu cá nhân)
├── /admin/thong-bao          Thông báo hệ thống
├── /admin/audit-log          Nhật ký hành động
└── /admin/cai-dat            Cài đặt hệ thống
```

---

## 18. Layout Admin

```
┌────────────┬──────────────────────────────────────────────┐
│ ASTRA      │  Breadcrumb            [🔍 Tìm nhanh] [🔔] [👤] │
│ Admin      ├──────────────────────────────────────────────┤
│            │                                              │
│ 📊 Tổng quan│                                              │
│ 📖 Bài viết │              Khu vực nội dung                │
│ 🎓 Học      │                                              │
│ 💬 Hỏi đáp  │                                              │
│  └ Báo cáo ●│                                              │
│ 👥 Người dùng│                                             │
│ 🔮 Lá số    │                                              │
│ 🔔 Thông báo│                                              │
│ 🧾 Audit log│                                              │
│ ⚙ Cài đặt   │                                              │
│            │                                              │
│ ← Về trang │                                              │
│   người dùng│                                             │
└────────────┴──────────────────────────────────────────────┘
```

- Sidebar thu gọn được (chỉ hiện icon).
- Badge đỏ ● số báo cáo chưa xử lý ở mục Hỏi đáp.
- Header có banner cố định **"Chế độ Admin"** (màu khác giao diện User, VD xanh than / cam) để phân biệt.
- Trên tablet: sidebar thành drawer.

---

## 19. Dashboard `/admin`

```
Tổng quan                              [ 7 ngày ▾ ]

┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
│ Lá số   │ │ User mới│ │ Câu hỏi │ │ Báo cáo │
│ 1,240   │ │ 86      │ │ 34      │ │ 5 chờ   │
│ ▲ 12%   │ │ ▲ 5%    │ │ ▼ 3%    │ │ xử lý   │
└─────────┘ └─────────┘ └─────────┘ └─────────┘

[ Biểu đồ: Lá số lập theo ngày ]   [ Biểu đồ: User mới ]

Bài Sổ tay xem nhiều nhất      Từ khoá tìm kiếm phổ biến
1. Tử Vi ......... 2.1k         1. tu vi
2. Cung Mệnh ..... 1.8k         2. cung menh
3. Tứ Hóa ........ 1.2k         3. tham lang

Cần xử lý                       Hoạt động gần đây
• 5 báo cáo mới                 • Editor A đã xuất bản "Thiên Cơ"
• 3 bài nháp chờ duyệt          • Mod B đã ẩn 1 câu trả lời
• 2 câu hỏi chưa có trả lời >7d • ...
```

- Thẻ số liệu bấm được → đi tới trang quản lý tương ứng.
- Bộ lọc thời gian: 24h / 7 ngày / 30 ngày.
- Mục **Cần xử lý** là trung tâm công việc hằng ngày.
- Từ khoá tìm kiếm không có kết quả được đánh dấu để biết cần viết thêm bài nào.

---

## 20. Quản lý bài viết Sổ tay `/admin/bai-viet`

### Danh sách

```
Bài viết   [ + Tạo bài ]   [🔍 Tìm...]  Loại ▾  Danh mục ▾  Trạng thái ▾

☐ Tiêu đề        Loại     Danh mục      Trạng thái   Cập nhật   ⋯
☐ Tham Lang      Sao      Chính tinh    ● Đã xuất bản  2 giờ    ⋯
☐ Cung Phu Thê   Cung     12 Cung       ○ Nháp         1 ngày   ⋯
☐ Tứ Hóa căn bản Khái niệm Tứ Hóa       ● Đã xuất bản  3 ngày   ⋯

Đã chọn 2:  [Xuất bản] [Chuyển nháp] [Xoá]        Trang 1 2 3 ›
```

- Loại: `Sao / Cung / Cách cục / Khái niệm / Bài viết`.
- Hành động hàng loạt: xuất bản, chuyển nháp, đổi danh mục, xoá.
- Menu ⋯: Sửa, Xem trước, Nhân bản, Xem trên site, Xoá.

### Trình soạn bài `/admin/bai-viet/:id`

```
┌──────────────────────────────────┬──────────────────────┐
│ Tiêu đề: [..................]    │ Xuất bản             │
│ Slug:    [tham-lang]             │ Trạng thái: ○ Nháp   │
│                                  │ [Lưu nháp] [Xuất bản]│
│ [B][I][H2][Link][Ảnh][Bảng]      │ Lịch xuất bản: [..]  │
│ ┌──────────────────────────────┐ ├──────────────────────┤
│ │ Markdown / WYSIWYG           │ │ Phân loại            │
│ │ ...                          │ │ Loại: [Sao ▾]        │
│ │                              │ │ Danh mục: [..▾]      │
│ └──────────────────────────────┘ │ entity_key:[tham-lang]│
│ [ Soạn ] [ Xem trước ] [ Song song]│ Tóm tắt: [......]   │
│                                  ├──────────────────────┤
│                                  │ Bài liên quan (Đọc tiếp)│
│                                  │ + Thái Âm  + Cự Môn   │
│                                  ├──────────────────────┤
│                                  │ SEO: meta title/desc │
│                                  │ Lịch sử phiên bản ›  │
└──────────────────────────────────┴──────────────────────┘
```

- **Tự động lưu nháp** mỗi vài giây, hiển thị "Đã lưu lúc 14:32".
- Chế độ soạn: Markdown, Xem trước, Song song.
- Chèn nhanh: link sao/cung (autocomplete theo `entity_key`), lá số mẫu, ảnh.
- Template theo loại (VD loại Sao có sẵn khung: Tổng quan / Đặc điểm / Ở 12 cung / Khi gặp sao khác / Ví dụ / Đọc tiếp).
- Lịch sử phiên bản: so sánh và khôi phục.
- Cảnh báo khi rời trang chưa lưu; kiểm tra thiếu slug / `entity_key` trước khi xuất bản.

### Danh mục `/admin/bai-viet/danh-muc`

Cây danh mục kéo-thả để đổi thứ tự và cấp cha-con; thêm / sửa / xoá (chặn xoá nếu còn bài).

---

## 21. Quản lý học `/admin/hoc`

- **Lộ trình**: danh sách bước, kéo-thả đổi thứ tự (Bước 1 → 6), bật/tắt hiển thị.
- **Bài học** `/admin/hoc/bai/:id`:

```
Tên bài: [Cung Mệnh]       Bài viết liên kết: [Cung Mệnh ▾]
Thứ tự: [3]                Lộ trình: [Căn bản ▾]

Quiz
┌ Câu 1 ───────────────────────────────┐
│ Câu hỏi: [................]           │
│ ○ Đáp án A  ● Đáp án B (đúng)  ○ C  ○ D│
│ Giải thích: [................]        │
│ [↑][↓][Nhân bản][Xoá]                 │
└───────────────────────────────────────┘
[ + Thêm câu hỏi ]        [ Xem trước quiz ]
```

- Xem thống kê: tỉ lệ hoàn thành, câu hay sai nhất.

---

## 22. Quản lý hỏi đáp `/admin/hoi-dap`

### Danh sách câu hỏi / câu trả lời

Bảng có lọc: danh mục, tag, trạng thái (hiển thị / bị ẩn), có báo cáo, chưa có trả lời. Hành động: Xem, Ẩn, Khôi phục, Xoá, Ghim, Đổi danh mục, Gộp trùng.

### Hàng đợi báo cáo `/admin/hoi-dap/bao-cao`

```
Báo cáo   [Chờ xử lý (5)] [Đã xử lý] [Tất cả]

┌──────────────────────────────────────────────────────┐
│ ⚑ Câu trả lời  · "Tham Lang ở Phu Thê chắc chắn..."  │
│ Lý do: Spam / Nội dung xúc phạm   ·  3 báo cáo       │
│ Người đăng: user_123   ·  2 giờ trước                │
│ [Xem ngữ cảnh]                                       │
│ [ Bỏ qua ] [ Ẩn nội dung ] [ Xoá ] [ Cảnh cáo user ] │
└──────────────────────────────────────────────────────┘
```

- Xem đầy đủ ngữ cảnh (câu hỏi + chuỗi trả lời) trong panel bên phải, không rời trang.
- Phím tắt: `J/K` chuyển báo cáo, `H` ẩn, `I` bỏ qua.
- Mọi quyết định ghi vào audit log; có ô ghi chú nội bộ.
- Nội dung đính kèm lá số: chỉ hiển thị dữ liệu mà người đăng đã chia sẻ.

---

## 23. Quản lý người dùng `/admin/nguoi-dung`

```
Người dùng  [🔍 Tìm email / tên]  Vai trò ▾  Trạng thái ▾

Tên          Email           Vai trò    Trạng thái  Tham gia   ⋯
Nguyễn A     a@mail.com      User       ● Hoạt động 01/09/26   ⋯
Trần B       b@mail.com      Editor     ● Hoạt động 15/08/26   ⋯
Lê C         c@mail.com      User       ⛔ Bị khoá  20/09/26   ⋯
```

### Chi tiết user `/admin/nguoi-dung/:id`

- Thông tin tài khoản (tên, email, ngày tham gia, lần đăng nhập cuối).
- **Số liệu hoạt động**: số lá số, số câu hỏi/trả lời, số báo cáo bị nhận (chỉ số đếm, **không hiện nội dung lá số riêng tư**).
- Lịch sử xử lý vi phạm.
- Hành động: Đổi vai trò, Khoá / Mở khoá (có thời hạn), Cảnh cáo, Gửi email đặt lại mật khẩu, Xoá tài khoản (xác nhận nhập lại email).
- Chỉ **Admin** được đổi vai trò; Moderator chỉ khoá/cảnh cáo.

---

## 24. Thống kê lá số `/admin/la-so`

- Biểu đồ: số lá số theo ngày, tỉ lệ khách / user, phân bố chế độ chia sẻ (Riêng tư / Link / Công khai).
- Danh sách **lá số công khai** (để kiểm duyệt nội dung / tên): xem, ẩn khỏi công khai.
- Phân bố theo Mệnh chính tinh, Cục, v.v. (dữ liệu ẩn danh tổng hợp) phục vụ hiểu người dùng.
- Theo dõi lỗi engine: lá số tính lỗi / cảnh báo, `engine_version` đang dùng.
- ⚠️ Không có chức năng đọc lá số riêng tư của user.

---

## 25. Thông báo hệ thống `/admin/thong-bao`

- Tạo thông báo gửi: toàn bộ user / nhóm vai trò.
- Banner thông báo trên site (bảo trì, tính năng mới) với lịch bắt đầu / kết thúc và xem trước.
- Lịch sử đã gửi.

---

## 26. Audit log & Cài đặt

### Audit log `/admin/audit-log`

```
Thời gian        Người thực hiện   Hành động            Đối tượng        Chi tiết
09/10 14:32      mod_b             Ẩn câu trả lời       answer #4812     Lý do: spam
09/10 13:10      editor_a          Xuất bản bài         "Thiên Cơ"       v3
09/10 11:02      admin_x           Đổi vai trò          user_123 → Editor
```

- Lọc theo người, loại hành động, thời gian; xuất CSV.
- Chỉ đọc, không sửa / xoá được.

### Cài đặt `/admin/cai-dat` (chỉ Admin)

- Chung: tên site, logo, thông tin liên hệ, disclaimer.
- Đăng nhập: bật/tắt Google, magic link.
- Hỏi đáp: danh mục, danh sách tag cấm, giới hạn tốc độ (rate limit), từ khoá cấm.
- Tính năng: bật/tắt module (Hỏi đáp, Học) — dùng khi launch MVP trước Phase 5–6.
- SEO mặc định, Open Graph, sitemap.

---

## 27. Components & trạng thái riêng của Admin

### Components thêm

`AdminLayout`, `Sidebar`, `DataTable` (sort, filter, phân trang, chọn nhiều dòng), `FilterBar`, `StatCard`, `RoleBadge`, `StatusBadge`, `MarkdownEditor`, `DiffViewer` (so sánh phiên bản), `DragSortList`, `ConfirmDialog` (nhập lại để xác nhận), `AuditEntry`, `ReportCard`, `Charts`.

### Quy ước chung

- Hành động phá huỷ (xoá, khoá, đổi vai trò) luôn qua `ConfirmDialog`; hành động rủi ro cao yêu cầu gõ lại tên/email.
- Toast kết quả + nút **Hoàn tác** cho thao tác ẩn / chuyển nháp.
- Màu trạng thái: Đã xuất bản (xanh) · Nháp (xám) · Chờ duyệt (vàng) · Bị ẩn / khoá (đỏ).

### Trạng thái

|Màn hình|Loading|Empty|Error|
|---|---|---|---|
|Dashboard|Skeleton thẻ + biểu đồ|"Chưa có dữ liệu trong khoảng này"|Retry từng khối|
|Danh sách (bảng)|Skeleton dòng|"Không có kết quả. Xoá bộ lọc"|Retry|
|Trình soạn bài|Skeleton|—|Lỗi lưu: giữ nội dung, hiện "Lưu thất bại, thử lại"|
|Hàng đợi báo cáo|Skeleton|"🎉 Không còn báo cáo nào cần xử lý"|Retry|
|Khu vực không đủ quyền|—|—|Trang 403 + nút về Dashboard|

### Accessibility & responsive

- Điều hướng bàn phím toàn bộ bảng và form; focus ring rõ.
- Tương phản ≥ WCAG AA; không chỉ dựa vào màu cho trạng thái (kèm chữ / icon).
- Desktop ≥ 1024px là chuẩn; tablet dùng drawer; mobile chỉ hỗ trợ duyệt báo cáo & xem dashboard.

### Ánh xạ Admin theo Phase phát triển

|Phase|Giao diện Admin cần có|
|---|---|
|1–3|Khung `/admin`, kiểm tra vai trò, Dashboard cơ bản, quản lý user|
|4|Quản lý bài viết + danh mục + trình soạn|
|5|Quản lý hỏi đáp + hàng đợi báo cáo + audit log|
|6|Quản lý lộ trình, bài học, quiz|

> Lưu ý kỹ thuật: cần thêm cột `role` (`user | editor | moderator | admin`) vào `profiles`, bảng `audit_logs`, `user_bans`, và RLS theo vai trò.