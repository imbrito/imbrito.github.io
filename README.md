# imbrito.com.br

Site pessoal e portfólio profissional de **Ismael Moreira de Brito** — engenheiro de dados
e IA, Rio de Janeiro.

**Produção:** <https://imbrito.com.br> · **Deploy:** GitHub Pages, automático a cada push na `main`.

---

## Princípio de projeto

Página única, estática, **sem etapa de build**. Nenhum Node, bundler, Jekyll, framework ou
gerenciador de pacotes. Você edita `index.html` e dá push — é o site inteiro.

A consequência mais importante: **o site não carrega nada de terceiros em runtime.** Zero
CDN, zero Google Fonts, zero fonte de ícones. Isso não é purismo — é o que sustenta o LCP,
elimina pontos de falha externos e evita compartilhar o IP de cada visitante com três
empresas sem base legal.

| | |
|---|---|
| Peso total da página | ~94 KB (11,5 KB de HTML gzip + 84,7 KB de fontes) |
| Requisições a terceiros | 0 |
| Subrecursos | 2 (as duas fontes, mesmo domínio) |
| CSS / JS | inline no `index.html` |

---

## Estrutura

```
.
├── index.html              O site inteiro: HTML + CSS inline + JSON-LD + ~40 linhas de JS
├── 404.html                Página de erro (o GitHub Pages usa automaticamente)
├── CNAME                   imbrito.com.br  — define o domínio customizado
├── .nojekyll               Desliga o Jekyll. NÃO REMOVER (ver "Armadilhas")
├── robots.txt              Libera tudo + permite explicitamente crawlers de LLM
├── sitemap.xml             Uma URL, com lastmod
├── site.webmanifest        Ícones, theme-color, nome do app
├── favicon.ico             ICO com 3 PNGs embutidos (16/32/48 px)
├── assets/
│   ├── favicon.svg         Marca vetorial, sem dependência de fonte
│   ├── apple-touch-icon.png   180×180
│   ├── icon-192.png           192×192   (manifest, maskable)
│   ├── icon-512.png           512×512   (manifest, maskable)
│   ├── og-image.svg        Fonte editável do card social
│   ├── og-image.png        1200×630 — é este que o LinkedIn/WhatsApp leem
│   ├── ismael-moreira-de-brito.jpg   Foto de perfil 512×512 (hero + Person.image)
│   └── fonts/
│       ├── inter-latin-var.woff2        Inter variável, subset latin, 48 KB
│       └── fira-code-latin-var.woff2    Fira Code variável, subset latin, 36 KB
├── docs/
│   ├── seo-audit.md        Auditoria que originou a refatoração (histórico)
│   └── seo-checklist.md    Passos manuais pós-deploy — LEIA APÓS PUBLICAR
├── README.md
└── SECURITY.md
```

---

## Rodar localmente

Qualquer servidor estático serve. **Abrir `index.html` com `file://` não funciona** — os
caminhos são absolutos (`/assets/...`) e não resolvem fora de um servidor.

```bash
python3 -m http.server 4321
```

Depois abra <http://localhost:4321>. Para testar a `404.html`, acesse
<http://localhost:4321/404.html> diretamente (o `http.server` não a serve automaticamente;
o GitHub Pages serve).

---

## Editar

Tudo está em `index.html`, nesta ordem:

| Linhas (aprox.) | O quê |
|---|---|
| 1–60 | `<head>`: title, description, canonical, Open Graph, Twitter, ícones, CSP |
| 60–230 | Bloco JSON-LD (`Person`, `ProfilePage`, `WebSite`, `ImageObject`, `ScholarlyArticle`) |
| 230–830 | CSS: tokens em `:root`, depois componentes, cada bloco comentado |
| 830–1180 | HTML do conteúdo |
| 1180–1215 | JS: montagem do e-mail + `IntersectionObserver` |

**Ao mudar o conteúdo, mude também o JSON-LD.** Eles descrevem a mesma pessoa e divergir
é pior que não ter markup nenhum. Um cargo novo, uma empresa nova ou uma data nova precisa
aparecer nos dois lugares.

**Ao mudar o conteúdo, atualize `<lastmod>` no `sitemap.xml`** e `dateModified` no
`ProfilePage`. É o principal sinal de "recrawleie isto".

### Paleta e contraste

As cores estão em `:root`. Duas delas têm regra de uso:

- `--text-faint` (`#7c8ba1`, 5,69:1) — o mais escuro que passa WCAG AA em texto pequeno.
- `--text-muted` (`#64748b`, 4,24:1) — **só** para texto grande em negrito (≥18,66px/700),
  cujo limite é 3:1. Usar em texto de corpo reprova AA.

Ao introduzir cor nova, verifique o contraste contra o fundo composto antes de commitar.

---

## Regenerar os assets de imagem

Nenhum destes arquivos é gerado no deploy — todos estão versionados. Só regere se mudar a
marca ou o texto do card social.

### Imagem OG (1200×630)

