# Báo Cáo Thực Hành: Tái Cấu Trúc Lịch Sử Commit Bằng Interactive Rebase

## 1. Giới thiệu & Bối cảnh

Trước khi đẩy code lên remote, nhánh tính năng thường chứa nhiều commit thử nghiệm vụn vặt với thông điệp không rõ ràng. Báo cáo này trình bày cách dùng `git rebase -i` (Interactive Rebase) để:
- **Squash**: Gộp nhiều commit nhỏ thành một commit duy nhất có nghĩa.
- **Reword**: Đặt lại thông điệp commit sau khi gộp.
- **Drop**: Xóa bỏ hoàn toàn một commit không cần thiết.

---

## 2. Các bước thực hiện

### Bước 1: Chuẩn bị — Tạo 4 commit thử nghiệm

Khởi tạo repository và tạo lần lượt 4 commit mô phỏng tình huống thực tế:

```bash
git init
git config user.email "hocvien@example.com"
git config user.name "Hoc Vien"

# Initial commit
echo "# Auth Module" > README.md
git add . && git commit -m "Initial commit"

# Commit 1: Tạo module auth
echo "function login() { }" > auth.js
git add . && git commit -m "feat: khoi tao module auth"

# Commit 2: Sửa lỗi nhỏ
echo "function login() { /* login */ }" > auth.js
git add . && git commit -m "fix typo"

# Commit 3: Thêm hàm tiện ích
printf "function login() { }\nfunction logout() { }" > auth.js
git add . && git commit -m "adds utility functions"

# Commit 4: File rác tạm thời
echo "debug data" > temp.txt
git add . && git commit -m "add temp file for debug"
```

Kiểm tra lịch sử — **5 commit** lộn xộn:

```bash
git log --oneline
```

```
4438447 add temp file for debug
71ae60b adds utility functions
81cb8f8 fix typo
2d87078 feat: khoi tao module auth
d756c3a Initial commit
```

---

### Bước 2: Chạy Interactive Rebase

Lùi về 4 commit gần nhất để vào giao diện tương tác:

```bash
git rebase -i HEAD~4
```

Git mở trình soạn thảo hiển thị danh sách các commit theo thứ tự **từ cũ đến mới**:

```
pick 2d87078 feat: khoi tao module auth
pick 81cb8f8 fix typo
pick 71ae60b adds utility functions
pick 4438447 add temp file for debug

# Rebase d756c3a..4438447 onto d756c3a (4 commands)
#
# Commands:
# p, pick   = use commit
# r, reword = use commit, but edit the commit message
# s, squash = use commit, but meld into previous commit
# d, drop   = remove commit
```

---

### Bước 3: Cấu hình lệnh rebase

Chỉnh sửa file todo theo yêu cầu:

| Commit | Hành động ban đầu | Thay đổi thành | Lý do |
|--------|-------------------|----------------|-------|
| `feat: khoi tao module auth` | `pick` | `pick` | Giữ nguyên, đây là base commit |
| `fix typo` | `pick` | `squash` | Gộp vào commit trước |
| `adds utility functions` | `pick` | `squash` | Gộp vào commit trước |
| `add temp file for debug` | `pick` | `drop` | Xóa bỏ — file rác không cần thiết |

Nội dung file todo sau khi chỉnh sửa:

```
pick 2d87078 feat: khoi tao module auth
squash 81cb8f8 fix typo
squash 71ae60b adds utility functions
drop 4438447 add temp file for debug
```

Lưu và đóng trình soạn thảo.

---

### Bước 4: Chỉnh sửa thông điệp commit gộp

Git mở trình soạn thảo thứ hai để soạn thông điệp cho commit sau khi squash. Mặc định hiển thị tổng hợp 3 commit cũ:

```
# This is a combination of 3 commits.
# This is the 1st commit message:

feat: khoi tao module auth

# This is the commit message #2:

fix typo

# This is the commit message #3:

adds utility functions
```

Xóa toàn bộ, thay bằng thông điệp mới:

```
feat: hoan thien module authentication
```

Lưu và đóng — Git hoàn tất rebase.

---

### Bước 5: Kiểm tra kết quả

**Kiểm tra `git log`** — lịch sử đã gọn sạch:

```bash
git log --oneline
```

```
8212d9d feat: hoan thien module authentication
d756c3a Initial commit
```

**Kiểm tra nội dung `auth.js`** — code đầy đủ từ 3 commit đã gộp:

```bash
cat auth.js
```

```
function login() { }
function logout() { }
```

**Kiểm tra `temp.txt`** — file rác đã bị xóa bỏ hoàn toàn:

```bash
ls temp.txt
```

```
ls: cannot access 'temp.txt': No such file or directory
```

Output của Git khi rebase hoàn tất:

```
Rebasing (2/4)
Rebasing (3/4)
[detached HEAD 8212d9d] feat: hoan thien module authentication
 1 file changed, 2 insertions(+)
 create mode 100644 auth.js
Rebasing (4/4)
Successfully rebased and updated refs/heads/master.
```

---

## 3. Tổng kết

### So sánh trước và sau rebase

| | Trước rebase | Sau rebase |
|-|-------------|-----------|
| Số commit | 5 (Initial + 4 commit) | 2 (Initial + 1 commit sạch) |
| Thông điệp | Lộn xộn, không rõ nghĩa | `feat: hoan thien module authentication` |
| File rác | `temp.txt` còn tồn tại | Đã bị xóa bỏ (drop) |
| Code | Phân tán qua nhiều commit | Gộp đầy đủ trong 1 commit |

### Các lệnh và từ khóa rebase quan trọng

| Từ khóa | Viết tắt | Tác dụng |
|---------|----------|---------|
| `pick` | `p` | Giữ nguyên commit |
| `squash` | `s` | Gộp vào commit liền trước, giữ cả 2 thông điệp |
| `fixup` | `f` | Gộp vào commit liền trước, bỏ thông điệp của commit này |
| `reword` | `r` | Giữ commit nhưng cho phép sửa thông điệp |
| `drop` | `d` | Xóa bỏ hoàn toàn commit |

### Bài học rút ra

- `git rebase -i` chỉ nên dùng trên lịch sử **cục bộ, chưa push** lên remote — tránh gây xung đột cho người khác.
- Squash giúp lịch sử commit trở nên **có ngữ nghĩa và dễ review** hơn.
- Drop loại bỏ hoàn toàn commit lẫn các thay đổi file đi kèm — cẩn thận khi dùng.
