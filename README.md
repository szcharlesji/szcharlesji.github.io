# Charles Cheng Ji - Personal Website

A minimal, modern Jekyll website using the [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/) theme.

## Prerequisites

- **Ruby 3.4.7** (via Homebrew)
- **Bundler** (for managing Ruby gems)

## Setup

### 1. Install Ruby (if not already installed)

```bash
# Install Ruby via Homebrew
brew install ruby

# Add Homebrew Ruby to your PATH (Fish shell)
# Add these lines to ~/.config/fish/config.fish:
fish_add_path /opt/homebrew/opt/ruby/bin
fish_add_path (gem environment gemdir)/bin

# Reload your shell config
source ~/.config/fish/config.fish

# Verify Ruby version
ruby -v  # Should show 3.4.7
```

### 2. Install Dependencies

```bash
# Install Bundler
gem install bundler

# Install project dependencies
bundle install
```

### 3. Run Locally

```bash
# Start Jekyll server
bundle exec jekyll serve

# Or with live reload
bundle exec jekyll serve --livereload

# Site will be available at http://localhost:4000
```

## Project Structure

```
.
├── _config.yml          # Jekyll configuration
├── _posts/              # Blog posts (if any)
├── assets/              # Images, CSS, JS
│   └── images/          # Image assets
├── index.md             # Homepage
├── resume.md            # Resume/Experience page
├── 404.md               # 404 error page
└── Gemfile              # Ruby dependencies
```

## Configuration

Main configuration is in `_config.yml`. Key settings:

- **Site info**: Title, description, URL
- **Author info**: Name, bio, social links
- **Theme**: Minimal Mistakes with default skin
- **Plugins**: SEO, sitemap, feed, etc.

## Customization

### Change Theme Skin

Edit `_config.yml`:

```yaml
minimal_mistakes_skin: "default" # or "air", "aqua", "contrast", "dark", "mint", "neon", "plum", "sunrise"
```

### Update Author Info

Edit the `author` section in `_config.yml`:

```yaml
author:
  name: "Your Name"
  bio: "Your bio"
  email: "your@email.com"
  # Add/remove social links as needed
```

### Add Blog Posts

Create markdown files in `_posts/` with filename format: `YYYY-MM-DD-title.md`

```markdown
---
layout: single
title: "Post Title"
date: 2025-01-01
categories: [category]
---

Your content here...
```

## Deployment

### GitHub Pages

1. Push to GitHub
2. Enable GitHub Pages in repository settings
3. Set source to main/master branch
4. Site will be live at `https://yourusername.github.io`

### Custom Domain

1. Add `CNAME` file with your domain
2. Update `url` in `_config.yml`
3. Configure DNS with your domain provider

## Troubleshooting

### Ruby Version Issues

If you see Ruby version errors:

```bash
# Check your Ruby version
ruby -v

# Make sure Homebrew Ruby is first in PATH
which ruby  # Should show /opt/homebrew/opt/ruby/bin/ruby
```

### Bundle Install Fails

```bash
# Clean and reinstall
rm -rf vendor/bundle
rm Gemfile.lock
bundle install
```

### Jekyll Won't Start

```bash
# Clean Jekyll cache
bundle exec jekyll clean

# Rebuild
bundle exec jekyll serve
```

## Resources

- [Minimal Mistakes Documentation](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/)
- [Jekyll Documentation](https://jekyllrb.com/docs/)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)

## License

Content is yours, theme is [MIT licensed](https://github.com/mmistakes/minimal-mistakes/blob/master/LICENSE).
