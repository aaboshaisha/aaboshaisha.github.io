---
layout: post
title: "Setting Up a GitHub Pages Blog"
date: 2025-09-29
---
This is the process I used to set up this blog:

First thing to know id GitHub pages uses Jekyll to make blogs. **Jekyll** is a static site generator that converts plain text (markdown files) into websites. GitHub Pages uses Jekyll automatically - when you push markdown files to your repository, Jekyll converts them into HTML pages.

## Initial Setup
1. Create `yourusername.github.io` repository
2. Clone locally
3. To make this a blog using `_config.yml`
4. Enable Pages in Settings

### What is `_config.yml`?

`_config.yml` is Jekyll's main configuration file that controls how your site is built and behaves. It's basically the "settings file" that tells Jekyll how to turn your markdown files into a website.

**What it does:**
- Sets site-wide settings like title, description, author
- Chooses which theme to use
- Configures plugins and features
- Defines custom variables you can use throughout your site

**Basic example:**
```yaml
title: My Data Science Blog
description: Sharing machine learning experiments and insights
author: Your Name
email: your.email@example.com
theme: minima

# Optional settings
show_excerpts: true
plugins:
  - jekyll-feed
```

**How Jekyll uses it:**
- Reads this file when building your site
- Makes these values available in your templates (like `{{ site.title }}`)
- Applies the theme and plugins you specify
- Sets up navigation, RSS feeds, etc. based on your choices


The most simple `_config.yml` file can be

```yaml
title: Your Blog Name
description: Your blog description
theme: minima
```

**Minima** is just one of many visual themes for Jekyll sites. It determines how your blog looks - the colors, fonts, layout, etc. Think of it like choosing a template for your blog.

----
We have a blog. Now what? 

We need an index page - listing posts - and we need to know how to add posts.

You'll need two things: an index page and a posts folder.

**Create an index page:**
Create `index.md` in your repository root:
```yaml
---
layout: home
---

"Welcome to my blog! This is where I'll share my data science notebooks and thoughts."
```

**Create your first post:**
1. Create a `_posts` folder
2. Inside `_posts`, create a file named `2024-12-29-my-first-post.md` (use today's date)
3. Add this content:
```yaml
---
layout: post
title: "My First Post"
date: 2024-12-29
---

"This is my first blog post! Soon I'll be adding Jupyter notebook posts here."
```

The minima theme will automatically:
- Show your posts list on the home page
- Create individual post pages
- Add navigation

Commit and push both files, then check your site in a few minutes.


## YAML 

```yaml
---
layout: home
---
```

That's called **YAML front matter** - it's a Jekyll convention for adding metadata to pages and posts.

The part between the `---` lines is YAML (a data format) that tells Jekyll how to process the file:

This tells Jekyll:
- Use the "home" layout template (which shows a list of blog posts)
- Apply the theme's styling and navigation

Everything below the second `---` is regular markdown content that gets inserted into the layout.

**Other common front matter examples:**

For posts:
```yaml
---
layout: post
title: "My Post Title"
date: 2024-12-29
---
```

For regular pages:
```yaml
---
layout: page
title: "About Me"
---
```

Jekyll reads this metadata and uses it to:
- Choose which HTML template to wrap your content in
- Set the page title
- Organize posts by date
- Create navigation menus

Without the front matter, Jekyll treats the file as plain HTML/markdown without any theme styling.

## Organizing Content for Jekyll

Here's how Jekyll organizes content:

**`_posts` folder:**
- Blog posts only
- Must be named `YYYY-MM-DD-title.md`
- Automatically listed on your home page in chronological order

**Root directory (outside `_posts`):**
- `index.md` - your home page
- `about.md` - about page
- `contact.md` - contact page
- Any other standalone pages

**Other special folders:**
- `_drafts` - unpublished posts (won't appear on site)
- `assets` or `images` - for images, CSS, etc.
- `_data` - for data files (advanced)

**Example structure:**
```
yourusername.github.io/
├── _config.yml
├── index.md                    # Home page
├── about.md                    # About page
├── _posts/
│   ├── 2024-12-29-first-post.md
│   └── 2024-12-30-notebook-post.md
└── images/
    └── plot1.png
```

So your blog posts go in `_posts`, but regular pages like "About" or "Contact" go in the root directory.

## Publishing Jupyter Notebooks

To publish a Jupyter notebook as a blog post:

1. **Convert your notebook to markdown:**
```bash
jupyter nbconvert --to markdown your-notebook.ipynb
```

2. **Add YAML front matter** to the generated `.md` file:
```yaml
---
layout: post
title: "Your Notebook Title"
date: 2024-12-29
---
```

3. **Rename and move the file:**
   - Rename to `YYYY-MM-DD-title.md` format
   - Move it to the `_posts` folder

4. **Handle images:**
   - The conversion creates an `your-notebook_files` folder with plots/images
   - Move this folder to your repository root or `assets` folder
   - Update image paths in the markdown if needed

5. **Commit and push**

**Example workflow:**
- You have `data-analysis.ipynb`
- Run: `jupyter nbconvert --to markdown data-analysis.ipynb`
- Add front matter to `data-analysis.md`
- Rename to `2024-12-29-data-analysis.md`
- Move to `_posts/`
- Push to GitHub

Your notebook will appear as a blog post with all the code, outputs, and plots intact!