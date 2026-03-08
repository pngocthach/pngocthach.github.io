+++
title = 'Cách tôi setup blog cá nhân nhanh, đẹp, miễn phí'
date = 2026-03-08T19:25:00+07:00
draft = false
description = "Hướng dẫn nhanh thiết lập blog cá nhân với Hugo, GitHub Pages và Waline"
image = ""
categories = [
    "Tech",
    "Blog"
]
tags = [
    "hugo",
    "github pages",
    "waline",
    "comment system",
    "tutorial",
    "guide"
]
+++

Nếu bạn muốn có một blog cá nhân **nhanh, đẹp, miễn phí hoàn toàn**, không quảng cáo, load nhanh, và có phần comment với đầy đủ tính năng (emoji, reaction, anonymous...), thì combo này là lựa chọn tuyệt vời năm 2026:

- **Hugo** (static site generator siêu nhanh)  
- **Theme Stack** (thiết kế hiện đại, tối giản, dark mode đẹp)  
- **GitHub Pages** (host miễn phí)  
- **Waline** (comment system open-source, self-host trên Vercel + Neon Postgres, hỗ trợ anonymous, comment hiện ngay)

## Yêu cầu trước khi bắt đầu

- Tài khoản GitHub (public repo)  
- Tài khoản Vercel (miễn phí, đăng nhập bằng GitHub)  
- Tài khoản Neon (miễn phí, serverless Postgres)  
- Máy tính có Git + Hugo installed (extended version khuyến nghị)

## Bước 1: Clone template Hugo Stack và setup site cơ bản

1. Truy cập repo starter chính thức:  
   <https://github.com/CaiJimmy/hugo-theme-stack-starter>

2. Click **Use this template** → **Create a new repository**  
   - Đặt tên repo:  
     - Nếu muốn domain root: `username.github.io` (ví dụ: pngocthach.github.io)  
     - Nếu muốn subpath: bất kỳ tên nào (ví dụ: my-blog) → domain sẽ là `username.github.io/my-blog`

3. Clone repo về máy:

   ```bash {linenos=false}
   git clone https://github.com/username/username.github.io.git
   cd username.github.io
   ```

4. Cài Hugo nếu chưa có:
   - macOS: `brew install hugo`  
   - Windows: dùng Scoop/Chocolatey hoặc tải binary extended từ <https://github.com/gohugoio/hugo/releases>  
   - Linux (tùy distro): `sudo apt install hugo` (hoặc extended)

   Kiểm tra: `hugo version` (nên ≥ 0.120)

5. Chạy thử local:

   ```bash {linenos=false}13
   hugo server
   ```

   Mở <http://localhost:1313> → thấy trang demo Stack là OK.

6. Deploy lên GitHub Pages:
   - Vào repo Settings → Pages  
   - Source: Deploy from a branch → Branch: main → Folder: / (root) → Save  
   - Template đã có GitHub Actions workflow sẵn (`.github/workflows/hugo.yml`), chỉ cần push là tự build.

   Trang live sau 1-3 phút: <https://username.github.io> (hoặc /my-blog)

## Bước 2: Setup Waline comment backend (Vercel + Neon Postgres)

Waline là hệ thống comment open-source, nhẹ, hỗ trợ anonymous (comment không cần login), emoji/reaction đẹp.

1. Deploy Waline trên Vercel (miễn phí):
   - Truy cập: <https://waline.js.org/en/guide/deploy/vercel.html>  
   - Click nút **Deploy** (màu xanh)  
   - Đăng nhập Vercel bằng GitHub → Đặt tên project (ví dụ: `my-waline-backend`) → Create  
   - Chờ deploy xong (1-2 phút) → Lấy URL backend: <https://my-waline-backend.vercel.app>

2. Tạo database Neon Postgres (free tier):
   - Trong Vercel Dashboard → Tab **Storage** → **Create Database** → Chọn **Neon**  
   - Region: **Singapore (aws-ap-southeast-1)** (latency thấp nhất từ VN)  
   - Tạo DB → Open in Neon console  
   - Trong Neon SQL Editor → Paste script init từ:  
     <https://github.com/walinejs/waline/blob/main/assets/waline.pgsql>  
     → Run để tạo tables

