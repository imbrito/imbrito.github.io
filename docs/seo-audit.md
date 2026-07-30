# Auditoria técnica de SEO — imbrito.com.br

**Data:** 30/07/2026
**Escopo auditado:** `index.html` (262 linhas), `README.md`, `SECURITY.md`, `CNAME`, `.gitignore`
**Ambiente de produção verificado em runtime:** `https://imbrito.com.br` (GitHub Pages)

## Sumário executivo

O site é visualmente competente e **tecnicamente invisível**. Em 302 palavras de conteúdo
indexável não aparece uma única vez o nome do proprietário, não há `meta description`,
`canonical`, dados estruturados, `robots.txt`, `sitemap.xml` nem favicon. As três empresas
relevantes da carreira (Mercado Livre, Globo.com, B2W) aparecem apenas como texto decorativo
com `opacity-40`, colapsadas em um card genérico — o que impede o Google de associar a pessoa
a qualquer domínio de expertise.

Em performance, o site carrega **três dependências de terceiros bloqueantes** (Tailwind CDN,
Font Awesome CDN, Google Fonts via `@import`), sendo que a primeira é explicitamente
documentada pelo próprio Tailwind como "somente para desenvolvimento".

Contagem por severidade: **11 críticos**, **12 altos**, **9 médios**, **4 baixos**.

### Estado verificado da infraestrutura (bom — não precisa mexer)

Estes pontos foram testados e **já estão corretos**:

| Verificação | Resultado |
|---|---|
| `http://imbrito.com.br` → HTTPS | `301` → `https://imbrito.com.br/` ✅ Enforce HTTPS ativo |
| `https://www.imbrito.com.br` | `301` → `https://imbrito.com.br/` ✅ |
| `https://imbrito.github.io` | `301` → `https://imbrito.com.br/` ✅ |
| DNS apex | 4 registros A do GitHub Pages, corretos ✅ |
| HTTP/2 | Ativo ✅ |

**Consequência importante:** o único host canônico servindo conteúdo com `200` é o **apex**
(`https://imbrito.com.br/`). Ver item **C-01**.

---

## 1. Bloqueadores críticos

### C-01 — Domínio canônico: o pedido conflita com a realidade do deploy · CRÍTICO

O briefing pede `<link rel="canonical">` apontando para `https://www.imbrito.com.br/`.
Isso **não pode ser feito como está**: o arquivo `CNAME` contém o apex (`imbrito.com.br`),
e o GitHub Pages consequentemente redireciona `www` → apex com `301`.

Um `canonical` apontando para uma URL que responde `301` é um sinal contraditório: o Google
descarta o canonical declarado e escolhe a URL de destino por conta própria. Perde-se o
controle do sinal exatamente no ponto onde ele mais importa.

**Correção:** escolher **uma** das duas rotas, nunca as duas:
- **(A) Apex como canônico** — `canonical = https://imbrito.com.br/`. Zero mudança de
  infraestrutura, funciona hoje. Recomendado.
- **(B) `www` como canônico** — trocar `CNAME` para `www.imbrito.com.br`, criar registro
  DNS `CNAME www → imbrito.github.io`, e reemitir o certificado. O apex passa a redirecionar
  para `www`. Requer ~24h de propagação e reemissão de TLS pelo GitHub.

Todas as URLs absolutas do projeto (`canonical`, `og:url`, `sitemap.xml`, `@id` do JSON-LD,
`sameAs`) precisam usar **exatamente** o host escolhido, com barra final consistente.

### C-02 — Nome real ausente de todo o HTML · CRÍTICO

"Ismael Moreira de Brito" não aparece em `<title>`, `description` (que não existe), `<h1>`,
`og:*` ou no corpo do texto. A única pista de identidade é a string `IMBRITO` no logotipo do
nav e o `mailto:`.

Isso torna **impossível** rankear para a busca de maior intenção de conversão: o próprio nome.
Hoje, quem busca "Ismael Moreira de Brito" encontra o LinkedIn e nada mais.

**Correção:** nome completo em `title`, `meta description`, `<h1>`, JSON-LD `Person.name`,
`og:title`, e ao menos uma vez em prosa natural na seção "Sobre".

### C-03 — `<meta name="description">` inexistente · CRÍTICO

Sem description, o Google sintetiza o snippet da SERP a partir do primeiro texto que
encontrar — hoje, provavelmente "Construindo Ecossistemas de Dados Escaláveis". Perde-se o
controle do único texto de marketing que o recrutador lê antes de decidir clicar.

