# Bài 2: Tái cấu trúc lịch sử commit bằng Interactive Rebase

## Mục tiêu

- Sử dụng **Interactive Rebase** để chỉnh sửa lịch sử commit cục bộ.
- Gộp nhiều commit nhỏ lẻ (**squash**) thành một commit duy nhất có ý nghĩa.
- Đổi tên thông điệp commit (**reword**) và xóa bỏ commit không cần thiết (**drop**).

---

## Các bước thực hiện

### Bước 1: Khởi tạo dự án

```bash
mkdir rebase-demo && cd rebase-demo
git init
```

```
Initialized empty Git repository in /rebase-demo/.git/
```

### Bước 2: Tạo lần lượt 4 commit thử nghiệm

**Commit 1 — Khởi tạo module auth:**

```bash
echo "// Authentication module" > auth.js
echo "function login() { return true; }" >> auth.js
git add .
git commit -m "feat: khoi tao module auth"
```

```
[main (root-commit) a1b2c3d] feat: khoi tao module auth
 1 file changed, 2 insertions(+)
 create mode 100644 auth.js
```

**Commit 2 — Fix typo:**

```bash
sed -i 's/return true/return true;/' auth.js
echo "function logout() { return false; }" >> auth.js
git add .
git commit -m "fix typo"
```

```
[main b2c3d4e] fix typo
 1 file changed, 1 insertion(+)
```

**Commit 3 — Thêm utility functions:**

```bash
echo "function validateToken(token) { return token.length > 0; }" >> auth.js
echo "function hashPassword(pwd) { return btoa(pwd); }" >> auth.js
git add .
git commit -m "adds utility functions"
```

```
[main c3d4e5f] adds utility functions
 1 file changed, 2 insertions(+)
```

**Commit 4 — Tạo file rác temp.txt:**

```bash
echo "debug data - remove later" > temp.txt
git add .
git commit -m "add temp file for debug"
```

```
[main d4e5f6g] add temp file for debug
 1 file changed, 1 insertion(+)
 create mode 100644 temp.txt
```

### Bước 3: Kiểm tra log trước khi rebase

```bash
git log --oneline
```

```
d4e5f6g (HEAD -> main) add temp file for debug
c3d4e5f adds utility functions
b2c3d4e fix typo
a1b2c3d feat: khoi tao module auth
```

→ Có 4 commit lộn xộn, bao gồm commit rác `temp.txt`.

### Bước 4: Chạy Interactive Rebase

Lùi về 4 commit trước:

```bash
git rebase -i HEAD~4
```

Trình soạn thảo mở ra với nội dung:

```
pick a1b2c3d feat: khoi tao module auth
pick b2c3d4e fix typo
pick c3d4e5f adds utility functions
pick d4e5f6g add temp file for debug
```

### Bước 5: Cấu hình các hành động trong Interactive Rebase

Chỉnh sửa nội dung thành:

```
pick a1b2c3d feat: khoi tao module auth
squash b2c3d4e fix typo
squash c3d4e5f adds utility functions
drop d4e5f6g add temp file for debug
```

**Giải thích cấu hình:**

| Commit | Hành động | Lý do |
|--------|-----------|-------|
| `a1b2c3d` feat: khoi tao module auth | `pick` | Giữ nguyên làm commit gốc |
| `b2c3d4e` fix typo | `squash` | Gộp vào commit gốc |
| `c3d4e5f` adds utility functions | `squash` | Gộp vào commit gốc |
| `d4e5f6g` add temp file for debug | `drop` | Xóa bỏ hoàn toàn (file rác) |

Lưu và đóng trình soạn thảo (`Esc` → `:wq` trong vim).

### Bước 6: Soạn lại thông điệp commit gộp

Sau khi lưu cấu hình, Git mở trình soạn thảo yêu cầu nhập thông điệp cho commit gộp:

```
# This is a combination of 3 commits.
# This is the 1st commit message:

feat: khoi tao module auth

# This is the commit message #2:

fix typo

# This is the commit message #3:

adds utility functions
```

**Xóa toàn bộ nội dung cũ** và thay bằng:

```
feat: hoan thien module authentication
```

Lưu và đóng trình soạn thảo.

```
[detached HEAD f7g8h9i] feat: hoan thien module authentication
 Date: Mon Oct 5 19:00:00 2026 +0700
 1 file changed, 5 insertions(+)
 create mode 100644 auth.js
Successfully rebased and updated refs/heads/main.
```

### Bước 7: Kiểm tra kết quả sau rebase

**Kiểm tra git log:**

```bash
git log --oneline
```

```
f7g8h9i (HEAD -> main) feat: hoan thien module authentication
```

→ ✅ Chỉ còn **duy nhất 1 commit** sạch sẽ, rõ ràng!

**Kiểm tra file auth.js vẫn còn đầy đủ:**

```bash
cat auth.js
```

```
// Authentication module
function login() { return true; }
function logout() { return false; }
function validateToken(token) { return token.length > 0; }
function hashPassword(pwd) { return btoa(pwd); }
```

→ ✅ Toàn bộ code từ 3 commit đã được gộp lại nguyên vẹn.

**Kiểm tra file temp.txt đã bị xóa:**

```bash
ls temp.txt
```

```
ls: cannot access 'temp.txt': No such file or directory
```

→ ✅ File rác `temp.txt` đã bị xóa hoàn toàn nhờ lệnh `drop`.

---

## Tổng kết các hành động Interactive Rebase

| Hành động | Từ khóa | Mô tả |
|-----------|---------|-------|
| Giữ nguyên | `pick` (hoặc `p`) | Giữ commit không thay đổi |
| Gộp commit | `squash` (hoặc `s`) | Gộp vào commit trước đó, cho phép sửa message |
| Đổi tên | `reword` (hoặc `r`) | Giữ nội dung commit, chỉ đổi thông điệp |
| Xóa bỏ | `drop` (hoặc `d`) | Xóa commit hoàn toàn khỏi lịch sử |
| Sửa nội dung | `edit` (hoặc `e`) | Dừng lại để sửa nội dung commit |
| Đổi thứ tự | Kéo dòng lên/xuống | Thay đổi thứ tự áp dụng commit |

---

## Kết luận

Interactive Rebase (`git rebase -i`) là công cụ mạnh mẽ giúp:
- **Dọn dẹp lịch sử** trước khi push lên remote.
- **Gộp commit vụn vặt** thành commit có ý nghĩa.
- **Xóa bỏ commit rác** không mong muốn.
- **Giữ cho lịch sử Git** luôn sạch sẽ, chuyên nghiệp.

> ⚠️ **Lưu ý:** Chỉ sử dụng Interactive Rebase cho các commit **chưa push** lên remote. Nếu đã push, việc rebase sẽ thay đổi lịch sử và gây xung đột cho các thành viên khác.
