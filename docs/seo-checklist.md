# Checklist pós-deploy

Passos que **só você pode executar** — exigem login, propriedade do domínio, ou leem a URL
pública e portanto não funcionam antes do push.

URL canônica: **`https://imbrito.com.br/`** (apex, com barra final). Use exatamente essa
forma em toda ferramenta abaixo.

---

## 0. Antes de qualquer coisa: confirme que o deploy subiu

```bash
curl -sI https://imbrito.com.br/ | head -3
curl -s https://imbrito.com.br/ | grep -c "Ismael Moreira de Brito"
```

O segundo comando deve retornar um número maior que zero. Se retornar `0`, o GitHub Pages
ainda está publicando — espere um minuto e repita.

Depois confirme que os arquivos novos estão sendo servidos com o tipo certo:

```bash
for p in /robots.txt /sitemap.xml /site.webmanifest /favicon.ico \
         /assets/og-image.png /assets/favicon.svg \
         /assets/fonts/inter-latin-var.woff2; do
  printf "%-42s " "$p"
  curl -s -o /dev/null -w "%{http_code}  %{content_type}\n" "https://imbrito.com.br$p"
done
```

Todos devem responder `200`. E a 404 precisa responder **404**, não 200:

```bash
curl -s -o /dev/null -w "%{http_code}\n" https://imbrito.com.br/pagina-que-nao-existe
```

---

## 1. Google Search Console

- [x] Adicionar a propriedade em <https://search.google.com/search-console>

  Escolha **"Prefixo do URL"** com `https://imbrito.com.br/`, não "Domínio" — a verificação
  por prefixo aceita uma meta tag no `<head>`, enquanto "Domínio" exige registro TXT no DNS.
  Se preferir a verificação por DNS (cobre todos os subdomínios de uma vez), você vai
  precisar de acesso ao painel do registrador.

- [x] **Meta tag de verificação já está no ar** (adicionada em 30/07/2026, no `<head>` do
      `index.html`, logo abaixo de `<meta name="robots">`). **Não remova** — o Google
      revalida periodicamente e a propriedade cai se a tag sumir.

- [x] Enviar o sitemap: menu **Sitemaps** → `sitemap.xml` → Enviar.

- [x] Usar a **Inspeção de URL** em `https://imbrito.com.br/` e clicar em
      "Solicitar indexação". Isso enfileira o rastreamento em vez de esperar o ciclo natural.

- [ ] Em ~3 dias, verificar em **Páginas** se a URL aparece como "Indexada". Em ~2 semanas,
      verificar em **Desempenho** se já há impressões para "ismael moreira de brito".

## 2. Bing Webmaster Tools

- [x] <https://www.bing.com/webmasters> — permite **importar a propriedade direto do Google
      Search Console**, o que evita repetir a verificação. Vale o esforço: o Bing alimenta
      o índice usado pelo ChatGPT Search e pelo Copilot.
- [x] Enviar o mesmo `sitemap.xml`.

---

## 3. Validar os dados estruturados

- [x] **Schema Markup Validator** — esta é a ferramenta certa:
      <https://validator.schema.org/#url=https%3A%2F%2Fimbrito.com.br%2F>

      Esperado: 5 itens (`Person`, `ProfilePage`, `WebSite`, `ImageObject`,
      `ScholarlyArticle`), **0 erros**. Avisos sobre propriedades opcionais ausentes são
      aceitáveis.

- [x] **Rich Results Test** — <https://search.google.com/test/rich-results?url=https%3A%2F%2Fimbrito.com.br%2F>

      > ⚠️ Ele vai dizer **"Nenhum item elegível para rich results detectado"**. **Isso não
      > é erro.** `Person` não gera enfeite na SERP para pessoas físicas, então a ferramenta
      > não reporta nada. O que importa aqui é a aba de erros de sintaxe estar vazia.

- [x] Conferir se o `Person` bate com o LinkedIn: mesmo nome completo, mesma cidade, mesmos
      cargos e datas. Divergência entre o site e o perfil enfraquece a consolidação da
      entidade — que é justamente o objetivo.

---

## 4. Validar o HTML

- [x] <https://validator.w3.org/nu/?doc=https%3A%2F%2Fimbrito.com.br%2F>
- [x] Repetir para a 404: `https://imbrito.com.br/404.html`

Esperado: zero erros. Avisos do tipo "Section lacks heading" não devem aparecer — todas as
seções têm `aria-labelledby` apontando para um heading real.

---

## 5. Cards sociais

Os validadores **cacheiam agressivamente**. Se você já compartilhou o link antes desta
refatoração, o card velho (vazio) pode estar preso no cache por semanas — e os passos
abaixo são a única forma de invalidar.

