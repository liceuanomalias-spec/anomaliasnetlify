# 📋 Ficha Técnica da Aplicação

## Relato de Anomalias — Escola Secundária de Latino Coelho

---

## 1. Identificação

| Campo | Valor |
|---|---|
| **Nome do Projeto** | Relato de Anomalias · Latino Coelho |
| **Instituição** | Agrupamento de Escolas Latino Coelho, Lamego |
| **Tipo de Produto** | Aplicação Mobile + Website (PWA) |
| **Versão** | 1.0.0 |
| **Idioma** | Português (Portugal) |
| **Público-alvo** | Alunos e Professores do agrupamento |
| **Modalidade de Acesso** | Web (qualquer browser) e Mobile (Android/iOS) |

---

## 2. Objetivo do Projeto

Plataforma digital que permite à comunidade escolar (alunos e professores) **reportar de forma rápida e estruturada** anomalias detetadas em instrumentos e espaços da escola — projetores, computadores, mobiliário, portas, janelas, etc. — substituindo processos informais (queixas verbais, papel) por um fluxo centralizado, rastreável e com notificação automática à direção.

### Problema resolvido
- Comunicação ineficiente de avarias entre comunidade escolar e equipa de manutenção
- Perda/esquecimento de relatos verbais
- Falta de histórico para gestão e estatísticas
- Ausência de notificação imediata aos responsáveis

---

## 3. Stack Tecnológica

### Frontend (App Mobile + Website)
| Camada | Tecnologia | Versão |
|---|---|---|
| Framework | **React Native (Expo SDK 54)** | Latest |
| Linguagem | TypeScript | 5.x |
| Routing | Expo Router (file-based) | 4.x |
| Web rendering | react-native-web | Latest |
| Navegação | @react-navigation | 6.x |
| Armazenamento local | AsyncStorage / localStorage (cross-platform) | — |
| Ícones | @expo/vector-icons (Ionicons) | — |

### Backend (API)
| Camada | Tecnologia |
|---|---|
| Framework | **FastAPI** (Python 3.11) |
| Servidor ASGI | Uvicorn |
| Validação | Pydantic v2 |
| Autenticação | JWT (PyJWT) + bcrypt |
| Email | Gmail SMTP (smtplib, App Password 2FA) |
| ORM Async | Motor (driver MongoDB assíncrono) |

### Base de Dados
- **MongoDB** (NoSQL document store)
- Coleções: `users`, `reports`, `recipients`
- Identificadores: UUID v4 (sem ObjectId expostos)

### Infraestrutura / DevOps
- Containerização: Kubernetes
- Process manager: supervisord
- Reverse proxy / Ingress: redireciona `/api/*` → backend `:8001`
- Variáveis sensíveis em ficheiros `.env`
- Tunelamento dev: Expo Tunnel (ngrok)

---

## 4. Arquitetura de Alto Nível

```
┌─────────────────────────────────────────────────────────────────┐
│  Utilizadores                                                    │
│  ├─ Alunos / Professores  (browser ou app mobile)               │
│  └─ Administrador          (browser, painel separado)            │
└────────────────────┬────────────────────────────────────────────┘
                     │ HTTPS
                     ▼
        ┌────────────────────────────┐
        │     Frontend Expo          │
        │  (Mesma codebase serve)    │
        │  • iOS / Android (nativo)  │
        │  • Web (react-native-web)  │
        └────────────┬───────────────┘
                     │ /api/*
                     ▼
        ┌────────────────────────────┐
        │     Backend FastAPI        │
        │  • Auth JWT                │
        │  • Validação Pydantic      │
        │  • Domain restriction      │
        └─────┬──────────────┬───────┘
              │              │
              ▼              ▼
     ┌──────────────┐  ┌──────────────────┐
     │   MongoDB    │  │  Gmail SMTP      │
     │  (Reports,   │  │  (Notificação    │
     │   Users,     │  │   direção)       │
     │   Recipients)│  └──────────────────┘
     └──────────────┘
```

