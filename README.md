# David Ferreira Blog

Blog pessoal construído com **Hugo** sobre o tema **PaperMod**, com foco em:

- Cloud
- Platform Engineering
- DevOps
- Governança
- Projetos open source

Site publicado em: `https://davidferreira.blog/`

## Stack

- **Static site generator**: Hugo
- **Theme**: PaperMod
- **Customizações do projeto**: `layouts/`, `layouts/partials/`, `assets/css/extended/` e `static/`
- **Conteúdo**: Markdown em `content/`

## Rodar localmente

Para subir o site localmente:

```bash
hugo server -D
```

O blog ficará disponível em `http://localhost:1313`.

Observações:

- `-D` inclui conteúdos com `draft: true`
- para testar só conteúdo publicado, rode `hugo server` sem `-D`

## Build

Para gerar o site estático:

```bash
hugo
```

Ou com minificação:

```bash
hugo --minify
```

Saída gerada em:

- `public/`

## Estrutura do projeto

Diretórios principais:

- `content/posts/`: posts do blog
- `content/categories/`: páginas das categorias
- `content/sobre/`: página Sobre
- `layouts/`: overrides e templates do projeto
- `layouts/partials/`: partials reutilizáveis
- `assets/css/extended/`: CSS customizado do site
- `static/images/`: imagens e capas dos posts
- `themes/PaperMod/`: tema base como submódulo

## Convenções do projeto

Este repositório segue algumas regras simples para reduzir acoplamento ao tema:

- não editar `themes/PaperMod` diretamente
- preferir overrides em `layouts/` e `layouts/partials/`
- preferir estilos isolados em `assets/css/extended/`
- tratar `public/` e `resources/` como artefatos gerados
- manter compatibilidade com o PaperMod sempre que possível

## Categorias atuais

O blog hoje organiza conteúdo principalmente nestas categorias:

- `kubernetes`
- `projetos`
- `cloud`
- `devops`
- `governanca`
- `terraform`

## Criar um novo post

O caminho padrão é criar um arquivo em `content/posts/` usando o mesmo formato dos posts existentes.

Exemplo de front matter:

```toml
---
title: "Título do post"
slug: "titulo-do-post"
date: 2026-03-26T10:00:00-03:00
description: "Resumo curto do conteúdo."
summary: "Resumo curto do conteúdo."
tags: ["cloud", "devops"]
categories: ["cloud"]
draft: false
cover:
  image: "images/hero.png"
  alt: "Descrição curta da imagem"
---
```

Boas práticas:

- usar `description` e `summary` curtos e claros
- manter `tags` e `categories` consistentes
- usar capa em `static/images/` quando o post tiver destaque visual
- preferir slug explícito para evitar mudanças futuras de URL

## Posts de projetos

Projetos open source e soluções próprias entram como posts em `content/posts/` e também podem usar a categoria:

- `projetos`

Isso faz o conteúdo aparecer automaticamente em:

- `/posts/`
- `/categories/projetos/`

## Mermaid

O blog tem suporte a diagramas Mermaid nos posts.

Use normalmente blocos como:

```md
~~~mermaid
flowchart TD
  A[Origem] --> B[Processamento]
  B --> C[Resultado]
~~~
```

A renderização é feita no cliente por um loader adicionado via override local, sem alterar o tema original.

## Customizações visuais atuais

O projeto já possui customizações relevantes fora do PaperMod:

- home customizada
- listagem de posts com hero, busca e cards próprios
- página Sobre customizada
- estilo customizado para post individual
- suporte a Mermaid
- paleta ajustada para o visual atual do site

## Deploy

O deploy é feito via GitHub Actions para o site público.

Fluxo esperado:

1. editar conteúdo ou customizações
2. validar localmente com `hugo` ou `hugo server -D`
3. enviar para a branch principal

## Validação recomendada

Depois de qualquer alteração relevante, rodar:

```bash
hugo
```

E verificar:

- build sem erro de template
- página afetada renderizando corretamente
- layout mobile
- dark theme
- busca, navegação e filtros quando aplicável

## Tema

Tema base:

- `PaperMod`

O projeto usa overrides locais para evitar acoplamento desnecessário ao submódulo do tema.
