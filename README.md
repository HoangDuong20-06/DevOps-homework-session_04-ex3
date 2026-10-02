# Bài 3: Cấu hình xác thực SSH và Đẩy dự án lên GitHub

## 1. Mục tiêu
- Tạo cặp khóa SSH thuật toán Ed25519 để xác thực với GitHub (không dùng HTTPS/password/token).
- Liên kết repository cục bộ `ss4/bai3` với GitHub qua giao thức SSH.
- Push mã nguồn + lịch sử commit lên GitHub thành công.

## 2. Môi trường
- OS: Windows 11 (chạy lệnh trong Git Bash)
- Git: `git --version` (đã cài)
- Thư mục dự án: `ss4/bai3`
- Repo GitHub: https://github.com/HoangDuong20-06/DevOps-homework-session_04-ex3
- Remote SSH: `git@github.com:HoangDuong20-06/DevOps-homework-session_04-ex3.git`

## 3. Quá trình thực hiện

### Bước 1 — Sinh cặp khóa Ed25519
```bash
ssh-keygen -t ed25519 -C "hoangduong2062006@gmail.com" -f ~/.ssh/id_ed25519
```
- Nhấn Enter 2 lần để chấp nhận đường dẫn mặc định và bỏ qua passphrase (hoặc đặt passphrase nếu muốn).
- Kết quả sinh ra 2 file:
  - `~/.ssh/id_ed25519` (private key — TUYỆT ĐỐI KHÔNG nộp, không public)
  - `~/.ssh/id_ed25519.pub` (public key — đem lên GitHub). Nội dung dạng:
    ```
    ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGjmqGt/x60ZFtwJGWsaPQA0YsBZQ4EF/Qdv1f0eLd4z hoangduong2062006@gmail.com
    ```

### Bước 2 — Nạp key vào ssh-agent
```bash
eval "$(ssh-agent -s)"
# Agent pid 1336 (ví dụ)

ssh-add ~/.ssh/id_ed25519
ssh-add -l
# Kết quả mong đợi: 256 SHA256:... hoangduong2062006@gmail.com (ED25519)
```
> Lỗi gặp phải: `Could not open a connection to your authentication agent.`
> Nguyên nhân: chưa chạy `ssh-agent`. Fix bằng `eval "$(ssh-agent -s)"` trước khi `ssh-add`.
>
> Lỗi gặp phải: `/c/Users/admin/.ssh/id_ed25519: No such file or directory`
> Nguyên nhân: chưa chạy `ssh-keygen`. Fix bằng cách chạy lại Bước 1.

### Bước 3 — Thêm public key lên GitHub
1. Lấy key: `cat ~/.ssh/id_ed25519.pub` (copy full 1 dòng từ `ssh-ed25519` đến hết email).
2. Lên GitHub: Avatar > Settings > SSH and GPG keys > New SSH key.
3. Title: `laptop-win`, Key: dán dòng trên > Add SSH key.

### Bước 4 — Kiểm tra kết nối SSH (lệnh kiểm tra của bài)
```bash
ssh -T git@github.com
```
Kết quả thực tế đạt được:
```
Hi HoangDuong20-06! You've successfully authenticated, but GitHub does not provide shell access.
```
=> Xác thực thành công, đúng tài khoản GitHub.

### Bước 5 — Liên kết repo cục bộ với GitHub qua SSH và push
```bash
cd "C:/Users/admin/IdeaProjects/Devops/ss4/bai3"
git init
git add .
git commit -m "init bai3"
git branch -M main
# (repo này đang dùng nhánh master, nếu dùng main thì giữ nguyên, nếu dùng master thì: git branch -M master)
git remote add origin git@github.com:HoangDuong20-06/DevOps-homework-session_04-ex3.git
git remote -v
git push -u origin master
# (hoặc: git push -u origin main — tùy nhánh mặc định)
```

Kiểm tra remote (lệnh kiểm tra của bài):
```
origin  git@github.com:HoangDuong20-06/DevOps-homework-session_04-ex3.git (fetch)
origin  git@github.com:HoangDuong20-06/DevOps-homework-session_04-ex3.git (push)
```
=> Đúng dạng SSH `git@github.com:username/repository.git`, không dùng HTTPS.

## 4. Kết quả
- [x] `ssh -T git@github.com` trả về `Hi HoangDuong20-06! You've successfully authenticated...`
- [x] `git remote -v` hiện URL SSH cho cả fetch và push.
- [x] Code đã push lên: https://github.com/HoangDuong20-06/DevOps-homework-session_04-ex3
- [x] Chỉ nộp báo cáo này, không nộp file private key `id_ed25519`.

## 5. Lưu ý bảo mật
- Không commit, không upload `~/.ssh/id_ed25519` lên GitHub.
- File `.gitignore` đã loại trừ key/secret nếu có.
- Nếu lộ private key: vào GitHub xóa SSH key cũ, tạo lại cặp key mới bằng `ssh-keygen -t ed25519`.