**Correção:** 150–160 caracteres, com nome + cargo + empresas + localização.

### C-04 — Sem `<link rel="canonical">` · CRÍTICO

Embora os redirects já consolidem os hosts, a ausência de canonical deixa vulneráveis:
variações com query string (`?utm_source=...`, `?fbclid=...`) e com fragmento, que o
Google pode tratar como URLs distintas.

**Correção:** `<link rel="canonical" href="https://imbrito.com.br/">` (auto-referencial).

### C-05 — Zero dados estruturados · CRÍTICO

Nenhum JSON-LD, microdata ou RDFa. Sem isso o site não é elegível a rich results, não
alimenta Knowledge Graph, e crawlers de LLM precisam inferir a entidade a partir de prosa —
com resultado imprevisível.

**Correção:** um bloco `application/ld+json` com `@graph` contendo `Person`, `ProfilePage`
e `WebSite`. Detalhes na Fase 2 do plano.

### C-06 — Sem `robots.txt` (404 confirmado) · CRÍTICO

`https://imbrito.com.br/robots.txt` → **404**. Não é bloqueante para indexação (ausência
= tudo liberado), mas impede apontar o sitemap e impede declarar explicitamente permissão
para `GPTBot`, `ClaudeBot`, `PerplexityBot`, `Google-Extended`.

### C-07 — Sem `sitemap.xml` (404 confirmado) · CRÍTICO

`https://imbrito.com.br/sitemap.xml` → **404**. Sem sitemap não há sinal de `lastmod`, o que
para um site de página única é o principal mecanismo de "por favor, recrawleie, o conteúdo
mudou".

### C-08 — `og:image` comentado; sem Twitter Cards · CRÍTICO

`index.html:10` — a tag está literalmente dentro de um comentário HTML. Ausentes também:
`og:url`, `og:locale`, `og:site_name`, `og:image:alt`, `og:image:width/height`, e o bloco
inteiro de `twitter:*`. Além disso, `og:type` está como `website` quando deveria ser `profile`.

Impacto direto: compartilhar o link no LinkedIn ou WhatsApp — exatamente o canal onde um
recrutador recebe o link — gera um card cinza sem imagem.

### C-09 — `<html lang="pt-pt">` · CRÍTICO

`index.html:2`. Declara português europeu. Sinal errado de idioma e de geografia para
Google e Bing, e conflita com o alvo "Rio de Janeiro". Deve ser `pt-BR`.

### C-10 — Tailwind CDN compilando CSS no browser · CRÍTICO (performance)

`index.html:12` — `https://cdn.tailwindcss.com`. A própria documentação do Tailwind declara
este CDN como *"for development purposes only"*. O script baixa ~100 KB de JavaScript,
faz parsing do DOM, gera CSS em runtime e injeta no `<head>`. Consequências: JS bloqueante,
TBT alto, e FOUC — a página aparece sem estilo até o script terminar.

**Correção:** substituir por CSS próprio, apenas com as classes efetivamente usadas,
inline no `<head>`. O CSS real deste layout cabe em ~8–10 KB não comprimido.

### C-11 — Conteúdo raso e não verificável · CRÍTICO

302 palavras, sem uma única data, número, produto ou realização atribuível. O texto usa
adjetivos de LinkedIn ("sólida trajetória", "gigantes como", "soluções sustentáveis") no
lugar de fatos. As três empresas são mencionadas em uma frase e depois como logotipos
decorativos com `opacity-40`.

O caso mais grave é o card **"Especialista em Ecossistemas de Dados — Mercado Livre / Globo /
B2W"** (`index.html:157-158`): três empregos, três períodos e três escopos distintos
colapsados em uma entrada sem data. Isso destrói toda possibilidade de associação de
entidades — o Google não consegue inferir "esta pessoa trabalha no Mercado Livre desde 2022".

Não há nenhum sinal de E-E-A-T: sem formação, sem publicação, sem tecnologia em contexto de
uso real.

---

## 2. Severidade alta

### A-01 — Sem `<main>` e hierarquia de headings quebrada · ALTO

Problemas encontrados, em ordem de documento:
- `index.html:86` — `<h2>` (eyebrow "Construindo Ecossistemas...") **antes** do `<h1>`
- `index.html:87` — `<h1>` correto em quantidade (um só), mas sem o nome
- `index.html:113` — seção `#dominios` **sem heading algum**
- `index.html:119,128,137` — salto direto para `<h4>` (não existe `h3` nesse ramo)
- `index.html:211,220` — `<h5>` sob um `<h3>`, pulando `h4`
- `index.html:236` — `<h2>` no footer, hierarquicamente errado

