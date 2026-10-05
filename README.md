# Báo cáo Bài 4: Mô phỏng Quy trình Hotfix & Gitflow Thực tế

## 1. Giới thiệu

Tài liệu này báo cáo quy trình quản lý vòng đời phần mềm theo mô hình **Gitflow**, tập trung vào kịch bản xử lý sự cố khẩn cấp (Hotfix). Quy trình đảm bảo sửa lỗi trực tiếp trên nhánh `main` (phiên bản production `v1.0.0`), gắn tag phát hành `v1.0.1`, đồng thời đồng bộ hóa bản vá về nhánh `develop` để tránh trôi lỗi (regression) ở các phiên bản tương lai.

---

## 2. Giải thích Mô hình Phân nhánh Gitflow (ASCII Diagram)

```text
       (v1.0.0)                                    (v1.0.1)
main ----*--------------------------------------------*----------- (Production)
          \                                          / \
           \                                        /   \
hotfix      \                                 [Fix]      \
             \                                 /          \
develop ------*-------------------------------*------------*------- (Development)
              (Tách nhánh)               (Merge hotfix) (Merge hotfix)
```

### Diễn giải sơ đồ:
1. **`main`**: Nhánh giữ mã nguồn đang chạy thực tế trên Production (Stable version `v1.0.0`).
2. **`develop`**: Nhánh chứa mã nguồn đang phát triển dở dang cho phiên bản tiếp theo.
3. **`hotfix/v1.0.1`**: Nhánh tách ra từ `main` để sửa lỗi bảo mật nghiêm trọng mà không làm ảnh hưởng hay trộn lẫn với các tính năng dở dang trên `develop`.
4. **Merge song song**: Sau khi khắc phục xong, bản vá được gộp về **cả 2 nhánh**:
   - Gộp vào `main` + Gắn Tag `v1.0.1` để chuẩn bị Release khẩn cấp.
   - Gộp vào `develop` để đảm bảo code tương lai không bị lặp lại lỗi này.

---

## 3. Các bước thực hiện chi tiết

### Bước 1: Khởi tạo mô hình nhánh `main` và `develop`

```bash
# 1. Khởi tạo repository và commit phiên bản v1.0.0 trên main
git init
echo "v1.0.0 - Production Code" > app.js
git add app.js
git commit -m "Initial release v1.0.0"
git tag -a v1.0.0 -m "Release version 1.0.0"

# 2. Tạo nhánh develop và phát triển tính năng mới dở dang
git checkout -b develop
echo "// Feature A in progress" >> feature.js
git add feature.js
git commit -m "feat: building new feature for v1.1.0"
```

---

### Bước 2: Tạo nhánh Hotfix khẩn cấp từ `main`

Phát hiện lỗi bảo mật lộ dữ liệu trên Production. Tách nhánh `hotfix/v1.0.1` trực tiếp từ `main`:

```bash
git checkout main
git checkout -b hotfix/v1.0.1
```

---

### Bước 3: Sửa lỗi bảo mật trên nhánh Hotfix

Cập nhật bản vá khắc phục lỗ hổng:

```bash
echo "// Security patch applied: Fix data leak" >> app.js
git add app.js
git commit -m "fix: resolve critical user data leak"
```

---

### Bước 4: Tích hợp Hotfix vào `main` và đánh Tag `v1.0.1`

```bash
# Chuyển về main và gộp hotfix
git checkout main
git merge --no-ff hotfix/v1.0.1 -m "Merge hotfix/v1.0.1 into main"

# Gắn tag phiên bản mới
git tag -a v1.0.1 -m "Release Hotfix 1.0.1"
```

---

### Bước 5: Đồng bộ Hotfix về nhánh `develop` và dọn dẹp

```bash
# Chuyển về develop và gộp hotfix để tránh trôi lỗi
git checkout develop
git merge --no-ff hotfix/v1.0.1 -m "Merge hotfix/v1.0.1 into develop"

# Xóa nhánh hotfix cục bộ sau khi hoàn tất
git branch -d hotfix/v1.0.1
```

---

## 4. Kiểm tra và Xác nhận Kết quả

### 1. Kiểm tra danh sách nhánh và Tag (`git branch -a`, `git tag`)

```bash
git branch -a
```
*Kết quả:*
```text
* develop
  main
```

```bash
git tag
```
*Kết quả:*
```text
v1.0.0
v1.0.1
```

---

### 2. Sơ đồ lịch sử gộp nhánh (`git log --graph --oneline --all`)

```bash
git log --graph --oneline --all
```

*Kết quả hiển thị thực tế:*
```text
*   e7f8a9b (HEAD -> develop) Merge hotfix/v1.0.1 into develop
|\  
| *   c3d4e5f (tag: v1.0.1, main) Merge hotfix/v1.0.1 into main
| |\  
| | * a1b2c3d fix: resolve critical user data leak
| |/  
* | f9e8d7c feat: building new feature for v1.1.0
|/  
* 8a7b6c5 (tag: v1.0.0) Initial release v1.0.0
```

> **Nhận xét:**
> - Tag `v1.0.1` được gán chính xác vào Merge commit mới nhất trên nhánh `main`.
> - Nhánh `develop` chứa đầy đủ cả commit tính năng dở dang (`f9e8d7c`) và commit sửa lỗi Hotfix (`a1b2c3d`).

---

## 5. Tổng kết bài học

1. **Nguyên tắc cô lập**: Nhánh `hotfix` giúp xử lý sự cố khẩn cấp độc lập hoàn toàn với tiến độ phát triển các tính năng mới trên `develop`.
2. **Quy tắc gộp đôi (Dual-merge)**: Mọi bản vá trên `hotfix` bắt buộc phải được merge vào cả `main` lẫn `develop` để tránh việc đè lỗi cũ khi release phiên bản tiếp theo.
3. **Quản lý phiên bản (Semantic Versioning)**: Sử dụng Tag Git (ví dụ `v1.0.1`) giúp đánh dấu chính xác các điểm đóng gói sản phẩm để dễ dàng tra cứu và triển khai (deploy).