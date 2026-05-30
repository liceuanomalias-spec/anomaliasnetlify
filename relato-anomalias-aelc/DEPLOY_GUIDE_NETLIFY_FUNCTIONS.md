# 📘 Guia Detalhado: Deploy da App no Netlify (Passo a Passo)

> **Tempo estimado**: 45-60 minutos na primeira vez
> **Custo**: 0€/mês (tudo em planos gratuitos)
> **Resultado**: URL público estável tipo `https://anomalias-aelc.netlify.app` que toda a comunidade escolar pode usar

---

## 📋 O que vai precisar antes de começar

Tenha à mão:
- ✅ Um email que vai usar para criar 3 contas (MongoDB Atlas, GitHub e Netlify)
- ✅ A App Password do Gmail (`xeog btmj yruk puyv`)
- ✅ ~1 hora sem interrupções

Não precisa de ter:
- ❌ Cartão de crédito (tudo grátis)
- ❌ Conhecimentos técnicos (este guia explica tudo)
- ❌ Instalar nada no seu computador

---

# Parte 1 · MongoDB Atlas (Base de Dados Grátis)

> **O que é?** Um serviço da MongoDB que oferece base de dados gratuita na nuvem.
> **Para que serve?** Guardar os relatos, utilizadores e destinatários.

## 1.1 · Criar Conta

1. Abra https://www.mongodb.com/cloud/atlas/register no browser
2. Preencha:
   - Nome
   - Email
   - Password (mínimo 8 caracteres)
3. Aceite os termos → **"Create your Atlas account"**
4. Verifique o email e clique no link de confirmação

## 1.2 · Criar o Cluster (servidor de base de dados)

Depois de fazer login, vai ver uma página com escolhas:

1. Quando perguntarem **"What kind of database would you like?"** → escolha **M0 FREE** (o cartão amarelo grátis, NÃO o azul Dedicated)
2. **Provider**: AWS (escolha o que estiver pré-selecionado)
3. **Region**: escolha **Frankfurt (eu-central-1)** ou **Ireland (eu-west-1)** — está mais próximo de Portugal
4. **Cluster Name**: deixe `Cluster0` ou mude para `aelc-anomalias`
5. Clique **"Create Deployment"**
6. Aguarde 1-3 minutos enquanto criam o cluster

## 1.3 · Criar o Utilizador da Base de Dados

Assim que o cluster terminar, aparece uma janela "Connect to Cluster0":

1. Em **"Create a database user"**:
   - **Username**: `aelc_admin`
   - **Password**: clique **"Autogenerate Secure Password"** e **GUARDE A PASSWORD** num bloco de notas (vai precisar dela já a seguir)
2. Clique **"Create Database User"**

## 1.4 · Permitir Ligação de Qualquer Lado

Na mesma janela, em **"How would you like to connect?"**:

1. Escolha **"Drivers"**
2. Driver: **Node.js**
3. Version: a mais recente
4. Vai aparecer uma **connection string** parecida com:
   ```
   mongodb+srv://aelc_admin:<password>@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority
   ```
5. **GUARDE ESTA STRING** num bloco de notas e **substitua `<password>` pela password real** que gerou no passo anterior:
   ```
   mongodb+srv://aelc_admin:SUA_PASSWORD_REAL@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority
   ```
6. Clique **"Done"**

## 1.5 · Network Access (passo crítico)

1. No menu lateral esquerdo do MongoDB Atlas, clique **"Network Access"**
2. Clique no botão verde **"Add IP Address"**
3. Clique em **"Allow Access from Anywhere"** → fica `0.0.0.0/0`
4. **Confirm**

> Sem este passo, o Netlify não consegue ligar-se à base de dados.

✅ **MongoDB Atlas pronto!** Tem agora:
- Uma connection string (guardada num bloco de notas)
- Acesso permitido de qualquer lado

---

# Parte 2 · GitHub (Guardar o Código)

> **O que é?** O sítio onde fica o código fonte da sua app.
> **Para que serve?** O Netlify lê o código de lá automaticamente.

## 2.1 · Criar Conta no GitHub (se ainda não tem)

1. Vá a https://github.com/signup
2. Preencha email, password, escolha username (ex: `aelc-lamego`)
3. Verifique o email

## 2.2 · Guardar o Código no GitHub