`assets/og-image.svg` é a fonte editável. Para rasterizar com a tipografia correta é preciso
que a Inter esteja disponível ao renderizador, então usa-se um wrapper HTML que declara a
`@font-face` apontando para o `.woff2` local:

```bash
cd "$(git rev-parse --show-toplevel)" && python3 - <<'EOF' && "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --disable-gpu --hide-scrollbars --allow-file-access-from-files --force-device-scale-factor=1 --screenshot=assets/og-image.png --window-size=1200,630 "file://$PWD/_og.html" && rm -f _og.html
import os, re
svg = open('assets/og-image.svg', encoding='utf-8').read()
svg = re.sub(r'<\?xml.*?\?>', '', svg)
css = ("@font-face{font-family:'Inter';src:url('%s/assets/fonts/inter-latin-var.woff2') format('woff2');font-weight:300 800;font-display:block}"
       "@font-face{font-family:'Fira Code';src:url('%s/assets/fonts/fira-code-latin-var.woff2') format('woff2');font-weight:300 700;font-display:block}"
       "*{margin:0;padding:0}html,body{width:1200px;height:630px;overflow:hidden;background:#020617}svg{display:block}") % (os.getcwd(), os.getcwd())
open('_og.html','w',encoding='utf-8').write('<!DOCTYPE html><meta charset="utf-8"><style>%s</style>%s' % (css, svg))
EOF
```

Confira o resultado: `sips -g pixelWidth -g pixelHeight assets/og-image.png` deve dizer
1200 × 630. Depois de publicar, force o LinkedIn a recarregar o cache pelo Post Inspector
(ver `docs/seo-checklist.md`).

### Foto de perfil

`assets/ismael-moreira-de-brito.jpg` é **auto-hospedada de propósito**. A URL original do
CDN do LinkedIn é assinada e temporária (expirava em 20/08/2026) e é host de terceiro —
seria bloqueada pela CSP e quebraria sozinha. Para trocar a foto:

```bash
sips -s format jpeg -s formatOptions 82 -z 512 512 nova-foto.png \
     --out assets/ismael-moreira-de-brito.jpg
```

O nome do arquivo é descritivo de propósito: é sinal de SEO de imagem. Se mudar o tamanho,
atualize `width`/`height` na tag `<img>` **e** no nó `#foto` do JSON-LD.

### Favicons e ícones

Derivados de `assets/favicon.svg`. Renderize a marca em 512 px sem cantos arredondados
(o iOS e o Android aplicam a própria máscara), depois reduza com `sips`:

```bash
sips -z 192 192 assets/icon-512.png --out assets/icon-192.png
sips -z 180 180 assets/icon-512.png --out assets/apple-touch-icon.png
```

O `favicon.ico` é um container com três PNGs (16, 32, 48 px). Para reconstruí-lo, gere os
três PNGs com `sips -z N N` e empacote:

```bash
python3 - <<'EOF'
import struct
srcs = [(16, '/tmp/i16.png'), (32, '/tmp/i32.png'), (48, '/tmp/i48.png')]
blobs = [(s, open(p, 'rb').read()) for s, p in srcs]
off = 6 + 16 * len(blobs)
entries, data = b'', b''
for size, blob in blobs:
    entries += struct.pack('<BBBBHHII', size, size, 0, 0, 1, 32, len(blob), off)
    data += blob; off += len(blob)
open('favicon.ico', 'wb').write(struct.pack('<HHH', 0, 1, len(blobs)) + entries + data)
EOF
```

---

## Armadilhas

**`.nojekyll` não pode sumir.** Sem esse arquivo, o GitHub Pages passa todo `.html` pelo
parser de template Liquid, que **consome silenciosamente** qualquer `{{` ou `{%`. O bloco
JSON-LD e o JavaScript ficariam corrompidos em produção sem erro nenhum, e funcionando
perfeitamente no seu servidor local. É o tipo de bug que custa uma tarde.

**A CSP no `<head>` é restritiva de propósito.** `default-src 'none'` — se você adicionar
qualquer script, fonte, imagem ou stylesheet de outro host (inclusive analytics), precisa
liberar o host na meta tag ou ele será bloqueado sem aviso visível na página. Se isso
atrapalhar mais do que ajuda, a tag é uma linha só e pode ser removida.

**Caminhos são absolutos.** `/assets/...`, não `assets/...`. Isso mantém tudo funcionando
se um dia surgir uma subpágina, e é por isso que `file://` não funciona.

**Uma URL canônica só.** `https://imbrito.com.br/` (apex, com barra final). O `www`, o
`imbrito.github.io` e o `http` todos redirecionam com 301 para lá. Se você mudar o `CNAME`,
tem que atualizar junto: `canonical`, `og:url`, `sitemap.xml`, todos os `@id` do JSON-LD e
o `Sitemap:` do `robots.txt`.

---

## Deploy

```bash
git add -A && git commit -m "feat: descrição da mudança" && git push
```

O GitHub Pages publica em cerca de um minuto. **Depois de publicar, siga
[`docs/seo-checklist.md`](docs/seo-checklist.md)** — há passos que só você pode executar
(Search Console, validadores, invalidação de cache do LinkedIn).
