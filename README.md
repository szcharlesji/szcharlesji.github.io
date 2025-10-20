# Personal Website

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
# Start Jekyll server (minimal output)
bundle exec jekyll serve --quiet

# Or with live reload
bundle exec jekyll serve --livereload

# Verbose mode (shows all Sass warnings from theme)
bundle exec jekyll serve

# Site will be available at http://localhost:4000
```