**Opção A · A partir do Emergent (mais fácil)**

1. Na plataforma Emergent (onde está esta conversa), procure o botão **"Save to GitHub"** no canto superior direito
2. Clique e autorize a integração com GitHub
3. Escolha **"Create new repository"**
4. Nome: `relato-anomalias-aelc`
5. Visibilidade: **Public** (Private também funciona, mas Public é mais simples para começar)
6. Clique **"Create and push"**
7. Aguarde — quando terminar, vai ter um repo em `https://github.com/SEU_USERNAME/relato-anomalias-aelc`

**Opção B · Manual**

Se não tem o botão "Save to GitHub" no Emergent, peça-me para gerar o ZIP do projeto, descarregue-o e faça upload manual no GitHub:
1. https://github.com/new → criar repo `relato-anomalias-aelc`
2. Seguir as instruções "uploading an existing file" no GitHub

✅ **Código guardado no GitHub!**

---

# Parte 3 · Netlify (Deploy do Site + Backend)

> **O que é?** Serviço que aloja websites e funções serverless.
> **Para que serve?** Hospedar a sua app inteira (frontend + backend).

## 3.1 · Criar Conta no Netlify

1. Vá a https://app.netlify.com/signup
2. Clique em **"Sign up with GitHub"** (usa a mesma conta GitHub que criou na Parte 2)
3. Autorize o Netlify a ver os seus repositórios

## 3.2 · Importar o Repositório

1. No painel principal, clique **"Add new site"** → **"Import an existing project"**
2. Escolha **"Deploy with GitHub"**
3. Se for a primeira vez, clique **"Configure Netlify on GitHub"** e dê permissão ao repositório `relato-anomalias-aelc` (ou "All repositories")
4. Volte ao Netlify, e na lista de repositórios escolha **`relato-anomalias-aelc`**

## 3.3 · Configurar o Build

Netlify vai mostrar uma página "Site configuration". **A maioria já está pré-configurada** pelo ficheiro `netlify.toml`, mas confirme:

| Campo | Valor |
|---|---|
| **Branch to deploy** | `main` |
| **Base directory** | `frontend` |
| **Build command** | `npx expo export -p web` |
| **Publish directory** | `frontend/dist` ou `dist` (depende da deteção) |
| **Functions directory** | `netlify/functions` |

> Se algum destes campos estiver diferente, **edite manualmente** para coincidir com a tabela.

## 3.4 · Adicionar as Variáveis de Ambiente (passo CRÍTICO)

Antes de fazer deploy, **NÃO clique "Deploy site" ainda**. Primeiro vá adicionar as env vars:

1. Na mesma página de configuração, procure a secção **"Environment variables"** (pode estar logo abaixo dos campos de build, ou em "Advanced settings")
2. Clique **"New variable"** e adicione **TODAS** estas variáveis, **uma a uma**:

| Chave | Valor | Notas |
|---|---|---|
| `MONGO_URL` | `mongodb+srv://aelc_admin:SUA_PASS@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority` | A string da Parte 1.4 com password substituída |
| `DB_NAME` | `aelc_anomalias` | |
| `SMTP_HOST` | `smtp.gmail.com` | |
| `SMTP_PORT` | `587` | |
| `SMTP_USER` | `liceuanomalias@gmail.com` | |
| `SMTP_PASSWORD` | `xeog btmj yruk puyv` | Pode ter espaços, código limpa-os |
| `EMAIL_SENDER_NAME` | `Escola Secundária de Latino Coelho` | |
| `DEFAULT_RECIPIENT` | `nuno.ribeiro@aelc-lamego.pt` | Será criado automaticamente como destinatário ativo |
| `JWT_SECRET` | (clique em "Generate a value" se houver — ou coloque uma string aleatória de 64+ chars) | Ex: `j8H!9kL@2mN#4pQ$5rS%6tU^7vW&8xY*9zA1bC2dE3fG4hI5jK6lM7nO8pQ9rS0` |
| `JWT_EXPIRATION_HOURS` | `168` | (7 dias) |
| `ADMIN_USERNAME` | `admin` | |
| `ADMIN_PASSWORD` | Escolha uma password forte e GUARDE | Ex: `LatinoCoelho@2026!` |
| `ALLOWED_DOMAIN` | `aelc-lamego.pt` | Sem o "@" inicial |

