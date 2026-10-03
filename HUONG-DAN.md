# Davis Studio — Hướng dẫn triển khai (GitHub + Firebase + Vercel)

## Các file trong gói
| File | Dùng để |
|---|---|
| `index.html` | Trang cho khách (`/`) — không có nút quản trị |
| `admin.html` | Trang quản trị (`/admin`) — đăng nhập để sửa nội dung, ảnh, thông tin |
| `firebase-config.js` | Nơi dán cấu hình Firebase (file nhỏ, sửa ngay trên GitHub) |
| `vercel.json` | Cho Vercel: link `/admin` gọn, không cho Google index trang admin |
| `firestore.rules`, `storage.rules` | Luật bảo mật: ai cũng xem được, chỉ email admin được sửa |
| `cors.json` | Cho phép trang đọc ảnh từ Firebase Storage để phối vân da |

Cách hoạt động: anh sửa ở `/admin` → bấm **Lưu lên máy chủ** → dữ liệu lưu trên Firebase → khách mở `/` thấy ngay. GitHub + Vercel chỉ chứa giao diện, chỉ đổi khi nâng cấp tính năng.

---

## Bước 1 — Firebase (làm trước, vì cần lấy cấu hình)
Vào https://console.firebase.google.com → chọn project Firebase anh đang có.

**1.1. Nâng lên gói Blaze** (bắt buộc để dùng Storage lưu ảnh)
- Góc dưới bên trái: **Upgrade** → chọn **Blaze** → gắn thẻ thanh toán.
- Vẫn có hạn mức miễn phí; nên đặt cảnh báo ngân sách thấp (VD 5 USD) ở Google Cloud Billing → Budgets & alerts để yên tâm.

**1.2. Bật đăng nhập Email/Password**
- **Build → Authentication → Get started → Sign-in method → Email/Password → Enable → Save**.
- Tab **Users → Add user**: nhập email admin + mật khẩu mạnh. Đây là tài khoản anh dùng để vào `/admin`.

**1.3. Tạo Firestore Database**
- **Build → Firestore Database → Create database** → vị trí `asia-southeast1 (Singapore)` → **Start in production mode**.
- Tab **Rules**: xoá hết, dán nội dung file `firestore.rules`, đổi `EMAIL_ADMIN@gmail.com` thành email ở bước 1.2 (viết thường) → **Publish**.

**1.4. Tạo Storage**
- **Build → Storage → Get started** → chọn vùng `US-CENTRAL1` (có hạn mức miễn phí "Always Free") → production mode.
- Tab **Rules**: dán nội dung `storage.rules`, đổi email giống bước 1.3 → **Publish**.
- Ghi lại tên bucket hiện ở đầu trang Files, dạng `gs://TEN-PROJECT.firebasestorage.app`.

**1.5. Bật CORS cho Storage** (để phối vân da/màu da không lỗi)
- Mở https://console.cloud.google.com → chọn đúng project → bấm biểu tượng **Cloud Shell** (`>_`) góc trên phải.
- Dán lệnh sau (thay `TEN-BUCKET` bằng tên bucket ở 1.4, không có `gs://`), Enter:

```
echo '[{"origin":["*"],"method":["GET","HEAD"],"responseHeader":["Content-Type"],"maxAgeSeconds":3600}]' > cors.json && gcloud storage buckets update gs://TEN-BUCKET --cors-file=cors.json
```

**1.6. Lấy cấu hình web**
- **⚙ Project settings → General → Your apps**. Chưa có web app thì bấm **`</>`**, đặt tên `davis-studio`, **không** tick Firebase Hosting → Register.
- Phần **SDK setup and configuration → Config**: copy 6 giá trị `apiKey`, `authDomain`, `projectId`, `storageBucket`, `messagingSenderId`, `appId`.

---

## Bước 2 — GitHub
Vào repo `github.com/Jinzo-Davis/davis-studio`.

**2.1. Tải file lên**
- **Add file → Upload files** → kéo thả tất cả file trong gói (thay luôn `index.html` cũ) → **Commit changes**.

**2.2. Dán cấu hình Firebase**
- Mở `firebase-config.js` → bấm biểu tượng bút chì ✏ → dán 6 giá trị từ bước 1.6 vào giữa các dấu ngoặc kép → **Commit changes**.
- Cấu hình này là thông tin công khai của web app (không phải mật khẩu), an toàn để nằm trên GitHub — bảo mật thật nằm ở Rules (bước 1.3, 1.4).

---

## Bước 3 — Vercel
**3.1. Nếu repo đã nối với Vercel** (site test đang chạy): mỗi lần commit ở bước 2, Vercel tự triển khai lại sau khoảng 1 phút. Xem ở https://vercel.com → project → **Deployments** (trạng thái **Ready**).

**3.2. Nếu chưa nối**: **Add New → Project → Import** repo `davis-studio` → Framework Preset: **Other** → để trống Build Command và Output Directory → **Deploy**.

**3.3. Kiểm tra**
- `https://<ten-site>.vercel.app/` → trang khách.
- `https://<ten-site>.vercel.app/admin` → trang quản trị, hiện hộp đăng nhập.

**3.4. (Tuỳ chọn) Tên miền riêng**, VD `thietke.davis.vn`: Vercel → project → **Settings → Domains → Add** → làm theo hướng dẫn thêm bản ghi CNAME ở nơi quản lý tên miền davis.vn. Sau đó vào Firebase → **Authentication → Settings → Authorized domains → Add domain** thêm tên miền đó (và cả `<ten-site>.vercel.app`).

---

## Bước 4 — Nhập nội dung lần đầu
1. Mở `/admin` → đăng nhập bằng tài khoản ở bước 1.2.
2. Lần đầu, trang hiện dữ liệu mẫu. Lần lượt sửa các tab: **Sản phẩm** (ảnh mẫu A, B, C…) → **Màu da** (ảnh da thật) → **Mẫu khắc** → **Bộ sưu tập** (ảnh thật đã khắc) → **Giới thiệu** → **Liên hệ** (số Zalo, trang Messenger).
3. Bấm **Lưu lên máy chủ** (góc trên phải). Ảnh được tải lên Storage, nội dung lưu vào Firestore.
4. Bấm **Mở trang khách ↗** để xem kết quả.
5. Tab **Đăng và sao lưu → Sao lưu mẫu (.json)**: tải một bản sau mỗi lần sửa lớn.

Nếu bấm Lưu báo "chưa được cấp quyền": kiểm tra email trong Rules (bước 1.3, 1.4) có đúng y hệt email đăng nhập.

---

## Bước 5 — Đưa lên davis.vn
- Đơn giản nhất: thêm nút/menu "Tự thiết kế khắc laser" trỏ tới link trang khách.
- Link từng mục để chạy quảng cáo: `/#gioi-thieu`, `/#bo-suu-tap`, `/#thiet-ke`.
- Không đưa link `/admin` lên website.

## Thêm người quản trị
Firebase → Authentication → Users → **Add user**, rồi thêm email đó vào danh sách trong cả `firestore.rules` và `storage.rules` (Firebase Console → Rules) → Publish.
