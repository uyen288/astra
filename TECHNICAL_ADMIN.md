# ASTRA — Technical Spec (ADMIN)

> 📌 Tài liệu kỹ thuật cho **phía Admin**: biên tập nội dung, kiểm duyệt cộng đồng, quản lý người dùng, thống kê, cài đặt. Phía User nằm ở [`technical-user.md`](https://claude.ai/chat/technical-user.md). Hai file **dùng chung** stack, database, Supabase project và Vercel deployment; file này chỉ mô tả phần **bổ sung** cho Admin. Giao diện tương ứng: Phần B của `ui.md`.

---

## 1. Mục tiêu & phạm vi

- Cung cấp trang quản trị `/admin/*` để vận hành sản phẩm mà **không phải sửa DB bằng tay**.
- Chức năng đầy đủ:
    1. Dashboard & thống kê
    2. Quản lý bài viết / danh mục Sổ tay
    3. Quản lý lộ trình, bài học, quiz
    4. Kiểm duyệt hỏi đáp + hàng đợi báo cáo
    5. Quản lý người dùng & phân quyền
    6. Thống kê lá số (ẩn dữ liệu cá nhân)
    7. Thông báo hệ thống & banner
    8. Audit log
    9. Cài đặt hệ thống & bật/tắt tính năng
- Nguyên tắc: **least privilege**, **riêng tư là mặc định** (Admin không đọc lá số riêng tư), **mọi hành động nhạy cảm đều có audit log**.

---

## 2. Vai trò & ma trận quyền

|Chức năng|Khách/User|Moderator|Editor|Admin|
|---|:-:|:-:|:-:|:-:|
|Vào `/admin`|✗|✓|✓|✓|
|Dashboard (tổng quan)|✗|✓ (giới hạn)|✓ (giới hạn)|✓|
|Tạo/sửa/xuất bản bài Sổ tay|✗|✗|✓|✓|
|Quản lý danh mục|✗|✗|✓|✓|
|Quản lý lộ trình/bài học/quiz|✗|✗|✓|✓|
|Xem hàng đợi báo cáo|✗|✓|✗|✓|
|Ẩn/xoá/khôi phục câu hỏi, trả lời|✗|✓|✗|✓|
|Cảnh cáo / khoá tạm user|✗|✓|✗|✓|
|Khoá vĩnh viễn / xoá tài khoản|✗|✗|✗|✓|
|Đổi vai trò user|✗|✗|✗|✓|
|Xem thống kê lá số (tổng hợp)|✗|✗|✗|✓|
|Thông báo hệ thống / banner|✗|✗|✗|✓|
|Xem audit log|✗|✓ (của mình)|✓ (của mình)|✓ (toàn bộ)|
|Cài đặt & feature flags|✗|✗|✗|✓|
|**Đọc lá số riêng tư của user**|✗|✗|✗|**✗ (không ai)**|

Vai trò lưu ở `profiles.role` (`user | editor | moderator | admin`), được kiểm tra **ở 3 lớp**: UI (ẩn menu) → RLS/RPC (chặn thật) → Edge Function (hành động đặc quyền).

---

## 3. Kiến trúc

```
React SPA (Vercel)
 ├── Khu User   : /, /lap-la-so, /so-tay, ...
 └── Khu Admin  : /admin/*   (lazy-loaded, bundle riêng)
        │
        │ supabase-js (JWT user, RLS + role)
        ▼
 Supabase
 ├── Postgres: bảng + RLS + RPC security definer
 ├── Auth: JWT chứa role (custom claim) 
 ├── Storage: ảnh bài viết (bucket `articles`)
 ├── Edge Functions (service role, chạy server): các hành động đặc quyền
 └── pg_cron: xuất bản theo lịch, dọn dữ liệu
```

Quyết định chính:

- **Admin là một phần của cùng app React**, tách bằng code-splitting (`React.lazy`) để user thường không tải mã admin.
- **Service role key chỉ ở Edge Function**, không bao giờ ở client.
- Hành động đặc quyền (đổi vai trò, khoá/xoá user, gửi thông báo hàng loạt) đi qua **Edge Function**, ghi audit log trong cùng giao dịch.
- Thống kê dùng **RPC / view tổng hợp**, không cho Admin `select` thẳng bảng `charts` của người khác.

---

## 4. Stack bổ sung cho Admin

|Nhu cầu|Công nghệ|
|---|---|
|Bảng dữ liệu (sort/filter/phân trang)|TanStack Table + phân trang server-side|
|Biểu đồ|Recharts|
|Trình soạn bài|CodeMirror (Markdown) + react-markdown để preview; hoặc Tiptap nếu muốn WYSIWYG|
|Kéo-thả (danh mục, bước học)|dnd-kit|
|Diff phiên bản bài|`diff` / react-diff-viewer|
|Upload ảnh|Supabase Storage + nén phía client|
|Lập lịch|pg_cron (Supabase)|
|Gửi email thông báo (tuỳ chọn)|Resend / Supabase SMTP qua Edge Function|
|Phím tắt|react-hotkeys-hook|

---

## 5. Cấu trúc thư mục (bổ sung)

```
src/
├── features/
│   └── admin/
│       ├── layout/           # AdminLayout, Sidebar, RoleGuard
│       ├── dashboard/
│       ├── articles/         # list, editor, revisions, categories
│       ├── learning/         # paths, lessons, quiz editor
│       ├── moderation/       # questions, answers, reports queue
│       ├── users/
│       ├── chart-stats/
│       ├── announcements/
│       ├── audit/
│       ├── settings/
│       └── components/       # DataTable, FilterBar, ConfirmDialog, StatCard...
supabase/
├── migrations/               # thêm các migration admin
└── functions/
    ├── admin-set-role/
    ├── admin-moderate-user/
    ├── admin-delete-user/
    ├── admin-send-announcement/
    ├── admin-reindex-search/
    └── admin-export-audit/
```

---

## 6. Routes & guard

```
/admin                         Dashboard                    moderator, editor, admin
/admin/bai-viet                Danh sách bài                editor, admin
/admin/bai-viet/moi            Tạo bài                      editor, admin
/admin/bai-viet/:id            Sửa bài                      editor, admin
/admin/bai-viet/danh-muc       Danh mục                     editor, admin
/admin/hoc/lo-trinh            Lộ trình                     editor, admin
/admin/hoc/bai/:id             Sửa bài học + quiz           editor, admin
/admin/hoi-dap                 Quản lý Q&A                  moderator, admin
/admin/hoi-dap/bao-cao         Hàng đợi báo cáo             moderator, admin
/admin/nguoi-dung              Danh sách user               moderator (xem), admin
/admin/nguoi-dung/:id          Chi tiết user                moderator (hạn chế), admin
/admin/la-so                   Thống kê lá số               admin
/admin/thong-bao               Thông báo hệ thống           admin
/admin/audit-log               Nhật ký                      theo vai trò
/admin/cai-dat                 Cài đặt                      admin
```

```tsx
// RoleGuard: chặn ở UI (lớp 1), quyền thật vẫn do RLS/RPC quyết định
<Route element={<RequireAuth><RoleGuard allow={['editor','admin']} /></RequireAuth>}>
  <Route path="/admin/bai-viet" element={<ArticlesPage />} />
</Route>
```

Không đủ quyền → trang 403; chưa đăng nhập → chuyển `/dang-nhap?redirect=/admin/...`.

---

## 7. Database — phần bổ sung

> `profiles.role`, `profiles.status`, `profiles.suspended_until` đã được định nghĩa trong `technical-user.md`.

```sql
-- ============ Helper: kiểm tra vai trò ============
create or replace function public.current_role() returns text
language sql stable security definer set search_path = public as $$
  select role from profiles where id = auth.uid()
$$;

create or replace function public.has_role(roles text[]) returns boolean
language sql stable security definer set search_path = public as $$
  select coalesce(public.current_role() = any(roles), false)
$$;

-- ============ Audit log (chỉ ghi thêm, không sửa/xoá) ============
create table audit_logs (
  id bigint generated always as identity primary key,
  actor_id uuid references profiles(id) on delete set null,
  actor_role text,
  action text not null,                 -- 'article.publish', 'user.ban', 'answer.hide'...
  target_type text,                     -- 'article' | 'user' | 'answer' | ...
  target_id text,
  detail jsonb,                         -- {reason, before, after}
  ip inet,
  created_at timestamptz default now()
);
create index on audit_logs (actor_id, created_at desc);
create index on audit_logs (action, created_at desc);

-- ============ Bài viết: mở rộng cho quy trình biên tập ============
alter table articles
  add column author_id uuid references profiles(id),
  add column status text not null default 'draft'
    check (status in ('draft','review','published','archived')),
  add column scheduled_at timestamptz,          -- xuất bản theo lịch
  add column published_at timestamptz,
  add column seo_title text,
  add column seo_description text,
  add column cover_url text,
  add column deleted_at timestamptz;            -- soft delete
-- (cột `published` boolean cũ có thể thay bằng status = 'published')

create table article_revisions (
  id uuid primary key default gen_random_uuid(),
  article_id uuid not null references articles(id) on delete cascade,
  version int not null,
  title text, content_md text, summary text,
  edited_by uuid references profiles(id),
  created_at timestamptz default now(),
  unique (article_id, version)
);

create table article_relations (               -- "Đọc tiếp" / bài liên quan
  article_id uuid references articles(id) on delete cascade,
  related_id uuid references articles(id) on delete cascade,
  sort_order int default 0,
  primary key (article_id, related_id)
);

-- ============ Kiểm duyệt Q&A ============
alter table questions
  add column is_hidden boolean default false,
  add column hidden_by uuid references profiles(id),
  add column hidden_reason text,
  add column is_pinned boolean default false;
alter table answers
  add column is_hidden boolean default false,
  add column hidden_by uuid references profiles(id),
  add column hidden_reason text;

alter table reports
  add column status text not null default 'open'
    check (status in ('open','dismissed','actioned')),
  add column resolved_by uuid references profiles(id),
  add column resolution text,
  add column internal_note text,
  add column resolved_at timestamptz;
create index on reports (status, created_at);

-- ============ Xử lý người dùng ============
create table user_sanctions (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references profiles(id) on delete cascade,
  type text not null check (type in ('warn','suspend','ban')),
  reason text not null,
  expires_at timestamptz,                      -- null = vĩnh viễn (với ban)
  created_by uuid references profiles(id),
  created_at timestamptz default now(),
  revoked_at timestamptz
);

-- ============ Thông báo hệ thống ============
create table announcements (
  id uuid primary key default gen_random_uuid(),
  title text not null, body text,
  audience text default 'all'                  -- 'all' | 'editor' | 'user'...
    check (audience in ('all','user','editor','moderator','admin')),
  kind text default 'banner' check (kind in ('banner','notification','email')),
  starts_at timestamptz, ends_at timestamptz,
  created_by uuid references profiles(id),
  created_at timestamptz default now()
);

-- ============ Cài đặt & feature flags ============
create table site_settings (
  key text primary key,                        -- 'site.name', 'qa.rate_limit', 'banned_words'...
  value jsonb not null,
  updated_by uuid references profiles(id),
  updated_at timestamptz default now()
);
create table feature_flags (
  key text primary key,                        -- 'qa_enabled', 'learning_enabled'
  enabled boolean not null default false,
  updated_by uuid references profiles(id),
  updated_at timestamptz default now()
);

-- ============ Log tìm kiếm (ẩn danh) ============
create table search_logs (
  id bigint generated always as identity primary key,
  query text not null,
  result_count int,
  created_at timestamptz default now()
);
create index on search_logs (created_at desc);
```

Việc ghi `audit_logs` do **Edge Function / trigger** thực hiện, client không được `insert/update/delete` trực tiếp.

---

## 8. RLS & quyền (theo vai trò)

```sql
alter table audit_logs enable row level security;
alter table article_revisions enable row level security;
alter table user_sanctions enable row level security;
alter table announcements enable row level security;
alter table site_settings enable row level security;
alter table feature_flags enable row level security;
alter table search_logs enable row level security;
```

|Bảng|SELECT|INSERT/UPDATE/DELETE|
|---|---|---|
|`audit_logs`|Admin: tất cả; editor/mod: dòng của mình|**Không ai** (chỉ Edge Function/service role)|
|`articles`|Public: `status='published' and deleted_at is null`; editor/admin: tất cả|editor, admin|
|`article_revisions`|editor, admin|trigger tự ghi khi sửa bài|
|`article_categories`, `article_relations`|public / editor, admin|editor, admin|
|`learning_paths`, `lessons`|public (đang bật) / editor, admin|editor, admin|
|`questions`, `answers`|Public: `is_hidden=false`; moderator/admin: tất cả|Chính chủ (tạo/sửa), moderator/admin (ẩn, xoá)|
|`reports`|moderator, admin|User đăng nhập (tạo); moderator/admin (xử lý)|
|`user_sanctions`|moderator, admin; user xem của chính mình|moderator (warn/suspend), admin (ban) qua Edge Function|
|`announcements`|Public (banner đang hiệu lực)|admin|
|`site_settings`, `feature_flags`|feature flags: public đọc; settings: admin|admin|
|`search_logs`|admin|client (insert ẩn danh qua RPC)|
|`profiles.role/status`|—|**Chỉ Edge Function** (`admin-set-role`, `admin-moderate-user`)|

Ví dụ policy:

```sql
create policy "published articles are public" on articles
  for select using (status = 'published' and deleted_at is null);

create policy "editors manage articles" on articles
  for all using (public.has_role(array['editor','admin']))
  with check (public.has_role(array['editor','admin']));

create policy "mods see all questions" on questions
  for select using (is_hidden = false or public.has_role(array['moderator','admin']));

-- Ngăn user tự nâng quyền: không cho UPDATE cột role/status từ client
revoke update (role, status, suspended_until) on profiles from authenticated;
```

### Bảo vệ lá số riêng tư (quan trọng)

```sql
-- KHÔNG có policy nào cho phép admin select charts của người khác.
-- Thống kê chỉ qua RPC trả về số liệu tổng hợp, ẩn danh:
create function admin_chart_stats(days int default 30)
returns table (day date, total int, guests int, users int, private int, link int, public int)
language sql security definer set search_path = public as $$
  select ... -- chỉ COUNT/GROUP BY, không trả input/data/user_id
  where public.has_role(array['admin'])
$$;
```

Mọi RPC admin đều **kiểm tra `has_role` bên trong hàm** (security definer không tự kiểm quyền).

---

## 9. Edge Functions (hành động đặc quyền)

|Function|Quyền|Việc làm|Audit|
|---|---|---|---|
|`admin-set-role`|admin|Đổi `profiles.role`; không cho tự hạ quyền admin cuối cùng|`user.role_change`|
|`admin-moderate-user`|moderator (warn/suspend), admin (ban)|Ghi `user_sanctions`, cập nhật `profiles.status`, gửi email báo|`user.warn/suspend/ban`|
|`admin-delete-user`|admin|Xoá tài khoản + dữ liệu liên quan (cascade), yêu cầu xác nhận|`user.delete`|
|`admin-send-announcement`|admin|Tạo notification hàng loạt / email theo `audience`|`announcement.send`|
|`admin-reindex-search`|admin/editor|Làm mới `tsvector`/chỉ mục sau import hàng loạt|`search.reindex`|
|`admin-export-audit`|admin|Xuất CSV theo bộ lọc|`audit.export`|

Quy ước chung:

- Xác thực JWT → đọc `role` từ DB (không tin client) → kiểm tra quyền → thực thi → ghi `audit_logs` (cùng giao dịch).
- Trả lỗi có mã rõ (`FORBIDDEN`, `CONFLICT`, `VALIDATION`).
- Rate limit theo `actor_id`.

---

## 10. Đặc tả từng module

### 10.1 Dashboard

- RPC `admin_dashboard_stats(range)`: số lá số, user mới, câu hỏi mới, báo cáo chờ, so sánh kỳ trước.
- Khối "Cần xử lý": báo cáo `open`, bài `review`, câu hỏi chưa trả lời > 7 ngày.
- Top bài xem nhiều, top từ khoá tìm kiếm, từ khoá **không có kết quả** (từ `search_logs`).
- Moderator/Editor chỉ thấy các khối liên quan vai trò của mình.

### 10.2 Quản lý bài viết Sổ tay

- **Danh sách:** lọc loại/danh mục/trạng thái/tác giả, tìm, phân trang server-side, thao tác hàng loạt (xuất bản, chuyển nháp, đổi danh mục, xoá mềm).
- **Trình soạn:** Markdown + preview; **autosave** (debounce ~3s, ghi bản nháp, không tạo revision mỗi lần); template theo loại (Sao, Cung, Cách cục...); chèn link sao/cung bằng autocomplete theo `entity_key`; chèn ảnh (upload Storage).
- **Quy trình trạng thái:** `draft → review → published → archived`; Editor gửi duyệt, Admin duyệt (cấu hình được: cho phép editor tự xuất bản hay không).
- **Revision:** trigger `before update` ghi `article_revisions` khi nội dung đổi lúc xuất bản/lưu thủ công; giao diện diff + khôi phục.
- **Xuất bản theo lịch:** `scheduled_at` + pg_cron chạy mỗi phút chuyển `status='published'`.
- **Validate trước xuất bản:** có `slug` duy nhất, `title`, `entity_key` (với loại sao/cung), tóm tắt; cảnh báo link nội bộ hỏng.
- **Danh mục:** cây kéo-thả (dnd-kit) cập nhật `parent_id`, `sort_order`; chặn xoá nếu còn bài.
- **Bài liên quan:** quản lý `article_relations` (mục "Đọc tiếp").
- **Import:** nhập hàng loạt từ Markdown/CSV (seed nội dung ban đầu) → gọi `admin-reindex-search`.

### 10.3 Quản lý học

- CRUD `learning_paths`, `lessons`; kéo-thả đổi thứ tự; bật/tắt hiển thị.
- Trình soạn quiz (lưu `lessons.quiz` jsonb) với schema Zod: `{question, options[], correctIndex, explanation}`; xem trước quiz.
- Thống kê: tỉ lệ hoàn thành, câu sai nhiều nhất (tổng hợp từ `lesson_progress`).

### 10.4 Kiểm duyệt hỏi đáp

- **Hàng đợi báo cáo:** nhóm theo đối tượng (nhiều báo cáo cùng nội dung gộp 1 thẻ), sắp theo số báo cáo/thời gian; xem ngữ cảnh đầy đủ trong panel; hành động: Bỏ qua / Ẩn / Xoá / Cảnh cáo user; ghi chú nội bộ; phím tắt `J/K/H/I`.
- **Quản lý Q&A:** ẩn/khôi phục/xoá, ghim, đổi danh mục, gộp câu hỏi trùng, sửa tag.
- **Tự động hoá nhẹ:** nội dung bị ≥ N báo cáo tự ẩn tạm chờ duyệt (cấu hình trong `site_settings`); lọc từ khoá cấm khi đăng.
- Lá số đính kèm: moderator chỉ xem phần người đăng đã chia sẻ (tuân quyền `visibility`).

### 10.5 Quản lý người dùng

- Danh sách: tìm email/tên, lọc vai trò/trạng thái, phân trang server-side (đọc qua RPC `admin_list_users`, không lộ dữ liệu sinh).
- Chi tiết: thông tin tài khoản, **số liệu đếm** (lá số, câu hỏi, câu trả lời, báo cáo nhận) — **không hiển thị nội dung lá số riêng tư**; lịch sử `user_sanctions`.
- Hành động: đổi vai trò (admin), cảnh cáo/khoá tạm (moderator+), khoá vĩnh viễn/xoá (admin), gửi email đặt lại mật khẩu.
- Khoá: middleware/RLS kiểm tra `profiles.status` và `suspended_until` để chặn tạo nội dung; tự mở khoá khi hết hạn (pg_cron).

### 10.6 Thống kê lá số

- Biểu đồ theo ngày (tổng / khách / user), phân bố `visibility`, phân bố Mệnh–Cục (tổng hợp ẩn danh).
- Danh sách **lá số công khai** để kiểm duyệt (chỉ `visibility='public'`); nút gỡ khỏi công khai (đưa về `private`, ghi audit, báo cho chủ sở hữu).
- Giám sát engine: số lần tính lỗi (gửi từ client qua Sentry/endpoint log), phiên bản `engine_version` đang dùng.

### 10.7 Thông báo hệ thống

- Banner (có thời gian hiệu lực, xem trước), thông báo trong app (ghi `notifications` hàng loạt), email tuỳ chọn.
- Client User đọc banner đang hiệu lực qua view `active_announcements`.

### 10.8 Audit log

- Bảng chỉ đọc: lọc theo người/hành động/thời gian; xem `detail` (before/after); xuất CSV.
- Danh sách `action` chuẩn hoá: `article.create/publish/unpublish/delete`, `category.*`, `lesson.*`, `question.hide/delete`, `answer.hide/delete`, `report.dismiss/action`, `user.role_change/warn/suspend/ban/delete`, `announcement.send`, `settings.update`, `flag.toggle`, `audit.export`.
- Giữ tối thiểu 12 tháng; không có UI sửa/xoá.

### 10.9 Cài đặt & feature flags

- `site_settings`: tên site, disclaimer, liên hệ, SEO mặc định, rate limit Q&A, từ khoá cấm, ngưỡng tự ẩn.
- `feature_flags`: `qa_enabled`, `learning_enabled`, `signup_enabled`, `maintenance_mode`.
- Client User đọc flags khi khởi động (cache TanStack Query) để ẩn/hiện Hỏi đáp, Học — cho phép **launch MVP trước Phase 5–6**.
- Mọi thay đổi ghi `audit_logs`.

---

## 11. Bảo mật Admin

- **Bắt buộc 2FA (TOTP)** cho vai trò `admin` (Supabase MFA); khuyến nghị cho editor/moderator.
- Session ngắn hơn cho khu admin; yêu cầu **xác thực lại** (re-auth) trước hành động rủi ro cao (đổi vai trò, xoá user, xuất audit).
- Service role key chỉ lưu trong secrets của Edge Function; không đưa vào biến `VITE_*`.
- Kiểm quyền **ở server** cho mọi RPC/Edge Function; không tin `role` từ client.
- Không thể hạ/xoá admin cuối cùng; không thể tự đổi vai trò của chính mình.
- Sanitize Markdown bài viết (rehype-sanitize), whitelist domain ảnh/iframe.
- CSP chặt cho `/admin`, header chống clickjacking.
- Ghi `ip` vào audit log (qua Edge Function); cảnh báo đăng nhập admin từ thiết bị/địa điểm mới (tuỳ chọn).
- **Soft delete** cho bài viết, câu hỏi/trả lời để khôi phục khi nhầm; xoá cứng chỉ qua Admin.
- Tuân thủ quyền riêng tư: Admin chỉ thấy dữ liệu tổng hợp; mọi truy cập dữ liệu người dùng cụ thể (nếu có yêu cầu pháp lý) phải qua quy trình riêng và ghi log.

---

## 12. Hiệu năng

- Phân trang **server-side** (cursor/limit) cho mọi bảng; không tải toàn bộ.
- Index theo cột lọc/sắp xếp phổ biến (`status`, `created_at`, `action`).
- Dashboard dùng RPC tổng hợp + cache 1–5 phút (materialized view cho số liệu nặng, refresh bằng pg_cron).
- Bundle admin tách riêng, không ảnh hưởng tốc độ tải của User.
- Trình soạn bài lazy-load CodeMirror.

---

## 13. Kiểm thử

- **Test RLS theo từng vai trò** (bắt buộc, chạy trong CI với Supabase local):
    - user thường không đọc được `audit_logs`, `user_sanctions` của người khác, không sửa được `role`.
    - editor không ẩn được câu trả lời; moderator không xuất bản được bài.
    - **không vai trò nào** đọc được `charts` private của người khác.
- Unit test: validate quiz schema, hàm chuyển trạng thái bài, logic ban/suspend hết hạn.
- E2E (Playwright): editor tạo → gửi duyệt → admin xuất bản → bài hiện ở Sổ tay; user báo cáo → moderator ẩn → nội dung biến mất; admin đổi vai trò + audit log ghi nhận.
- Test Edge Function: từ chối khi thiếu quyền, rollback khi lỗi giữa chừng.

---

## 14. Lộ trình phát triển theo Phase

### Phase A1 — Khung Admin, vai trò, audit _(song song Phase 1–3 của User)_

**Mục tiêu:** Nền tảng phân quyền và nhật ký.

- [ ] Thêm `role`, `status` vào `profiles`; hàm `has_role`, `current_role`
- [ ] Chặn user tự sửa `role/status` (revoke + test)
- [ ] Bảng `audit_logs` + helper ghi log trong Edge Function
- [ ] `AdminLayout`, `Sidebar` theo vai trò, `RoleGuard`, trang 403
- [ ] Bundle admin tách (lazy)
- [ ] Gán admin đầu tiên (seed/SQL một lần), bật MFA cho admin
- [ ] Trang Audit log (chỉ đọc)

**DoD:** Chỉ đúng vai trò vào được từng route admin; test RLS vai trò pass; mọi hành động đặc quyền đã có audit log.

---

### Phase A2 — Dashboard & Quản lý người dùng _(song song Phase 2–3)_

- [ ] RPC `admin_dashboard_stats`, `admin_chart_stats` (chỉ số tổng hợp)
- [ ] Dashboard: thẻ số liệu, biểu đồ, khối "Cần xử lý"
- [ ] Trang thống kê lá số + danh sách lá số công khai + gỡ công khai
- [ ] Danh sách user (RPC `admin_list_users`) + chi tiết user (chỉ số đếm)
- [ ] Edge Function: `admin-set-role`, `admin-moderate-user`, `admin-delete-user`
- [ ] Bảng `user_sanctions`, kiểm tra `status` khi tạo nội dung, tự mở khoá khi hết hạn

**DoD:** Admin đổi vai trò / khoá user được và có log; xác minh Admin **không** đọc được lá số private.

---

### Phase A3 — Quản lý bài viết, danh mục, tìm kiếm _(song song Phase 4)_

- [ ] Mở rộng `articles` (status, scheduled_at, seo, deleted_at), `article_revisions`, `article_relations`
- [ ] Danh sách bài + lọc + thao tác hàng loạt
- [ ] Trình soạn bài: Markdown, preview, autosave, template theo loại, chèn link sao/cung, upload ảnh
- [ ] Quy trình `draft → review → published`, xuất bản theo lịch (pg_cron)
- [ ] Revision + diff + khôi phục
- [ ] Danh mục kéo-thả, bài liên quan
- [ ] Import Markdown/CSV hàng loạt + `admin-reindex-search`
- [ ] Ghi `search_logs`, hiển thị từ khoá không có kết quả trên Dashboard

**DoD:** Editor tạo và xuất bản một bài hoàn chỉnh không cần đụng SQL; bài xuất bản xuất hiện trong Sổ tay và tìm kiếm được.

> 🚀 **Mốc launch MVP:** cần xong A1 + A2 (rút gọn) + A3 song song với User Phase 1–4.

---

### Phase A4 — Kiểm duyệt hỏi đáp & báo cáo _(song song Phase 5)_

- [ ] Cột `is_hidden/hidden_by/hidden_reason/is_pinned` cho `questions`, `answers`
- [ ] Mở rộng `reports` (status, resolved_by, resolution, internal_note)
- [ ] Hàng đợi báo cáo: gộp theo đối tượng, panel ngữ cảnh, phím tắt, ghi chú nội bộ
- [ ] Quản lý Q&A: ẩn/khôi phục/xoá, ghim, đổi danh mục, gộp trùng, sửa tag
- [ ] Tự ẩn tạm khi ≥ N báo cáo; lọc từ khoá cấm
- [ ] Liên kết hành động kiểm duyệt ↔ cảnh cáo user ↔ audit log

**DoD:** Luồng _User báo cáo → Moderator xử lý → nội dung ẩn → ghi log_ chạy trọn vẹn; moderator không vượt quyền.

---

### Phase A5 — Quản lý học _(song song Phase 6)_

- [ ] CRUD lộ trình, bài học; kéo-thả thứ tự; bật/tắt
- [ ] Trình soạn quiz + schema Zod + xem trước
- [ ] Liên kết bài học ↔ bài viết Sổ tay
- [ ] Thống kê hoàn thành / câu sai nhiều

**DoD:** Editor dựng được toàn bộ lộ trình 6 bước, quiz hiển thị đúng bên User.

---

### Phase A6 — Thông báo, cài đặt, hardening _(xuyên suốt, hoàn thiện cuối)_

- [ ] Thông báo hệ thống: banner, notification, email; `admin-send-announcement`
- [ ] `site_settings`, `feature_flags` + UI; client User đọc flags
- [ ] `admin-export-audit` (CSV)
- [ ] Bắt buộc MFA admin, re-auth cho hành động rủi ro, session policy
- [ ] Rà soát RLS toàn bộ + test theo vai trò trong CI
- [ ] Soft delete & quy trình khôi phục; chính sách lưu trữ audit ≥ 12 tháng
- [ ] Tài liệu vận hành (runbook): xử lý báo cáo, xoá dữ liệu theo yêu cầu, sự cố

**DoD:** Bật/tắt Hỏi đáp & Học bằng flag không cần deploy; kiểm thử bảo mật Admin đạt; có runbook vận hành.

---

## 15. Rủi ro & giảm thiểu

|Rủi ro|Giảm thiểu|
|---|---|
|Lộ quyền (user tự nâng role)|Revoke update cột role, chỉ đổi qua Edge Function, test RLS trong CI|
|Admin xem được lá số riêng tư|Không có policy đọc; thống kê chỉ qua RPC tổng hợp; test tự động|
|Tài khoản admin bị chiếm|MFA bắt buộc, re-auth, cảnh báo thiết bị lạ, audit log|
|Xoá nhầm nội dung|Soft delete, revision, nút Hoàn tác, xác nhận 2 bước|
|Kiểm duyệt quá tải|Gộp báo cáo, tự ẩn tạm theo ngưỡng, lọc từ khoá, phím tắt|
|Bắt đầu quá nặng (admin phình to)|Làm theo Phase A1→A6 song song User; MVP chỉ cần A1, A2 rút gọn, A3|
|Bảng admin chậm khi dữ liệu lớn|Phân trang server-side, index, materialized view|

---

## 16. Tiêu chí chấp nhận (Admin)

- [ ] Đúng vai trò mới vào được đúng màn hình; test RLS theo vai trò pass trong CI
- [ ] Editor tạo → (review) → xuất bản bài, có revision và xuất bản theo lịch
- [ ] Moderator xử lý báo cáo, ẩn/khôi phục nội dung, cảnh cáo/khoá tạm user
- [ ] Admin đổi vai trò, khoá/xoá user, gửi thông báo, chỉnh cài đặt & feature flags
- [ ] Dashboard hiển thị số liệu đúng, **không lộ dữ liệu lá số cá nhân**
- [ ] Mọi hành động nhạy cảm có trong audit log, không sửa/xoá được log
- [ ] MFA bật cho admin; không có service key ở client
- [ ] Bundle admin tách riêng, không làm chậm trang User