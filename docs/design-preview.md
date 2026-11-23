# Prévia pública do redesign

A versão visual da branch `design/preview-editorial-v2` é publicada **somente em `/preview/`**, no mesmo GitHub Pages do blog, sem configurar DNS:

**[https://davidferreira.blog/preview/](https://davidferreira.blog/preview/)**

A home e os artigos existentes em `https://davidferreira.blog/` não são substituídos pelo processo de prévia.

## Como funciona

- Um workflow GitHub Actions da própria branch executa `hugo --minify --baseURL https://davidferreira.blog/preview/`.
- O deploy usa `destination_dir: preview` e `keep_files: true` na branch `gh-pages`, preservando os arquivos da raiz publicada.
- A prévia é **pública**. Não insira segredos, dados privados ou rascunhos confidenciais nesta branch.
- O deploy normal da `main` atual usa `force_orphan: true`, então **uma futura publicação da `main` poderá apagar a pasta `/preview/`**. Para restaurá-la, basta atualizar a branch de preview e o workflow republicará somente essa pasta. A prévia não é permanente.
- Para publicar o redesign definitivamente, é necessário aprovar e mesclar a PR. **Não faça merge apenas para testar a prévia.**
- Se a URL ainda não carregar, confira o workflow **Publish isolated design preview** em Actions e o GitHub Pages; aguarde o Pages concluir a implantação.

## Revisão

Navegue a home, /preview/posts/, categorias, artigos e página Sobre em desktop e celular. Os links internos devem permanecer dentro de `/preview/`. Links externos para GitHub/LinkedIn abrem seus destinos reais.

A alternativa Codespaces continua disponível, mas não é necessária para navegar nesta prévia.