Não existe `<main>`. Leitores de tela não têm landmark de conteúdo principal, e o Google
perde o mapa de tópicos da página.

### A-02 — `href` em `<div>` e `<span>`: HTML inválido · ALTO

`index.html:68` — `<div href="#" class="w-8 h-8 ...">#</div>`
`index.html:69` — `<span href="#" class="mono ...">IMBRITO</span>`

`href` não é atributo válido em `div`/`span`. Reprova no validador do W3C e os elementos
não são clicáveis nem focáveis por teclado — a intenção era claramente um link para o topo.

### A-03 — Links de ícone sem nome acessível · ALTO

`index.html:239-240` — GitHub e LinkedIn são `<a>` contendo apenas `<i class="fab fa-...">`.
Font Awesome renderiza via pseudo-elemento `::before`, portanto o nome acessível calculado é
**string vazia**. Um leitor de tela anuncia "link" sem destino. Também faltam `aria-label`
e `rel="noopener noreferrer"`.

### A-04 — Contraste reprovando WCAG AA · ALTO

Ratios calculados contra o fundo `#020617`:

| Elemento | Cor efetiva | Ratio | AA normal (4,5:1) |
|---|---|---|---|
| Copyright do footer (`text-slate-700`) | `#334155` | **1,95** | ❌ reprova até para texto grande |
| Logos das empresas (`opacity-40`) | `#525968` | **2,87** | ❌ |
| Footer parágrafo e ícones (`text-slate-500`) | `#64748b` | **4,24** | ❌ (falta pouco) |
| Parágrafos (`text-slate-400`) | `#94a3b8` | 7,87 | ✅ |
| Body (`text-slate-300`) | `#cbd5e1` | 13,59 | ✅ |
| Destaque do h1 (`text-indigo-500`) | `#6366f1` | 4,52 | ✅ (no limite) |

O copyright em `1,95:1` é praticamente invisível. Os logos em `opacity-40` são o pior caso
porque carregam informação relevante de SEO/credibilidade (os nomes das empresas) em contraste
ilegível.

### A-05 — Font Awesome inteiro para 3 ícones · ALTO (performance)

`index.html:13` — CSS de ~30 KB + arquivos de fonte de ~150 KB baixados de
`cdnjs.cloudflare.com` para renderizar `fa-microchip`, `fa-brain`, `fa-shield-halved`,
`fa-github` e `fa-linkedin`. Bloqueante de render (é `<link rel="stylesheet">`).

**Correção:** SVG inline. Custo real: ~1,5 KB para os cinco ícones.

### A-06 — `@import` de Google Fonts dentro de `<style>` · ALTO (performance)

`index.html:15`. É o pior padrão possível de carregamento de fonte: o browser precisa
(1) baixar o HTML, (2) parsear o `<style>`, (3) descobrir o `@import`, (4) resolver DNS de
`fonts.googleapis.com`, (5) baixar o CSS, (6) descobrir o `@font-face`, (7) resolver DNS de
`fonts.gstatic.com`, (8) baixar o `.woff2`. Oito passos serializados antes do primeiro
glifo. Nenhum `preconnect`.

### A-07 — Sem favicon, `apple-touch-icon` ou `theme-color` · ALTO

Nenhum ícone declarado. O browser requisita `/favicon.ico` e recebe 404 (uma requisição
desperdiçada em toda navegação). Aba sem identidade, favorito sem identidade, "adicionar
à tela de início" no iOS gera um screenshot da página como ícone.

### A-08 — Handler de scroll com layout thrashing — e que não funciona · ALTO

`index.html:248-259`. Dois problemas:

1. **Performance:** `scroll` sem throttle, chamando `getBoundingClientRect()` em N elementos
   por evento. Cada chamada força *layout reflow* sincronizado. Em scroll rápido isso são
   centenas de reflows por segundo.
2. **É código morto.** O script *seta* `opacity: 1` e `transform: translateY(0)`, mas
   nenhum dos alvos (`.glass-card`, `.relative.pl-12`) tem `opacity: 0` como estado inicial.
   Não há animação para revelar. A classe `.reveal` (que tem a animação real) é aplicada
   apenas ao hero, que já está visível no load. Além disso, nada executa antes do primeiro
   evento de scroll.

