 # Blog - Hugo

Site pessoal em Hugo com foco em Cloud, Platform Engineering e DevOps.

Endereço do blog: `https://davidferreira21.github.io/blog/`

## Rodar localmente

```bash
hugo server -D
```

- O site fica em `http://localhost:1313`
- `-D` inclui rascunhos (`draft: true`)

## Build

```bash
hugo --minify
```

O resultado vai para `public/`.

## Estrutura

- `content/posts/`: posts do blog
- `content/sobre/`: pagina Sobre
- `static/`: imagens e arquivos estaticos
- `assets/`: imagens processadas/pipe do tema
- `layouts/`: overrides do tema (se houver)
- `hugo.toml`: configuracao do site

## Criar um novo post

Copie um post existente e ajuste o front matter:

```toml
---
title: "Titulo do post"
date: 2026-01-29T09:00:00-03:00
tags: ["cloud", "devops"]
categories: ["cloud"]
draft: false
---
```

## Deploy (GitHub Pages)

O deploy eh feito via GitHub Actions. Basta enviar para a branch principal.

## Tema

Tema: PaperMod.
github.com/ghodss/yaml="v1.0.0"
github.com/go-openapi/jsonpointer="v0.21.0"
github.com/go-openapi/swag="v0.23.0"
github.com/go-sourcemap/sourcemap="v2.1.4+incompatible"
github.com/gobuffalo/flect="v1.0.3"
github.com/gobwas/glob="v0.2.3"
github.com/gohugoio/go-i18n/v2="v2.1.3-0.20230805085216-e63c13218d0e"
github.com/gohugoio/hashstructure="v0.5.0"
github.com/gohugoio/httpcache="v0.7.0"
github.com/gohugoio/hugo-goldmark-extensions/extras="v0.2.0"
github.com/gohugoio/hugo-goldmark-extensions/passthrough="v0.3.0"
github.com/gohugoio/locales="v0.14.0"
github.com/gohugoio/localescompressed="v1.0.1"
github.com/golang/freetype="v0.0.0-20170609003504-e2365dfdc4a0"
github.com/google/go-cmp="v0.6.0"
github.com/google/pprof="v0.0.0-20250208200701-d0013a598941"
github.com/gorilla/websocket="v1.5.3"
github.com/hairyhenderson/go-codeowners="v0.7.0"
github.com/hashicorp/golang-lru/v2="v2.0.7"
github.com/jdkato/prose="v1.2.1"
github.com/josharian/intern="v1.0.0"
github.com/kr/pretty="v0.3.1"
github.com/kr/text="v0.2.0"
github.com/kyokomi/emoji/v2="v2.2.13"
github.com/lucasb-eyer/go-colorful="v1.2.0"
github.com/mailru/easyjson="v0.7.7"
github.com/makeworld-the-better-one/dither/v2="v2.4.0"
github.com/marekm4/color-extractor="v1.2.1"
github.com/mattn/go-colorable="v0.1.13"
github.com/mattn/go-isatty="v0.0.20"
github.com/mattn/go-runewidth="v0.0.9"
github.com/mazznoer/csscolorparser="v0.1.5"
github.com/mitchellh/mapstructure="v1.5.1-0.20231216201459-8508981c8b6c"
github.com/mohae/deepcopy="v0.0.0-20170929034955-c48cc78d4826"
github.com/muesli/smartcrop="v0.3.0"
github.com/niklasfasching/go-org="v1.7.0"
github.com/oasdiff/yaml3="v0.0.0-20241210130736-a94c01f36349"
github.com/oasdiff/yaml="v0.0.0-20241210131133-6b86fb107d80"
github.com/olekukonko/tablewriter="v0.0.5"
github.com/pbnjay/memory="v0.0.0-20210728143218-7b4eea64cf58"
github.com/pelletier/go-toml/v2="v2.2.3"
github.com/perimeterx/marshmallow="v1.1.5"
github.com/pkg/browser="v0.0.0-20240102092130-5ac0b6a4141c"
github.com/pkg/errors="v0.9.1"
github.com/rivo/uniseg="v0.4.7"
github.com/rogpeppe/go-internal="v1.13.1"
github.com/russross/blackfriday/v2="v2.1.0"
github.com/sass/libsass="3.6.6"
github.com/spf13/afero="v1.11.0"
github.com/spf13/cast="v1.7.1"
github.com/spf13/cobra="v1.8.1"
github.com/spf13/fsync="v0.10.1"
github.com/spf13/pflag="v1.0.6"
github.com/tdewolff/minify/v2="v2.20.37"
github.com/tdewolff/parse/v2="v2.7.15"
github.com/tetratelabs/wazero="v1.8.2"
github.com/webmproject/libwebp="v1.3.2"
github.com/yuin/goldmark-emoji="v1.0.4"
github.com/yuin/goldmark="v1.7.8"
go.uber.org/automaxprocs="v1.5.3"
golang.org/x/crypto="v0.33.0"
golang.org/x/exp="v0.0.0-20250210185358-939b2ce775ac"
golang.org/x/image="v0.24.0"
golang.org/x/mod="v0.23.0"
golang.org/x/net="v0.35.0"
golang.org/x/sync="v0.11.0"
golang.org/x/sys="v0.30.0"
golang.org/x/text="v0.22.0"
golang.org/x/tools="v0.30.0"
golang.org/x/xerrors="v0.0.0-20240903120638-7835f813f4da"
gonum.org/v1/plot="v0.15.0"
google.golang.org/protobuf="v1.36.5"
gopkg.in/yaml.v2="v2.4.0"
gopkg.in/yaml.v3="v3.0.1"
oss.terrastruct.com/d2="v0.6.9"
oss.terrastruct.com/util-go="v0.0.0-20241005222610-44c011a04896"
rsc.io/qr="v0.2.0"
software.sslmate.com/src/go-pkcs12="v0.2.0"
```
</details>
