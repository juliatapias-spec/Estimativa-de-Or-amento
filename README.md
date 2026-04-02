# Estimador de Orçamento — Eventos & Cenografia

Ferramenta interna para estimativa de orçamentos com base em projetos anteriores.

---

## Pré-requisitos

- Conta gratuita no [Supabase](https://supabase.com)
- Conta gratuita no [Vercel](https://vercel.com)
- Chave de API da Anthropic (obtenha em [console.anthropic.com](https://console.anthropic.com))

---

## Passo 1 — Criar o banco de dados no Supabase

1. Acesse [supabase.com](https://supabase.com) e crie um projeto novo
2. No menu lateral, clique em **SQL Editor**
3. Cole e execute o SQL abaixo:

```sql
create table if not exists items (
  id bigserial primary key,
  cat text not null,
  item text not null,
  desc text,
  qty numeric,
  unit numeric,
  total numeric,
  sup text,
  date text,
  created_at timestamptz default now()
);

create table if not exists config (
  id int primary key default 1,
  markup numeric default 20,
  rules text default ''
);

insert into config (id, markup, rules)
values (1, 20, '')
on conflict do nothing;

alter table items enable row level security;
alter table config enable row level security;

create policy "acesso publico items leitura"  on items for select using (true);
create policy "acesso publico items escrita"  on items for insert with check (true);
create policy "acesso publico items exclusao" on items for delete using (true);
create policy "acesso publico config leitura" on config for select using (true);
create policy "acesso publico config escrita" on config for update using (true);
```

4. Vá em **Project Settings → API**
5. Copie a **Project URL** e a **anon public key**

---

## Passo 2 — Configurar o arquivo index.html

Abra o arquivo `index.html` e localize as linhas no topo do `<script>`:

```js
const SUPABASE_URL = 'COLE_AQUI_SUA_URL';
const SUPABASE_KEY = 'COLE_AQUI_SUA_CHAVE_ANONIMA';
```

Substitua pelos valores copiados no Passo 1.

---

## Passo 3 — Publicar no Vercel

1. Acesse [vercel.com](https://vercel.com) e faça login
2. Clique em **Add New Project → Deploy from existing files** (ou use o [Vercel CLI](https://vercel.com/docs/cli))
3. Faça upload da pasta `orcamento-eventos` inteira
4. Antes de confirmar o deploy, vá em **Environment Variables** e adicione:
   - Nome: `ANTHROPIC_API_KEY`
   - Valor: sua chave da Anthropic (começa com `sk-ant-...`)
5. Clique em **Deploy**

Pronto! O Vercel vai te dar um link `.vercel.app` para compartilhar com a equipe.

---

## Estrutura dos arquivos

```
orcamento-eventos/
  index.html        ← app principal
  api/
    estimate.js     ← proxy seguro para a API da Anthropic
  vercel.json       ← configuração do Vercel
  README.md         ← este arquivo
```

---

## Observações

- A chave da Anthropic **nunca fica exposta** no navegador — ela fica apenas no servidor do Vercel
- Os dados ficam no Supabase e são compartilhados por toda a equipe em tempo real
- Para adicionar autenticação de usuários, consulte a documentação do Supabase Auth
