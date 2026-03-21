
# Configuração e Deploy de um Site no GitHub Pages

## 1. Preparar o Repositório

```bash
git init
git add .
git commit -m "Initial commit"
```

## 2. Criar Repositório no GitHub

- Acesse [github.com](https://github.com)
- Clique em "New repository"
- Nomeie como `seu-usuario.github.io`
- Clique em "Create repository"

## 3. Adicionar Remote e Push

```bash
git branch -M main
git remote add origin https://github.com/seu-usuario/seu-usuario.github.io.git
git push -u origin main
```

## 4. Habilitar GitHub Pages

- Vá para **Settings** do repositório
- Na seção **Pages**, selecione a branch `main`
- Escolha a pasta `/root` ou `/docs`
- Clique em **Save**

## 5. Aguardar Deploy

- O GitHub Pages irá construir e publicar automaticamente
- Seu site estará disponível em: `https://seu-usuario.github.io`

## 6. Estrutura Mínima

```
seu-usuario.github.io/
├── index.html
├── style.css
└── README.md
```

## 7. Push de Atualizações

```bash
git add .
git commit -m "Update content"
git push
```

O deploy ocorre automaticamente a cada push na branch principal.

## 8. Configurar um Domínio Dedicado

### 8.1. Adquirir um Domínio

- Escolha um registrador de domínios (ex: Registro.br, GoDaddy, Namecheap, etc.)
- Registre e pague por seu domínio personalizado
- Anote os detalhes de acesso ao painel de controle do registrador

### 8.2. Criar o Arquivo CNAME

Crie um arquivo chamado `CNAME` na raiz do repositório com o nome do seu domínio:

```bash
echo "seu-dominio.com.br" > CNAME
```

**Conteúdo do arquivo CNAME:**
```
seu-dominio.com.br
```

> **Nota:** Use apenas o domínio, sem `https://` ou `www` neste arquivo

### 8.3. Configurar Registros DNS

Acesse o painel de controle do seu registrador de domínios e configure os registros DNS:

**Opção A - Usando registros A (recomendado):**

Adicione os seguintes registros A apontando para os IPs do GitHub Pages:

```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

Configure assim no seu registrador:
- Host: `@` (ou deixe em branco)
- Type: `A`
- Value: `185.199.108.153` (repita para cada IP)

**Opção B - Usando CNAME:**

Se o registrador permitir CNAME no domínio raiz:
- Host: `@` (ou deixe em branco)
- Type: `CNAME`
- Value: `seu-usuario.github.io`

Se quiser usar o subdomínio `www`:
- Host: `www`
- Type: `CNAME`
- Value: `seu-usuario.github.io`

### 8.4. Validar no GitHub Pages

- Vá para **Settings** do repositório
- Na seção **Pages**, no campo "Custom domain", insira seu domínio
- Clique em **Save**
- O GitHub irá verificar os registros DNS (pode levar até 24 horas)
- Após a validação, você pode marcar "Enforce HTTPS" para usar certificado SSL

### 8.5. Verificar a Configuração

```bash
# Verificar registros DNS (exemplo para Linux/Mac)
dig seu-dominio.com.br
nslookup seu-dominio.com.br

# Deve retornar os IPs do GitHub ou apontar para seu-usuario.github.io
```

### 8.6. Solução de Problemas

**Domínio não resolve:**
- Espere até 24 horas para propagação dos registros DNS
- Verifique se o arquivo CNAME está no repositório
- Confirme que os registros DNS foram adicionados corretamente no registrador

**HTTPS não funciona:**
- Aguarde mais tempo para o certificado SSL ser gerado
- Verifique se os registros DNS estão corretos
- Tente remover e adicionar novamente o domínio em GitHub Pages

**O site redireciona para GitHub Pages:**
- Verifique se o arquivo CNAME contém o domínio correto
- Faça push do arquivo CNAME para o repositório
- Aguarde alguns minutos para a propagação