3. Kết nối Neon với Vercel:
   - Trong Vercel → Storage → Chọn Neon DB vừa tạo → **Connect**  
   - Tick **tất cả Environments**: Development, Preview, Production  
   - Tick **Preview** cho branching (bỏ Production nếu không cần)  
   - Custom Prefix: Để **trống**  
   - Connect → Vercel tự inject env vars

4. Tạo admin account:
   - Truy cập: <https://my-waline-backend.vercel.app/ui/register>  
   - Đăng ký user đầu tiên → thành admin  
   - Login tại `/ui` để quản lý comment sau này (nếu cần review spam)

## Bước 3: Tích hợp Waline vào Hugo Stack

1. Chỉnh file config (`config/_default/params.toml`):

   ```toml {linenos=false}
   [comments]
     enabled = true
     provider = "waline"

     [comments.waline]
       serverURL = "https://waline-backend-iota.vercel.app"  # Thay bằng URL Vercel của bạn
       lang = "vi"
       reaction = true
       emoji = [
         "https://unpkg.com/@waline/emojis@1.0.1/tw-emoji",
         "https://unpkg.com/@waline/emojis@1.0.1/weibo"
       ]
   ```

2. **Override để comment hiển thị full URL (dễ quản lý trong admin)**:
   - Quá trình deploy bạn có thể gặp tình trạng comment lưu trên Vercel chỉ nhận dạng bằng đường dẫn path relative `\p\bai-viet` thay vì `https:\\domain\p\bai-viet`. Khiến cực khó phân biệt bài viết.
   - Để xử lý việc này, hãy tạo file: `layouts/partials/comments/provider/waline.html` đè lên file của theme.

   - Nội dung file:

     ```html
     <script src='//unpkg.com/@waline/client@v2/dist/waline.js'></script>
     <link href='//unpkg.com/@waline/client@v2/dist/waline.css' rel='stylesheet'/>
     <div id="waline" class="waline-container"></div>
     <style>
         .waline-container {
             background-color: var(--card-background);
             border-radius: var(--card-border-radius);
             box-shadow: var(--shadow-l1);
             padding: var(--card-padding);
             --waline-font-size: var(--article-font-size);
         }
         .waline-container .wl-count {
             color: var(--card-text-color-main);
         }
     </style>

     {{- $permalink := .Permalink -}}
     {{- with .Site.Params.comments.waline -}}
     {{- $config := dict "el" "#waline" "dark" `html[data-scheme="dark"]` "path" $permalink -}}
     {{- $replaceKeys := dict "serverurl" "serverURL" "requiredmeta" "requiredMeta" "wordlimit" "wordLimit" "pagesize" "pageSize" "imageuploader" "imageUploader" "texrenderer" "texRenderer" "turnstilekey" "turnstileKey" -}}

     {{- range $key, $val := . -}}
         {{- if ne $val nil -}}  
             {{- $replaceKey := index $replaceKeys $key -}}
             {{- $k := default $key $replaceKey -}}

             {{- $config = merge $config (dict $k $val) -}}
         {{- end -}}
     {{- end -}}

     <script id="waline-config" type="application/json">
         {{ $config | jsonify | safeJS }}
     </script>
     <script>
         /// Waline client configuration see: https://waline.js.org/en/reference/client.html
         const walineConfig = JSON.parse(document.getElementById('waline-config').textContent);
         Waline.init(walineConfig);
     </script>
     {{- end -}}
     ```

   - `.Permalink` sẽ tự động lấy full URL (bao gồm domain), giúp admin Waline hiển thị comment dưới dạng `https://yourblog.com/p/bai-viet/` thay vì path tương đối → cực kì dễ nhận biết blog nào.

## Bước 4: Cách viết bài mới và cập nhật blog

Sau khi đã setup xong "bộ khung", việc viết bài hằng ngày sẽ cực kỳ đơn giản:

