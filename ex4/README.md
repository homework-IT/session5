# Bài 4: Mô phỏng quy trình Hotfix & Gitflow thực tế

## Mục tiêu

- Áp dụng mô hình phân nhánh **Gitflow** chuẩn trong quản lý vòng đời phần mềm.
- Thực hành tạo, kiểm thử và tích hợp nhánh **Hotfix**.
- Đồng bộ hóa bản vá nóng về nhánh phát triển (`develop`) để tránh trôi lỗi.

---

## Sơ đồ phân nhánh Gitflow

```
main:     ──●──────────────●──────────── (v1.0.0)──●──── (v1.0.1)
             \              \                      /  \
hotfix:       \              \        ●───●───────    |
               \              \      /                |
develop:        ●──────●───●───●────────────────────●─── (merge hotfix)
```

**Giải thích:**
- `main`: Nhánh production ổn định, luôn có tag phiên bản.
- `develop`: Nhánh phát triển tính năng mới.
- `hotfix/v1.0.1`: Nhánh sửa lỗi khẩn cấp, tách từ `main`, gộp ngược vào cả `main` và `develop`.

---

## Các bước thực hiện

### Bước 1: Khởi tạo dự án và thiết lập nhánh `main`

```bash
mkdir gitflow-demo && cd gitflow-demo
git init
```

Tạo file ứng dụng mô phỏng sản phẩm v1.0.0:

```bash
cat > app.js << 'EOF'
// App v1.0.0 - Production
function getUserData(userId) {
  // LỖI BẢO MẬT: trả về toàn bộ dữ liệu kể cả password
  return database.query("SELECT * FROM users WHERE id = " + userId);
}

function startServer() {
  console.log("Server running on port 3000");
}

module.exports = { getUserData, startServer };
EOF

git add .
git commit -m "Release v1.0.0 - Initial production"
git tag -a v1.0.0 -m "Version 1.0.0 - Initial Release"
```

```
[main (root-commit) aaa1111] Release v1.0.0 - Initial production
 1 file changed, 11 insertions(+)
 create mode 100644 app.js
```

### Bước 2: Tạo nhánh `develop` và phát triển tính năng mới

```bash
git checkout -b develop
```

Thêm tính năng mới đang phát triển dở:

```bash
cat > feature-payment.js << 'EOF'
// Feature: Payment Module (WIP - đang phát triển)
function processPayment(amount) {
  // TODO: Tích hợp cổng thanh toán
  console.log("Processing payment: " + amount);
}
EOF

git add .
git commit -m "WIP: bat dau phat trien module thanh toan"
```

```
[develop bbb2222] WIP: bat dau phat trien module thanh toan
 1 file changed, 6 insertions(+)
 create mode 100644 feature-payment.js
```

Thêm thêm code đang dở:

```bash
cat > feature-notification.js << 'EOF'
// Feature: Notification (WIP)
function sendEmail(to, subject) {
  // TODO: Cấu hình SMTP
  console.log("Sending email to: " + to);
}
EOF

git add .
git commit -m "WIP: bat dau module thong bao"
```

```
[develop ccc3333] WIP: bat dau module thong bao
 1 file changed, 6 insertions(+)
 create mode 100644 feature-notification.js
```

→ Nhánh `develop` đang phát triển dở dang, **không thể deploy ngay**.

### Bước 3: Phát hiện lỗi bảo mật nghiêm trọng trên production!

> ⚠️ **Tình huống khẩn cấp:** Hệ thống trên `main` (v1.0.0) bị phát hiện lỗi lộ dữ liệu người dùng — hàm `getUserData()` trả về cả password!

### Bước 4: Tạo nhánh Hotfix từ `main`

```bash
git checkout main
git checkout -b hotfix/v1.0.1
```

```
Switched to a new branch 'hotfix/v1.0.1'
```

### Bước 5: Sửa lỗi bảo mật trên nhánh Hotfix

```bash
cat > app.js << 'EOF'
// App v1.0.1 - Security Patch
function getUserData(userId) {
  // ĐÃ SỬA: Loại bỏ trường password, sử dụng parameterized query
  const query = "SELECT id, name, email FROM users WHERE id = ?";
  return database.query(query, [userId]);
}

function startServer() {
  console.log("Server running on port 3000");
}

module.exports = { getUserData, startServer };
EOF

git add .
git commit -m "hotfix: vá lỗi bảo mật lộ dữ liệu người dùng"
```

```
[hotfix/v1.0.1 ddd4444] hotfix: vá lỗi bảo mật lộ dữ liệu người dùng
 1 file changed, 4 insertions(+), 2 deletions(-)
```

