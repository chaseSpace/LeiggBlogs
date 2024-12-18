![Hexo Logo](https://github.com/vercel/vercel/blob/main/packages/frameworks/logos/hexo.svg)

# Hexo Example

使用Vercel创建的Hexo模板。

## Deploy Your Own

Deploy your own Hexo project with Vercel.

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https://github.com/vercel/vercel/tree/main/examples/hexo&template=hexo)

_Live Example: https://hexo-template.vercel.app_

## Source markdown

```shell
➜ tree hexo/source 
hexo/source
└── _posts
    ├── about-eq-ord-trait.md
    ├── build-site-with-vercel.md
    ├── rust-cross-compile-on-mac.md
    └── welcome.md
```

### How We Created This Example

To get started with Hexo for deployment with Vercel, you can use the [Hexo CLI](https://hexo.io/docs/index.html#Installation) to initialize the project:

```shell
$ hexo init project-name

# Creates a new article
$ hexo new [layout] <title>

# Generates static files.
$ hexo generate

# Starts a local server.
$ hexo server

# Cleans the cache file (db.json) and generated files (public).
$ hexo clean

# Lists all routes.
$ hexo list <type>

$ hexo version
$ hexo --debug
$ hexo --silent
$ hexo --draft # Displays draft posts (stored in the source/_drafts folder).
$ hexo --cwd /path/to/cwd
```
