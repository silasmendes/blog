# Silas Mendes — Data Engineering Notes

My personal blog where I share field notes, lessons learned, and practical guides on data engineering — covering topics like **Microsoft Fabric**, **Databricks**, **Synapse Analytics**, and modern data platforms in general.

## About

I'm a Data Professional working as an Embedded Escalation Engineer. This blog is a space for me to document things I learn along the way and hopefully help others facing similar challenges. The opinions and content here are entirely my own and do not represent my employer.

## Built with

- [Hugo](https://gohugo.io/) — A fast, open-source static site generator
- [Hextra](https://github.com/imfing/hextra) — A modern Hugo theme built with Tailwind CSS

## Running locally

### Prerequisites

- [Hugo Extended](https://gohugo.io/installation/) (v0.140.0+)
- [Git](https://git-scm.com/)

### Setup

```bash
# Clone the repository with the theme submodule
git clone --recurse-submodules https://github.com/silasmendes/blog.git
cd blog

# Start the development server
hugo server --buildDrafts
```

The site will be available at `http://localhost:1313`.

### Creating a new post

```bash
hugo new content blog/my-new-post/index.md
```

## License

Blog content (under `content/`) is my own. The [Hextra](https://github.com/imfing/hextra) theme is licensed under the MIT License.
