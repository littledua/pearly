# Pearly: ghi chú cho Claude

Pearly là sổ tiền cá nhân của chủ shop (Tiệm Dứa + Pinya Printing). Đây là app web cài được lên điện thoại (PWA), giao diện chủ đề biển màu pastel.

## Cách nói chuyện
- Chủ nhắn tiếng Việt kiểu thân, gọi Claude là "bà", tự xưng "tui". Trả lời tiếng Việt, ngắn, dễ hiểu, không dùng thuật ngữ nếu không cần.
- Chủ hỏi "hoạt động như thế nào" thì **chỉ giải thích, đừng làm**. Chỉ bắt đầu khi chủ nói rõ "làm đi".
- Trước khi sửa database Supabase (tạo/sửa bảng, hàm, xoá dữ liệu), phải nói rõ sẽ làm gì và **chờ chủ đồng ý**.
- Sửa xong và tự kiểm tra rồi thì đẩy thẳng lên `main` (trang chạy từ `main`).

## Chỗ để mọi thứ
- **Code:** repo này. Web tĩnh, không build. `index.html` là cả app (HTML, CSS, JS chung một file). `sw.js` cho app mở được khi mất mạng; đổi `index.html` thì nhớ đổi chuỗi `V` trong `sw.js` để điện thoại tải bản mới. `manifest.webmanifest`, `icons/`, `sea/` (hình nền, rong, san hô, bong bóng, cá bơi `m_*_L/R.webp`).
- **Link app:** https://littledua.github.io/pearly/ (GitHub Pages, nhánh `main`, thư mục gốc).
- **Dữ liệu không nằm trong repo.** Nằm ở Supabase:
  - **Tiệm Dứa**, project `aornraushxlloskdcqne` (https://aornraushxlloskdcqne.supabase.co):
    - `pearly_entries(user_id, id, data jsonb, updated_at)`: mỗi dòng là một khoản ghi.
    - `pearly_profile(user_id, data jsonb, updated_at)`: ví/thẻ, mục chi, ngân sách, số dư gốc (anchor), offset.
    - `pearly_owners(user_id)`: danh sách chủ được dùng Pearly. RLS chặn mọi người khác.
    - Hàm `pearly_is_owner()` và `pearly_td_events(from_code text)`: trả tiền khách CK đã xác nhận, CK đang chờ, phí ship (khi hàng về kho) tính từ đơn `from_code`.
  - **Pinya**, project `sdplwfpmbavcfjfxfecf` (https://sdplwfpmbavcfjfxfecf.supabase.co): có `pearly_owners` và hàm `pearly_pp_events()` (đọc bảng `records`: order, printjob, jobfee, logistic). Project này nằm ở nhóm Supabase khác, `list_projects` không thấy, nhưng gọi thẳng bằng mã project thì được.
- Khoá công khai (publishable) của hai project nằm trong `SBCONF` ở `index.html`. Không đưa khoá bí mật vào repo.
- Đăng nhập: email + mật khẩu tài khoản admin. Cả hai project đều có tài khoản của chủ. App tự đăng nhập Pinya bằng cùng mật khẩu, sai thì hiện nút "Đăng nhập Pinya".
- Code gọi Supabase bằng REST tự viết (`sbClient`), không dùng thư viện supabase-js. Máy Claude bị chặn tải npm/CDN.
- Ghi chép trên máy: `pearly-cache-v1` (bản sao sổ), `pearly-outbox-v1` (thay đổi chờ mạng), `pearly-auth-td`/`pearly-auth-pp` (phiên đăng nhập). Mất mạng vẫn ghi được, có mạng lại thì tự lưu lên.

## Lịch sử
- Lúc đầu Pearly là một artifact trong Claude, tên cũ "Sổ Xèng". Sổ khi đó lưu trong kho dữ liệu của artifact. Artifact cũ giờ có dòng nhắc chuyển qua app.
- Ngày 12/10/2026 chuyển sang PWA và dời dữ liệu qua Supabase. Hồi đó app nằm ở `littledua/pinyaprinting`, thư mục `pearly/` (https://littledua.github.io/pinyaprinting/pearly/).
- Ngày 22/11/2026 tách sang repo riêng này.

## Việc còn dở (làm theo thứ tự)
1. Kiểm tra https://littledua.github.io/pearly/ đã chạy chưa. Chủ phải tự bật Pages: Settings, Pages, Deploy from a branch, `main`, `/ (root)`. Claude không bật được vì proxy chặn API Pages.
2. Khi link mới chạy rồi, trong repo `littledua/pinyaprinting`:
   - Thay `pearly/index.html` bằng trang tự chuyển sang `https://littledua.github.io/pearly/`.
   - Thay `pearly/sw.js` bằng bản tự gỡ (xoá cache `pearly-*` rồi unregister) để điện thoại khỏi giữ bản cũ.
   - Xoá phần còn lại trong `pearly/`.
   - Sửa dòng nhắc Pearly trong `CLAUDE.md` của repo đó.
   - Repo đó có quy tắc riêng: làm trên nhánh `claude/...`, `git fetch origin main` trước khi đẩy `main`.
3. Báo chủ thêm Pearly vào màn hình chính bằng link mới. Không cần đăng nhập lại vì cùng tên miền `littledua.github.io`.

## Quy tắc tính tiền (chủ đã chốt, đừng đổi nếu chưa hỏi)
- **Số dư ngân hàng** = số dư thật chủ nhập (anchor) + tiền vào − chi trả bằng ngân hàng − trả nợ thẻ/ví − đổi tệ. Khoản quẹt thẻ hay ví trả sau **không** trừ ngân hàng lúc quẹt, chỉ trừ khi trả nợ.
- **Loại khoản** (`data.kind`): không có kind là chi thường; `thu` là tiền vào; `doite` là đổi tệ (có `cny`); `tra_no` là trả nợ thẻ/ví (`acct`).
- **Tiền shop:**
  - Tiệm Dứa tính từ đơn **DH00040**: cộng khi chủ xác nhận khách CK, trừ phí ship khi hàng về kho. **Không** trừ tiền hàng, tiền in.
  - Pinya tính từ ngày **07/10/2026**.
  - Đơn chưa có thanh toán thì không cộng gì.
- **Thẻ/ví:**
  - MoMo, ZaloPay, SPayLater: hạn ngày **10**.
  - Thẻ tín dụng HC: hạn ngày **12**, sao kê ngày 31.
  - Thẻ trả sau HC: hạn cố định ngày **28**, sao kê ngày 20, rơi vào thứ Bảy/Chủ nhật thì dời sang **thứ Hai** (`wkShift`).
  - Trả bất kỳ số tiền nào sau hạn kỳ trước thì kỳ hiện tại tính là đã trả, hạn nhảy sang tháng sau.
  - **Còn dùng** = hạn mức − tiền gốc chưa trả (không tính lãi).
  - Nợ có sẵn: `debtAdj` + `adjDate`. `adjNext` nghĩa là nợ đó chưa sao kê, trả vào kỳ sau.
- **Trả góp:** 3/6/9/12 kỳ. Gốc và lãi **mỗi kỳ khác nhau**, lưu trong `inst.rows=[{due,goc,lai}]`. Ô app tự đoán hiện nền tím. Ví dụ thật: quẹt 3.500.000 trên thẻ trả sau HC ngày 07/10/2026:
  - 28/10: gốc 1.152.383, lãi 90.617
  - 30/11: gốc 1.147.487, lãi 95.513
  - 28/12: gốc 1.200.130, lãi 41.430

## Giao diện
- Chủ đề biển pastel, hình nền khác nhau cho máy tính và điện thoại, cá/sứa/mực/rùa bơi với cỡ và tốc độ ngẫu nhiên, bong bóng nhiều cỡ.
- Thanh tab là các ô vuông, ô đang chọn nhô lên.
- **Mỗi mục chi phải có biểu tượng riêng, không trùng nhau** (bảng `SEA_ICONS`, hàm `seaIcon`).
- Không vẽ lại hay chép nhân vật hay tranh của người khác. Chỉ dùng hình chủ đưa.
- Ô nhập trên điện thoại cỡ chữ tối thiểu 16px để khỏi tự zoom.

## Kiểm tra
- Máy Claude không gọi được Supabase từ trình duyệt thử. Chạy `python3 -m http.server`, rồi Playwright (Chromium có sẵn) giả lập Supabase bằng `page.route(/supabase\.co/)`, trả dữ liệu mẫu. Thử ở cỡ 390px trước.
- Muốn xem dữ liệu thật thì dùng Supabase MCP `execute_sql`, giả làm chủ bằng `set_config('request.jwt.claims', …)` cộng `set local role authenticated`, bọc trong `begin … rollback` khi chỉ đọc.
