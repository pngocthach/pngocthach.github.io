+++
title = 'How I Set Up My Personal Blog — Fast, Beautiful, and Free'
date = 2026-03-08T19:25:00+07:00
draft = false
description = "A quick guide to setting up a personal blog with Hugo, GitHub Pages, and Waline"
image = ""
categories = [
    "Tech",
    "Blog",
    "Tutorial"
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

If you want a personal blog that is **fast, beautiful, and completely free** — no ads, quick loading, and a full-featured comment system (emoji, reactions, anonymous commenting...) — this combo is the perfect choice in 2026:

- **Hugo** (a blazing-fast static site generator)  
- **Stack Theme** (modern, minimalist design with a gorgeous dark mode)  
- **GitHub Pages** (free hosting)  
- **Waline** (open-source comment system, self-hosted on Vercel + Neon Postgres, supports anonymous comments that appear instantly)

## Prerequisites

- A GitHub account (public repo)  
- A Vercel account (free, sign in with GitHub)  
- A Neon account (free, serverless Postgres)  
- A machine with Git + Hugo installed (extended version recommended)

## Step 1: Clone the Hugo Stack Template and Set Up the Basic Site

1. Go to the official starter repo:  
   <https://github.com/CaiJimmy/hugo-theme-stack-starter>

2. Click **Use this template** → **Create a new repository**  
   - Name your repo:  
     - For a root domain: `username.github.io` (e.g., pngocthach.github.io)  
     - For a subpath: any name (e.g., my-blog) → your domain will be `username.github.io/my-blog`

3. Clone the repo to your machine:

   ```bash {linenos=false}
   git clone https://github.com/username/username.github.io.git
   cd username.github.io
   ```

4. Install Hugo if you haven't already:
   - macOS: `brew install hugo`  
   - Windows: use Scoop/Chocolatey or download the extended binary from <https://github.com/gohugoio/hugo/releases>  
   - Linux (varies by distro): `sudo apt install hugo` (or extended)

   Verify: `hugo version` (should be ≥ 0.120)

5. Run the site locally:

   ```bash {linenos=false}
   hugo server
   ```

   Open <http://localhost:1313> — if you see the Stack demo page, you're good to go.

6. Deploy to GitHub Pages:
   - Go to your repo's Settings → Pages  
   - Source: Deploy from a branch → Branch: main → Folder: / (root) → Save  
   - The template already includes a GitHub Actions workflow (`.github/workflows/hugo.yml`), so every push will automatically trigger a build.

   Your site will be live within 1–3 minutes at <https://username.github.io> (or `/my-blog`)

## Step 2: Set Up the Waline Comment Backend (Vercel + Neon Postgres)

Waline is a lightweight, open-source comment system that supports anonymous commenting and beautiful emoji/reactions.

1. Deploy Waline on Vercel (free):
   - Visit: <https://waline.js.org/en/guide/deploy/vercel.html>  
   - Click the **Deploy** button (blue)  
   - Sign in to Vercel with GitHub → Name your project (e.g., `my-waline-backend`) → Create  
   - Wait for deployment (1–2 minutes) → Copy your backend URL: <https://my-waline-backend.vercel.app>

2. Create a Neon Postgres database (free tier):
   - In Vercel Dashboard → **Storage** tab → **Create Database** → Select **Neon**  
   - Region: **Singapore (aws-ap-southeast-1)** (lowest latency from Southeast Asia)  
   - Create the DB → Open in Neon console  
   - In the Neon SQL Editor → Paste the init script from:  
     <https://github.com/walinejs/waline/blob/main/assets/waline.pgsql>  
     → Run to create the tables

3. Connect Neon to Vercel:
   - In Vercel → Storage → Select the Neon DB you just created → **Connect**  
   - Check **all Environments**: Development, Preview, Production  
   - Check **Preview** for branching (uncheck Production if not needed)  
   - Custom Prefix: Leave **blank**  
   - Connect → Vercel will automatically inject the environment variables

4. Create the admin account:
   - Visit: <https://my-waline-backend.vercel.app/ui/register>  
   - Register the first user → this account becomes the admin  
   - Log in at `/ui` to manage comments later (e.g., review spam)

## Step 3: Integrate Waline into Hugo Stack

1. Edit the config file (`config/_default/params.toml`):

   ```toml {linenos=false}
   [comments]
     enabled = true
     provider = "waline"

     [comments.waline]
       serverURL = "https://waline-backend-iota.vercel.app"  # Replace with your Vercel URL
       lang = "en"
       reaction = true
       emoji = [
         "https://unpkg.com/@waline/emojis@1.0.1/tw-emoji",
         "https://unpkg.com/@waline/emojis@1.0.1/weibo"
       ]
   ```

2. **Override the template to display full URLs (easier to manage in the admin panel)**:
   - During deployment, you may notice that Waline stores comments using only the relative path (e.g., `/p/post-name`) instead of the full URL (`https://yourdomain.com/p/post-name`), making it very hard to identify which post a comment belongs to.
   - To fix this, create the file: `layouts/partials/comments/provider/waline.html` to override the theme's default.

   - File content:

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

- Using `.Permalink` will automatically include the full URL (with domain), so the Waline admin panel displays comments as `https://yourblog.com/p/post-name/` instead of a relative path — making them much easier to identify.

## Step 4: Writing New Posts and Updating Your Blog

Once the scaffolding is set up, writing new posts is incredibly simple:

1. **Create a new post**: Open a terminal and run:

   ```bash {linenos=false}
   hugo new content post/your-post-title/index.md
   ```

   This creates a new folder inside `content/post/` along with an `index.md` file — this is where you'll write your post content.

2. **Write your content**: Open the newly created `index.md`. You'll see a section at the top (between `+++`) called the **Front Matter** — this is where metadata like title, date, description, and cover image goes. Everything below is your post content, written in standard Markdown.

3. **Preview locally**: Run `hugo server` to see how your post looks before publishing.

4. **Publish (This is all it takes)**: When you're happy with your post, run these 3 basic Git commands:

   ```bash {linenos=false}
   git add .
   git commit -m "Add new post: Your Post Title"
   git push origin main
   ```

   As soon as you push, GitHub Actions will handle everything else. Within about a minute, your new post will appear live on your blog!

## Conclusion

- Total cost: **$0** (the free tiers of GitHub Pages, Vercel, and Neon are more than sufficient for a personal blog).
- Advantages: A full-featured comment system with anonymous support, instant display, emoji, and reactions.
- **Automation**: Once the setup is done, your only job is to write in Markdown and push to GitHub. GitHub Actions takes care of building and deploying — your post goes live automatically.
- Drawback: If you experience a sudden traffic spike, you may need to upgrade your Neon/Vercel plan — but this is extremely rare for a personal blog.

In just a few simple steps and at zero cost, you have a sleek, professional blog to write on freely. Now it's just a matter of making time to publish! Thanks for reading, and see you in the next post. If you run into any issues or have questions, feel free to leave a comment below!

## References

1. **Hugo Theme Stack Starter Template** (official repo to clone and get started quickly):  
   <https://github.com/CaiJimmy/hugo-theme-stack-starter>

2. **Hugo Theme Stack Official Documentation** (comments config, Waline, and general customization):  
   <https://stack.jimmycai.com/>  
   Comments section: <https://stack.jimmycai.com/config/comments>

3. **Hugo Official Guide: Host and Deploy on GitHub Pages** (deployment with GitHub Actions):  
   <https://gohugo.io/host-and-deploy/host-on-github-pages/>

4. **Waline Official Documentation** (Vercel deployment, client config, ecosystem):  
   <https://waline.js.org/en/>  
   Deploy on Vercel: <https://waline.js.org/en/guide/deploy/vercel.html>  
   Client config (path, lang, reaction): <https://waline.js.org/en/reference/client/props.html>

5. **Neon Docs: Integrate with Vercel** (connecting Neon Postgres with Vercel, env vars, branching):  
   <https://neon.com/docs/guides/vercel-overview>

6. **Source code of this blog** (for a real-world reference):  
   <https://github.com/pngocthach/pngocthach.github.io>
