# Phát triển blog trên máy local

Tài liệu này dùng cho repository `pngocthach.github.io`. Site dùng Hugo Modules và được deploy lên GitHub Pages khi push vào nhánh `main`.

## Yêu cầu

- Git
- Go
- Hugo **extended**

Kiểm tra Hugo trước khi bắt đầu:

```sh
hugo version
```

Output phải có chữ `extended`; site dùng Sass nên bản Hugo không có extended có thể không build được.

## Cài đặt

```sh
git clone git@github.com:pngocthach/pngocthach.github.io.git
cd pngocthach.github.io
```

Lần build đầu tiên Hugo tự tải theme được khai báo trong `go.mod`. Nếu cần làm mới dependency module:

```sh
hugo mod tidy
```

Không thêm thư mục `_vendor/` vào Git. Repository dùng version theme được khóa trong `go.mod` và CI tải dependency đó khi build.

## Chạy local

Khởi động development server tại root của repository:

```sh
hugo server --baseURL http://localhost:1313/
```

Mở <http://localhost:1313/>. Hugo theo dõi thay đổi trong `content/`, `layouts/`, `assets/` và `config/`, sau đó tự build lại và reload trình duyệt.

Để xem cả bài đang là draft:

```sh
hugo server -D --baseURL http://localhost:1313/
```

Dừng server bằng `Ctrl+C`.

## Kiểm tra production build

Chạy cùng kiểu build với workflow deploy, nhưng dùng base URL production:

```sh
hugo --gc --minify --baseURL https://pngocthach.github.io/
```

Thư mục `public/` và `resources/` là output/cache local, đã được `.gitignore`; không commit chúng.

## Tạo bài viết mới

Mỗi bài nằm trong một thư mục dưới `content/post/`. Dùng slug chữ thường, ngăn cách bằng dấu gạch ngang.

### Bài tiếng Việt

```sh
hugo new content post/<slug>/index.vi.md
```

Ví dụ:

```sh
hugo new content post/hugo-local-workflow/index.vi.md
```

### Bài song ngữ

Tạo hai file trong cùng thư mục bài viết:

```sh
hugo new content post/<slug>/index.vi.md
hugo new content post/<slug>/index.en.md
```

Mở file vừa tạo và điền front matter. Ví dụ tối thiểu:

```toml
+++
title = 'Tiêu đề bài viết'
date = 2026-07-28T09:00:00+07:00
draft = true
description = 'Mô tả ngắn cho bài viết'
categories = ['Tech']
tags = ['hugo']
+++
```

Viết nội dung Markdown bên dưới front matter. Giữ `draft = true` trong lúc soạn và chạy `hugo server -D` để preview. Đổi thành `draft = false` trước khi xuất bản.

## Xuất bản

1. Chạy production build ở trên.
2. Kiểm tra thay đổi rồi commit đúng các file nguồn:

   ```sh
   git status
   git add content/post/<slug>
   git commit -m "post: add <slug>"
   git push origin main
   ```

3. Workflow `.github/workflows/deploy.yml` sẽ build và deploy lên GitHub Pages.
