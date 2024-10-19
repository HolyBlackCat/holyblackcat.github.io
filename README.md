This is the sources for my personal blog. Read the blog at https://holyblackcat.github.io.

The rest of this file is the documentation (primarily for myself) on how to maintain the blog.

## Maintaining a blog:

### Basic setup

Install Ruby: `sudo pacman -S --needed ruby`

* Only when creating a new blog from scratch:

  ```sh
  # Install jekyll manually:
  gem install jekyll
  # Create a new project:
  mkdir my_blog && cd my_blog
  ~/.ruby_bundles/ruby/3.2.0/bin/jekyll new .
  ```

  And add the `.bundle` directory to `.gitignore`. Everything else should already be added to it by default.

```sh
# Tell `bundle` to install packages relative to the user dir (to match where `gem` installs them).
cd my_blog
bundle config set --local path '~/.local/share/gem'
# Install dependencies:
bundle install
```

Run `bundle exec jekyll serve` to start a local webserver. The content autoupdates when (most?) files are modified.

### Creating a post

Create `YYYY-MM-DD-shortname.md` in `_posts`. The date **must** not be in the future, otherwise it will be ignored by default.

The file should start with:
```yaml
---
layout: post
title: Long title
---
```
### Creating a top-level page
You can create `.md` files directly at the top level, they will be added to the blog's header bar.

```yaml
---
layout: page
title: Title
---
```