- [x] **LinkedIn Post Inspector** — <https://www.linkedin.com/post-inspector/>

      Cole `https://imbrito.com.br/` e clique em **"Inspect"**. Isso força o refetch. Você
      deve ver a imagem 1200×630 com seu nome, o título e a descrição. É o mais importante
      dos três: é onde recrutador clica.

- [x] **Facebook / WhatsApp Sharing Debugger** — <https://developers.facebook.com/tools/debug/>

      O WhatsApp usa o mesmo cache do Facebook. Clique em **"Scrape Again"**.

- [x] **X / Twitter** — <https://cards-dev.twitter.com/validator> (exige login). Opcional:
      não há handle configurado nas meta tags, porque você não me passou um.

- [x] Teste real: mande o link para você mesmo no WhatsApp e veja o card renderizar.

---

## 6. Performance — a medição que falta

Nada disso foi medido localmente. **A nota só existe contra a URL publicada.**

- [x] **PageSpeed Insights** — <https://pagespeed.web.dev/analysis?url=https%3A%2F%2Fimbrito.com.br%2F>

      Rode **mobile e desktop**. Metas: 100 em Performance, Acessibilidade, Práticas
      recomendadas e SEO. LCP < 1,5 s, CLS < 0,05, TBT ~0.

      Se Performance vier abaixo de 100, o suspeito número um é o **LCP da fonte**: o `<h1>`
      usa Inter com `font-display: swap`. O `.woff2` da Inter tem `preload`, então deveria
      chegar junto com o HTML — mas confirme na aba "Diagnóstico" se aparece
      "Ensure text remains visible during webfont load".

- [x] Se der 100/100/100/100 nos dois, **salve os prints**. É o baseline para detectar
      regressão depois de qualquer edição futura.

- [x] Opcional: <https://www.webpagetest.org> a partir de São Paulo, para ver a latência
      real do CDN do GitHub Pages para o Brasil.

### Baseline registrado — 30/07/2026

Mobile, Lighthouse 13.4.1, Moto G Power emulado, limitação lenta de 4G.
Duas execuções no mesmo dia, antes e depois da correção de CLS.

**Execução 1 — commit `0225ad4`, antes da correção**

| | Valor | Score | Peso | Pontos |
|---|---|---|---|---|
| First Contentful Paint | 1,1 s | 1,00 | 10 | 10,00 |
| Largest Contentful Paint | 1,7 s | 0,99 | 25 | 24,75 |
| Total Blocking Time | 0 ms | 1,00 | 30 | 30,00 |
| Cumulative Layout Shift | **0,109** | 0,87 | 25 | 21,75 |
| Speed Index | 2,4 s | 0,98 | 10 | 9,80 |
| | | | | **96** |

Categorias: Desempenho **96**, Acessibilidade **100**, Práticas recomendadas **100**,
SEO **100**.

Um único layout shift, atribuído pelo Lighthouse a
`main#conteudo > section#perfil > div.wrap > ul.logos`. A origem, porém, era o parágrafo
acima: `.hero-lead` passava de 5 para 4 linhas no swap da fonte e empurrava 27,2 px de
conteúdo. Corrigido em `681ddf0` com o `@font-face 'Inter Fallback'` de métricas casadas.

**Execução 2 — commit `681ddf0`, depois da correção**

Categorias: Desempenho **100**, Acessibilidade **100**, Práticas recomendadas **100**,
SEO **100**.

> Os valores individuais das cinco métricas desta execução não foram capturados — só as
> quatro notas de categoria. Se quiser um baseline completo para comparação futura, rode o
> PageSpeed de novo e cole os cinco tempos aqui.

**Como usar este baseline.** Depois de qualquer edição no `index.html`, rode o PageSpeed e
compare. Se o Desempenho cair, a suspeita nº 1 é CLS, e a nº 2 é peso novo no caminho
crítico. Reproduzir CLS localmente exige **viewport de 412×823** — foi medir a 375 px que
me fez estimar 0,008 quando o real era 0,109. Quebra de linha é função degrau: um parágrafo
na fronteira de quebra desloca uma linha inteira, não uma fração proporcional.

---

## 7. Acessibilidade — o que ferramenta automática não pega

O contraste já foi verificado programaticamente em todos os nós de texto (zero reprovações
em WCAG AA). O que falta é o que só teste manual pega:

- [x] Navegar a página inteira **só com Tab**, sem mouse. O foco deve ser sempre visível
      (contorno indigo) e a ordem deve ser: skip-link → marca → 4 links do menu → CONTATO →
      2 botões do hero → 3 botões de contato → 2 ícones do rodapé.