---

## 5. Funcionalidades Principais

### 🔐 Autenticação e Identidade
- **Registo restrito ao domínio institucional** `@aelc-lamego.pt` (validado backend + frontend)
- Login com email + palavra-passe
- Palavra-passe armazenada com **bcrypt** (não reversível)
- Sessão persistente via token **JWT** (validade 7 dias)
- Logout com confirmação de segurança
- Endpoint de emergência `/logout` para limpeza forçada de sessão

### 📝 Submissão de Relato
- Nome completo (obrigatório)
- Perfil: **Aluno** ou **Professor** (obrigatório, seleção visual)
- Local da Anomalia (obrigatório)
- Número da Sala (opcional)
- Descrição do Problema (obrigatório)
- Informações Adicionais (opcional)
- Validação dupla (cliente + servidor)
- Submissão com indicador de progresso

### 📨 Notificação Automática
- Envio de email HTML formatado para a **lista configurável de destinatários** após cada relato
- Cabeçalho com identidade institucional
- Marcação automática `email_sent: true/false`
- Persistência garantida mesmo em caso de falha SMTP

### 📊 Histórico Pessoal
- Lista cronológica de todos os relatos do utilizador
- Estado de cada relato (Enviado / Pendente)
- Pull-to-refresh para atualização rápida

### 🛠️ Painel de Administração
- Login separado com utilizador `admin` (autenticação independente, token JWT próprio)
- **Gestão de destinatários**:
  - Listar emails atuais
  - Adicionar novos destinatários
  - Ativar/Desativar individualmente (sem perda de dados)
  - Remover permanentemente
- **Visualização de todos os relatos** submetidos por toda a comunidade:
  - Filtragem por estado de email
  - Identificação do submitente (nome + email + perfil)
  - Histórico completo cronológico

### 📱 Experiência Multi-Dispositivo
- **Telemóvel**: layout otimizado em ecrã completo
- **Desktop / Tablet**: cartão centrado em forma de "telemóvel" para preservar a UX
- Detecção automática via media query (`@media min-width: 768px`)
- Toque adaptado a iOS/Android com `hitSlop` em botões críticos

---

## 6. Segurança

| Vetor | Mitigação |
|---|---|
| Acesso indevido | Restrição por domínio institucional `@aelc-lamego.pt` |
| Armazenamento de passwords | Hash **bcrypt** com salt único |
| Sessões | Tokens **JWT** assinados (HS256), expiração 7 dias |
| Injeção SQL/NoSQL | Pydantic + parametrização Motor (sem queries string) |
| CORS | Configurado no FastAPI |
| Privilégio admin | Token JWT com `role` distinto, validação por dependency injection |
| Credenciais SMTP | App Password Gmail (revogável independentemente da password da conta) |
| Exposição de IDs internos | UUIDs aleatórios; `_id` MongoDB nunca exposto |
| HTTPS | Garantido pelo Ingress Kubernetes em produção |

---

## 7. Endpoints da API (Backend)

### Autenticação
| Método | Endpoint | Descrição |
|---|---|---|
| POST | `/api/auth/register` | Registo (valida domínio) |
| POST | `/api/auth/login` | Login utilizador |
| POST | `/api/auth/admin-login` | Login administrador |
| GET | `/api/auth/me` | Utilizador atual |

### Utilizador
| Método | Endpoint | Descrição |
|---|---|---|
| POST | `/api/user/type` | Definir perfil (Aluno/Professor) |
| GET | `/api/reports/mine` | Histórico do utilizador |
| POST | `/api/reports` | Submeter novo relato |

### Administração
| Método | Endpoint | Descrição |
|---|---|---|
| GET | `/api/admin/reports` | Lista todos os relatos |
| GET | `/api/admin/recipients` | Lista destinatários |
| POST | `/api/admin/recipients` | Adicionar destinatário |
| PATCH | `/api/admin/recipients/{id}/toggle` | Ativar/Desativar |
| DELETE | `/api/admin/recipients/{id}` | Remover destinatário |

