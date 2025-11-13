Comic Store API

API REST simples para gerenciar uma loja de quadrinhos (comics).
Autenticação via Supabase Auth; banco e queries via Supabase. Feito para rodar no Render — pronto pra deploy e nota A no trabalho.

⚠️ Observação rápida: este backend usa a SERVICE_ROLE key do Supabase no servidor — não exponha essa chave no frontend. Use o access_token do Supabase no cliente para autenticar requisições.

Tecnologias

Node.js (>=18)

Express

supabase-js

Joi (validação)

Render (deploy)

Supabase (Auth + Postgres)

Estrutura do projeto
comic-store-api/
├─ .env.example
├─ package.json
├─ README.md
├─ server.js
├─ src/
│  ├─ routes/
│  │  └─ comics.js
│  ├─ controllers/
│  │  └─ comicsController.js
│  ├─ middlewares/
│  │  └─ auth.js
│  ├─ services/
│  │  └─ supabaseClient.js
│  └─ validators/
│     └─ comicsValidator.js
└─ migrations/
   └─ 001_create_comics.sql

Requisitos (do prof)

CRUD completo para comics (listar, buscar, criar, atualizar, excluir).

Organização de pastas e código limpo.

Validações e tratamento de erros (Joi + middleware).

Retornos JSON com códigos HTTP corretos.

Autenticação e DB via Supabase.

Deploy no Render.

Repositório GitHub com commits claros.

README com instruções e exemplos.

Bom — tá tudo aqui. 😉

Variáveis de ambiente (.env)

Cria um .env a partir do .env.example:

PORT=3000
SUPABASE_URL=https://<your-project>.supabase.co
SUPABASE_SERVICE_ROLE_KEY=service_role_xxx...
NODE_ENV=production


SUPABASE_SERVICE_ROLE_KEY: service_role key (admin) — mantém privado no Render.

O cliente (frontend) NÃO usa essa chave. O cliente deve usar o SDK do Supabase para autenticação e enviar o access_token ao backend (Authorization: Bearer <token>).

Migração / Tabela SQL

Execute este SQL no SQL Editor do Supabase:

migrations/001_create_comics.sql

create extension if not exists "pgcrypto";

create table if not exists public.comics (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  author text not null,
  publisher text not null,
  price numeric not null,
  stock integer not null,
  description text,
  owner uuid not null,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

create index if not exists comics_owner_idx on public.comics(owner);


Sugestão: ative Row Level Security (RLS) e crie policies para que owner = auth.uid() apenas acesse seus registros — opcional, mas recomendado.

Como rodar localmente

Clone o repo:

git clone https://github.com/<seu-user>/comic-store-api.git
cd comic-store-api


Copia o .env e preenche as variáveis:

cp .env.example .env
# edita .env com SUPABASE_URL e SUPABASE_SERVICE_ROLE_KEY


Instala dependências:

npm install


Rodar em modo dev:

npm run dev
# ou: npm start


API disponível em http://localhost:3000 (ou porta definida em .env).

Endpoints (todos retornam JSON)

Cabeçalho obrigatório:
Authorization: Bearer <ACCESS_TOKEN> — token obtido via Supabase Auth (cliente).

Listar quadrinhos

GET /comics
Resposta: 200 + lista de comics do usuário.

Buscar um quadrinho

GET /comics/:id
Resposta: 200 + objeto do comic ou 404 se não encontrado.

Criar quadrinho

POST /comics
Body (JSON):

{
  "title": "Batman Ano Um",
  "author": "Frank Miller",
  "publisher": "DC Comics",
  "price": 39.9,
  "stock": 12,
  "description": "Clássico."
}


Resposta: 201 + comic criado.

Atualizar quadrinho

PUT /comics/:id
Body: qualquer campo válido (pelo menos 1).
Resposta: 200 + comic atualizado.

Deletar quadrinho

DELETE /comics/:id
Resposta: 204 (sem conteúdo) ou 404.

Exemplos rápidos (cURL)

Listar

curl -H "Authorization: Bearer <TOKEN>" https://<your-api>.onrender.com/comics


Criar

curl -X POST https://<your-api>.onrender.com/comics \
 -H "Authorization: Bearer <TOKEN>" \
 -H "Content-Type: application/json" \
 -d '{"title":"Sandman Vol.1","author":"Neil Gaiman","publisher":"Vertigo","price":79.90,"stock":5,"description":"Mítico."}'


Atualizar

curl -X PUT https://<your-api>.onrender.com/comics/<id> \
 -H "Authorization: Bearer <TOKEN>" \
 -H "Content-Type: application/json" \
 -d '{"stock":10}'


Deletar

curl -X DELETE https://<your-api>.onrender.com/comics/<id> \
 -H "Authorization: Bearer <TOKEN>"

Tratamento de erros & códigos HTTP

400 — Erro de validação (Joi) com mensagem.

401 — Token ausente ou inválido.

404 — Recurso não encontrado.

201 — Criado.

204 — Deletado com sucesso.

500 — Erro interno (logs no servidor).

Autenticação (fluxo resumido)

Cliente autentica via Supabase (email/senha, OAuth, etc.) usando o SDK do Supabase no frontend.

Supabase retorna access_token (JWT).

Cliente envia Authorization: Bearer <access_token> ao backend.

Backend valida com supabase.auth.getUser(access_token) e obtém user.id.

Usa user.id na coluna owner para filtrar/autorizar recursos no DB.

Deploy no Render (passo a passo rápido)

Faz push do repo no GitHub com commits claros.

No Render: New → Web Service → Connect to GitHub.

Escolhe o repositório e branch (main).

Build command: npm install
Start command: npm start

Em Environment → Add Environment Variables:

SUPABASE_URL = https://<project>.supabase.co

SUPABASE_SERVICE_ROLE_KEY = service_role_xxx... (privado)

NODE_ENV=production

Deploya e monitora logs. URL pública do serviço será https://<your-service>.onrender.com.

Sugestão de commits (pra montar o Git bonito)

feat: init comic-store-api

feat: add supabase client

feat: implement auth middleware

feat: add comics CRUD endpoints

feat: add validation with Joi

chore: add db migration

docs: add README

ci: add render deploy config (opcional)

Testes e QA

Sugestão de testes: Jest + supertest para endpoints (cobre status codes, validações, auth).

Testes básicos que deves incluir:

Criar comic (201) com token válido.

Tentar criar sem token (401).

Listar comics do usuário (200).

Atualizar/Deletar com outro user (deve retornar 404 ou 403 conforme escolha).

Segurança & Boas práticas

Nunca commitar SUPABASE_SERVICE_ROLE_KEY.

Use RLS no Supabase para proteger linhas por auth.uid().

Limita rate (opcional) e logging sensível (não logar tokens).

Se for expor admin endpoints, proteja com roles e checks adicionais.

Próximos passos (se quiser nota A+)

Implemento endpoints para authors, publishers e relacionamentos.

Carrinho de compras e fluxo de pedidos (orders).

Swagger / OpenAPI docs.

CI com GitHub Actions (testes + deploy).

Script de seed com exemplos de quadrinhos.
