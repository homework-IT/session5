# Bài 3: Xử lý xung đột phức tạp trong quá trình Rebase

## Mục tiêu

- Hiểu sự khác biệt giữa giải quyết xung đột khi **Merge** và khi **Rebase**.
- Áp dụng quy trình giải quyết xung đột từng bước trong Rebase.
- Đưa lịch sử nhánh tính năng lên trên đầu nhánh chính một cách thẳng hàng (linear history).

---

## Các bước thực hiện

### Bước 1: Khởi tạo dự án và commit ban đầu trên `main`

```bash
mkdir rebase-conflict-demo && cd rebase-conflict-demo
git init
```

Tạo file `config.json` với nội dung ban đầu:

```bash
cat > config.json << 'EOF'
{
  "port": 8080,
  "debug": false
}
EOF

git add config.json
git commit -m "init config"
```

```
[main (root-commit) aaa1111] init config
 1 file changed, 4 insertions(+)
 create mode 100644 config.json
```

### Bước 2: Tạo nhánh `feature-api` và thêm 2 commit

```bash
git checkout -b feature-api
```

```
Switched to a new branch 'feature-api'
```

**Commit 1 trên feature-api — Đổi port thành 9000:**

```bash
cat > config.json << 'EOF'
{
  "port": 9000,
  "debug": false
}
EOF

git add config.json
git commit -m "feat: change port"
```

```
[feature-api bbb2222] feat: change port
 1 file changed, 1 insertion(+), 1 deletion(-)
```

**Commit 2 trên feature-api — Bật debug:**

```bash
cat > config.json << 'EOF'
{
  "port": 9000,
  "debug": true
}
EOF

git add config.json
git commit -m "feat: enable debug"
```

```
[feature-api ccc3333] feat: enable debug
 1 file changed, 1 insertion(+), 1 deletion(-)
```

### Bước 3: Quay lại `main` và thêm 2 commit (mô phỏng thay đổi từ đồng nghiệp)

```bash
git checkout main
```

**Commit 1 trên main — Đổi port thành 8081:**

```bash
cat > config.json << 'EOF'
{
  "port": 8081,
  "debug": false
}
EOF

git add config.json
git commit -m "update port on main"
```

```
[main ddd4444] update port on main
 1 file changed, 1 insertion(+), 1 deletion(-)
```

**Commit 2 trên main — Thêm env config:**

```bash
cat > config.json << 'EOF'
{
  "port": 8081,
  "debug": false,
  "env": "production"
}
EOF

git add config.json
git commit -m "add env config"
```

```
[main eee5555] add env config
 1 file changed, 2 insertions(+), 1 deletion(-)
```

### Bước 4: Kiểm tra lịch sử trước khi rebase

```bash
git log --graph --oneline --all
```

```
* eee5555 (HEAD -> main) add env config
* ddd4444 update port on main
| * ccc3333 (feature-api) feat: enable debug
| * bbb2222 feat: change port
|/
* aaa1111 init config
```

→ Hai nhánh đã phân tách, cùng sửa `config.json`.

### Bước 5: Chuyển sang `feature-api` và bắt đầu Rebase

```bash
git checkout feature-api
git rebase main
```

```
Auto-merging config.json
CONFLICT (content): Merge conflict in config.json
error: could not apply bbb2222... feat: change port
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
Could not apply bbb2222... feat: change port
```

→ ❌ **Xung đột lần 1** xảy ra tại commit `feat: change port`.

### Bước 6: Giải quyết xung đột lần 1

**Mở file `config.json` để xem conflict:**

```bash
cat config.json
```

```
{
<<<<<<< HEAD
  "port": 8081,
  "debug": false,
  "env": "production"
=======
  "port": 9000,
  "debug": false
>>>>>>> bbb2222 (feat: change port)
}
```

**Giải thích conflict marker:**

| Phần | Nội dung | Nguồn |
|------|----------|-------|
| `<<<<<<< HEAD` | Code từ `main` (đích rebase) | `port: 8081`, có thêm `env` |
| `=======` | Ranh giới phân cách | — |
| `>>>>>>> bbb2222` | Code từ `feature-api` commit đang apply | `port: 9000` |

