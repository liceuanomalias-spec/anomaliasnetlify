# 🚀 Guia de Deploy — Relato de Anomalias AELC

Este guia explica como publicar a aplicação em produção usando serviços **gratuitos**:

| Componente | Serviço | Custo |
|---|---|---|
| Base de dados | **MongoDB Atlas** (M0) | Grátis para sempre |
| Backend (API) | **Render** | Grátis (com cold-start) |
| Frontend (Website) | **Netlify** | Grátis |

Resultado: um URL público estável, p. ex. `https://anomalias-aelc.netlify.app`.

---

## 📋 Pré-requisitos

- Conta GitHub (para guardar o código)
- Email para criar contas em MongoDB Atlas, Render e Netlify
- 30-45 minutos da primeira vez

---

## Passo 1 · Guardar o Código no GitHub

1. No painel Emergent, clique em **"Save to GitHub"** (canto superior direito)
2. Autorize a integração e crie um repositório novo (ex: `relato-anomalias-aelc`)
3. O código fica disponível no seu GitHub para os passos seguintes

> Alternativa: descarregar o ZIP da Emergent, criar repo manualmente no GitHub e fazer push.

---

## Passo 2 · MongoDB Atlas (Base de Dados Grátis)

1. Aceda a https://www.mongodb.com/cloud/atlas/register e crie conta
2. Crie um **cluster gratuito M0** (qualquer região próxima de Portugal, ex: Frankfurt)
3. Em **Database Access** → criar utilizador com password
4. Em **Network Access** → adicionar IP `0.0.0.0/0` (permite ligações de qualquer lado, necessário para Render)
5. Em **Database** → "Connect" → "Drivers" → Python → copie a **connection string**
   - Formato: `mongodb+srv://user:password@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority`
   - Anote para o passo 3

---

## Passo 3 · Backend no Render

1. Aceda a https://render.com e crie conta (pode usar GitHub login)
2. Clique em **"New +"** → **"Web Service"**
3. Conecte o repositório GitHub do projeto
4. Configure:
   - **Name**: `anomalias-aelc-backend`
   - **Region**: Frankfurt
   - **Branch**: main
   - **Root Directory**: `backend`
   - **Runtime**: Python 3
   - **Build Command**: `pip install -r requirements.txt`
   - **Start Command**: `uvicorn server:app --host 0.0.0.0 --port $PORT`
   - **Plan**: Free
5. Em **"Environment"** adicione **TODAS** estas variáveis:

   | Chave | Valor |
   |---|---|
   | `MONGO_URL` | (a string do MongoDB Atlas do passo 2) |
   | `DB_NAME` | `aelc_anomalias` |
   | `SMTP_HOST` | `smtp.gmail.com` |
   | `SMTP_PORT` | `587` |
   | `SMTP_USER` | `liceuanomalias@gmail.com` |
   | `SMTP_PASSWORD` | `xeog btmj yruk puyv` |
   | `EMAIL_SENDER_NAME` | `Escola Secundária de Latino Coelho` |
   | `DEFAULT_RECIPIENT` | `nuno.ribeiro@aelc-lamego.pt` |
   | `JWT_SECRET` | *(clique "Generate" para gerar 64+ chars)* |
   | `JWT_ALGORITHM` | `HS256` |
   | `JWT_EXPIRATION_HOURS` | `168` |
   | `ADMIN_USERNAME` | `admin` |
   | `ADMIN_PASSWORD` | *(escolha uma password forte)* |
   | `ALLOWED_DOMAIN` | `aelc-lamego.pt` |

6. Clique **"Create Web Service"** e aguarde ~5 min para o build
7. **Copie o URL atribuído** (ex: `https://anomalias-aelc-backend.onrender.com`) — vai precisar no passo 4

### ⚠️ Importante: Cold start do plano grátis
- O Render Free dorme após 15 min sem requests
- Primeiro request demora ~30s a "acordar"
- Para produção real, considere o plano Starter ($7/mês) que não dorme

---

## Passo 4 · Compilar o Frontend (Local)

1. Clone o repositório do GitHub para o seu computador:
   ```bash
   git clone https://github.com/SEU_USER/relato-anomalias-aelc.git
   cd relato-anomalias-aelc/frontend
   ```

2. Instale as dependências:
   ```bash
   yarn install
   ```

3. **Atualize o ficheiro `.env`** na pasta `frontend/`:
   ```env
   EXPO_PUBLIC_BACKEND_URL=https://anomalias-aelc-backend.onrender.com
   ```
   (use o URL do Render do passo 3 — sem barra final, sem `/api`)

4. Gere o build estático para web:
   ```bash
   npx expo export -p web
   ```