> ⚠️ **NÃO defina** `EXPO_PUBLIC_BACKEND_URL` — deixe esta variável NÃO criada.
> Quando está vazia, o frontend chama as funções na mesma origem, que é o que queremos.

> 💡 **Gerar JWT_SECRET aleatório**: Pode usar https://www.uuidgenerator.net/api/version4 e juntar 2-3 UUIDs, ou simplesmente bater no teclado durante 5 segundos.

## 3.5 · Deploy

1. Após adicionar todas as variáveis → clique **"Deploy site"** (ou "Deploy [nome do repo]")
2. Vai para a página do site com status **"Building..."** ou **"Site deploy in progress"**
3. Aguarde **2-5 minutos** para o primeiro build
4. Pode acompanhar logs em tempo real clicando no deploy a decorrer

### Se o build falhar
- Clique no deploy falhado → ver logs
- Erros comuns:
  - `Cannot find module 'expo'` → confirme que `Base directory` é `frontend`
  - `Build script not found` → confirme `Build command` = `npx expo export -p web`
  - `Function failed to bundle` → veja o ficheiro `netlify/functions/api.mjs` está no repo

## 3.6 · Renomear o Site (opcional)

Por defeito Netlify dá um nome aleatório tipo `wonderful-name-12345.netlify.app`. Para mudar:

1. Site settings → **"Change site name"**
2. Escreva `anomalias-aelc` (ou outro nome que goste)
3. Confirm — o site fica em `https://anomalias-aelc.netlify.app`

✅ **App em produção!**

---

# Parte 4 · Validação (testar se tudo funciona)

## 4.1 · Teste Rápido do Backend

Abra o terminal do computador (ou o "Shell" no Netlify) e cole:

```bash
# Substituir SEU_SITE pelo URL atribuído pelo Netlify
SEU_SITE="https://anomalias-aelc.netlify.app"

curl $SEU_SITE/api/
```

**Resultado esperado**:
```json
{"message":"Escola Secundária de Latino Coelho - Reporte de Anomalias API","domain":"aelc-lamego.pt"}
```

Se obtiver isto, o backend está a funcionar! Se não, vá ver logs em **Netlify → Site → Functions → api**.

## 4.2 · Teste End-to-End no Browser

1. Abra `https://anomalias-aelc.netlify.app` no browser
2. **Registar** uma conta:
   - Clique "Ainda não tem conta? Registar"
   - Email: `teste@aelc-lamego.pt`
   - Password: `Teste123`
   - Confirmar e clicar "Registar"
3. Selecionar perfil: **Professor** (ou Aluno)
4. Clicar **"Novo Relato"**:
   - Nome: `Teste Inicial`
   - Perfil: Professor
   - Local: `Biblioteca`
   - Descrição: `Teste de envio`
   - Submeter
5. Deverá ver tela "**Relato Submetido**" com badge verde "Email enviado"
6. Verificar que **email chegou** a `nuno.ribeiro@aelc-lamego.pt`

## 4.3 · Teste do Painel Admin

1. Faça logoff (botão "Sair")
2. Na tela de login, clique **"Acesso Administrador"** em baixo
3. Username: `admin`, Password: a que configurou em `ADMIN_PASSWORD`
4. Deve ver o painel com 2 abas:
   - **Destinatários**: vê `nuno.ribeiro@aelc-lamego.pt` ativo
   - **Reportes**: vê o relato de teste submetido no passo 4.2

✅ **Tudo a funcionar!** Pode partilhar o URL com toda a comunidade escolar.

---

# Parte 5 · Atualizações Futuras (deploy automático)

Já não precisa de fazer mais nada manualmente. **A partir de agora**:

1. Cada vez que eu (ou você) faz uma alteração ao código e dá push para GitHub:
2. O Netlify deteta automaticamente
3. Faz rebuild e re-deploy
4. Em ~2 minutos, a alteração está online

Para forçar um rebuild manualmente:
- Netlify → Site → **"Deploys"** → **"Trigger deploy"** → **"Clear cache and deploy site"**

---

# Parte 6 · Domínio Próprio (Opcional)

Para usar `anomalias.aelc-lamego.pt` em vez de `anomalias-aelc.netlify.app`:

