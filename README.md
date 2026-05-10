# Eduard Heijkoop — Academic Portfolio

Personal portfolio site built with [Jekyll](https://jekyllrb.com/) and the [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/) theme. Features blog posts, geospatial portfolio pages with interactive Folium maps, and an about page.

🌐 **Live site:** https://eduardheijkoop.github.io

---

## Structure

```
.
├── _config.yml              # Main Jekyll configuration
├── index.html               # Homepage with feature row
├── Gemfile                  # Ruby dependencies
├── _pages/
│   ├── about.md             # About me page
│   ├── portfolio.md         # Portfolio listing
│   └── blog.md              # Blog listing
├── _portfolio/              # Portfolio project pages
│   ├── icesat2-coastal-dem.md
│   ├── sea-level-rise-impact.md
│   └── vertical-land-motion.md
├── _posts/                  # Blog posts
│   ├── 2025-03-15-icesat2-getting-started.md
│   ├── 2025-01-20-dem-bias-coastal.md
│   └── 2024-11-05-folium-coastal-maps.md
├── _data/
│   └── navigation.yml       # Site navigation
├── assets/
│   ├── css/custom.css       # Style overrides
│   ├── images/              # → Add your photos here
│   └── maps/                # Interactive Folium map HTML files
│       ├── icesat2_coverage.html
│       ├── slr_impact.html
│       └── vlm_map.html
└── .github/workflows/
    └── deploy.yml           # Auto-deploy to GitHub Pages
```

---

## Deployment (First Time)

### Option A: GitHub Pages (Recommended)

1. **Create a repo** named `EduardHeijkoop.github.io` on GitHub

2. **Clone and push:**
   ```bash
   git clone https://github.com/EduardHeijkoop/EduardHeijkoop.github.io
   # Copy all these files into it
   git add .
   git commit -m "Initial portfolio"
   git push origin main
   ```

3. **Enable GitHub Pages:**
   - Go to repo Settings → Pages
   - Source: **GitHub Actions**
   - The `deploy.yml` workflow handles everything automatically

4. **Your site is live** at `https://eduardheijkoop.github.io` within ~2 minutes.

### Option B: Local Preview

```bash
# Install Ruby 3.x, then:
bundle install
bundle exec jekyll serve --livereload
# Open http://localhost:4000
```

---

## Customization Checklist

- [ ] **Avatar:** Add `assets/images/avatar.jpg` (your photo, ~400×400px)
- [ ] **Header images:** Add `assets/images/header-coastal.jpg` and feature images
- [ ] **`_config.yml`:** Update email, LinkedIn, Google Scholar URLs
- [ ] **About page:** Update publications list with real DOIs
- [ ] **Portfolio projects:** Add your actual GitHub repo links
- [ ] **Blog posts:** Add more posts or update dates/content

### Adding a New Portfolio Project

Create `_portfolio/my-project.md`:

```yaml
---
title: "My Project Title"
excerpt: "One-line description shown in the grid."
header:
  teaser: /assets/images/my-project-thumb.jpg
tags:
  - ICESat-2
  - Python
---

Project content in Markdown...

<!-- Embed a Folium map: -->
<iframe src="/assets/maps/my_map.html" width="100%" height="500" frameborder="0"></iframe>
```

### Adding a New Blog Post

Create `_posts/YYYY-MM-DD-post-slug.md`:

```yaml
---
title: "Post Title"
date: 2025-06-01
categories:
  - Tutorial
tags:
  - Python
  - Remote Sensing
excerpt: "Short description for the listing page."
---

Post content...
```

---

## Generating Real Folium Maps

Replace the demo HTML files in `assets/maps/` with outputs from your Python scripts:

```python
import folium
# ... build your map ...
m.save('assets/maps/my_map.html')
```

Then commit and push — the map will appear in the portfolio page.

---

## Theme

[Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/) by Michael Rose, MIT License.  
Skin: `dark` — change in `_config.yml` (`minimal_mistakes_skin`) to `"default"`, `"air"`, `"aqua"`, `"neon"`, etc.
