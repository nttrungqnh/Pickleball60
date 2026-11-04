# PICKLEBALL 60 — Firebase + GitHub Pages

Website tĩnh dùng HTML5, CSS3, JavaScript ES Modules và Firebase. GitHub Pages phục vụ trực tiếp `index.html`; trình duyệt kết nối Firebase Authentication và Cloud Firestore. Không cần backend/server runtime hoặc bước build.

## Cấu trúc

- `index.html`: ứng dụng tĩnh, giao diện tiếng Việt.
- `assets/css/`: design system và responsive styles.
- `assets/js/app.js`: đăng nhập, CRUD, nhật ký, challenge, thống kê và biểu đồ.
- `assets/images/`: biểu tượng ứng dụng dạng SVG.
- `firebase-config.js`: cấu hình Web App công khai của Firebase.
- `firestore.rules`: quy tắc giới hạn dữ liệu theo UID và kiểm tra dữ liệu.
- `manifest.webmanifest`, `service-worker.js`: khả năng cài lên màn hình chính và cache app shell.

## 1. Tạo Firebase Project

1. Mở [Firebase Console](https://console.firebase.google.com/) và tạo project.
2. Thêm một Web App trong Project settings → General → Your apps.
3. Sao chép cấu hình Firebase Web App; cấu hình này là thông tin client công khai, không phải mật khẩu.

## 2. Bật đăng nhập Google

1. Trong Firebase Console mở Authentication → Sign-in method.
2. Bật Google và chọn email hỗ trợ.
3. Trong Authentication → Settings → Authorized domains, thêm domain GitHub Pages, ví dụ `your-name.github.io`. Chỉ nhập hostname, không thêm `https://` hay tên repository.

## 3. Tạo Firestore

1. Mở Firestore Database → Create database.
2. Chọn vị trí phù hợp và tạo database.
3. Không bật test mode để triển khai công khai.

Dữ liệu được đặt dưới `users/{uid}/profile/main`, `challenges/{challengeId}`, `exercises/{exerciseId}` và `trainingLogs/{logId}`. Nhật ký dùng ID xác định `{challengeId}_{YYYY-MM-DD}` để mỗi thử thách chỉ có một nhật ký trong một ngày; lưu lại ngày đó sẽ cập nhật bản ghi hiện có.

## 4. Cấu hình ứng dụng

`firebase-config.js` hiện đã có cấu hình Web App. Kiểm tra trong Firebase Console → Project settings → General → Your apps để chắc chắn cấu hình đó thuộc đúng project, và thay các giá trị trong tệp nếu bạn dùng project khác. Firebase web config có thể nằm trong source GitHub Pages; quyền truy cập dữ liệu được bảo vệ bằng Authentication và Firestore Rules, không bằng cách giấu API key.

Khi bấm nút đăng nhập, ứng dụng dùng Google popup và báo hướng xử lý cụ thể nếu Firebase trả về `unauthorized-domain`, `operation-not-allowed` hoặc popup bị chặn. Thêm đúng hostname đang chạy vào Authorized domains, ví dụ `username.github.io` cho GitHub Pages hoặc `localhost` khi kiểm tra trên localhost. Không thêm đường dẫn repository vào danh sách domain.

Các thư viện ngoài được ghim phiên bản trong source: Firebase JavaScript SDK `10.12.5`, Chart.js `4.4.3`.

## 5. Triển khai Firestore Security Rules

Trong Firebase Console → Firestore Database → Rules, dán nội dung `firestore.rules` rồi bấm Publish. Rules yêu cầu người dùng đã đăng nhập và UID trên đường dẫn phải bằng `request.auth.uid`. Quy tắc giới hạn kích thước/kiểu dữ liệu, phút 1–1440, điểm 1–10, ghi chú, video HTTPS và tối đa 20 kỹ thuật trong một nhật ký. Nhật ký phân biệt `practice` (luyện tập), `social` (giao lưu) và `rest` (nghỉ tập); giao lưu tính là ngày hoạt động, nghỉ tập không cộng chuỗi.

Mọi dữ liệu của người dùng khác bị từ chối. Không thay rules bằng quyền đọc/ghi công khai.

## 6. Đẩy source lên GitHub

Commit toàn bộ source, bao gồm `index.html`, `assets/`, `firebase-config.js`, `manifest.webmanifest`, `service-worker.js` và `firestore.rules` lên repository. Không có bước `npm install` hoặc build.

## 7. Bật GitHub Pages

1. Repository → Settings → Pages.
2. Ở Build and deployment chọn Deploy from a branch.
3. Chọn branch `main`, folder `/ (root)`, rồi Save.
4. Mở địa chỉ GitHub Pages sau khi GitHub hoàn tất deploy. Các đường dẫn ứng dụng đều tương đối nên hoạt động tại `https://your-name.github.io/repository-name/`.

## 8. Kiểm tra website

1. Đăng nhập Google.
2. Tạo, sửa và xóa bài tập; thêm link video.
3. Chọn Luyện tập, Giao lưu hoặc Nghỉ tập cho từng ngày; ghi nhật ký, đổi ngày, sửa và xóa nhật ký; xác nhận một ngày/thử thách chỉ có một bản ghi.
4. Kiểm tra lịch 60 ngày, thời lượng, điểm tự đánh giá và biểu đồ.
5. Thử với một tài khoản Google khác để xác nhận chỉ thấy dữ liệu của tài khoản đó.

## Nhập dữ liệu từ phiên bản MVC cũ

Phiên bản MVC trước đây trong project này lưu dữ liệu trình duyệt ở `localStorage` với khóa `pb60` (không có SQLite database cần chuyển). Trên cùng trình duyệt/origin cũ, mở DevTools Console và chạy:

```js
const oldData = localStorage.getItem('pb60');
if (!oldData) throw new Error('Không tìm thấy dữ liệu PICKLEBALL 60 cũ');
const file = new Blob([oldData], { type: 'application/json' });
const link = document.createElement('a');
link.href = URL.createObjectURL(file);
link.download = 'pickleball60-backup.json';
link.click();
setTimeout(() => URL.revokeObjectURL(link.href), 1000);
```

Đăng nhập vào website mới, bấm ⚙ ở cuối sidebar, chọn tệp JSON và nhập. Quá trình nhập tạo/cập nhật bài tập và nhật ký trong tài khoản đang đăng nhập; không xóa dữ liệu khỏi ứng dụng cũ. Nhật ký cũ của minigame được chuyển số lượng sang trường số game.

## Lưu ý vận hành

- Firebase config mẫu cần được thay trước khi đăng nhập hoạt động.
- Service worker cache giao diện; đọc/ghi Firestore và đăng nhập vẫn cần Internet.
- Một nhật ký hỗ trợ tối đa 20 bài tập mỗi ngày, mỗi bài 1–1440 phút, điểm 1–10, tối đa 500 rổ/game và ghi chú giới hạn độ dài.
- Project đã được dọn các file ASP.NET Core MVC và thư mục build .NET; repository hiện chỉ còn website tĩnh cùng cấu hình Firebase và rules.