5. Após terminar, vai existir uma pasta **`dist/`** com:
   - `index.html`
   - `_expo/static/js/...`
   - `_expo/static/css/...`
   - assets

   Esta pasta é o site pronto para deploy.

---

## Passo 5 · Frontend no Netlify

### Opção A · Drag & Drop (mais rápida)
1. Aceda a https://app.netlify.com/drop
2. Arraste a pasta `frontend/dist` inteira para a área indicada
3. Ao fim de ~20 segundos o site fica online com um URL tipo `https://wonderful-name-12345.netlify.app`
4. Pode renomear o subdomínio em Site Settings → Domain Management → opção *"Options" → "Edit site name"*

### Opção B · Auto-deploy via GitHub (recomendado)
1. https://app.netlify.com → **"Add new site" → "Import an existing project"**
2. Conecte o GitHub e selecione o repositório
3. Configure:
   - **Base directory**: `frontend`
   - **Build command**: `npx expo export -p web`
   - **Publish directory**: `frontend/dist`
4. Em **"Environment variables"** adicione:
   - `EXPO_PUBLIC_BACKEND_URL` = `https://anomalias-aelc-backend.onrender.com`
5. Clique **"Deploy"**

Vantagem: cada push ao GitHub dispara um novo deploy automático.

---

## Passo 6 · Domínio Próprio (Opcional)

Para usar `anomalias.aelc-lamego.pt`:

1. No Netlify: Site Settings → Domain Management → **"Add custom domain"**
2. Escreva `anomalias.aelc-lamego.pt`
3. Netlify mostra registos DNS a configurar — peça ao responsável de TI do agrupamento para adicionar no DNS:
   - Tipo: **CNAME**
   - Nome: `anomalias`
   - Valor: `wonderful-name-12345.netlify.app` (o seu URL Netlify)
4. Após propagação DNS (~10 min a 1h), o domínio próprio fica ativo com HTTPS automático (Let's Encrypt)

---

## 🧪 Validação Pós-Deploy

Após cada deploy, valide rapidamente:

```bash
# 1. Backend responde?
curl https://anomalias-aelc-backend.onrender.com/api/

# Esperado:
# {"message":"Escola Secundária de Latino Coelho - Reporte de Anomalias API","domain":"aelc-lamego.pt"}

# 2. Login admin funciona?
curl -X POST https://anomalias-aelc-backend.onrender.com/api/auth/admin-login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"SUA_PASSWORD"}'
```

E no browser:
- Abrir o URL Netlify
- Fazer registo com email `@aelc-lamego.pt`
- Submeter um relato de teste
- Confirmar que email chega a `nuno.ribeiro@aelc-lamego.pt`

---

## 🆘 Problemas Comuns

| Sintoma | Causa | Solução |
|---|---|---|
| "Network request failed" no frontend | `EXPO_PUBLIC_BACKEND_URL` errado ou backend cold-start | Aguardar 30s, ou pingar o backend antes |
| Email não chega | Variáveis SMTP em falta no Render | Verificar todas as variáveis SMTP_* |
| Login admin falha | `ADMIN_PASSWORD` diferente | Conferir variável no Render |
| 404 ao recarregar `/admin/dashboard` | Netlify SPA fallback em falta | O `netlify.toml` já inclui — verificar que foi feito deploy |
| CORS error | Frontend num domínio diferente | Backend tem `allow_origins=["*"]` — não deve acontecer |
| Build expo falha | Versão Node | Use Node 18+ (`node -v`) |

---

## 💰 Custos Totais

| Serviço | Plano | Custo/mês |
|---|---|---|
| MongoDB Atlas | M0 Free | 0 € |
| Render Free | Free | 0 € (com cold-start) |
| Netlify | Starter | 0 € |
| **Total** | | **0 € / mês** |

Para evitar cold-start e ter SLA melhor (recomendado se for usado por toda a escola):
- Render Starter: $7/mês (≈6,50 €/mês)
- MongoDB Atlas continua grátis até ~512 MB de dados
- Netlify continua grátis até 100 GB/mês de tráfego

---

## 🔐 Checklist de Segurança

Antes de partilhar o URL com alunos e professores:

- [ ] `JWT_SECRET` no Render é diferente do exemplo (gerado aleatoriamente)
- [ ] `ADMIN_PASSWORD` é forte (>12 chars, mistura de tipos)
- [ ] MongoDB Atlas com password forte
- [ ] HTTPS ativo em ambos os URLs (Netlify e Render fazem automaticamente)
- [ ] App Password Gmail revogável (não a password real)
- [ ] Variáveis sensíveis **só** no painel Render, nunca commit ao GitHub

---

*Guia atualizado em Maio 2026 · versão 1.0*
