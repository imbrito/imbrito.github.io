# Política de segurança

Este repositório contém o site estático pessoal de Ismael Moreira de Brito, publicado em
<https://imbrito.com.br> pelo GitHub Pages.

## Escopo

O site é inteiramente estático: HTML, CSS e cerca de 40 linhas de JavaScript. Não há
backend, banco de dados, formulário, autenticação, sessão, cookie nem `localStorage`.
Nenhum dado de visitante é coletado, e o site não faz nenhuma requisição a terceiros —
o que é garantido por uma `Content-Security-Policy` com `default-src 'none'` declarada
em `index.html`.

Não existe versionamento de release: o que está na branch `main` é o que está no ar.
Por isso não há tabela de "versões suportadas" — só existe uma.

Superfície relevante:

- o conteúdo servido pelo GitHub Pages
- a configuração de DNS e do domínio customizado (`CNAME` + registros A)
- o próprio repositório e suas permissões

## Como reportar

Prefira o canal privado do GitHub:

**<https://github.com/imbrito/imbrito.github.io/security/advisories/new>**

Se não conseguir, use o LinkedIn: <https://www.linkedin.com/in/imbrito>

Por favor **não abra issue pública** para algo que exponha o domínio ou o repositório a
abuso. Confirmo o recebimento em até 7 dias.

## O que vale reportar

- conteúdo injetado ou desfiguração da página publicada
- comprometimento do domínio, do DNS ou do certificado
- bypass da `Content-Security-Policy` declarada
- qualquer requisição a host de terceiro partindo do site (deveriam ser zero)
- exposição de dado pessoal além do que está publicado deliberadamente

## Fora de escopo

- ausência de cabeçalhos HTTP que o GitHub Pages não permite configurar
  (HSTS customizado, `X-Frame-Options`, `Referrer-Policy`, `Permissions-Policy`)
- saída de scanner automatizado sem impacto demonstrável
- ausência de SPF/DKIM/DMARC: este domínio não envia e-mail
- engenharia social, phishing ou ataque de negação de serviço
- versões de dependências: o projeto não tem nenhuma