## 6.1 · No Netlify

1. **Site settings → Domain management**
2. **"Add a domain"** → escreva `anomalias.aelc-lamego.pt`
3. Netlify vai mostrar 1 ou 2 registos DNS para configurar
4. Anote esses registos

## 6.2 · No DNS do `aelc-lamego.pt`

Peça ao responsável de TI do agrupamento para adicionar no DNS:

| Tipo | Nome | Valor |
|---|---|---|
| CNAME | `anomalias` | `anomalias-aelc.netlify.app` |

## 6.3 · Aguardar Propagação

- DNS demora 5min a 2h a propagar globalmente
- Volte ao Netlify, clique **"Verify DNS configuration"**
- HTTPS é ativado automaticamente (Let's Encrypt) — leva mais 1-2 min
- Pronto: `https://anomalias.aelc-lamego.pt`

---

# 🆘 Resolução de Problemas

## Build falha em Netlify

| Erro | Causa | Solução |
|---|---|---|
| `expo: command not found` | Node antigo | Site settings → Build → Environment → adicione `NODE_VERSION=20` |
| `Cannot find module 'mongodb'` | `package.json` não copiado | Verificar que `frontend/netlify/functions/package.json` está no repo |
| Build trava | Cache corrompida | Trigger deploy → **Clear cache and deploy site** |

## Função devolve erro 500

1. Netlify → **Functions** → clicar em `api`
2. Ver "Recent invocations" e logs detalhados
3. Erros comuns:
   - `Missing env var: MONGO_URL` → adicionou as env vars depois do deploy? Faça novo deploy
   - `Authentication failed` (MongoDB) → password errada na connection string
   - `EAUTH` (Gmail) → App Password errada

## "Unexpected end of JSON input" no frontend

- Significa que a função crashou silently
- Solução: ver logs da função (Netlify → Functions → api)

## Email não chega

1. Verificar em **Admin → Destinatários** que `nuno.ribeiro@aelc-lamego.pt` está **Ativo**
2. Ver logs da função no Netlify para encontrar erro SMTP específico
3. Confirmar que `SMTP_PASSWORD` em Netlify env é a App Password do Gmail (16 chars, ignorando espaços)
4. Se Gmail bloqueia: aceder a https://myaccount.google.com/security e verificar que 2FA está ativo

## Backend dorme após algum tempo

**Não acontece com Netlify Functions** — diferente do Render Free. Cada pedido acorda a função em ~1s mesmo após inatividade.

---

# 💰 Limites do Plano Grátis

| Recurso | Limite Free | Uso típico de escola |
|---|---|---|
| Pedidos por mês | 125.000 | ~4.000 (largo de sobra) |
| Minutos de função/mês | 125.000 | ~1.000 (largo de sobra) |
| Tráfego (bandwidth) | 100 GB/mês | ~1 GB |
| Builds | 300 min/mês | ~10 min |
| Sites | 500 | 1 |

Se um dia ultrapassar (improvável), Netlify avisa por email antes de cobrar.

---

# ✅ Checklist Final

Antes de partilhar o URL com a comunidade escolar:

- [ ] App responde em `https://SEU_SITE.netlify.app`
- [ ] Registo funciona com email `@aelc-lamego.pt`
- [ ] Registo bloqueia emails de outros domínios
- [ ] Submeter relato envia email real (verificou na caixa)
- [ ] Admin login funciona
- [ ] Admin pode adicionar/remover destinatários
- [ ] `JWT_SECRET` é único e aleatório (não o exemplo)
- [ ] `ADMIN_PASSWORD` é forte
- [ ] HTTPS ativo (cadeado verde no browser)
- [ ] (Opcional) Domínio próprio configurado

---

# 🎉 Pronto!

A app está em produção e pode ser usada por toda a comunidade escolar. Pode:

- Partilhar o URL com alunos e professores
- Imprimir QR codes do site para os alunos digitalizarem
- Adicionar mais destinatários no painel admin (ex: `direcao@aelc-lamego.pt`, `antonio.goncalves@aelc-lamego.pt`)

Em caso de dúvidas, volte aqui e descreva o problema com o máximo de detalhe — log de erros, screenshots, URL onde ocorreu.

---

*Guia atualizado em Maio 2026 · versão 2.0*
