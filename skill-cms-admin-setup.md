# How to Add a CMS Admin Panel to an Astro Site (Sveltia CMS + GitHub)

## What This Does
Adds a visual content editor at `/admin` on your website. You log in with GitHub and can create/edit blog posts, manage data (like product listings), and publish — all without touching code. Changes commit to GitHub automatically, which triggers a redeploy on Vercel/Netlify.

## Prerequisites
- An Astro site deployed on Vercel (or Netlify)
- A GitHub repo containing the site
- A GitHub account that owns or has write access to the repo

---

## Step 1: Create the Admin Page

Create `public/admin/index.html`:

```html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Site Admin</title>
    <link href="https://unpkg.com/@sveltia/cms/dist/sveltia-cms.css" rel="stylesheet" />
  </head>
  <body>
    <script src="https://unpkg.com/@sveltia/cms/dist/sveltia-cms.js" type="module"></script>
  </body>
</html>
```

This loads Sveltia CMS entirely in the browser — no server required.

---

## Step 2: Create the CMS Config

Create `public/admin/config.yml`:

```yaml
backend:
  name: github
  repo: YOUR_GITHUB_USERNAME/YOUR_REPO_NAME
  branch: main
  auth_type: pkce
  app_id: YOUR_GITHUB_OAUTH_CLIENT_ID

media_folder: public/images
public_folder: /images

collections:
  # Blog posts (stored as Markdown files)
  - name: blog
    label: "Blog Posts"
    folder: src/content/blog
    create: true
    slug: "{{slug}}"
    fields:
      - { label: "Title", name: "title", widget: "string" }
      - { label: "Description", name: "description", widget: "string" }
      - { label: "Date", name: "date", widget: "datetime" }
      - { label: "Tags", name: "tags", widget: "list", default: [] }
      - { label: "Body", name: "body", widget: "markdown" }

  # Data files (stored as JSON)
  - name: settings
    label: "Settings"
    files:
      - name: books
        label: "Puzzle Books"
        file: src/data/books.json
        fields:
          - label: "Books"
            name: "books"
            widget: "list"
            fields:
              - { label: "Title", name: "title", widget: "string" }
              - { label: "Description", name: "description", widget: "text" }
              - { label: "Category", name: "category", widget: "string" }
              - { label: "Link", name: "link", widget: "string" }
```

### Config explained:
- **backend**: Connects to GitHub using OAuth PKCE (client-side, no server needed)
- **media_folder**: Where uploaded images are stored in the repo
- **collections**: Defines what content types the CMS can edit
  - `folder` collections = one file per entry (blog posts)
  - `files` collections = a single file with structured data (settings, product lists)

---

## Step 3: Set Up GitHub OAuth App

1. Go to **github.com → Settings → Developer settings → OAuth Apps → New OAuth App**
2. Fill in:
   - **Application name:** Your CMS name
   - **Homepage URL:** `https://your-site.vercel.app`
   - **Authorization callback URL:** `https://your-site.vercel.app/admin/`
3. Click **Register application**
4. Copy the **Client ID** and put it in `config.yml` as `app_id`

**Important:** This must be an **OAuth App**, NOT a **GitHub App**. They are different things in GitHub Developer Settings.

---

## Step 4: Authentication

### Option A: OAuth Login (one-click)
Click "Sign In with GitHub" on the admin page. This uses the OAuth App you created.

### Option B: Personal Access Token (simpler, always works)
If OAuth gives trouble:
1. Go to **github.com → Settings → Developer settings → Personal access tokens → Tokens (classic)**
2. Generate a token with `repo` scope
3. Click "Sign In with GitHub Using Token" on the admin page
4. Paste the token

---

## Step 5: Make Your Astro Pages Read from CMS-Managed Files

### For blog posts:
Astro content collections already work. Set up `src/content/config.ts`:

```typescript
import { defineCollection, z } from 'astro:content';

const blog = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    description: z.string(),
    date: z.coerce.date(),
    tags: z.array(z.string()).default([]),
  }),
});

export const collections = { blog };
```

### For data files (like product listings):
Store data in a JSON file (e.g., `src/data/books.json`):

```json
{
  "books": [
    {
      "title": "My Book",
      "description": "About the book",
      "category": "Category",
      "link": "https://amazon.com/dp/XXXX"
    }
  ]
}
```

Import it in your Astro page:

```astro
---
import booksData from '../data/books.json';
const { books } = booksData;
---
```

---

## How the Flow Works

```
You edit in CMS → Sveltia commits to GitHub → Vercel detects push → Site rebuilds → Live in ~15 seconds
```

No server, no database, no WordPress. The CMS is just a browser-based UI for editing files in your GitHub repo.

---

## Tools Used

| Tool | What it does | Cost |
|------|-------------|------|
| **Sveltia CMS** | Browser-based content editor UI | Free |
| **GitHub OAuth App** | Authenticates you to edit the repo | Free |
| **GitHub API** | Reads/writes files in the repo | Free |
| **Vercel** | Auto-deploys when files change | Free tier |
| **Astro** | Builds the site from Markdown + JSON | Free |

## Key Files

```
public/
  admin/
    index.html      ← Loads the CMS script
    config.yml      ← Defines content types and GitHub connection
src/
  content/
    config.ts       ← Astro content collection schema
    blog/
      my-post.md    ← Blog posts (created/edited via CMS)
  data/
    books.json      ← Structured data (edited via CMS)
```
