# conley-labs.github.io

Static website built with Jekyll, deployed to GitHub Pages.

## Development

```bash
# Run locally
bundle install
bundle exec jekyll serve

# Build for production
jekyll build
```

## Writing Content

Add or edit `.md` files in the root directory. Jekyll automatically converts markdown to HTML.

- `index.md` - Homepage
- `proxmox.md` - Proxmox homelab project

## Deployment

Push to the `main` branch. GitHub Pages automatically builds and deploys the site.

## Requirements

- Ruby 3.0+
- Jekyll 4.0+
- Bundler