1. **Tạo bài viết mới**: Bạn sử dụng terminal và chạy lệnh:

   ```bash {linenos=false}
   hugo new content post/ten-bai-viet-cua-ban/index.md
   ```

   Lệnh này sẽ tạo ra một thư mục mới trong `content/post/` kèm theo file `index.md`. Đây là nơi bạn sẽ viết nội dung bài viết.

2. **Soạn thảo nội dung**: Mở file `index.md` vừa tạo. Bạn sẽ thấy phần đầu (nằm giữa cặp `+++`) gọi là **Front Matter**—nơi chứa thông tin meta như tiêu đề, ngày tháng, mô tả, ảnh bìa... Phía dưới đó là nơi bạn viết nội dung bằng cú pháp Markdown quen thuộc.

3. **Kiểm tra cục bộ**: Chạy `hugo server` để xem bài viết hiển thị như thế nào trên máy tính mình trước.

4. **Đăng bài (Cực kỳ quan trọng)**: Khi đã ưng ý, bạn chỉ cần thực hiện 3 lệnh Git cơ bản:

   ```bash {linenos=false}
   git add .
   git commit -m "Thêm bài viết mới: Tên bài viết"
   git push origin main
   ```

   Ngay sau khi push, GitHub Actions sẽ tự động làm mọi việc còn lại. Sau khoảng 1 phút, bài viết mới của bạn sẽ xuất hiện "chễm chệ" trên blog live!

## Kết luận

- Tổng chi phí: **0 đồng** (tận dụng free tier của GitHub Pages, Vercel và Neon là quá đủ dùng cho blog cá nhân).
- Ưu điểm: Hệ thống bình luận đầy đủ tính năng, hỗ trợ ẩn danh (anonymous), hiển thị ngay lập tức cùng với emoji và reaction dễ thương.
- **Tự động hóa**: Sau khi setup xong, công việc duy nhất của bạn là tập trung viết bài bằng Markdown và push code lên GitHub. GitHub Actions sẽ lo liệu phần còn lại (build và deploy) để bài viết của bạn xuất hiện trên blog ngay lập tức.
- Nhược điểm: Nếu traffic tăng cao đột biến, có thể cần nâng cấp Neon/Vercel (nhưng rất hiếm gặp với blog cá nhân).

Vậy là chỉ với vài bước đơn giản và không tốn bất kỳ chi phí nào, bạn đã có cho mình một trang blog xịn sò để thỏa sức viết lách rồi. Giờ chỉ còn việc sắp xếp thời gian để ra bài viết thôi. Cảm ơn đã đọc và hẹn gặp lại ở các bài viết sau! Nếu gặp phải lỗi hoặc cần hỗ trợ gì, bạn cứ để lại comment bên dưới nhé!

## Tài liệu tham khảo

1. **Hugo Theme Stack Starter Template** (repo chính thức để clone và bắt đầu nhanh):  
   <https://github.com/CaiJimmy/hugo-theme-stack-starter>

2. **Hugo Theme Stack Official Documentation** (config comments, Waline, và tùy chỉnh chung):  
   <https://stack.jimmycai.com/>  
   Phần Comments: <https://stack.jimmycai.com/config/comments>

3. **Hugo Official Guide: Host and Deploy on GitHub Pages** (deploy với GitHub Actions):  
   <https://gohugo.io/host-and-deploy/host-on-github-pages/>

4. **Waline Official Documentation** (deploy trên Vercel, config client, ecosystem):  
   <https://waline.js.org/en/>  
   Deploy Vercel: <https://waline.js.org/en/guide/deploy/vercel.html>  
   Client config (path, lang, reaction): <https://waline.js.org/en/reference/client/props.html>

5. **Neon Docs: Integrate with Vercel** (kết nối Neon Postgres với Vercel, env vars, branching):  
   <https://neon.com/docs/guides/vercel-overview>

6. **Source code của chính blog này** (để bạn tham khảo thực tế):  
   <https://github.com/pngocthach/pngocthach.github.io>