### Bước 6: Gộp Hotfix vào `main` và tạo tag phiên bản

```bash
git checkout main
git merge hotfix/v1.0.1
```

```
Updating aaa1111..ddd4444
Fast-forward
 app.js | 6 ++++--
 1 file changed, 4 insertions(+), 2 deletions(-)
```

Tạo tag phiên bản mới:

```bash
git tag -a v1.0.1 -m "Release Hotfix v1.0.1 - Vá lỗi bảo mật lộ dữ liệu"
```

**Kiểm tra tag:**

```bash
git tag
```

```
v1.0.0
v1.0.1
```

→ ✅ Tag `v1.0.1` đã được tạo trên `main`.

### Bước 7: Gộp ngược Hotfix vào `develop`

Đây là bước **bắt buộc** trong Gitflow — đảm bảo nhánh phát triển cũng được sửa lỗi:

```bash
git checkout develop
git merge hotfix/v1.0.1
```

```
Auto-merging app.js
Merge made by the 'ort' strategy.
 app.js | 6 ++++--
 1 file changed, 4 insertions(+), 2 deletions(-)
```

→ ✅ Code sửa lỗi đã được đồng bộ vào `develop`.

### Bước 8: Xóa nhánh Hotfix (dọn dẹp)

```bash
git branch -d hotfix/v1.0.1
```

```
Deleted branch hotfix/v1.0.1 (was ddd4444).
```

### Bước 9: Kiểm tra kết quả cuối cùng

**Kiểm tra danh sách nhánh và tag:**

```bash
git branch -a
```

```
  develop
* main
```

```bash
git tag
```

```
v1.0.0
v1.0.1
```

**Kiểm tra sơ đồ lịch sử đầy đủ:**

```bash
git log --graph --oneline --all
```

```
*   eee5555 (develop) Merge branch 'hotfix/v1.0.1' into develop
|\
| * ddd4444 (HEAD -> main, tag: v1.0.1) hotfix: vá lỗi bảo mật lộ dữ liệu người dùng
* | ccc3333 WIP: bat dau module thong bao
* | bbb2222 WIP: bat dau phat trien module thanh toan
|/
* aaa1111 (tag: v1.0.0) Release v1.0.0 - Initial production
```

→ ✅ **Kết quả đúng yêu cầu:**
- Tag `v1.0.1` chỉ vào commit sửa lỗi trên `main`.
- Nhánh `develop` chứa commit sửa lỗi từ hotfix (thông qua merge commit).
- Nhánh `hotfix/v1.0.1` đã được dọn dẹp.

---

## Sơ đồ ASCII — Luồng Gitflow Hotfix

```
        main                hotfix/v1.0.1            develop
         |                                              |
    [v1.0.0]                                            |
         |                                         [WIP payment]
         |                                              |
         |                                         [WIP notify]
         |                                              |
         |─────────> [tạo hotfix] ──────>               |
         |           [sửa lỗi bảo mật]                  |
         |                |                             |
         |<── merge ──────┘                             |
    [v1.0.1]                                            |
         |                                              |
         |                      merge hotfix ──────────>|
         |                                         [merge commit]
         |                                              |
         ▼                                              ▼
    (Production                                   (Development
     sẵn sàng                                     tiếp tục với
     deploy)                                      bản vá đã có)
```

---

## Tổng kết quy trình Gitflow Hotfix

| Bước | Lệnh | Mô tả |
|------|-------|-------|
| 1 | `git checkout main` | Chuyển về nhánh production |
| 2 | `git checkout -b hotfix/v1.0.1` | Tạo nhánh hotfix từ main |
| 3 | *(sửa code)* + `git commit` | Thực hiện sửa lỗi |
| 4 | `git checkout main` → `git merge hotfix/v1.0.1` | Gộp hotfix vào main |
| 5 | `git tag -a v1.0.1 -m "..."` | Tạo tag phiên bản mới |
| 6 | `git checkout develop` → `git merge hotfix/v1.0.1` | Đồng bộ hotfix vào develop |
| 7 | `git branch -d hotfix/v1.0.1` | Xóa nhánh hotfix |

---

## Kết luận

Quy trình **Gitflow Hotfix** đảm bảo:
- 🔒 **Lỗi bảo mật** được vá nhanh chóng trên production mà không ảnh hưởng code đang phát triển.
- 🔄 **Đồng bộ ngược** về `develop` tránh tình trạng lỗi cũ tái xuất hiện trong phiên bản tương lai.
- 🏷️ **Tag phiên bản** giúp theo dõi rõ ràng từng bản release.
- 🧹 **Dọn dẹp nhánh** giữ repository gọn gàng, dễ quản lý.
