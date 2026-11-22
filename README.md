# Pearly

Sổ tiền cá nhân (PWA). Mở: https://littledua.github.io/pearly/

- Code: trang tĩnh, không build. `index.html` là cả app, `sw.js` giúp mở được khi mất mạng, `sea/` là hình nền, `icons/` là biểu tượng.
- Dữ liệu không nằm trong repo này. Sổ tiền nằm ở Supabase Tiệm Dứa (bảng `pearly_entries`, `pearly_profile`, có khoá theo tài khoản). Tiền shop đọc qua hàm `pearly_td_events` (Tiệm Dứa) và `pearly_pp_events` (Pinya), chỉ tài khoản chủ gọi được.
- Đăng nhập bằng email + mật khẩu trang admin Tiệm Dứa.
