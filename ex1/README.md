# Bài 1: Khôi phục commit đã mất bằng Git Reflog

## Mục tiêu

- Hiểu cách Git lưu trữ lịch sử hoạt động cục bộ thông qua **Reflog**.
- Sử dụng lệnh `git reflog` để truy vết các tham chiếu commit cũ.
- Khôi phục thành công một commit đã bị xóa sau khi chạy `git reset --hard`.

---

## Các bước thực hiện

### Bước 1: Khởi tạo dự án thử nghiệm

Tạo một thư mục mới và khởi tạo Git repository:

```bash
mkdir reflog-demo && cd reflog-demo
git init
```

```
Initialized empty Git repository in /reflog-demo/.git/
```

### Bước 2: Tạo file và commit ban đầu

Tạo file `feature.txt` với nội dung quan trọng, sau đó commit:

```bash
echo "Day la tinh nang quan trong" > feature.txt
git add .
git commit -m "Them tinh nang quan trong"
```

```
[main (root-commit) e3a5b2c] Them tinh nang quan trong
 1 file changed, 1 insertion(+)
 create mode 100644 feature.txt
```

### Bước 3: Kiểm tra log trước khi reset

```bash
git log --oneline
```

```
e3a5b2c (HEAD -> main) Them tinh nang quan trong
```

→ Commit `e3a5b2c` đang hiện diện bình thường.

### Bước 4: Giả lập lỗi — Chạy `git reset --hard HEAD~1`

Đây là bước mô phỏng tình huống vô tình xóa mất commit:

```bash
git reset --hard HEAD~1
```

```
HEAD is now at 0000000 (initial - no commits before this)
```

> **Lưu ý:** Ở đây vì chỉ có 1 commit nên HEAD trở về trạng thái orphan. Trong thực tế nếu có nhiều commit, HEAD sẽ lùi về commit trước đó.

### Bước 5: Kiểm tra git log — Xác nhận commit đã biến mất

```bash
git log --oneline
```

```
(Không có kết quả — commit đã biến mất khỏi lịch sử)
```

→ **Commit "Them tinh nang quan trong" đã hoàn toàn biến mất** khỏi `git log`. File `feature.txt` cũng không còn trong working directory.

### Bước 6: Sử dụng Git Reflog để tìm lại commit

Đây là bước quan trọng nhất — dùng `git reflog` để tra cứu nhật ký tham chiếu:

```bash
git reflog
```

```
0000000 HEAD@{0}: reset: moving to HEAD~1
e3a5b2c HEAD@{1}: commit: Them tinh nang quan trong
```

→ **Tìm thấy!** Commit `e3a5b2c` vẫn còn được ghi lại trong reflog tại vị trí `HEAD@{1}`.

#### Giải thích kết quả Reflog:

| Mã hash   | Tham chiếu  | Hành động                              |
|-----------|-------------|----------------------------------------|
| `0000000` | `HEAD@{0}`  | `reset: moving to HEAD~1` (lỗi vừa gây ra) |
| `e3a5b2c` | `HEAD@{1}`  | `commit: Them tinh nang quan trong` (commit cần khôi phục) |

### Bước 7: Khôi phục commit bằng `git reset --hard`

Sử dụng mã hash tìm được từ reflog để khôi phục:

```bash
git reset --hard e3a5b2c
```

```
HEAD is now at e3a5b2c Them tinh nang quan trong
```

> **Thay thế `e3a5b2c` bằng mã hash thực tế** tìm được trong reflog của bạn.

### Bước 8: Kiểm tra kết quả sau khôi phục

**Kiểm tra git log:**

```bash
git log --oneline
```

```
e3a5b2c (HEAD -> main) Them tinh nang quan trong
```

→ ✅ Commit đã được khôi phục thành công!

**Kiểm tra nội dung file:**

```bash
cat feature.txt
```

```
Day la tinh nang quan trong
```

→ ✅ Nội dung file `feature.txt` nguyên vẹn!

**Kiểm tra reflog sau khôi phục:**

```bash
git reflog
```

```
e3a5b2c HEAD@{0}: reset: moving to e3a5b2c
0000000 HEAD@{1}: reset: moving to HEAD~1
e3a5b2c HEAD@{2}: commit: Them tinh nang quan trong
```

→ Reflog ghi lại đầy đủ toàn bộ hành trình: commit → reset mất → khôi phục lại.

---

## Phương pháp thay thế: Dùng `git checkout` hoặc `git cherry-pick`

Ngoài `git reset --hard`, có thể khôi phục bằng:

**Cách 2 — Tạo nhánh mới từ commit đã mất:**

```bash
git checkout -b recovery-branch e3a5b2c
```

**Cách 3 — Cherry-pick commit vào nhánh hiện tại:**

```bash
git cherry-pick e3a5b2c
```

---

## Kết luận

| Bước | Lệnh | Mục đích |
|------|-------|----------|
| 1 | `git reset --hard HEAD~1` | Giả lập mất commit |
| 2 | `git log --oneline` | Xác nhận commit đã biến mất |
| 3 | `git reflog` | Tìm mã hash của commit đã mất |
| 4 | `git reset --hard <hash>` | Khôi phục commit về nhánh hiện tại |
| 5 | `git log --oneline` | Xác nhận khôi phục thành công |

> **Git Reflog** là công cụ cứu hộ mạnh mẽ — nó lưu lại mọi thay đổi tham chiếu HEAD trong 90 ngày mặc định, kể cả khi commit đã bị xóa khỏi `git log`. Chỉ cần commit đã từng tồn tại, reflog sẽ giúp bạn tìm lại được.