**Correção:** `IntersectionObserver`, com estado inicial de fato oculto e fallback para
`prefers-reduced-motion`.

### A-09 — Três dependências de terceiros em runtime · ALTO

`cdn.tailwindcss.com`, `cdnjs.cloudflare.com`, `fonts.googleapis.com`, `fonts.gstatic.com`.
Quatro hosts, quatro handshakes TLS, quatro pontos de falha, e o IP de cada visitante
compartilhado com três empresas distintas sem base legal nem aviso.

Vale registrar o ponto de posicionamento: o site de alguém que se apresenta como
especialista em LGPD faz transferência internacional de dados pessoais (IP + User-Agent)
para três terceiros sem consentimento. É uma inconsistência que um entrevistador atento nota.

### A-10 — Sem `prefers-reduced-motion` · ALTO (acessibilidade)

`index.html:55-59` define `@keyframes fadeIn` com `translateY(20px)`, e
`index.html:2` usa `scroll-smooth`. Nenhum dos dois respeita a preferência do sistema por
movimento reduzido — gatilho conhecido de desconforto vestibular.

### A-11 — Sem navegação em mobile · ALTO

`index.html:71` — o menu é `class="hidden md:flex"`. Abaixo de 768px **não existe
navegação alguma**, e não há botão de menu, drawer ou substituto. Em mobile o usuário só tem
o botão "CONTATO" e scroll manual por ~5 telas.

### A-12 — Sem skip-link e sem estilo de foco explícito · ALTO

Não há link "pular para o conteúdo". Todos os estilos de interação são `hover:*`; nenhum
`focus-visible`. Navegação por teclado depende do outline default do browser, que em fundo
escuro é de baixa visibilidade — e será removido junto com o reset caso o CSS próprio
inclua um.

---

## 3. Severidade média

### M-01 — Ausência de `.nojekyll` · MÉDIO

O GitHub Pages roda Jekyll por padrão em repositórios sem esse arquivo, o que significa que
todo `.html` passa pelo parser de template Liquid. Sequências `{{` e `{%` são interpretadas
e removidas do output.

Isso é um **risco concreto** para este projeto: vamos adicionar um bloco JSON-LD e um
`site.webmanifest`. Qualquer construção que gere `{{` (por exemplo CSS com `@media` aninhado
mal formatado, ou um template de string em JS) pode ser silenciosamente corrompida no deploy
sem erro visível localmente.

**Correção:** criar `.nojekyll` vazio na raiz. Bônus: builds mais rápidos.

### M-02 — Sem `404.html` · MÉDIO

`https://imbrito.com.br/pagina-inexistente` → 404 com a página genérica do GitHub Pages,
fora da identidade visual e sem link de retorno.

### M-03 — Âncoras aterrissam sob o nav fixo · MÉDIO

O nav é `fixed` com `h-16` (64px). As seções-alvo não têm `scroll-margin-top`. Clicar em
"Trajetória" posiciona o topo da seção exatamente atrás da barra, escondendo o heading.

### M-04 — E-mail exposto em texto puro · MÉDIO

`index.html:77` — `href="mailto:imbrito1986@gmail.com"`. Coletável por qualquer scraper
trivial. Como o endereço é pessoal e de uso permanente, o custo de vazamento é alto.

### M-05 — `SECURITY.md` é o template do GitHub, não editado · MÉDIO

