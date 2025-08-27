# Lochlan's Blog

This is the repository for my personal blog, [lochgeo.github.io](https://lochgeo.github.io/). It is built with Jekyll and based on the "simple-blog" theme.

## Local Development

To run the site locally, you'll need to have Ruby and Bundler installed.

1.  **Clone the repository:**
    ```sh
    git clone https://github.com/lochgeo/lochgeo.github.io.git
    cd lochgeo.github.io
    ```

2.  **Install dependencies:**
    ```sh
    bundle install
    ```

3.  **Run the Jekyll server:**
    ```sh
    bundle exec jekyll serve --livereload
    ```

    The site will be available at `http://localhost:4000`. The `--livereload` flag will automatically refresh the page when you make changes to the files.

## Creating a New Post

To create a new blog post, add a new Markdown file to the `_posts` directory. The filename must follow the Jekyll convention:

`YYYY-MM-DD-your-post-title.md`

The file should start with the standard Jekyll front matter, for example:

```yaml
---
layout: post
title: "Your Post Title"
date: YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [example, category]
tags: [tag1, tag2]
---

Your post content starts here...
```

## Project Structure

- **`_config.yml`**: Main Jekyll configuration.
- **`_posts`**: Blog posts.
- **`_pages`**: Static pages (About, Archive, etc.).
- **`_layouts`**: HTML templates for pages and posts.
- **`_includes`**: Reusable HTML snippets.
- **`assets`**: CSS, JavaScript, and other static files.
- **`_sass`**: Sass files for styling.

## Credits

This blog is based on the [simple-blog theme](https://github.com/AmitMerchant/simple-blog) by Amit Merchant. It also utilizes the following open-source projects:

- [Simple Jekyll Search](https://github.com/christian-fei/simple-jekyll-search)
- [Pullquote.js](https://github.com/csb3/pullquote.js)
- [normalize.css](https://github.com/necolas/normalize.css/)
- [SVG icons](https://icomoon.io/)

## License

The theme is available as open source under the terms of the [MIT License](LICENSE).
