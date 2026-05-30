# 📱 Relato de Anomalias — Escola Secundária de Latino Coelho

Aplicação mobile + website para reporte de anomalias em instrumentos e espaços da escola.
Mesma codebase (Expo + React Native) compila para iOS, Android e Web.

## 🚀 Quick Start

### Para fazer deploy em produção
Siga o guia detalhado em **[`DEPLOY_GUIDE_NETLIFY_FUNCTIONS.md`](./DEPLOY_GUIDE_NETLIFY_FUNCTIONS.md)** — tudo correrá no Netlify (grátis) com MongoDB Atlas (grátis).

### Para correr localmente

```bash
# Backend (Python FastAPI)
cd backend
cp .env.example .env  # editar com valores reais
pip install -r requirements.txt
uvicorn server:app --reload --port 8001

# Frontend (Expo)
cd frontend
cp .env.example .env  # editar com valores reais
yarn install
yarn expo start  # menu para iOS/Android/Web
```

## 📂 Estrutura

```
.
├── backend/                     # FastAPI + MongoDB (Python) — usado em dev
│   ├── server.py                # Toda a lógica do backend
│   ├── requirements.txt
│   └── tests/
│       └── test_aelc_api.py     # 18 testes pytest
│
├── frontend/                    # Expo (TypeScript + React Native)
│   ├── app/                     # Ecrãs (Expo Router file-based routing)
│   │   ├── index.tsx            # Login
│   │   ├── register.tsx
│   │   ├── user-type.tsx
│   │   ├── home.tsx             # Dashboard
│   │   ├── report.tsx           # Form
│   │   ├── confirmation.tsx
│   │   ├── logout.tsx           # Emergency logout
│   │   └── admin/               # Painel admin
│   │       ├── login.tsx
│   │       └── dashboard.tsx
│   │
│   ├── src/
│   │   ├── auth/AuthContext.tsx
│   │   ├── theme.ts
│   │   └── utils/
│   │       ├── api.ts           # Robust fetch helper
│   │       └── storage/         # Cross-platform key-value
│   │
│   ├── assets/images/
│   │   └── school-logo.png      # Logo institucional
│   │
│   ├── netlify/
│   │   └── functions/
│   │       ├── api.mjs          # ✨ BACKEND SERVERLESS (produção)
│   │       └── package.json     # Deps das functions
│   │
│   ├── netlify.toml             # Config Netlify
│   ├── app.json                 # Config Expo
│   └── package.json
│
├── memory/
│   ├── PRD.md                   # Especificação do produto
│   └── test_credentials.md
│
├── FICHA_TECNICA.md             # Ficha técnica para apresentação
├── DEPLOY_GUIDE.md              # Guia deploy Render + Netlify
├── DEPLOY_GUIDE_NETLIFY_FUNCTIONS.md  # Guia deploy só Netlify
└── README.md                    # Este ficheiro
```

## 🏗 Arquitetura

```
Frontend Expo (Web/iOS/Android)
   │
   ▼  /api/*
┌──────────────────────────┐
│ Backend                  │
│ • Em dev: Python FastAPI │
│ • Em prod: Netlify       │
│   Functions (api.mjs)    │
└──────────┬───────────────┘
           │
   ┌───────┴───────┐
   ▼               ▼
MongoDB Atlas   Gmail SMTP
```

Tanto o backend Python como o backend serverless implementam **exatamente os mesmos endpoints** com a mesma lógica. Pode usar qualquer um deles.

## 🔐 Segurança

- ✅ Restrição por domínio institucional `@aelc-lamego.pt`
- ✅ Passwords com bcrypt
- ✅ Tokens JWT (HS256)
- ✅ Validação Pydantic / Zod-style
- ✅ Variáveis sensíveis fora do código (`.env` / Netlify env vars)
- ✅ HTTPS automático

## 📚 Mais documentação

- **[Ficha Técnica completa](./FICHA_TECNICA.md)** — para apresentação à direção
- **[Guia de Deploy (Netlify Functions)](./DEPLOY_GUIDE_NETLIFY_FUNCTIONS.md)** — passo a passo
- **[PRD](./memory/PRD.md)** — especificação detalhada

## 📝 Licença

Projeto interno do Agrupamento de Escolas Latino Coelho, Lamego.
