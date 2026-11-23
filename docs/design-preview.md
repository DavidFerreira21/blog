# Prévia do redesign — sem alterar o site publicado

As modificações ficam isoladas na branch `design/preview-editorial-v2`. **Nenhuma alteração em `main` ou `gh-pages` é necessária para revisar a interface.**

## Abrir versão navegável pelo navegador

1. Na PR do redesign, acesse **Code → Codespaces → Create codespace on this branch**. Você também pode entrar em [github.com/codespaces](https://github.com/codespaces) e escolher este repositório e branch.
2. O ambiente usa `.devcontainer/devcontainer.json`, instala Hugo Extended 0.146.0 e baixa o submódulo PaperMod.
3. Ao iniciar, o Hugo Server expõe a porta **1313**. Abra a aba **Ports** do Codespaces e clique no endereço HTTPS da porta 1313; se o servidor não estiver ativo, execute no terminal: `hugo server --bind 0.0.0.0 --port 1313 --baseURL /`.
4. Navegue pela home, listagem de posts, artigos e página Sobre. Edite os arquivos da branch e o Hugo atualizará a visualização. A URL da porta pode ser privada e exige sessão GitHub.

**Alternativa local:** `git checkout design/preview-editorial-v2 && git submodule update --init --recursive && hugo server -D`, abrir `http://localhost:1313/`.

## Verificações antes de publicar

- Desktop e celular: layout, cards, menu, categorias e navegação.
- Texto: largura, índice automático, bloco de código horizontal, leitura e imagens.
- Acessibilidade: contraste, foco de teclado, links que funcionam e alt de imagens.
- Rodapé: GitHub e LinkedIn reais; não há mais formulário sem integração.
- Performance: imagens grandes precisam de otimização; nenhum asset original foi removido.
- `hugo --minify` deve concluir com sucesso.

A PR não deve ser mesclada até a aprovação visual. O deploy de produção continua exclusivo de pushes na `main`.
