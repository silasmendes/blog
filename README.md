# Silas Blog (Hextra)

Blog pessoal construído com [Hugo](https://gohugo.io/) e o tema [Hextra](https://github.com/imfing/hextra).

## Pré-requisitos

- [Hugo Extended](https://gohugo.io/installation/) (v0.140.0+)
- [Git](https://git-scm.com/)

## Desenvolvimento local

```bash
# Clonar o repositório (com submodules)
git clone --recurse-submodules <url-do-repo>

# Iniciar servidor de desenvolvimento
hugo server --buildDrafts
```

O site estará disponível em `http://localhost:1313`.

## Criar novo post

```bash
hugo new content blog/meu-novo-post.md
```

## Build para produção

```bash
hugo --minify
```

O site será gerado na pasta `public/`.

## Estrutura

```
├── content/
│   ├── _index.md          # Página inicial
│   ├── about.md           # Página Sobre
│   └── blog/
│       ├── _index.md      # Lista de posts
│       └── *.md           # Posts do blog
├── hugo.yaml              # Configuração do Hugo
└── themes/hextra/         # Tema Hextra (submodule)
```
