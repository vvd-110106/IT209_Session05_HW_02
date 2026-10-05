# Báo Cáo Thực Hành: Tái Cấu Trúc Lịch Sử Commit Bằng Interactive Rebase

## 1. Giới thiệu & Bối cảnh
Trong quá trình phát triển tính năng, lịch sử Git thường chứa nhiều commit thử nghiệm vụn vặt (`fix typo`, `temp`, `adds utility functions`). Báo cáo này hướng dẫn cách sử dụng lệnh `git rebase -i` (Interactive Rebase) để làm sạch lịch sử commit: gộp các commit nhỏ (squash), đổi tên thông điệp (reword) và xóa commit rác (drop) trước khi đẩy lên remote.

---

## 2. Các bước thực hiện

### Bước 1: Khởi tạo các commit giả lập
Tạo nhánh tính năng và thực hiện 4 commit thử nghiệm theo thứ tự:

```bash
# Commit 1: Khoi tao module auth
echo "// Auth module" > auth.js
git add auth.js
git commit -m "feat: khoi tao module auth"

# Commit 2: Sua loi nho
echo "// Auth module fixed" >> auth.js
git add auth.js
git commit -m "fix typo"

# Commit 3: Them tien ich
echo "// Helper functions" >> auth.js
git add auth.js
git commit -m "adds utility functions"

# Commit 4: File temp rac
echo "debug data" > temp.txt
git add temp.txt
git commit -m "add temp file for debug"