**Quyết định giải quyết:** Giữ port `9000` từ feature, đồng thời giữ `env` từ main:

```bash
cat > config.json << 'EOF'
{
  "port": 9000,
  "debug": false,
  "env": "production"
}
EOF
```

**Đánh dấu đã giải quyết và tiếp tục rebase:**

```bash
git add config.json
git rebase --continue
```

```
[detached HEAD fff6666] feat: change port
 1 file changed, 1 insertion(+), 1 deletion(-)
Auto-merging config.json
CONFLICT (content): Merge conflict in config.json
error: could not apply ccc3333... feat: enable debug
Could not apply ccc3333... feat: enable debug
```

→ ❌ **Xung đột lần 2** xảy ra tại commit `feat: enable debug`.

### Bước 7: Giải quyết xung đột lần 2

**Xem nội dung conflict:**

```bash
cat config.json
```

```
{
  "port": 9000,
<<<<<<< HEAD
  "debug": false,
  "env": "production"
=======
  "debug": true
>>>>>>> ccc3333 (feat: enable debug)
}
```

**Quyết định giải quyết:** Giữ `debug: true` từ feature, giữ `env` từ main:

```bash
cat > config.json << 'EOF'
{
  "port": 9000,
  "debug": true,
  "env": "production"
}
EOF
```

**Đánh dấu và tiếp tục:**

```bash
git add config.json
git rebase --continue
```

```
[detached HEAD ggg7777] feat: enable debug
 1 file changed, 1 insertion(+), 1 deletion(-)
Successfully rebased and updated refs/heads/feature-api.
```

→ ✅ **Rebase hoàn tất thành công!**

### Bước 8: Kiểm tra kết quả

**Kiểm tra trạng thái:**

```bash
git status
```

```
On branch feature-api
nothing to commit, working tree clean
```

**Kiểm tra lịch sử:**

```bash
git log --graph --oneline --all
```

```
* ggg7777 (HEAD -> feature-api) feat: enable debug
* fff6666 feat: change port
* eee5555 (main) add env config
* ddd4444 update port on main
* aaa1111 init config
```

→ ✅ **Lịch sử thẳng tắp (linear)!** Không có merge commit phụ. Các commit của `feature-api` nằm nối tiếp ngay sau commit mới nhất của `main`.

**Kiểm tra nội dung file cuối cùng:**

```bash
cat config.json
```

```
{
  "port": 9000,
  "debug": true,
  "env": "production"
}
```

→ ✅ File chứa đầy đủ cả thay đổi từ `main` (`env`) và `feature-api` (`port`, `debug`).

---

## So sánh Merge vs Rebase khi xử lý xung đột

| Tiêu chí | Merge | Rebase |
|----------|-------|--------|
| **Lịch sử** | Có merge commit, dạng cây phân nhánh | Thẳng tắp, tuyến tính |
| **Xử lý conflict** | Giải quyết 1 lần cho toàn bộ | Giải quyết từng commit một |
| **Kết quả** | `* Merge branch 'feature'` xuất hiện | Không có merge commit phụ |
| **Khi nào dùng** | Merge vào nhánh chung (main, develop) | Cập nhật nhánh cá nhân với nhánh chính |

---

## Các lệnh hữu ích trong quá trình Rebase

| Lệnh | Mục đích |
|-------|----------|
| `git rebase main` | Bắt đầu rebase lên main |
| `git status` | Xem file nào đang conflict |
| `git add <file>` | Đánh dấu đã giải quyết conflict |
| `git rebase --continue` | Tiếp tục rebase sau khi resolve |
| `git rebase --abort` | Hủy bỏ rebase, quay lại trạng thái ban đầu |
| `git rebase --skip` | Bỏ qua commit đang conflict |

---

## Kết luận

Rebase là phương pháp mạnh mẽ để giữ lịch sử Git sạch sẽ và dễ đọc. Khi gặp xung đột trong quá trình rebase:
1. **Đọc kỹ conflict marker** để hiểu nguồn gốc xung đột.
2. **Quyết định giữ code nào** dựa trên yêu cầu nghiệp vụ.
3. **Giải quyết từng commit một** theo thứ tự rebase.
4. Luôn **kiểm tra kết quả** bằng `git log --graph --oneline` sau khi hoàn tất.