O arquivo inteiro é boilerplate: instruções para o autor ("Use this section to tell people
about which versions..."), e uma tabela de versões fictícias `5.1.x` / `4.0.x` que não têm
relação alguma com um site estático de página única.

Não afeta o SEO do site diretamente, mas o repositório é público e será um dos alvos de
`sameAs`. Para quem avalia o perfil técnico, é um artefato copiado sem revisão — o mesmo
problema editorial dos lusismos, em outro arquivo.

### M-06 — `README.md` não documenta o projeto · MÉDIO

São 147 linhas de tutorial genérico sobre como publicar qualquer site no GitHub Pages,
com placeholders `seu-usuario` e `seu-dominio.com.br`. Não há uma linha sobre a estrutura
*deste* projeto, como rodá-lo localmente, ou o que verificar após o deploy.

### M-07 — `.gitignore` é template Ruby/Jekyll irrelevante · MÉDIO

76 linhas geradas por `gitignore.io` para Ruby + Jekyll: `.byebug_history`, `.rvmrc`,
`build-iPhoneSimulator/`, `/vendor/bundle`. O projeto não tem Ruby. Nada aqui quebra o
build, mas duas entradas merecem atenção: `/doc/` (ignorado — usaremos `docs/`, que é
diferente, ok) e `build/` (ignorado — evitar esse nome se criarmos scripts de geração).

### M-08 — Dependência de `scroll-smooth` do Tailwind · MÉDIO

`index.html:2` — `class="scroll-smooth"`. Ao remover o Tailwind CDN, o scroll suave
desaparece silenciosamente. Precisa virar `scroll-behavior: smooth` no CSS próprio,
condicionado a `prefers-reduced-motion`.

### M-09 — `min-h-screen` usando `100vh` em mobile · MÉDIO

`index.html:84`. Em Safari/Chrome mobile, `100vh` inclui a área da barra de endereço, que
some ao rolar — o hero "salta" de altura. Usar `100svh` com fallback para `100vh`.

---

## 4. Severidade baixa

### B-01 — `<h2>` usado como eyebrow decorativo · BAIXO

`index.html:86` — heading escolhido por aparência, não por estrutura. É a causa raiz do
`h2`-antes-do-`h1` em A-01; registro separado porque a correção é semântica (virar `<p>`),
não de reordenação.

### B-02 — Sem `og:image:width` / `og:image:height` · BAIXO

Sem as dimensões declaradas, LinkedIn e WhatsApp precisam baixar a imagem antes de decidir
o layout do card, o que atrasa (e às vezes falha) o primeiro preview.

### B-03 — Sem `meta name="robots"` explícito · BAIXO

O default (`index, follow`) já é o desejado, mas declarar `index, follow, max-image-preview:large`
habilita preview de imagem grande na SERP e no Discover — relevante justamente porque
teremos uma imagem OG bem feita.

### B-04 — Sem `Content-Security-Policy` · BAIXO

Não é possível enviar headers no GitHub Pages, mas após eliminar todos os terceiros o site
passa a ser 100% self-contained, e uma CSP restritiva via
`<meta http-equiv="Content-Security-Policy">` torna-se trivial de escrever e efetiva.
Hardening opcional, coerente com o posicionamento de quem mantém um `SECURITY.md`.

---

## 5. Itens do briefing verificados como corretos

Para fechar o escopo, estes pontos do diagnóstico prévio foram checados e **não** são
problemas:

- **Nenhuma imagem sem `alt`** — o site não tem um único `<img>`. Portanto não há `alt`
  faltando; há uma **oportunidade não usada** de SEO de imagem (a ser criada: imagem OG,
  favicons, e opcionalmente uma foto de perfil, que também alimentaria `Person.image`).
- **CLS** — sem imagens e sem conteúdo assíncrono, a única fonte de CLS hoje é o swap de
  fonte. Definir `width`/`height` "em toda imagem" só se aplica às imagens que vamos
  introduzir.
- **HTTPS e consolidação de host** — já corretos, conforme tabela no sumário.

---

## 6. Lusismos e problemas editoriais encontrados

| Linha | Atual | Correção |
|---|---|---|
| `index.html:139` | monitorização | monitoramento |
| `index.html:180` | equipas | equipes |
| `index.html:2` | `lang="pt-pt"` | `lang="pt-BR"` |

Adjetivação vazia a remover (não é lusismo, é ruído de relevância):
"sólida trajetória" (`:91`), "gigantes como" (`:91`), "soluções sustentáveis" (`:163`),
"Base sólida" (`:199`), "rentabilidade organizacional e escala sustentável" (`:237`).

---

## 7. Baseline para comparação pós-refatoração

Medido no código atual, antes de qualquer mudança:

| Métrica | Antes |
|---|---|
| Palavras de conteúdo indexável | 302 |
| Requisições a terceiros | 4 hosts (Tailwind, cdnjs, fonts.googleapis, fonts.gstatic) |
| JS de terceiros | ~100 KB (Tailwind CDN) + fonte de ícones ~150 KB |
| Meta tags de SEO | 0 (`description`, `canonical`, `robots`, `author` ausentes) |
| Tags Open Graph funcionais | 3 de ~9 (`og:image` comentado) |
| Twitter Cards | 0 |
| Blocos JSON-LD | 0 |
| Menções ao nome "Ismael Moreira de Brito" | 0 |
| Entradas de experiência datadas | 0 |
| Erros de validação W3C esperados | ≥ 2 (`href` em `div`/`span`) |
| Elementos reprovando contraste AA | 3 classes (`slate-700`, `slate-500`, `opacity-40`) |
