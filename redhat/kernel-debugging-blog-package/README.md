# Kernel Debugging Blog Package

This package contains a blog-ready Markdown article and the extracted image assets.

## Files

- `kernel-debugging-blog.md` - main blog article
- `assets/` - images referenced by the Markdown article

## How to use

Upload both `kernel-debugging-blog.md` and the whole `assets/` directory to your blog repository.

Do not upload the Markdown file alone; otherwise the images will be broken.

For example, in a Hugo/Jekyll/static blog, keep this structure:

```text
posts/kernel-debugging-blog.md
posts/assets/01-crash-command-debuginfo.png
posts/assets/02-crash-sys-summary-oops.png
...
```

If your platform uses a separate media folder, upload the images there and update the relative paths in the Markdown.