- [ ] Pressionar Tab uma vez ao carregar: o link "Pular para o conteúdo principal" deve
      aparecer no topo.
- [ ] Ativar "Reduzir movimento" no sistema (macOS: Ajustes → Acessibilidade → Tela) e
      recarregar: nada deve animar, e todo o conteúdo deve estar visível.
- [ ] Testar com VoiceOver (`Cmd+F5`) percorrendo os headings (`Ctrl+Opt+Cmd+H`): a sequência
      deve ser Ismael Moreira de Brito → Sobre → Experiência → Stack → Formação e publicação
      → Vamos conversar.
- [ ] Desativar o JavaScript e recarregar. Todo o conteúdo deve continuar visível — e o
      botão de e-mail deve mostrar `imbrito1986 [arroba] gmail.com`, ainda legível.

---

## 8. Domínio e HTTPS

Já verificado em 30/07/2026 e **está correto** — reconfira só se mexer no DNS:

| Verificação | Estado |
|---|---|
| `http://imbrito.com.br` → 301 → HTTPS | ✅ Enforce HTTPS ativo |
| `https://www.imbrito.com.br` → 301 → apex | ✅ |
| `https://imbrito.github.io` → 301 → apex | ✅ |
| Registros A do GitHub Pages | ✅ os quatro |

```bash
for u in http://imbrito.com.br https://www.imbrito.com.br https://imbrito.github.io; do
  printf "%-32s " "$u"
  curl -sI "$u" | awk '/^HTTP|^[Ll]ocation/{printf "%s ", $2} END{print ""}'
done
```

- [x] Em **Settings → Pages**, confirmar que "Enforce HTTPS" continua marcado.

---

## 9. Repositório

- [ ] **Settings → Code security → Private vulnerability reporting: Enable.** Sem isso, o
      link de advisory que o `SECURITY.md` divulga retorna 404.
- [ ] Preencher a descrição e o site do repositório (aparece na busca do GitHub e reforça
      o `sameAs` do JSON-LD).
- [x] Conferir se o perfil do GitHub aponta para `imbrito.com.br` e se o LinkedIn tem o site
      no campo de website. `sameAs` é **bidirecional na prática**: os três apontando entre si
      é o que consolida a entidade.

---

## 10. Analytics

**Decisão tomada em 30/07/2026: nenhum, por enquanto.** O Search Console já entrega
impressões, cliques, CTR e posição média por consulta — que é a métrica que importa para o
objetivo deste site. Analytics de página agrega pouco num one-pager.

Se um dia mudar de ideia, em ordem de preferência:

| | Custo | Cookies | Peso | Observação |
|---|---|---|---|---|
| **GoatCounter** | grátis (pessoal) | não | ~3 KB | o mais leve; sem dado pessoal |
| **Plausible** | ~US$ 9/mês | não | ~1 KB | painel melhor, LGPD/GDPR |
| **Cloudflare Web Analytics** | grátis | não | ~1 KB | exige passar o DNS pela Cloudflare |

**GA4 não está na lista de propósito**: cookies, transferência internacional de dados
pessoais e um banner de consentimento que você teria que construir e manter — num site cujo
diferencial declarado inclui LGPD.

> ⚠️ Qualquer um dos três **vai ser bloqueado pela CSP** até você liberar o host. Adicione
> o domínio do script em `script-src` e `connect-src` na meta tag do `<head>`. Sem isso o
> analytics falha silenciosamente e você vai achar que o site não tem visita.

---

## 11. Acompanhamento

| Quando | O quê |
|---|---|
| 3 dias | Search Console → Páginas: URL indexada? |
| 2 semanas | Search Console → Desempenho: impressões para "ismael moreira de brito"? |
| 1 mês | Buscar o próprio nome numa janela anônima. Posição? |
| 1 mês | Perguntar ao ChatGPT / Claude / Perplexity "quem é Ismael Moreira de Brito?" e ver se citam o site |
| Trimestral | Rodar PageSpeed de novo e comparar com o baseline do item 6 |

### Expectativa calibrada

O trabalho técnico te torna **elegível**; ele não compra posição. Para a busca pelo seu
nome, o LinkedIn tem autoridade de domínio incomparavelmente maior e provavelmente vai
liderar por um bom tempo. O que decide no médio prazo é a consistência dos sinais entre
site, LinkedIn e GitHub — e tempo de domínio.

O vetor de crescimento que este site ainda não tem é **conteúdo novo e recorrente**. Uma
página estática, por melhor que esteja, compete por um punhado de termos. A seção de
artigos ficou deliberadamente de fora (decisão de 30/07/2026) e continua sendo a maior
alavanca disponível quando você quiser puxá-la.