---

## 8. Modelo de Dados

### `users`
```
id (UUID), email (UNIQUE), password (bcrypt hash),
user_type (aluno|professor|null), created_at (ISO 8601)
```

### `reports`
```
id (UUID), user_id, user_email, user_type,
nome, tipo (aluno|professor),
local, sala (opcional), descricao, info_adicional (opcional),
created_at (ISO 8601), email_sent (boolean), email_error (string|null)
```

### `recipients`
```
id (UUID), email (UNIQUE), active (boolean), created_at (ISO 8601)
```

---

## 9. Integração de Email

- **Servidor**: `smtp.gmail.com:587` (STARTTLS)
- **Conta remetente**: `liceuanomalias@gmail.com`
- **Autenticação**: Gmail **App Password** (2FA obrigatório)
- **Limite**: 500 emails/dia (plano gratuito Gmail)
- **Formato**: Email HTML responsivo, branding institucional
- **Fallback**: Em caso de falha, relato é persistido com `email_sent: false` para reenvio manual

---

## 10. Validação e Testes

- **18 testes automatizados** (pytest) no backend cobrindo:
  - Restrição de domínio
  - Fluxo completo de auth (registo → login → logout)
  - CRUD de relatos
  - CRUD de destinatários (admin)
  - Autorização e isolamento de papéis (user vs admin)
- **Testes E2E** (Playwright) em browser mobile-sized e desktop
- **Validação manual** de envio real de email para `nuno.ribeiro@aelc-lamego.pt` confirmada

---

## 11. Métricas e Estado Atual

- ✅ MVP completo e operacional
- ✅ Emails entregues com sucesso via Gmail SMTP
- ✅ Suporte web e mobile na mesma codebase (zero duplicação)
- ✅ Painel admin funcional com gestão de destinatários
- ✅ Histórico individual e global

---

## 12. Roadmap (Possíveis Evoluções)

| Prioridade | Funcionalidade |
|---|---|
| Alta | Dashboard com **gráficos** (relatos por sala/mês, tempo médio de resolução) |
| Alta | **Exportação CSV/Excel** para gestão pela direção |
| Média | Estado do relato (Recebido → Em análise → Resolvido) |
| Média | Anexo de **fotografia** ao relato |
| Média | Categorização de instrumento (eletrónica, mobiliário, infraestrutura) |
| Média | Login federado com **Microsoft 365 institucional** |
| Baixa | Notificações push (mobile) |
| Baixa | Modo offline com sincronização |
| Baixa | Aplicação Android/iOS na Play Store / App Store |

---

## 13. Distribuição

| Canal | Estado | URL |
|---|---|---|
| **Website (PWA)** | Em preview | URL temporário até publicação |
| **Mobile** | Pronto para Expo Go (QR code) | Build APK/IPA disponível sob pedido |
| **Domínio próprio** | Não configurado | Sugestão: `anomalias.aelc-lamego.pt` (CNAME) |

---

## 14. Equipa / Créditos

- **Desenvolvimento**: Construído com Emergent E1 (agente de programação full-stack)
- **Stack escolhida por**: requisitos de multi-plataforma com codebase única
- **Manutenção futura**: através da plataforma Emergent ou GitHub (export disponível)

---

## 15. Resumo Executivo

> Aplicação **completa, segura e moderna** que une mobile e web numa única codebase TypeScript. Permite que **toda a comunidade escolar reporte anomalias em menos de 30 segundos**, com **notificação imediata por email** aos responsáveis, e oferece à direção um **painel centralizado** com histórico completo e gestão flexível dos destinatários — substituindo processos manuais por um fluxo digital rastreável, eficiente e escalável.

---

*Documento gerado em Maio de 2026 · Versão 1.0*
