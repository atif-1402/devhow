+++
title = "How I Built and Deployed a Hugo Site on GitHub Pages for Free"
date = 2026-04-13
tags = ["Linux", "Hugo", "GitHub", "self-hosting"]
summary = "A complete walkthrough of how I migrated from Blogger to a custom Hugo site, built my own theme from scratch, and deployed it on GitHub Pages with a custom domain — all for free."
+++

I was running DevHow on Blogger for a while. It worked, kind of. But there was a redirect error that broke Google indexing, the theme was limiting, and every time I wanted to change something I had to fight the platform. After spending a month trying to fix the indexing issue with no luck, I decided to just start over.

This is the full story of how I rebuilt DevHow from scratch using Hugo, wrote a custom theme, and got it live on GitHub Pages with a custom domain in a single evening.

## Why Hugo

Hugo is a static site generator. You write your posts in Markdown, run one command, and it spits out a complete website as plain HTML, CSS and JS files. No database, no server-side code, nothing to maintain.

For a tutorial site like this, that's ideal. Static files are fast, secure, and can be hosted for free on GitHub Pages. Hugo itself is also extremely fast — it builds hundreds of pages in milliseconds.

The other option I considered was Jekyll, but Hugo is faster and doesn't require Ruby. Since I'm on Arch Linux, installing Hugo was just one command:

```bash
sudo pacman -S hugo
```

Verify it installed:

```bash
hugo version
```

You should see something like `hugo v0.159.1+extended`.

## Creating the Site

Navigate to wherever you want to keep your project and run:

```bash
hugo new site devhow
cd devhow
git init
```

This creates the basic Hugo folder structure:

- `content/` — your posts and pages as Markdown files
- `layouts/` — HTML templates
- `static/` — images, fonts, CSS that get served as-is
- `themes/` — theme files
- `hugo.toml` — your site config

The `hugo.toml` is the brain of the whole operation. Open it and set your basics:

```toml
baseURL = 'https://devhow.xyz/'
languageCode = 'en-us'
title = 'DevHow'
theme = 'devhow'
```

## Building a Custom Theme

Instead of using a prebuilt theme, I built my own. This gave me full control over the design and kept the codebase clean.

First create the theme folder structure:

```bash
mkdir -p themes/devhow/layouts/{_default,partials}
mkdir -p themes/devhow/static/css
```

Hugo themes work through a hierarchy of templates. The most important ones are:

- `baseof.html` — the master skeleton every page uses
- `index.html` — the home page
- `_default/single.html` — individual post pages
- `_default/list.html` — pages that list posts
- `partials/header.html` — reusable header component
- `partials/footer.html` — reusable footer component

The key concept is **partials** — instead of writing the same header HTML on every page, you write it once in `partials/header.html` and include it everywhere with `{{ partial "header.html" . }}`. Change it once, updates everywhere.

The `baseof.html` is the skeleton:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{ if .Title }}{{ .Title }} | {{ site.Title }}{{ else }}{{ site.Title }}{{ end }}</title>
  <link rel="stylesheet" href="/css/style.css">
</head>
<body>
  {{ partial "header.html" . }}
  <main>
    {{ block "main" . }}{{ end }}
  </main>
  {{ partial "footer.html" . }}
</body>
</html>
```

The `{{ block "main" . }}{{ end }}` is where each page injects its own unique content. Think of it like a socket — every page plugs its content into that socket.

## Writing Posts

Posts live in `content/posts/`. Each post is a Markdown file with a front matter block at the top:

```markdown
+++
title = "How to Install Arch Linux"
date = 2026-04-13
tags = ["Linux", "Arch Linux"]
summary = "A beginner friendly guide to installing Arch Linux from scratch."
+++

Your post content starts here.
```

The front matter (the `+++` block) is metadata Hugo uses to build the site — title, date, tags, summary. The rest is plain Markdown.

## Previewing Locally

While building, you can preview the site live:

```bash
hugo server
```

Open `http://localhost:1313` in your browser. Hugo watches for file changes and reloads automatically. This makes it really fast to iterate on the design.

## Deploying to GitHub Pages

GitHub Pages hosts static sites for free. The workflow is:

1. Create a new empty repo on GitHub
2. Push your Hugo project to the `main` branch
3. Build the site with `hugo --minify` which creates a `public/` folder
4. Push the `public/` folder to a `gh-pages` branch
5. Set GitHub Pages to serve from `gh-pages`

First add a `.gitignore` so the public folder doesn't clutter your main branch:

```bash
echo "public/" > .gitignore
```

Connect your repo and push:

```bash
git remote add origin https://github.com/yourusername/devhow.git
git add .
git commit -m "initial commit"
git branch -M main
git push -u origin main
```

Build and deploy to gh-pages:

```bash
hugo --minify
git add public/ -f
git commit -m "deploy"
git push origin `git subtree split --prefix public main`:gh-pages --force
```

In your GitHub repo go to **Settings → Pages** and set the source branch to `gh-pages`.

## Custom Domain

If you have a custom domain, point it to GitHub Pages by adding these DNS records with your registrar:

| Type | Name | Data |
|------|------|------|
| A | @ | 185.199.108.153 |
| A | @ | 185.199.109.153 |
| A | @ | 185.199.110.153 |
| A | @ | 185.199.111.153 |
| CNAME | www | yourusername.github.io |

Then add a CNAME file to your public folder:

```bash
echo "yourdomain.xyz" > public/CNAME
```

Redeploy. GitHub Pages will automatically provision an SSL certificate within a few minutes. No need to buy SSL separately.

## The Result

DevHow went from a broken Blogger site with indexing issues to a fast, clean, fully custom site on a real domain — all in one evening. The whole thing costs nothing to host. The only recurring cost is the domain renewal.

Every time I write a new post now, I just create a Markdown file in `content/posts/`, run `hugo --minify`, and push. Done.

If you're running a blog or tutorial site on Blogger or WordPress and feeling limited by the platform, Hugo is genuinely worth the switch. The learning curve is a few hours, and after that it gets out of your way completely.
