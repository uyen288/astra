# ASTRA — Technical Spec (USER)

> 📌 Đây là tài liệu kỹ thuật cho **phía User** (khách + người dùng đã đăng nhập). Phần **Admin** (quản trị, kiểm duyệt, biên tập) nằm ở file riêng: [`technical-admin.md`](https://claude.ai/chat/technical-admin.md). Hai file dùng chung: stack, database, Chart Engine và hạ tầng Vercel/Supabase (định nghĩa ở file này). Mỗi Phase bên dưới có Phase Admin tương ứng ở file admin.

> Stack: **React + TypeScript · Supabase (Auth + PostgreSQL + Storage) · Deploy Vercel**

---

## 1. Mục tiêu & phạm vi

- Web app Tử Vi gồm 3 trụ cột: **Lập lá số · Sổ tay · Hỏi đáp**, kèm Tài khoản và Học.
- Ưu tiên MVP: **Lập lá số + Sổ tay + Account**. Forum và Learning làm sau.
- Nguyên tắc: không xây forum khi chưa có người dùng (forum rỗng nhìn "chết").

---

## 2. Tech stack

|Lớp|Công nghệ|Ghi chú|
|---|---|---|
|Frontend|React 18 + TypeScript + Vite|SPA, dễ deploy Vercel|
|Routing|React Router v6|Route theo tiếng Việt không dấu|
|UI|Tailwind CSS + shadcn/ui (Radix)|Accessible sẵn|
|Server state|TanStack Query|Cache, retry, optimistic update|
|Client state|Zustand|State UI (zoom, đại hạn đang chọn)|
|Form|React Hook Form + Zod|Validate đầu vào lá số|
|Backend|Supabase|Auth, Postgres, RLS, Storage, Edge Functions|
|Nội dung Sổ tay|MDX / Markdown trong repo **hoặc** bảng `articles`|Xem mục 7|
|Search|Postgres Full-Text Search (`unaccent`, `pg_trgm`)|Giai đoạn đầu không cần dịch vụ ngoài|
|Hosting|Vercel|Preview deploy mỗi PR|
|Test|Vitest + Testing Library + Playwright|Unit cho engine, E2E cho flow chính|
|Lint/Format|ESLint + Prettier + Husky||
|Monitoring|Sentry + Vercel Analytics||

> Nếu cần SEO tốt cho Sổ tay (nội dung công khai), cân nhắc Next.js hoặc prerender trang Sổ tay. Với yêu cầu hiện tại (React + Vercel) có thể bắt đầu bằng Vite SPA + prerender (vite-ssg) cho `/so-tay/*`.

---

## 3. Kiến trúc tổng quan

```
┌──────────────────────────────────────────┐
│               React SPA (Vercel)         │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐  │
│  │ Chart UI │ │ Sổ tay   │ │ Hỏi đáp  │  │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘  │
│       │   Chart Engine (TS, thuần, chạy client)
└───────┼────────────┼────────────┼────────┘
        │  supabase-js (RLS)      │
┌───────▼────────────▼────────────▼────────┐
│ Supabase: Auth · Postgres · Storage · Edge│
└───────────────────────────────────────────┘
```

**Quyết định quan trọng: Chart Engine là thư viện TypeScript thuần**, không phụ thuộc React/Supabase:

- Chạy được ở client (khách lập lá số không cần server).
- Test unit dễ, tái sử dụng trong Edge Function nếu cần.
- Input: `BirthInput` → Output: `ChartData` (JSON).

---

## 4. Cấu trúc thư mục

```
astra/
├── src/
│   ├── app/                 # router, providers, layout
│   ├── features/
│   │   ├── chart/           # form, ChartGrid, PalaceCell, panels, share
│   │   ├── account/
│   │   ├── handbook/        # sổ tay, search, bookmark
│   │   ├── qa/              # hỏi đáp
│   │   └── learning/        # lộ trình, bài, quiz
│   ├── engine/              # Chart Engine (thuần TS)
│   │   ├── calendar/        # dương ↔ âm, can chi, canh giờ
│   │   ├── stars/           # an chính tinh, phụ tinh, tứ hóa
│   │   ├── limits/          # đại hạn, tiểu hạn
│   │   ├── data/            # bảng tra (JSON)
│   │   └── index.ts
│   ├── lib/                 # supabase client, utils
│   ├── components/          # UI dùng chung
│   ├── content/             # MDX sổ tay (nếu dùng file)
│   └── types/
├── supabase/
│   ├── migrations/
│   ├── seed.sql
│   └── functions/
├── e2e/
└── vercel.json              # rewrite SPA
```

---

## 5. Routes

```
/                          Trang chủ
/lap-la-so                 Form lập lá số
/la-so/:id                 Xem lá số
/dang-nhap                 Đăng nhập
/tai-khoan
  /la-so                   Lá số của tôi
  /bai-viet-da-luu
  /ghi-chu
/so-tay
  /nhap-mon  /sao  /cung  /cach-cuc  /bai-viet
  /:category/:slug         Trang bài
/hoc
  /lo-trinh
  /bai/:slug
/hoi-dap
  /cau-hoi/:id
  /dat-cau-hoi
```

Route cần auth: `/tai-khoan/*`, `/hoi-dap/dat-cau-hoi`. Còn lại truy cập tự do (lá số tuỳ visibility).

---

## 6. Chart Engine (lõi của sản phẩm)

### 6.1 Input / Output

```ts
type BirthInput = {
  name?: string;
  gender: 'male' | 'female';
  calendar: 'solar' | 'lunar';
  year: number; month: number; day: number;
  isLeapMonth?: boolean;       // nếu nhập âm lịch
  hour: number; minute?: number; // → canh giờ
  birthplace?: string;
  viewYear: number;             // năm xem hạn
};

type ChartData = {
  meta: { canChiYear; canChiMonth; canChiDay; canChiHour; amDuong; napAm; cuc; menhCungIndex; thanCungIndex };
  palaces: Palace[];            // 12 cung
  tuHoa: TuHoaMap;
  daiHan: DaiHan[];             // 12 đại hạn
  tieuHan?: TieuHan;
  version: string;              // phiên bản engine, để migrate khi sửa thuật toán
};

type Palace = {
  index: number;                // 0..11 theo địa chi
  name: PalaceName;             // Mệnh, Phụ Mẫu, ...
  canChi: { can: string; chi: string };
  majorStars: StarInstance[];
  minorStars: StarInstance[];
  isThan: boolean;
  daiHanRange: [number, number];
};
```

### 6.2 Các module cần làm

1. **Lịch**: đổi dương ↔ âm (kể cả tháng nhuận), Can Chi năm/tháng/ngày/giờ, 12 canh giờ.
2. **Định cung**: an cung Mệnh, Thân theo tháng + giờ; xác định 12 cung theo chiều thuận/nghịch.
3. **Cục**: tính Ngũ hành cục từ Can năm + cung Mệnh.
4. **Chính tinh**: an Tử Vi (theo ngày + cục) → vòng Tử Vi; an Thiên Phủ → vòng Thiên Phủ.
5. **Phụ tinh**: nhóm theo năm (Lộc Tồn, Kình Dương, Đà La, Thiên Khôi/Việt...), theo tháng (Tả Phụ, Hữu Bật...), theo giờ (Văn Xương, Văn Khúc, Địa Không, Địa Kiếp), vòng Tràng Sinh, Bác Sĩ...
6. **Tứ Hóa**: theo Thiên Can năm sinh (Lộc / Quyền / Khoa / Kỵ).
7. **Đại hạn**: khởi từ số Cục, chiều thuận/nghịch theo Âm Dương + giới tính.
8. **Tiểu hạn / lưu niên**: theo năm xem.
9. **Tam hợp - Xung chiếu**: hàm tiện ích trả về chỉ số cung liên quan.

### 6.3 Dữ liệu & kiểm thử

- Bảng tra lưu ở `engine/data/*.json` (tách khỏi logic).
- **Test bằng golden cases**: lấy 20–50 lá số chuẩn (đối chiếu với phần mềm/ sách tử vi uy tín) làm fixture; mỗi case so sánh toàn bộ sao trên 12 cung.
- Ghi rõ **trường phái / quy ước** được dùng (VD cách an Tiểu hạn, Tứ Hóa theo trường phái nào) trong tài liệu và UI.
- Có trường `version` trong `ChartData` để sau này sửa thuật toán vẫn tái tính được lá số cũ.

> ⚠️ Đây là phần rủi ro nhất: sai một quy tắc là sai toàn bộ sản phẩm. Dành nhiều thời gian nhất cho Phase 2.

---

## 7. Database (Supabase PostgreSQL)

### 7.1 Quan hệ chính

```
auth.users
 ├── profiles (1–1)
 ├── birth_profiles (1–n) ── charts (1–n)
 ├── questions ── answers ── votes
 ├── article_bookmarks
 ├── notes
 └── lesson_progress
```

### 7.2 Schema (rút gọn)

```sql
-- Extensions
create extension if not exists unaccent;
create extension if not exists pg_trgm;

-- Profiles
create table profiles (
  id uuid primary key references auth.users on delete cascade,
  display_name text,
  avatar_url text,
  role text not null default 'user'
    check (role in ('user','editor','moderator','admin')),  -- chi tiết quyền: xem technical-admin.md
  status text not null default 'active'
    check (status in ('active','suspended','banned')),
  suspended_until timestamptz,
  created_at timestamptz default now()
);

-- Hồ sơ ngày sinh
create table birth_profiles (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references profiles(id) on delete cascade,
  label text,                        -- "Của tôi", "Bố", "Mẹ"
  gender text check (gender in ('male','female')) not null,
  calendar text check (calendar in ('solar','lunar')) not null,
  birth_year int not null, birth_month int not null, birth_day int not null,
  is_leap_month boolean default false,
  birth_hour int not null, birth_minute int,
  birthplace text,
  created_at timestamptz default now()
);

-- Lá số
create table charts (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references profiles(id) on delete cascade, -- null nếu khách (chart ẩn danh)
  birth_profile_id uuid references birth_profiles(id) on delete set null,
  input jsonb not null,              -- BirthInput
  data jsonb not null,               -- ChartData (cache kết quả)
  engine_version text not null,
  visibility text not null default 'private'
    check (visibility in ('private','link','public')),
  share_token text unique,           -- dùng cho visibility = 'link'
  created_at timestamptz default now()
);

-- Sổ tay
create table article_categories (
  id serial primary key,
  slug text unique not null,
  name text not null,
  parent_id int references article_categories(id),
  sort_order int default 0
);

create table articles (
  id uuid primary key default gen_random_uuid(),
  category_id int references article_categories(id),
  slug text unique not null,
  title text not null,
  summary text,
  content_md text not null,
  kind text check (kind in ('star','palace','pattern','concept','post')),
  entity_key text,                   -- VD 'tham-lang', để nối với sao trong lá số
  published boolean default false,
  search tsvector generated always as (
    to_tsvector('simple', unaccent(coalesce(title,'') || ' ' || coalesce(summary,'') || ' ' || coalesce(content_md,'')))
  ) stored,
  updated_at timestamptz default now()
);
create index articles_search_idx on articles using gin (search);
create index articles_title_trgm on articles using gin (title gin_trgm_ops);

create table article_bookmarks (
  user_id uuid references profiles(id) on delete cascade,
  article_id uuid references articles(id) on delete cascade,
  created_at timestamptz default now(),
  primary key (user_id, article_id)
);

create table notes (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references profiles(id) on delete cascade,
  article_id uuid references articles(id) on delete set null,
  chart_id uuid references charts(id) on delete set null,
  body text not null,
  created_at timestamptz default now()
);

-- Hỏi đáp
create table questions (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references profiles(id) on delete cascade,
  title text not null, body text not null,
  category text,
  chart_id uuid references charts(id) on delete set null,
  accepted_answer_id uuid,
  status text default 'open',
  created_at timestamptz default now()
);
create table question_tags (
  question_id uuid references questions(id) on delete cascade,
  tag text not null,
  primary key (question_id, tag)
);
create table answers (
  id uuid primary key default gen_random_uuid(),
  question_id uuid not null references questions(id) on delete cascade,
  user_id uuid not null references profiles(id) on delete cascade,
  body text not null,
  created_at timestamptz default now()
);
create table answer_references (        -- "Câu trả lời này dựa trên kiến thức nào?"
  answer_id uuid references answers(id) on delete cascade,
  article_id uuid references articles(id) on delete cascade,
  primary key (answer_id, article_id)
);
create table votes (
  user_id uuid references profiles(id) on delete cascade,
  target_type text check (target_type in ('question','answer')),
  target_id uuid not null,
  value smallint check (value in (1,-1)),
  primary key (user_id, target_type, target_id)
);
create table reports (
  id uuid primary key default gen_random_uuid(),
  reporter_id uuid references profiles(id),
  target_type text, target_id uuid, reason text,
  created_at timestamptz default now()
);
create table comments (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references profiles(id) on delete cascade,
  target_type text, target_id uuid, body text not null,
  created_at timestamptz default now()
);
create table notifications (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references profiles(id) on delete cascade,
  type text, payload jsonb, read boolean default false,
  created_at timestamptz default now()
);

-- Học
create table learning_paths (id serial primary key, slug text unique, title text, sort_order int);
create table lessons (
  id serial primary key,
  path_id int references learning_paths(id),
  slug text unique, title text,
  article_id uuid references articles(id),
  quiz jsonb,                         -- danh sách câu hỏi
  sort_order int
);
create table lesson_progress (
  user_id uuid references profiles(id) on delete cascade,
  lesson_id int references lessons(id) on delete cascade,
  status text check (status in ('started','completed')),
  score int, updated_at timestamptz default now(),
  primary key (user_id, lesson_id)
);
```

### 7.3 Row Level Security (bắt buộc bật cho mọi bảng)

|Bảng|Đọc|Ghi|
|---|---|---|
|profiles|Chính chủ (+ tên hiển thị public cho Q&A)|Chính chủ (**không được tự sửa `role`, `status`** — chỉ Admin qua Edge Function)|
|birth_profiles|Chính chủ|Chính chủ|
|charts|Chính chủ; hoặc `visibility='public'`; hoặc truy cập qua `share_token` (RPC)|Chính chủ|
|articles|Mọi người nếu `published`|Admin|
|article_bookmarks, notes, lesson_progress|Chính chủ|Chính chủ|
|questions, answers, comments|Mọi người|Chính chủ (sửa/xoá), user đăng nhập (tạo)|
|votes|Mọi người (đếm)|Chính chủ, 1 vote/target|
|reports|Admin|User đăng nhập (tạo)|

Ví dụ policy:

```sql
alter table charts enable row level security;

create policy "owner full access" on charts
  for all using (auth.uid() = user_id) with check (auth.uid() = user_id);

create policy "public charts readable" on charts
  for select using (visibility = 'public');

-- Link chia sẻ: dùng RPC security definer lấy theo share_token,
-- không mở select rộng cho bảng.
create function get_chart_by_token(token text) returns charts
language sql security definer as $$
  select * from charts where share_token = token and visibility = 'link'
$$;
```

### 7.4 Luồng lá số của khách

- Khách lập lá số hoàn toàn ở client (engine), lưu tạm ở `sessionStorage`/state.
- Khi bấm "Đăng nhập để lưu": đăng nhập xong → insert `birth_profiles` + `charts` gắn `user_id`.
- Nếu cần URL `/la-so/:id` cho khách: insert chart ẩn danh với `visibility='link'` + `share_token` (chống đoán bằng UUID/token dài, giới hạn rate).

### 7.5 Nội dung Sổ tay: file hay DB?

||MDX trong repo|Bảng `articles`|
|---|---|---|
|Ưu|Version control, build tĩnh, SEO tốt|Sửa nội dung không cần deploy, dễ làm CMS|
|Nhược|Sửa phải deploy|Cần xây trang admin|
|**Gợi ý**|**MVP: dùng bảng `articles` + seed từ Markdown**, thêm admin đơn giản sau||

---

## 8. Tìm kiếm

- Postgres FTS trên `articles.search` + `pg_trgm` cho gợi ý khi gõ.
- Hỗ trợ **gõ không dấu** qua `unaccent`.
- RPC `search_all(q text)` trả về nhóm: `star`, `palace`, `pattern`, `post`.
- Debounce 200ms phía client; cache bằng TanStack Query.
- Có thể nạp danh mục sao/cung/cách cục từ `entity_key` để link trực tiếp từ lá số.

---

## 9. Auth

- Supabase Auth: **Google OAuth + Email magic link**.
- Trigger tạo `profiles` khi có user mới (`handle_new_user`).
- Session ở client, guard route bằng `<RequireAuth>`.
- Gắn lá số khách vào tài khoản sau đăng nhập (xem 7.4).

---

## 10. Bảo mật & riêng tư

- RLS cho **tất cả** bảng; không dùng service key ở client.
- Dữ liệu sinh là dữ liệu cá nhân: mặc định `private`, cảnh báo khi chuyển `public`.
- Rate limit tạo chart ẩn danh, đặt câu hỏi, vote (Edge Function / DB constraint).
- Sanitize nội dung Markdown người dùng (DOMPurify / rehype-sanitize).
- Quy trình **xoá tài khoản + xoá dữ liệu** (tuân thủ quyền riêng tư).
- Security headers qua `vercel.json` (CSP, X-Frame-Options...).
- Có trang Điều khoản, Chính sách riêng tư, Disclaimer.

---

## 11. Hiệu năng & SEO

- Code-splitting theo route (lazy).
- Engine chạy < 50ms trên thiết bị thường; không block UI.
- Lá số render bằng SVG/CSS grid (nhẹ); virtualize danh sách dài (Q&A).
- Prerender / meta tags cho `/so-tay/*`; Open Graph cho trang bài và lá số công khai.
- Ảnh tối ưu, font subset tiếng Việt.

---

## 12. CI/CD & môi trường

|Môi trường|Mục đích|
|---|---|
|Local|Supabase CLI (`supabase start`) + Vite dev|
|Preview|Vercel preview mỗi PR + Supabase branch/dự án staging|
|Production|Vercel prod + Supabase prod|

- GitHub Actions: lint → typecheck → unit test (engine) → build → E2E smoke.
- Migration quản lý bằng `supabase/migrations`, chạy qua CLI.
- Biến môi trường: `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`, `VITE_SENTRY_DSN`.
- `vercel.json` rewrite mọi route về `index.html` (SPA).

---

## 13. Lộ trình phát triển theo Phase

### Phase 1 — Foundation

**Mục tiêu:** Nền móng chạy được, deploy được.

- [ ] Khởi tạo repo: React + TypeScript + Vite + Tailwind + shadcn/ui
- [ ] Cấu hình ESLint, Prettier, Husky, Vitest
- [ ] Router, layout (header, footer, bottom tab mobile), theme sáng/tối
- [ ] Tạo dự án Supabase, cấu hình client, biến môi trường
- [ ] Deploy Vercel + preview deploy, `vercel.json`
- [ ] CI GitHub Actions cơ bản
- [ ] Sentry + analytics

**Definition of Done:** Trang chủ + khung route deploy trên Vercel, kết nối được Supabase, CI xanh.

---

### Phase 2 — Lập lá số ⭐ (phải làm kỹ nhất)

**Mục tiêu:** Lập lá số chính xác, UI dùng tốt trên mobile.

**Engine**

- [ ] Đổi lịch dương ↔ âm, Can Chi năm/tháng/ngày/giờ
- [ ] An Mệnh/Thân, 12 cung, Cục
- [ ] An 14 chính tinh
- [ ] An phụ tinh (cát tinh, sát tinh, vòng Tràng Sinh...)
- [ ] Tứ Hóa
- [ ] Đại hạn, Tiểu hạn
- [ ] Tam hợp / xung chiếu
- [ ] **Bộ golden test** ≥ 20 lá số đối chiếu chuẩn; viết tài liệu quy ước trường phái

**UI**

- [ ] Form nhập (dương/âm, giờ, giới tính, nơi sinh) + validation
- [ ] `ChartGrid` 12 cung, `PalaceCell`, badge Tứ Hóa
- [ ] Zoom / pan, bottom sheet trên mobile
- [ ] Click cung / sao → panel giải thích (nội dung tạm, nối Sổ tay ở Phase 4)
- [ ] Chuyển giữa các đại hạn, chọn năm xem tiểu hạn
- [ ] Lập lá số cho khách (không cần đăng nhập)

**Definition of Done:** Mọi golden test pass; lập lá số mượt trên điện thoại; khách dùng được không cần tài khoản.

---

### Phase 3 — Account

**Mục tiêu:** Đăng nhập, lưu và quản lý lá số, chia sẻ.

- [ ] Supabase Auth (Google + magic link), trigger tạo `profiles`
- [ ] Bảng `birth_profiles`, `charts` + RLS
- [ ] Gắn lá số khách vào tài khoản sau đăng nhập
- [ ] Trang `/tai-khoan/la-so`: danh sách, đổi tên, xoá
- [ ] URL riêng `/la-so/:id`
- [ ] Chia sẻ: 🔒 Riêng tư / 🔗 Link / 🌎 Công khai (`share_token`, RPC)
- [ ] Trang 403/404 thân thiện, ẩn tên/ngày sinh khi chia sẻ
- [ ] Xoá tài khoản + dữ liệu

**Definition of Done:** Người dùng lưu, xem lại, chia sẻ lá số đúng quyền; RLS được test (user A không đọc được lá số private của user B).

---

### Phase 4 — Sổ tay

**Mục tiêu:** Knowledge base + search + bookmark, nối với lá số.

- [ ] Bảng `article_categories`, `articles`, seed nội dung ban đầu (14 chính tinh, 12 cung, Tứ Hóa, Can Chi...)
- [ ] Template trang bài + mục lục + "Đọc tiếp"
- [ ] Điều hướng cây Sổ tay
- [ ] **Search toàn cục** (FTS + không dấu + command palette `Ctrl/⌘ K`)
- [ ] Bookmark bài viết + trang bài đã lưu
- [ ] Ghi chú cá nhân (`notes`)
- [ ] Nối lá số ↔ Sổ tay: click sao/cung → link `entity_key` tới bài tương ứng
- [ ] Lá số mẫu trong bài ("Xem lá số mẫu")
- [ ] SEO: meta, Open Graph, prerender

**Definition of Done:** Tìm "tu vi" ra đúng sao/cung/bài; click sao trên lá số mở được bài giải thích; bookmark hoạt động.

> 🚀 **Mốc launch MVP đề xuất: sau Phase 4** (Lập lá số + Account + Sổ tay).

---

### Phase 5 — Q&A (Hỏi đáp)

**Mục tiêu:** Cộng đồng hỏi đáp chuyên Tử Vi, nối với Sổ tay.

- [ ] Bảng `questions`, `answers`, `question_tags`, `votes`, `reports`, `comments`, `answer_references`
- [ ] Danh sách câu hỏi: Mới nhất / Nhiều tương tác / Chưa có trả lời; lọc theo danh mục, tag
- [ ] Đặt câu hỏi (kèm đính kèm lá số của mình)
- [ ] Trả lời + editor chèn link Sổ tay, trích sao/cung
- [ ] Khối **"Câu trả lời này dựa trên kiến thức nào?"** (`answer_references`)
- [ ] Vote, chấp nhận câu trả lời tốt nhất
- [ ] Báo cáo (report) + công cụ moderation tối thiểu cho admin
- [ ] Thông báo (`notifications`)
- [ ] Rate limit, chống spam

**Definition of Done:** Luồng Question → Answer → Vote → Tag → Report hoạt động đầy đủ; lá số đính kèm tuân thủ quyền riêng tư.

---

### Phase 6 — Learning

**Mục tiêu:** Lộ trình học có tiến độ.

- [ ] Bảng `learning_paths`, `lessons`, `lesson_progress`
- [ ] Trang `/hoc/lo-trinh` với 6 bước (Âm Dương → ... → Đại hạn)
- [ ] Mỗi bài: Đọc → Quiz → Áp dụng vào lá số
- [ ] Progress bar, hiển thị "Đang học" ở trang chủ
- [ ] Về sau: flashcard / spaced repetition

**Definition of Done:** User hoàn thành một lộ trình, tiến độ được lưu và hiển thị đúng ở trang chủ.

---

## 13.1 Liên kết với Phase Admin

|Phase User (file này)|Phase Admin tương ứng (`technical-admin.md`)|
|---|---|
|1 — Foundation|A1 — Khung Admin, vai trò, audit log|
|2 — Lập lá số|A2 — Dashboard & thống kê lá số|
|3 — Account|A2 — Quản lý người dùng|
|4 — Sổ tay|A3 — Quản lý bài viết, danh mục, search|
|5 — Q&A|A4 — Kiểm duyệt hỏi đáp & báo cáo|
|6 — Learning|A5 — Quản lý lộ trình, bài học, quiz|
|(xuyên suốt)|A6 — Thông báo, cài đặt, hardening|

> Với Phase 4, nội dung Sổ tay cần có công cụ nhập liệu: nên làm **A3 song song** (hoặc tối thiểu seed từ Markdown) để không phải sửa nội dung bằng SQL.

---

## 14. Rủi ro & giảm thiểu

|Rủi ro|Giảm thiểu|
|---|---|
|Thuật toán an sao sai / khác trường phái|Golden tests, tài liệu hoá quy ước, `engine_version`|
|Lộ dữ liệu sinh|RLS chặt, mặc định private, test phân quyền|
|Forum rỗng, thiếu nội dung|Launch Phase 4 trước, mở Q&A khi có traffic|
|Nội dung Sổ tay tốn công viết|Bắt đầu với nhóm cốt lõi (14 chính tinh, 12 cung), mở rộng dần|
|Scope phình to ("con quái vật 6 tháng")|Giữ đúng thứ tự phase, mỗi phase có DoD rõ|
|SEO yếu với SPA|Prerender trang Sổ tay hoặc chuyển Next.js nếu cần|

---

## 15. Tiêu chí chấp nhận MVP (sau Phase 4)

- [ ] Khách lập được lá số đúng, không cần đăng nhập
- [ ] Lá số hiển thị đủ: 12 cung, chính/phụ tinh, Tứ Hóa, Mệnh/Thân, Đại hạn/Tiểu hạn, Can Chi
- [ ] Đăng nhập, lưu & quản lý nhiều lá số
- [ ] Chia sẻ bằng link với 3 mức riêng tư
- [ ] Sổ tay có nội dung cốt lõi, tìm kiếm được (kể cả không dấu)
- [ ] Bookmark + ghi chú
- [ ] Dùng tốt trên mobile, Lighthouse ≥ 90 (Performance/Accessibility)
- [ ] Deploy production trên Vercel, có monitoring